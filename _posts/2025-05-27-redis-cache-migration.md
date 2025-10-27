---
title: 'JWT Refresh Token을 Redis에서 관리하기'
description: 기존 서비스에서 RDBMS로 Refresh Token을 관리하면서 매 요청마다 발생하는 조회 쿼리와 만료된 토큰을 정리하기 위한 배치 스케줄러 관리가 부담스러웠다. 이번 기회에 Redis의 In-Memory 특성과 TTL 자동 만료 기능을 활용하여 이 문제를 개선해보려고 한다.
categories:
 - Backend
tags:
 - Redis
 - Spring Boot
---