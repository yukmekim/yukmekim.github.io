---
title: 'Kubernetes 다중 파드 환경에서 ShedLock으로 스케줄러 중복 실행 막기'
description: Spring Boot 애플리케이션을 Kubernetes에 replicas 2로 운영하면서 @Scheduled 배치가 파드마다 한 번씩, 총 두 번 실행되는 문제가 있었다. 임시 파일 정리 배치에서 발생한 에러로 원인을 확인했고, 이미 사용 중이던 Redis와 ShedLock을 이용해 배치가 한 파드에서만 실행되도록 개선했다.
categories:
 - Backend
tags:
 - Spring Boot
 - Kubernetes
 - ShedLock
 - Redis
---

Spring Boot 애플리케이션을 Kubernetes에 `replicas: 2`로 올려서 운영하고 있었다.

`@Scheduled`는 각 JVM 안에서 독립적으로 동작한다. 파드가 2개면 같은 시각에 같은 배치가 2번 실행된다는 뜻인데, 이 중복 실행을 막는 장치가 없었다.

## 문제 상황

임시 파일 정리 배치에서 아래와 같은 에러가 발생하고 있었다.

원인을 확인하기 위해 크론 표현식을 조정해서 두 파드가 같은 시각에 배치를 실행하도록 재현했고, 아래는 그때의 로그다. (실제 배치는 매일 02:00에 실행된다)

```
[인스턴스 B] 12:14:00.025  임시 파일 정리 완료: 처리=3개, 삭제=3개
[인스턴스 A] 12:14:00.066  Unexpected error occurred in scheduled task
             ObjectOptimisticLockingFailureException: Row was updated or deleted by another transaction
             : [com.example.entity.Attachment#22]
```

1. 두 인스턴스가 같은 시각(12:14:00)에 같은 배치를 실행했다.
2. 인스턴스 B가 먼저 임시 파일 3개를 삭제했다.
3. 인스턴스 A도 같은 파일을 삭제하려 했지만, 이미 B가 삭제한 행이라 `ObjectOptimisticLockingFailureException`이 발생했다.

A의 트랜잭션은 롤백되어 삭제도, 이력도 남지 않았다. 결과적으로 데이터에는 문제가 없었다.

하지만 이건 낙관적 락 예외가 두 번째 실행을 **우연히** 막아준 것이다.
매번 한쪽 파드는 실패하는 구조였고, 이런 충돌이 일어나지 않는 배치(예: 만료 예정 안내 발송)라면 중복 실행이 그대로 일어날 수 있다.

## 해결 방법 검토

| 방법 | 판단 |
|---|---|
| ShedLock + JDBC | 락 테이블 추가 필요 → 테이블 명명 검토 필요 |
| Kubernetes CronJob | 배치 구조 변경 필요, 정확히 한 번 실행을 보장하지 않음 |
| ShedLock + Redis | 채택 |

### ShedLock + Redis를 고른 이유

**1. Redis가 이미 있었다**

로그인 토큰 저장과 파드 간 메시지 전달에 Redis를 이미 쓰고 있었다. 새 인프라가 필요 없었다.
또 Redis가 죽으면 로그인부터 안 되는 구조라, 락 때문에 Redis 의존이 새로 생기는 것도 아니었다.

**2. DB 스키마를 건드리지 않는다**

JDBC 방식은 락 테이블이 필요하다. 이 프로젝트는 테이블·컬럼 이름을 공공기관 공통표준 용어로 맞춰야 해서, 테이블 하나 추가에도 명명 검토와 정의서 갱신이 따라온다. Redis 방식은 이 과정이 없다.

**3. 변경 범위가 작다**

배치 코드는 그대로 두고 어노테이션 한 줄씩만 붙였다.
중복이 문제 되는 작업 4개에만 붙이고, 파드마다 돌아야 하는 SSE heartbeat는 제외했다.

**4. 어느 방법도 "정확히 한 번"을 보장하지 않는다**

Kubernetes 공식 문서의 [CronJob limitations](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations)에는 아래와 같이 명시되어 있다.

> The scheduling is approximate because there are certain circumstances where two Jobs might be created, or no Job might be created. Kubernetes tries to avoid those situations, but does not completely prevent them. Therefore, the Jobs that you define should be _idempotent_.

CronJob도 Job을 두 번 만들거나 만들지 않을 수 있으니, Job을 멱등하게 만들라는 것이다.

ShedLock도 마찬가지로 정확히 한 번을 보장하지는 않는다.
어느 방법도 보장하지 못한다면, 배치 구조를 크게 바꾸는 CronJob보다 변경이 적은 ShedLock이 낫다고 판단했다.

## ShedLock 적용

| 항목 | 버전 |
|---|---|
| ShedLock | 5.16.0 (core·Redis provider 동일) |
| Spring Boot | 3.2.12 |
| Java | 21 |

**의존성**
```groovy
implementation 'net.javacrumbs.shedlock:shedlock-spring:5.16.0'
implementation 'net.javacrumbs.shedlock:shedlock-provider-redis-spring:5.16.0'
```

