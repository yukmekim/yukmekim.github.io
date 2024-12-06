---
title: Jenkins를 이용한 CI/CD 구축 1
description: 우리 회사는 별도의 배포 시스템이 갖춰져 있지 않다. 소위 말하는 SI 업무를 수행하다보면 정해진 날짜에만 배포를 진행하는 것이 아니라, 클라이언트의 요청으로 급하게 수정되는 건과 당일 또는 퇴근후 급하게 연락이와서 배포를 진행하는 경우도 태반이다. 배포를 진행하는것이 어렵지는 않으나 운영중인 서버 여러대를 일정한 텀을 두고 재시작을 해야한다는 점과 해당 업무를 수행할 수 있는 인력이 별도로 없어 클라이언트쪽에서 배포 진행 확인을 받기까지 무기한 기다림은 거의 나의 몫이었다. 이후 모든 사람이 쉽게 배포를 진행 할 수 있게 자동화를 구축하기로 했다.
categories:
 - DevOps
tags:
 - Jenkins
---


내가 신입 사원으로 입사후 업무를 진행하며 매일 같이 긴장을 놓지 못하는 순간중 하나가, 배포 일정이다.

우리 회사는 별도의 배포 시스템이 갖춰져 있지 않다. 소위 말하는 SI 업무를 수행하다보면 정해진 날짜에만 배포를 진행하는 것이 아니라, 클라이언트의 요청으로 급하게 수정되는 건과 당일 또는 퇴근후 급하게 연락이와서 배포를 진행하는 경우도 태반이다.

배포를 진행하는것이 어렵지는 않으나 운영중인 서버 여러대를 일정한 텀을 두고 재시작을 해야한다는 점과 해당 업무를 수행할 수 있는 인력이 별도로 없어 클라이언트쪽에서 배포 진행 확인을 받기까지 무기한 기다리는 경우도 많은 편이었다.

이후 모든 사람이 쉽게 배포를 진행 할 수 있게 자동화를 구축하기로 했다.

## Jenkins
### Jenkins 설치
자, 배포시스템이 갖춰져 있지 않아 홧김에 CI/CD를 구축해보려 하니 사전 지식조차 없다.  
  
  
<!-- ![Desktop Preview](/assets/images/post/jenkins_1/mudo_1.jpg)   -->
  
  
가장 기본이 되는 **[Jenkins 공식 문서](https://www.jenkins.io/doc)**와 **[젠킨스로 배우는 CI/CD 파이프라인 구축](https://product.kyobobook.co.kr/detail/S000212572110)** 도서를 참고하여 구축에 들어갔다.

도서에서는 윈도우 환경에서 jenkins를 설치해서 진행하는 방법을 소개하고 있지만 팀장님께 건의드릴 aws ec2환경에서 해보는게 좋을거 같고, aws는 무려 **750시간이라는 프리티어 서비스**를 제공하는데 한 번 이용해보려 한다.

![Desktop Preview](/assets/images/post/jenkins_1/jenkins_system_requirements.png)

찾아보니 젠킨스 문서에서 권장하는 최소 사양은 **256MB RAM**에 **2GB의 저장공간**이 필요하다고 나와있다. (도커를 사용할 경우 10GB의 저장공간)  
프리티어에서 ec2서버의 스펙은 1GB의 RAM 최대 20GB의 저장 공간을 제공해주는거 같다.  
  
이전에 SSEN(직장동료)이 프리티어로 젠킨스 구축을 시도해봤는데 서버도 다운되고 콘솔도 많이 느렸다고 알려줬다.  
많이 느려지면 서버 스펙을 조금 올리고 일단은 프리티어로 진행할 계획이다.

AWS EC2에 인스턴스를 만드는 과정은 생략하도록 하겠다.

EC2에 운영체제를 **amazon linux**를 사용하여 **[Jenkins 공식 문서](https://www.jenkins.io/doc/book/installing/linux/)**중 리눅스 설치법을 참고하여 진행하였다.

생성한 EC2 서버에 접속해 자바부터 설치하겠다.
jenkins는 java 기반의 CI/CD 툴이기 때문에 java설치는 필수이다.

**[Jenkins 공식 문서]()**에 지원되는 자바 버전별로 정리되어있는데, 가장 최신 버전의 jenkins를 이용할 계획이기 때문에 java 17 버전을 다운로드 받도록 하겠다.

![Desktop Preview](/assets/images/post/jenkins_1/jenkins_java_version.png)

자바를 설치하기 위해서 **레드햇 계열의 리눅스 배포판에서 사용하는 프로그램(패키지) 설치 관리 도구 yum**을 이용해 설치를 진행했다.

`sudo yum list | grep jdk` 명령어로 jdk를 찾아봤는데 리스트에 없어 aws 문서를 보고 `sudo yum list | grep java` 명령어로 검색해서 설치 했다.

```linux
sudo yum -y update // -y 옵션 자동으로 yes 실행
sudo yum list | grep java // yum 패키지 리스트에서 java 검색
sudo yum install java-17-amazon-corretto // amazon java 17 설치

java -version
```

`java -version`명령어를 입력해서 java 버전 확인이 되면 설치 끝이다!  

![Desktop Preview](/assets/images/post/jenkins_1/jenkins_java_search.png)







