---
title: '[Ruby] `undefined method` 에러'
description: GitHub Blog 만들기 (2)에서 로컬에서 블로그 실행 환경을 위해 Ruby와 Jekyll을 설치하고 다운로드 받은 테마를 적용후 실행했는데 `undefined method` 에러가 발생했다.Ruby 공식 문서를 확인한 결과 해당 메서드는 3.2.0 버전 이후로는 제거되었다는 사실을 알게 되었다.
categories:
 - 트러블슈팅
tags:
 - GitHub Blog
 - Ruby
---

**[GitHub Blog 만들기 (2)](http://yukmekim.github.io/github%20blog/2024/03/26/github-blog-2/)** 에서 로컬에서 블로그 실행 환경을 위해 Ruby와 Jekyll을 설치하고 다운로드 받은 테마를 적용후 실행했는데 `undefined method` 에러가 발생했다.

해당 오류를 수정하는데 꽤나 삽질을해서 혹시나 다른 테마를 적용하다 같은 에러를 만났을때 도움이 될 수 있을거 같다고 생각하여서 추가 포스팅을 하기로 했다.

[Ruby 공식 문서](https://ruby-doc.org/stdlib-2.7.1/libdoc/pathname/rdoc/Pathname.html)를 확인한 결과 해당 메서드는 3.2.0 버전 이후로는 제거되었다는 사실을 알게 되었다. (Ruby 2.1.0 버전 이상으로 쓰면 된다며:flushed: 문서 업데이트좀.. ㅠㅠ)

![Desktop Preview](/assets/images/post/gitblog_3/ruby_docs_untaint.png)

`bundle update` 로 bundle을 최신 버전으로 update를 해도 같은 오류가 반복되고 있다.
Ruby 버전을 다운그레이드 해야하나 고민하던중 GemFile에서 bundle 버전까지 관리한다는 사실이 기억났다.

우리가 다운로드 받은 테마는 GemFile까지 포함하고 있었고 그걸 그대로 옮겨 붙였으니 해당 파일에서 관리되는 `1.17.1` 버전의 bundle을 사용하고 있었고 해당 버전을 update받은 bundle 버전으로 수정했다.

![Desktop Preview](/assets/images/post/gitblog_3/theme_bundle_version.png)

Jekyll 서버를 재시작후 에러 없이 콘솔창에 `Server Running...` 문구가 출력되는걸 확인 할 수 있다.