**설정 클래스**
```java
@Configuration
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "PT30M")
public class SchedulingConfig {
    // 잠금 키 접두어. Redis 는 dev·prod 가 별도 인스턴스라 환경 구분 없이 고정한다
    private static final String LOCK_ENVIRONMENT = "app";

    // replicas=2 라 @SchedulerLock 을 붙인 작업은 잠금을 잡은 파드 한 곳에서만 실행된다
    @Bean
    public LockProvider lockProvider(RedisConnectionFactory redisConnectionFactory) {
        return new RedisLockProvider(redisConnectionFactory, LOCK_ENVIRONMENT);
    }
}
```

`LOCK_ENVIRONMENT`는 Redis 키 접두어로 쓰인다. dev와 prod가 별도 Redis 인스턴스라 고정값을 사용했다.
여러 환경이 같은 Redis를 함께 쓴다면 환경마다 다른 값을 줘야 한다.

**배치 메서드**
```java
@Scheduled(cron = "0 0 2 * * ?") // 매일 오전 2시
@SchedulerLock(name = "cleanupTemporaryFiles", lockAtLeastFor = "PT5M")
@Transactional
public void cleanupTemporaryFiles() {
    // ...
}
```

락을 적용한 작업은 아래 4개다.

| 작업 | 실행 주기 | lockAtLeastFor | lockAtMostFor |
|---|---|---|---|
| 임시 첨부파일 정리 | 매일 02:00 | 5분 | 30분 |
| 보유기간 만료 처리 | 매일 02:00 | 5분 | 30분 |
| 만료 예정 안내 | 매일 10:00 | 5분 | 30분 |
| 오래된 삭제 이력 정리 | 매월 1일 03:00 | 5분 | 30분 |

lockAtMostFor는 메서드마다 지정하지 않고, `@EnableSchedulerLock`의 `defaultLockAtMostFor`로 프로젝트 기본값 30분을 적용했다.

### lockAtLeastFor = 5분

처리할 대상이 없는 날은 배치가 약 6ms 만에 끝난다. 작업이 끝나면 락도 바로 풀리는데, 이때 두 파드의 시계가 조금만 달라도 늦게 깨어난 파드가 풀린 락을 다시 잡아 같은 작업을 한 번 더 실행할 수 있다.

lockAtLeastFor를 두면 작업이 일찍 끝나도 락이 최소 그 시간만큼 유지된다.
ShedLock Redis provider는 락을 해제할 때 lockAtLeastFor 시간이 남아 있으면 키를 지우지 않고, 남은 시간만큼 만료 시간을 다시 설정한다.

실행 주기가 하루 또는 한 달 단위라 5분 동안 락이 걸려 있어도 다음 실행에는 영향이 없다.

### lockAtMostFor = 30분

락을 잡은 파드가 작업 도중 죽으면 락을 해제하지 못한다.
lockAtMostFor가 지나면 락이 자동으로 풀리므로 락이 영구적으로 남는 것을 막을 수 있다.

## 적용 후 확인

```
[파드 A] 02:00:00.043  Locked 'cleanupTemporaryFiles', lock will be held at most until 17:30:00Z
[파드 B] 02:00:00.044  Not executing 'cleanupTemporaryFiles'. It's locked.
```

파드 A가 락을 잡고 배치를 실행했고, 1ms 뒤 파드 B는 락이 걸려 있어 실행하지 않았다.

로그 앞의 시각은 KST, 락 만료 시각은 UTC다. 02:00 KST는 UTC로 17:00이므로, `17:30:00Z`는 lockAtMostFor 30분이 적용된 값이다.

## 한계

**lockAtMostFor보다 작업이 오래 걸리면 다시 중복 실행될 수 있다**

작업이 아직 실행 중인데 lockAtMostFor가 지나면 락이 먼저 풀리고, 다른 파드가 락을 잡아 같은 작업을 실행할 수 있다.
처리 대상이 없는 날의 실행 시간은 확인했지만, 처리 대상이 많을 때의 실행 시간은 아직 측정하지 못했다. 보유기간 만료 처리처럼 데이터가 쌓일수록 처리량이 늘어나는 작업은 실행 시간을 주기적으로 확인해야 한다.

**작업 자체의 멱등성은 여전히 필요하다**

ShedLock은 같은 시각에 여러 파드가 실행하는 것을 막을 뿐, 정확히 한 번 실행을 보장하지는 않는다.
중복으로 실행되더라도 결과가 같도록 작업을 만들어 두는 것이 함께 필요하다.

## 마무리

로컬이나 단일 서버에서는 문제없던 `@Scheduled`가 파드를 2개로 늘리는 순간 2번 실행되었다.

이번에는 낙관적 락 예외 덕분에 데이터 문제 없이 지나갔지만, 에러가 나지 않는 배치였다면 중복 실행을 알아채기 더 어려웠을 것이다.

다중 인스턴스 환경에서 스케줄러를 쓴다면 "이 작업이 동시에 두 번 실행되면 어떻게 되는가"를 먼저 확인해 봐야 한다.
