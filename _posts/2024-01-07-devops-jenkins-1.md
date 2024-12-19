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
<!-- ![Desktop Preview](/assets/images/post/jenkins_1/mudo_1.jpg)   -->
  
배포시스템이 갖춰져 있지 않아 홧김에 CI/CD를 구축해보려 하니 사전 지식조차 없기에, 가장 기본이 되는 **[Jenkins 공식 문서](https://www.jenkins.io/doc)**와 **[젠킨스로 배우는 CI/CD 파이프라인 구축](https://product.kyobobook.co.kr/detail/S000212572110)** 도서를 참고하여 구축에 들어갔다.

도서에서는 윈도우 환경에서 jenkins를 설치해서 진행하는 방법을 소개하고 있어 젠킨스 내부 시스템에 대한 참고로 사용하고, 팀장님꼐 건의 드릴 ec2 서버를 이용해 보기로 했다. aws는 무려 **750시간이라는 프리티어 서비스**를 제공하는데 한 번 이용해보려 한다.

<!-- ![Desktop Preview](/assets/images/post/jenkins_1/jenkins_system_requirements.png) -->
<img src="/assets/images/post/jenkins_1/jenkins_system_requirements.png" width="80%" height="80%"/>

찾아보니 젠킨스 문서에서 권장하는 최소 사양은 **256MB RAM**에 **2GB의 저장공간**이 필요하다고 나와있다. (도커를 사용할 경우 10GB의 저장공간)  
프리티어에서 ec2서버의 스펙은 1GB의 RAM 최대 20GB의 저장 공간을 제공해주는거 같다.  
  
이전에 SSEN(직장동료)이 프리티어로 젠킨스 구축을 시도해봤는데 서버도 다운되고 콘솔도 많이 느렸다고 알려줬다.  
많이 느려지면 서버 스펙을 조금 올리고 일단은 프리티어로 진행할 계획이다.

AWS EC2에 인스턴스를 만드는 과정은 생략하도록 하겠다.

EC2에 운영체제를 **amazon linux**를 사용하여 **[Jenkins 공식 문서](https://www.jenkins.io/doc/book/installing/linux/)**중 리눅스 설치법을 참고하여 진행하였다.

생성한 EC2 서버에 접속해 자바부터 설치하겠다.
jenkins는 java 기반의 CI/CD 툴이기 때문에 java설치는 필수이다.

**[Jenkins 공식 문서]()**에 지원되는 자바 버전별로 정리되어있는데, 가장 최신 버전의 jenkins를 이용할 계획이기 때문에 java 17 버전을 다운로드 받도록 하겠다.

<!-- ![Desktop Preview](/assets/images/post/jenkins_1/jenkins_java_version.png) -->
<img src="/assets/images/post/jenkins_1/jenkins_java_version.png" width="80%" height="80%"/>


자바를 설치하기 위해서 **레드햇 계열의 리눅스 배포판에서 사용하는 프로그램(패키지) 설치 관리 도구 yum**을 이용해 설치를 진행했다.

`sudo yum list | grep jdk` 명령어로 jdk를 찾아봤는데 리스트에 없어 aws 문서를 보고 `sudo yum list | grep java` 명령어로 검색해서 설치 했다.

```linux
sudo yum -y update // -y 옵션 자동으로 yes 실행
sudo yum list | grep java // yum 패키지 리스트에서 java 검색
sudo yum install java-17-amazon-corretto // amazon java 17 설치

java -version
```

`java -version`명령어를 입력해서 java 버전 확인이 되면 자바 설치가 된것이다.  

![Desktop Preview](/assets/images/post/jenkins_1/jenkins_java_search.png)

다음으로 jenkins를 설치할 차례다.
Red Hat/Alma/Rocky 설치 매뉴얼을 따라서 진행하는데 `Long Term Support release`와 `Weekly release` 두 가지 배포 버전이 있다.

아래 표가 jenkins 문서를 찾아서 두 가지 배포 버전을 정리한 내용이다.

| | Long Term Support release | Weekly release | 
| --- | --- | --- |
| 릴리스 주기 |	3개월마다 업데이트, 안정화 | 매주 새로운 버전 릴리스 |
| 안정성 |	매우 안정적, 프로덕션 환경에 적합 | 기능이 실험적일 수 있어 불안정한 경우 있음 |
| 보안 패치 및 버그 수정 |	1년 이상 지원, 보안 패치와 안정성 수정 제공 | 최신 기능이 빠르게 추가되지만, 안정성은 부족할 수 있음 |
| 주요 사용 환경 |	프로덕션 환경, 안정성 중시 | 새로운 기능을 실험하거나 테스트하는 개발 환경 |
| 새로운 기능 |	비교적 천천히 추가 | 최신 기능과 개선사항이 빠르게 추가 |

매주 새로운 버전을 배포 받는것보다 안정적인 작업을 위해 `Long Term Support release` 버전으로 문서를 따라 설치를 진행했다.

``` linux
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-17-openjdk
sudo yum install jenkins
sudo systemctl daemon-reload
```

설치후 jenkins를 실행하는것까지 친절하게 문서에 나와있다.
```linux
sudo systemctl enable jenkins // 젠킨스 부팅시 시작 상태로 전환

sudo systemctl start jenkins // 젠킨스 실행 명령어

sudo systemctl status jenkins // 젠킨스 서비스 상태 확인
```

`sudo systemctl start jenkins` 명령어로 시작후 `sudo systemctl status jenkins` 다음 명령어로 확인했을때 
아래와 같이 출력되면 현재 젠킨스가 실행중이라는 뜻이다.

<img src="/assets/images/post/jenkins_1/jenkins_status.png" width="60%" height="20%"/>

자 이제 jenkins를 설치해서 실행까지 해줬으니 한번쯤은 봤을 젠킨스 화면을 보러가보자
jenkins는 설치후 기본적으로 8080포트로 포워딩 된다.

퍼블릭DNS에 해당 포트로 접속을 하면 초기에는 들어가지지 않는다.  
ec2에 인스턴스를 추가하면 보안그룹이 생기는데 기본적으로 ssh 접근 포트인 22번 포트만 열려있는 상태이기 때문이다.  
해당 포트로 퍼블릭DNS에 접근하기 위해서는 만들어둔 ec2 인스턴스 보안그룹에 8080포트가 열려있어야 하니 보안그룹 인바운드 규칙에 8080포트를 추가해주자.


<img src="/assets/images/post/jenkins_1/aws_tcp.png" width="80%" height="30%"/>

포트를 추가하고 퍼블릭DNS에 8080포트로 접속을 하게되면 아래와 같은 화면을 볼 수 있다.

<img src="/assets/images/post/jenkins_1/jenkins_main_1.png" width="80%" height="80%"/>

친절한 jenkins씨의 설명을 따라 jenkins 설치 경로에 `initialAdminPassword`을 찾아 붙여 넣어준다.  
계속 해서 진행해보면 `Getting Started` 화면이 나타나는데 여기도 두가지 옵션이 있다.  
  
**Install suggested plugins** : jenkins에서 제안하는 추천 플러그인을 다운로드  
**Select plugins to install** : 직접 필요한 플러그인을 다운로드  
  
처음 jenkins를 이용해보기에 `Install suggested plugins` 옵션을 선택하여 자동으로 추천되는 플러그인들을 설치후 프리티어 서버가 무거워지거나
필요한 플러그인이 있을 경우 추가적으로 조치하는 방향으로 갈 예정이다.

<img src="/assets/images/post/jenkins_1/jenkins_main_2.png" width="80%" height="80%"/>

플러그인 설치가 완료되면 jenkins에 접속할때 사용할 계정을 생성 또는 스킵할수있는데 간단한게 사용할 6anglebro 계정을 만들었다.

<img src="/assets/images/post/jenkins_1/jenkins_main_3.png" width="80%" height="80%"/>

생성한 계정으로 jenkins에 로그인하면 익숙한 그 화면을 볼 수 있다.
jeknins 설치만 끝났는데 벌써 구축 끝낸거 같은 기분

이어서 두번째 포스팅에서부터 위에 구매한 책을보고 파이프 라인을 작성하여 본격적으로 자동화 구축에 들어가도록 하겠다.
