---
title: GitHub Blog 만들기 (2)
description: GitHub Pages는 Jekyll을 사용하여 정적 사이트를 구축하고 호스팅하는 데 필요한 기술을 제공하기 때문에 Jekyll은 GitHub Pages에서 정적 웹사이트를 운영하기 위한 필수 요소라고 볼 수 있다. GitHub Pages에서 직접 지원하는 기본 테마들을 사용할 경우 반드시 로컬 환경에 Ruby와 Jekyll을 설치하지 않아도 되지만, 로컬에서 사이트를 전체적으로 미리 보거나 맞춤 테마를 적용하고자 할 때는 필요하다.
categories:
 - GitHub Blog
tags:
---

지난 포스팅에서 블로그로 사용할 GitHub Repository를 만들어 GitHub Actions를 통해 자동 빌드 및 배포가 되는것까지 확인했다.

GitHub Pages는 Jekyll을 사용하여 정적 사이트를 구축하고 호스팅하는 데 필요한 기술을 제공하기 때문에 Jekyll은 GitHub Pages에서 정적 웹사이트를 운영하기 위한 필수 요소라고 볼 수 있다.

GitHub Pages에서 직접 지원하는 기본 테마들을 사용할 경우 반드시 로컬 환경에 Ruby와 Jekyll을 설치하지 않아도 되지만, 로컬에서 사이트를 전체적으로 미리 보거나 맞춤 테마를 적용할때 필요하니 Ruby부터 설치하도록 하자

## Jekyll Theme
### Jekyll Theme 다운로드
지금 보고있는 블로그처럼 여러 사용자가 만들어둔 Jekyll Theme가 있다. 직접 페이지를 만들어서 운영할수도 있지만, 테마를 다운 받아 틀안에서 나만의 블로그로 커스텀을 거쳐 사용하도록 하겠다.

* [http://jekyllthemes.org/](http://jekyllthemes.org/)

많은 테마중 NexT 테마를 사용하기로 했다. (가장 깔끔하고 커스텀이 자유로워 보였다...)  
다른 테마를 선택하더라도 큰 틀은 비슷하고 디테일이 다르기 때문에 원하는 테마를 자유롭게 골라보자

### Ruby 설치
우리가 사용해야할 Jekyll은 Ruby로 작성된 정적 사이트 생성기로 Jekyll을 사용하기 위해서는 Ruby를 설치해야한다.

그럼 Ruby를 다운로드 받아 Jekyll Theme를 블로그에 입혀보자

* [https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)

위에 Ruby for Windows 다운로드 페이지로 들어가서 버전을 선택해서 다운로드하면 된다.

**여기서 중요한건 테마를 적용할때 Ruby 버전이 영향을 줄 수도 있다는 것이다.**{: style="color: #f74e62"}  
우리가 선택한 테마들은 특정 버전의 Ruby에서 가장 잘 작동되도록 설계가 되어 호환이 안될수도 있다.

![Desktop Preview](/assets/images/post/gitblog_2/nexT-ruby-version.png)

내가 선택한 테마의 경우 개발자가 친절하게 Ruby 2.1.0 버전 또는 그 이상으로 사용하라고 적어놨다.

![Desktop Preview](/assets/images/post/gitblog_2/ruby-download.png)

가장 최신 버전의 Ruby를 다운로드 받아 별다른 설정 없이 next만 눌러 설치를 완료후 window search에 `Start Command Prompt with Ruby` 라고 검색해보면 못보던 커맨드 창이 생긴걸 볼 수 있다.

커맨드창에 아래 명령어를 실행하여 설치된 Ruby의 버전을 확인할 수 있다.

```
ruby --version
```

정상적으로 설치되어 Ruby의 버전 정보를 확인 했다면 지난 시간에 GitHub에서 Clone 받은 블로그의 로컬 경로로 이동해 Jekyll을 설치하도록 하자

### Jekyll 설치
Ruby의 표준 패키지 관리 도구인 `gem` 을 통해 **Jekyll 패키지**와 **Bundler**, 작업을하며 매번 Repository에 push후 빌드된 페이지를 보는것은 불편하니 로컬에서 웹 서버 역할을 해줄 **WEBrick**도 함께 설치해주자

```
gem install jekyll bundler
gem install webrick
```

### Jekyll Theme 적용
이제 Ruby에서 로컬 환경해서 블로그 실행시 필요한 패키지를 다운 받았으니 Theme를 적용 시켜보자
지난 시간에 GitHub Actions로 자동 빌드/배포를 테스트하기 위해 작성하였던 index.html 파일을 삭제후 다운로드 받은 Theme를 통째로 옮겨주자

```
bundle exec jekyll serve
```

위에 명령어를 통해 Gemfile에 정의된 gem 환경에서 Jekyll 사이트를 로컬 서버에 빌드하고 실행후 Jekyll의 기본 설정 포트 *127.0.0.1:4000*로 접속하면 적용된 테마가 화면에 나와야 하는데... 뭐요 이게..?

```
undefined method `untaint' for "C:/Users/{LocalPath}/jekyll-theme-next":String (NoMethodError)
    current  = File.expand_path(SharedHelpers.pwd).untaint
```

다음 포스팅에서는 테마 적용후 로컬 빌드시 발생한 에러에 해결에 대해 정리해야겠다..
