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
SSD:          50-150μs (메모리보다 약 1,000배 느림)
HDD:          5-15ms   (메모리보다 약 100,000배 느림)
```

![Desktop Preview](/assets/images/post/redis_migration/redis-test-result.png)

- PostgreSQL: 평균 9.13ms (913ms ÷ 100 threads)
- Redis: 평균 2.30ms (230ms ÷ 100 threads)

## Redis 적용 후 달라진 점

### 1. 코드가 단순해졌다

**Before: RDBMS**
```java
@Entity
public class RefreshTokenEntity { ... }

@Repository
public interface RefreshTokenRepository { ... }

@Scheduled(cron = "0 0 2 * * *")  // 배치 스케줄러
public void deleteExpiredTokens() { ... }
```

**After: Redis**
```java
redisTemplate.opsForValue().set(key, value);
redisTemplate.expire(key, 7, TimeUnit.DAYS); // TTL로 자동 만료
```

엔티티, Repository, 배치 스케줄러가 모두 사라졌다.
단순한 Key-Value 구조로 충분했다.

### 2. 성능이 개선되었다

부하 테스트 결과 **4배** 빠른 응답 속도를 확인했다.

| | PostgreSQL | Redis | 개선 |
|---|---|---|---|
| 응답 시간 | 9.13ms | 2.30ms | 4배 |
| 처리량 | 10,953 req/sec | 43,478 req/sec | 4배 |

더 인상적이었던 건, **트래픽이 증가할수록 차이가 커진다**는 점이다.

순차 처리에서는 1.9배였던 차이가,
동시 100명 접속 시에는 4배로 벌어졌다.

PostgreSQL은 Connection Pool 경합과 Lock으로 인해
동시 접속이 많아질수록 성능이 급격히 저하되었다.

반면 Redis는 높은 처리량(초당 43,000건)으로
안정적인 성능을 유지했다.

### 3. DB 부하가 줄었다

매 API 요청마다 발생하던 토큰 검증 쿼리가 사라졌다.
```
초당 1만 건의 API 요청이 있다면:
- Before: 초당 1만 번의 DB 쿼리
- After: DB 쿼리 0번 (Redis로 분리)
```

## 마무리

Redis를 처음 공부했을 때 가장 큰 고민은 **"어디에 써야 하지?"**였다.

좋다는 건 알겠는데, 막상 적용하려니 막막했다.
아마 새로운 기술을 배울 때 누구나 겪는 고민일 것이다.

### 모든 데이터를 Redis에?

"단순한 Key-Value 데이터는 전부 Redis에 저장하면 되는 거 아닐까?"

하지만 Redis는 메모리 기반이라 **데이터 손실 가능성**이 있다.
결제 내역이나 회원 정보 같은 중요한 데이터를 Redis에 저장하는 건
상상만 해도 끔찍하다.

### Redis의 적재적소

Redis는 **휘발성 데이터**에 특화되어 있다.

- ✅ 세션
- ✅ JWT Refresh Token
- ✅ 캐시
- ✅ 실시간 랭킹

사라져도 재생성 가능하거나, 임시로만 필요한 데이터.
바로 이런 곳에 Redis를 써야 한다는 것.

Refresh Token 관리가 딱 맞는 첫 번째 사례였다.