---
title: 'QueryDSL HQL Injection 취약점(CVE-2024-49203)'
description: 수습기간 이후 새로운 프로젝트를 진행하게 되며 게시판 기능을 구현중 검색 필터와 복잡한 조인 구조의 쿼리 처리를 팀내 백엔드 개발자와 상의하여 가장 익숙하다고 하는 querydsl을 이용하기로 했다.
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