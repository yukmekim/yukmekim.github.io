---
title: Github Blog 만들기 (2)
description: GitHub Pages는 Jekyll을 사용하여 정적 사이트를 구축하고 호스팅하는 데 필요한 기술을 제공하기 때문에 Jekyll은 GitHub Pages에서 정적 웹사이트를 운영하기 위한 필수 요소라고 볼 수 있다. GitHub Pages에서 직접 지원하는 기본 테마들을 사용할 경우 반드시 로컬 환경에 Ruby와 Jekyll을 설치하지 않아도 되지만, 로컬에서 사이트를 전체적으로 미리 보거나 맞춤 테마를 적용하고자 할 때는 필요하다.
categories:
 - Github Blog
tags:
---

지난 포스팅에서 블로그로 사용할 GitHub Repository를 만들어 GitHub Actions를 통해 자동 빌드 및 배포가 되는것까지 확인했다.

GitHub Pages는 Jekyll을 사용하여 정적 사이트를 구축하고 호스팅하는 데 필요한 기술을 제공하기 때문에 Jekyll은 GitHub Pages에서 정적 웹사이트를 운영하기 위한 필수 요소라고 볼 수 있다.

GitHub Pages에서 직접 지원하는 기본 테마들을 사용할 경우 반드시 로컬 환경에 Ruby와 Jekyll을 설치하지 않아도 되지만, 로컬에서 사이트를 전체적으로 미리 보거나 맞춤 테마를 적용할때 필요하니 Ruby부터 설치하도록 하자

## Jekyll Theme 적용
### Jekyll Theme 다운로드
지금 보고있는 블로그처럼 여러 사용자가 만들어둔 Jekyll Theme가 있다. 직접 페이지를 만들어서 운영할수도 있지만, 테마를 다운 받아 틀안에서 나만의 블로그로 커스텀을 거쳐 사용하도록 하겠다.

* [http://jekyllthemes.org/](http://jekyllthemes.org/)

많은 테마중 NexT 테마를 사용하기로 했다. (가장 깔끔하고 커스텀이 자유로워 보였다...)  
다른 테마를 선택하더라도 큰 틀은 비슷하고 디테일이 다르기 때문에 원하는 테마를 자유롭게 골라보자

### Ruby 설치
우리가 사용해야할 Jekyll은 Ruby로 작성된 정적 사이트 생성기로 Jekyll을 사용하기 위해서는 Ruby를 설치해야한다.

그럼 Ruby를 다운로드 받아 Jekyll Theme를 블로그에 입혀보자

* [https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)

위에 Windows Ruby 다운로드 페이지로 들어가서 버전을 선택해서 다운로드하면 된다.

**여기서 중요한건 테마를 적용할때 Ruby 버전이 영향을 줄 수도 있다는 것이다.**{: style="color: #f74e62"}  
우리가 선택한 테마들은 특정 버전의 Ruby 버전에서 가장 잘 작동되도록 설계가 되어 호환이 안될수도 있다.

![Desktop Preview](/assets/images/post/gitblog_2/nexT-ruby-version.png)

내가 선택한 테마의 경우 개발자가 친절하게 Ruby 2.1.0 버전 또는 그 이상으로 사용하라고 적어놨다.

![Desktop Preview](/assets/images/post/gitblog_2/ruby-download.png)

가장 최신 버전의 Ruby를 다운로드 받아 별다른 설정 없이 next만 눌러 설치를 완료후 window search에 `Start Command Prompt with Ruby` 라고 검색해보면 못보던 커맨드 창이 생긴걸 볼 수 있다.

커맨드 창을 실행시켜 1편에서 GitHub에서 Clone 받은 블로그의 로컬 경로로 이동해 Jekyll을 설치하도록 하자

### Jekyll 설치
Ruby의 표준 패키지 관리 도구인 `gem` 을 통해 Jekyll 패키지와 Bundler를 다운로드 하자

```
gem install jekyll bundler
```

Bundler는 Gemfile.lock 파일을 생성하고 프로젝트에 설치된 gems의 정확한 버전 정보를 기록한다고하니 의존성 관리를 위해 함께 추가해주자

