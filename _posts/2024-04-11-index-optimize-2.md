---
title: '[MySQL] 여러 컬럼으로 인덱스 구성'
description: 테이블에 인덱스를 추가하고 나서 쿼리 속도가 많이 개선되었는데, 사내 조직 개편으로 새로운 팀으로 옮긴후 새로운 프로젝트 개인정보동의서명과 백오피스 부분에서 서명관리 파트를 개발하던중 예상치 못한 문제가 발생했다. 이전에 인덱스를 구성할때 인덱스의 키 크기에만 주의하며 사용했었는데, 컬럼에 아무리 인덱스를 추가해도 성능이 개선되지 않는 쿼리가 있었다.
categories:
 - MySQL
tags:
 - database
 - MySQL
 - mariaDB
---
해당 프로젝트를 진행하면서 한 테이블 5개의 컬럼에 인덱스를 추가 해보았지만 성능에 변화가 없었다.  
아이러니하다, 실행 계획을 돌려봤을때 인덱스 키를 전혀 사용하지 않고 있었고 여러개의 인덱스를 구성할때 컬럼의 순서가 영향을 준다는 사실을 알게 되었다.

## 여러개의 컬럼으로 인덱스 구성
### 카디널리티와 상관 관계
여러개의 컬럼으로 인덱스를 구성하는데 있어 영향을 주는것은 카디널리티이다.

카디널리티란 간단하게 말하면 튜플/행의 수이며 전체 행에 대한 특정 컬럼의 중복 수치를 나타내는 지표이다.  
즉,  
중복도가 ‘낮으면’ 카디널리티가 **‘높다’**고 표현한다.  
중복도가 ‘높으면’ 카디널리티가 **‘낮다’**고 표현한다.

그렇다면 여러 컬럼으로 인덱스를 잡는다면 어떤 순서로 인덱스를 구성해야 할까?
간단하다. 두 가지 테스트 케이스가 있으니 모두 실험해보면 되는 일이다.  
**카디널리티가 낮은->높은순으로 구성**  
**카디널리티가 높은->낮은순으로 구성**

아래 예제코드를 보며 확인 해보자
```sql
CREATE TABLE `salaries` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `emp_no` int(11) NOT NULL,
  `salary` int(11) NOT NULL,
  `from_date` date NOT NULL,
  `personal_no` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

전체 Row수는 100건 정도로 진행했고 카디널리티를 조회해보면 다음과 같다.

![Desktop Preview](/assets/images/post/index_optimize_2/cardinarity_select.png)

```sql
-- 카디널리티가 낮은순
CREATE INDEX IDX_SALARIES_INCREASE ON salaries 
(from_date, salary, personal_no, emp_no);

-- 카디널리티가 높은순
CREATE INDEX IDX_SALARIES_DECREASE ON salaries 
(emp_no, personal_no, salary, from_date);
```

첫번째 인덱스는 from_date, salary, personal_no, emp_no 카디널리티가 낮은순에서 높은순 (중복도가 높은 순에서 낮은순으로) 으로,
두번째 인덱스는 emp_no, personal_no, salary, from_date 카디널리티가 높은순에서 낮은순 (중복도가 낮은 순에서 높은순으로) 으로 생성했다.

| | IDX_SALARIES_INCREASE | IDX_SALARIES_DECREASE | 
| --- | --- | --- |
| 1 | 77ms | 69ms | 
| 2 | 76ms | 57ms | 
| 3 | 69ms | 45ms | 

수행 시간을 비교하는게 적은 데이터에서는 큰 차이가 보이지는 않지만 얼핏 보았을때 카디널리티가 **높은순에서 낮은순으로** 구성한 인덱스의 성능이 조금 더 좋아 보인다.
실행 계획 까지 본 후 더미 데이터를 추가하여 다시 비교해보록 하겠다.

### 컬럼 조건
개요에서 말했던 **'인덱스 키를 사용하지 않고 있었고'** 이 대목을 주목하며 위에서 만들어놓은 예제 코드로 실행 계획도 돌려보자

```sql
explain select * 
from salaries 
where from_date = '2024-12-02' 
and salary = 3000
and personal_no in (101,102, 103, 104, 105, 106);

explain select * 
from salaries 
where emp_no = 1000
and salary = 3000
and personal_no in (101,102, 103, 104, 105, 106);

explain select * 
from salaries 
where salary = 3000
and personal_no in (101, 102, 103, 104, 105, 106);
```

첫번째 실행 계획에 key와 extra를 확인해보면
![Desktop Preview](/assets/images/post/index_optimize_2/index_explain_1.png)

만들어둔 IDX_SALARIES_INCREASE 인덱스 키를 사용한 것을 확인 할 수 있다.

두번째 실행 계획을 보면 
![Desktop Preview](/assets/images/post/index_optimize_2/index_explain_2.png)

만들어둔 IDX_SALARIES_DECREASE 인덱스 키를 사용한 것을 확인 할 수 있다.  
첫번째와 두번째 실행 계획을 비교해보면 첫 번쨰 조건절에 사용된 컬럼이 생성했던 인덱스 컬럼의 순서와 관련이 있음을 확인할 수 있다.

세번째 실행 계획을 보면 
![Desktop Preview](/assets/images/post/index_optimize_2/index_explain_3.png)

인덱스를 사용하지 않고 where 조건으로 단순 조회만 하는것과 row수를 비교해 보면 인덱스를 사용한것과 사용하지 않은것이 확연히 차이가 난다.

위에 실행 계획을 돌려보며 인덱스를 추가했지만 인덱스를 사용하지 못했던 이유를 알 수 있었다.  

## 마무리
![Desktop Preview](/assets/images/post/index_optimize_2/zeri_ending.gif)

테스트를 진행하며 알게된 사실을 크게 3가지로 정리해봤다.
1. 인덱스의 조회속도는 카디널리티가 **높은순에서 낮은순으로** 구성한 인덱스의 성능이 조금 더 좋다.  
2. **조회 조건에 들어가는 컬럼의 순서**와 **인덱스 컬럼의 순서**는 서로 관계가 있다.  
3. 첫번째 인덱스 컬럼이 조회 쿼리에 없으면 인덱스를 타지 않는다는 점을 기억하면 될것 같다.

개발3팀으로 옮기게 되면서 팀장님이 주의를 주었던 인덱스를 추가할때 컬럼의 순서에 주의하며 공부하며 인덱스를 잘못 사용하고 있었음을 알게되었다.
분명 이번에 알게된 사실 말고도 많은 요소들이 성능에 영향을 줄거라고 생각을 하며 마무리 한다.