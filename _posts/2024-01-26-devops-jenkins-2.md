---
title: Jenkins를 이용한 CI/CD 구축 2
description: 지난번 젠킨스 설치에 이어서 배포 라인을 구축하기 위해 신규 사용자 추가 및 필요한 플러그인 설치를 진행후
 파인프라인에 간단한 스크립트를 만들어서 프로젝트 패키징까지 진행해보겠다.
categories:
 - DevOps
tags:
 - Jenkins
---

aws ec2에 젠킨스 서버를 구축하고 사용자 등록까지 진행했다.

젠킨스 서버에 올려놓고 github 저장조만 연동해서 자동화를 시킬수 있으면 좋겠지만, github 저장소 연동 부터 소스 패키징에 릴리스까지 전부 구축해야한다.

젠킨스 파이프라인을 통해 github 저장소의 소스 코드를 받고 **'Pipeline Maven integration'**[https://plugins.jenkins.io/pipeline-maven/]을 이용해서 maven clean후 패키징을 진행후 패키징된 파일을 확인하는 순서로 진행해보도록 하겠다.

### Github 계정 연동

jenkins 플러그인중 **'Pipeline Maven integration'**[https://plugins.jenkins.io/pipeline-maven/]을 이용해서 maven clean후 패키징을 진행