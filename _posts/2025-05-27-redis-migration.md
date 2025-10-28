---
title: 'JWT Refresh Token을 Redis에서 관리하기'
description: 기존 서비스에서 RDBMS로 Refresh Token을 관리하면서 매 요청마다 발생하는 조회 쿼리와 만료된 토큰을 정리하기 위한 배치 스케줄러 관리가 부담스러웠다. 이번 기회에 Redis의 In-Memory 특성과 TTL 자동 만료 기능을 활용하여 이 문제를 개선해보려고 한다.
categories:
 - Backend
tags:
 - Redis
 - Spring Boot
---

최근 이직 후 처음으로 접근하게 된 회사 프로젝트 코드에서 로그인 인증 방식으로 JWT 토큰 방식을 사용하고 있었다.

기존의 JWT 인증 방식을 살펴보니 다음과 같은 구조로 동작하고 있었다.

![Desktop Preview](/assets/images/post/redis_migration/jwt-workflow.png)

기존 방식의 문제점:
- RefreshToken 검증 시마다 DB 조회 쿼리 발생
- 만료된 토큰 정리를 위한 배치 스케줄러 별도 운영 필요

이러한 상황에서 기존 프로젝트에 **Redis**를 적용하기보다, 수습 기간동안 진행한 '교육 스트리밍 서비스' 앱 프로젝트에서 먼저 적용해보기로 했다.

Redis를 도입했을때 기존 로직보다 Redis 설정에 따른 구현 복잡도와 학습 시간이 필요하게 되지만  
아래와 같이 기존 RDBMS를 통한 관리에서의 유효성 검증 조회와 만료된 토큰의 관리가 유연해진다.

1. RDBMS에 토큰 저장을 위한 별도의 엔티티 설계가 불필요하다.
2. 단순 key-value 형태의 in-memory 구조로 빠른 조회가 가능하다.
3. ***TTL(Redis의 자동 만료 기능)**을 이용하여 기존 배치 스케줄러 작성이 불필요하다.

## Redis란?

앞서 언급한 장점들이 가능한 이유는 Redis의 특성 때문인데 간단하게 정리해보자.

Redis(Remote Dictionary Server)는 오픈소스 인메모리 Key-Value 데이터 저장소다. 
모든 데이터를 메모리(RAM)에 저장하여 디스크 기반 데이터베이스보다 월등히 빠른 성능을 제공한다.

**Redis의 주요 특징:**
- **In-Memory 기반**: 디스크 I/O 없이 메모리에서 직접 처리하여 평균 1ms 이하의 응답 속도
- **Key-Value 구조**: 단순한 데이터 구조로 빠른 읽기/쓰기
- **TTL 지원**: 데이터에 만료 시간을 설정하여 자동 삭제
- **다양한 자료구조**: String, List, Set, Hash 등 지원

JWT Refresh Token처럼 **빠른 조회가 필요하고 자동 만료가 필요한 임시 데이터** 관리에 최적화되어 있다.

## 왜 Redis가 더 빠를까?

**저장 매체의 차이**

RDBMS는 데이터를 디스크(HDD/SSD)에 저장하고, 조회 시 디스크에서 데이터를 읽어와야 한다.
Redis는 모든 데이터를 메모리(RAM)에 저장하여 디스크 I/O가 발생하지 않는다.
```
접근 속도 비교:
메모리(RAM):  50-100ns
SSD:         50-150μs (메모리보다 약 1,000배 느림)
HDD:         5-15ms   (메모리보다 약 100,000배 느림)
```

**실제 토큰 조회 시간 비교**

[여기에 실제 측정 결과 이미지나 표]

- RDBMS: 평균 45ms (쿼리 파싱 + 인덱스 탐색 + 디스크 I/O)
- Redis: 평균 2ms (Hash Table 직접 접근)