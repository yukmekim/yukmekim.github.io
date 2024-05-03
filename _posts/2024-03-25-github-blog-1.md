---
title: GitHub Blog 만들기 (1)
description: 깃헙 블로그(GitHub blog)의 정식 명칭은 깃헙 페이지(GitHub page)입니다. 개발자들이 이용하는 오픈소스 라이브러리 GitHub에서 제공하는 웹 호스팅 서비스로 사용자가 무료로 자유롭게 사이트를 제작하고 운영할 수 있는 서비스를 제공해주는 것 입니다.
categories:
 - GitHub Blog
tags:
---

회고와 공부한 내용을 정리할 공간이 필요해서 노션 사용을 시작했지만 카테고리별로 나눠 정리를 해도 히스토리 관리가 힘들어 블로그에 정리하기로 결정했다.

![Desktop Preview](/assets/images/post/gitblog_1/chimhaha.png)

vlog나 티스토리 사용을 고려해봤으나 우린 개발자니까 조금 더 친화적이고(?) 커스텀이 자유로운 Github Blog 만드는 과정을 공부하며 이전에 노션에 기록했던 내용은 차차 옮기기로하고 이번 포스팅에서는 Github 블로그에 대해 작성 하도록 하자.

## GitHub Blog(깃허브 블로그)란?

깃헙 블로그(GitHub blog)의 정식 명칭은 **[깃헙 페이지(GitHub page)](https://docs.github.com/ko/pages/quickstart)**다.

개발자들이 이용하는 오픈소스 라이브러리 GitHub에서 제공하는 웹 호스팅 서비스로 사용자가 무료로 자유롭게 사이트를 제작하고 운영할 수 있도록 서비스를 제공해주는 것으로, 깃 블로그를 만들기 위해서는 GitHub 계정이 필요하니 GitHub 계정이 없다면 **[GitHub](https://github.com/)**으로 접속하여 계정생성(회원가입)을 먼저 진행하도록 하자.

## GitHub Repository 생성
생성한 계정으로 로그인후 프로필(https://github.com/{username})에서 Repository를 생성한다.

![Desktop Preview](/assets/images/post/gitblog_1/git-repository.png)

Repository를 생성할때 **[GitHub docs](https://docs.github.com/ko/pages/quickstart)**에 명시 되어 있는데로 `{username}.github.io`로 작성 해야 한다.

 내용을 무시하고 다른 이름으로 Repository를 만들어 봤지만 정상 동작하지 않으며 사용자 고유 식별과 GitHub와의 원활한 통합을 위해 Repository 이름을 `{username}.github.io`로 작성 해야 한다고 하니 고생하지 말고 문서를 따르자.. 

![Desktop Preview](/assets/images/post/gitblog_1/git-page-settings.png)

생성한 Repository로 이동하여 Github Page 생성을 진행하도록 하자. Setiings를 클릭하여 좌측 하단부의 Github Page에 Repository의 Source Branch를 main Branch 설정해주면 Github 블로그 주소가 완성 된다.

```html
<html>
	<body>
		육각형!의 블로그
	</body>
</html>
```

생성한 Repository를 local에 clone 받은후 root path에 index.html 파일을 만들고 다음 코드를 넣은 다음 git repository에 push 해보자

GitHub Pages로 등록된 Repository는 저장소 이벤트(예: 푸시, 풀 리퀘스트, 릴리즈 등)를 감지하여 **[Github Actions](https://docs.github.com/ko/actions)**를 통해 자동으로 빌드 및 배포가 된다.

![Desktop Preview](/assets/images/post/gitblog_1/git-actions.png)

Actions에 빌드가 완료되었으면 블로그 주소(`{username}.github.io`)로 접속해서 '육각형!의 블로그'가 화면에 보인다면 성공이다.

다음 포스팅에서는 빈폐이지를 블로그답게 만들어줄 Jekyll 테마를 적용하기 위해 Ruby 설치에 대해 정리하도록 하겠다.