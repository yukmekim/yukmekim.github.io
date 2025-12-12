---
title: 'QueryDSL HQL Injection 취약점(CVE-2024-49203)'
description: 수습기간 이후 새로운 프로젝트를 진행하게 되며 게시판 기능을 구현중 검색 필터와 복잡한 조인 구조의 쿼리 처리를 팀내 백엔드 개발자와 상의하여 가장 익숙하다고 하는 querydsl을 이용하기로 했다.  의존성 추가를 위해 maven repository에서 querydsl 가장 최신 버전을 build.gradle 추가하게 되었으며, 이 과정에서 IDE에서 사용 경고가 발생했다.
categories:
 - Backend
tags:
 - QueryDSL
 - Security
 - Spring Boot
---
수습기간 이후 새로운 프로젝트를 진행하게 되며 게시판 기능을 구현중 검색 필터와 복잡한 조인 구조의 쿼리 처리를 팀내 백엔드 개발자와 상의하여 가장 익숙하다고 하는 querydsl을 이용하기로 했다.

의존성 추가를 위해 maven repository에서 querydsl 가장 최신 버전을 build.gradle 추가하게 되었으며, 이 과정에서 IDE에서 사용 경고가 발생했다.

![Desktop Preview](/assets/images/post/querydsl_hql_injection/querydsl_ide_warning.png)

경고 내용으로는 Querydsl이 orderBy에서의 HQL injection에 취약하다는 내용이다.

내용을 자세히 찾아보니 2024년11월9일 CSIRT.SK에서 QueryDSL 5.1.0에서 HQL Injection 취약점([CVE-2024-49203](https://nvd.nist.gov/vuln/detail/CVE-2024-49203))을 보고하면서 등록된 경고이다.

**핵심: Order By Injection**
정렬 파라미터에 악의적인 값을 넣으면 임의의 HQL 구문을 실행할 수 있다는 것.

아래와 같이 악의적인 요청을 보냈을 때:
```
GET :8000/products?orderBy=name+INTERSECT+SELECT+t+FROM+Test+t+WHERE+(SELECT+SUBSTRING(email,1,1)+FROM+users+WHERE+username='admin')='a'+ORDER+BY+t.id
```

다음과 같은 네이티브 쿼리로 수행된다.
```sql
(SELECT * FROM test_table t1 ORDER BY t1.name)
INTERSECT
(SELECT * FROM test_table t 
 WHERE (SELECT LENGTH(email) FROM users WHERE username='admin')=17);
```

해당 쿼리의 수행 결과로 데이터가 존재 했을 경우 조건이 참이라는 뜻으로 실제 데이터를 유추하는게 가능하다는 얘기이다.  
```
HTTP/1.1 200
Content-Type: application/json
Date: Fri, 01 Aug 2025 13:34:57 GMT
Content-Length: 27


[{"id":1,"name":"test123"}]
```

취약점을 알게 되었으니 개선 방법을 고민 해봐야한다.


Querydsl은 쿼리 빌더이기 때문에 본질적으로 안전하지 않은 쿼리를 생성할 수 있다. 사용자나 개발자가 입력한 값으로 쿼리를 동적으로 생성하는 구조상, 입력 값을 적절히 검증하면 문제를 해결할 수 있다.

하지만 입력 검증을 도입하는 것이 완벽한 해결책이 되기는 어렵다. 데이터베이스마다 허용하는 문법이 다르기 때문이다. 예를 들어, 일부 데이터베이스는 구문에서 탭이나 이스케이프 문자와 같은 특수한 공백 문자를 허용하므로, 모든 경우를 고려한 검증 로직을 작성하는 것은 간단한 작업이 아니다.

가장 이상적인 방법은 Querydsl에서 직접 수정한 버전을 사용하는 것이다. 그러나 공식 Querydsl 프로젝트는 2024년 1월 29일에 릴리즈된 5.1.0 버전 이후로 업데이트되고 있지 않으며, Spring 공식 문서에서도 기존 Querydsl 유지보수가 중단 되었음이 명시되어있다.  이에 OpenFeign 팀에서 Querydsl 프로젝트를 Fork하여 지속적으로 유지보수 작업을 진행하고 있다.

OpenFeign 팀은 HQL Injection 취약점(CVE-2024-49203)을 [6.10.1](https://github.com/OpenFeign/querydsl/releases/tag/6.10.1) 버전 이후로 수정했다고 한다.
기존 Querydsl 코드베이스와 API를 그대로  유지하면서 취약점을 해결했기 때문에, 기존 사용자들이 큰 마이그레이션 부담 없이 안전한 버전으로 전환할 수 있다는 장점이 있다.

실제 프로젝트에 변경한 의존성 버전은 아래 릴리즈 내용을 바탕으로 현재 프로젝트 버전에 최적화 되어있는 7.0 버전을 사용하였다.  

1. JPA 3.2.0 + Hibernate 7.0, Java 17 최적화되어 있음
2. Java 17 베스트 프랙티스 적용
3. 코드 현대화 (var, switch 패턴 매칭, text blocks)


