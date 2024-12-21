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
젠킨스 서버에 올려놓고 github 저장조만 연동해서 자동화를 시킬수 있으면 좋겠지만, github 저장소 연동 부터 소스 패키징에 릴리즈까지 전부 구축해야한다.  
  
젠킨스 파이프라인을 통해 github 저장소의 소스 코드를 받고 **['Pipeline Maven integration'](https://plugins.jenkins.io/pipeline-maven/)**을 이용해서 maven clean후 패키징을 진행후 패키징된 파일을 확인하는 순서로 진행해보도록 하겠다.

## Github 계정 연동
파이프라인에서 github 비공개 저장소에 접근하기 위해서는 jenkins에 credential을 추가 해줘야한다.  
<img src="/assets/images/post/jenkins_2/jenkins_credentials_1.png" width="80%" height="80%"/>
<img src="/assets/images/post/jenkins_2/jenkins_credentials_2.png" width="80%" height="80%"/>

## 파이프라인 코드 작성
### Github 저장소 소스 clone
프라이빗 프로젝트에 접근하기 위한 credential을 이용하여 저장소의 코드 클론을 진행했다.

``` pipeline
pipeline {
    agent any
    
    environment {
        GIT_URL = "저장소 주소"
        POM_FILE = "pom 파일"
        APP_VERSION = "배포 버전을 관리하기 위한 변수"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "${GIT_URL}", branch: "master", poll: true, credentialsId: ""
            }
        }
    }
}
```

`agent any` : 모든 에이전트에서 파이프라인이 동작 가능  
`enviroment` : 환경 변수 설정  
`stages` : 파이프라인 작업 단위(stage)를 정의  

`enviroment`에 파이프라인 동작에 사용할 환경 변수를 선언해두었다. 저장소 url이나 pom파일 위치등은 작업 단위에 직업 작성하는것 보다는 변수를 만들어 사용하는게 편하다.  
  
`stages` 안에 `stage`는 파이프라인의 작업 단위이며 저장소에 Checkout 받는 단계로 정의해두었다.
`${GIT_URL}`은 환경변수에 선언한 저장소 주소이며 `branch`는 clone 받는 브랜치를 설정 `poll` 저장소의 커밋 또는 변경 사항을 자동으로 감지.  
비공개 저장소의 경우 `credentialsId`에 만들어둔 crendential ID를 넣어주면 된다.

해당 파이프라인을 실행했을때 `/var/lib/jenkins/workspace/젠킨스 프로젝트 명/` 디렉토리에 패키징되지 않은 소스를 받을 수 있었다.  
이후 프로젝트를 배포 파일로 패키징 하기 위해서 `Pipeline Maven integration` 플러그인을 사용할 예정이다.

<img src="/assets/images/post/jenkins_2/jenkins_plugins.png" width="80%" height="80%"/>

플러그인을 설치후 `stage`에 [젠킨스 문서에 Pipeline Maven integration](https://plugins.jenkins.io/pipeline-maven/)을 참고하여 Build 단계를 추가했다.  

``` pipeline
stage ('Build') {
    steps {
        withMaven {
            sh "mvn clean package -DskipTests=true -f ${POM_FILE}"
        }
    }
}
```

이전에 빌드한 아티펙트를 clean하고 환경변수에 정의한 pom 파일을 통해 패키징하는 과정이다.

작성한 파이프라인을 실행 시켜 봤다는데,프리티어에서 계속해서 테스트를 진행할수 없게 되었다.  
node agent를 통해서 파이프라인을 실행시켜야하는데 메모리 부족으로인해 node가 오프라인 상태로 유지가 되어 파이프라인이 작동하지 않는 상태이다.

ec2 서버의 메모리를 올려서 계속해서 진행 또는 맥북에 젠킨스를 설치하고 ec2 서버에 소스를 배포하는 방향으로 진행하겠다.