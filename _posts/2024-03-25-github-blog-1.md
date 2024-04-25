---
title: Github Blog 만들기 (1)
description: 깃헙 블로그(Github blog)의 정식 명칭은 깃헙 페이지(Github page)입니다. 개발자들이 이용하는 오픈소스 라이브러리 Github에서 제공하는 웹 호스팅 서비스로 사용자가 무료로 자유롭게 사이트를 제작하고 운영할 수 있는 서비스를 제공해주는 것 입니다.
categories:
 - Github Blog
tags:
---

회고와 공부한 내용을 정리할 공간이 필요해서 노션 사용을 시작했지만 카테고리별로 나눠 정리를 해도 히스토리 관리가 힘들어 블로그에 정리하기로 결정했다.

![Desktop Preview](/assets/images/post/gitblog/chimhaha.png)

vlog나 티스토리 사용을 고려해봤으나 우린 개발자니까 조금 더 친화적이고(?) 커스텀이 자유로운 Github Blog 만드는 과정을 공부하며 이전에 노션에 기록했던 내용은 차차 옮기기로하고 이번 포스팅에서는 Github 블로그에 대해 작성 하도록 하자.

## Github Blog(깃허브 블로그)란?

깃헙 블로그(Github blog)의 정식 명칭은 **[깃헙 페이지(Github page)](https://docs.github.com/ko/pages/quickstart)**다.

개발자들이 이용하는 오픈소스 라이브러리 Github에서 제공하는 웹 호스팅 서비스로 사용자가 무료로 자유롭게 사이트를 제작하고 운영할 수 있도록 서비스를 제공해주는 것으로, 깃 블로그를 만들기 위해서는 Github 계정이 필요함으로 Github 계정이 없다면 **[Github](https://github.com/)**으로 접속하여 계정생성(회원가입)을 먼저 진행하도록 하자.

## Github Repository 생성
생성한 계정으로 로그인후 프로필(https://github.com/{username})에서 Repository를 생성한다.

![Desktop Preview](/assets/images/post/gitblog/git-repository.png)

Repository를 생성할때 **[Github docs](https://docs.github.com/ko/pages/quickstart)**에 명시 되어 있는데로 `{username}.github.io`로 작성 해야 한다.

 내용을 무시하고 다른 이름으로 Repository를 만들어 봤지만 정상 동작하지 않으며 사용자 고유 식별과 Github와의 원활한 통합을 위해 Repository 이름을 `{username}.github.io`로 작성 해야 한다고 하니 고생하지 말고 개발 문서를 따르자.. 

생성한 Repository로 이동하여 Github Page 생성을 진행하도록 하자. Setiings를 클릭하여 좌측 하단부의 Github Page에 Repository 이름과 동일한 주소 등록을 완료하면 해당 주소가 Github 블로그 주소가 된다.

다음 포스팅에서는 jekyll 테마 사용을 위한 ruby 설치에 대해 정리하도록 하겠다.