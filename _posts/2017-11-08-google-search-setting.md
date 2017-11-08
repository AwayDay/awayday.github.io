---
layout: post
title: "구글 검색 설정"
description: "검색하니 내 블로그가 나왔어요! 라는 로망이 있었다. 그러나..."
tags: [web]
---

# 들어가기 전에

...그러나 내 블로그는 이름도 (지나치게) 단순하고 포스트 제목들도 하나같이 검색에서 잡힐 리 없는 괴상하고 망측한 문자열들이라(거기다 포스트 주제도 이상하다!) 검색에서 잡히질 않는다.

# Search Console

jekyll + github.io 조합으로 블로그를 생성하면 구글 검색에 잡히질 않는다. 검색이 되게 하려면 다음과 같은 과정을 거쳐야 한다.

## 웹 사이트 추가 및 소유권 확인

1. [서치 콘솔](https://www.google.com/webmasters/tools/home?hl=ko) 접근.
1. __속성 추가__ 버튼을 눌러 블로그 주소를 추가.
1. 진실로 자신의 블로그(웹 사이트)가 맞다는 것을 입증하기 위해 구글에서 제공하는 파일(`file-name.html`)을 내려받아 블로그 프로젝트 폴더 __최상위__ 에 추가하고 커밋 + 푸시.
1. `/{file-name.html}` 페이지에 정상적으로 접근이 된다면 서치 콘솔로 돌아가 확인.

## 사이트맵 추가

1. `site.url` 옵션을 사용하기 때문에, `_config.yml` 파일의 `url` 속성이 정의되어 있어야 한다.

```yml
# 이런 식으로
url: "https://awayday.github.io"
```


1. `sitemap.xml` 파일을 프로젝트 폴더 최상위에 추가한다.
1. 파일 내용은 아래와 같이 하여 커밋 + 푸시 하면 되는데, 몇 몇 테마에서는 `/sitemap.xml` 경로로 접근 시 웬 에러 페이지가 표시된다.

<script src="https://gist.github.com/AwayDay/447442f145a3f3e070dbbe98f13627d6.js"></script>

1. ...레이아웃이 적용되어 그렇다. 이런 경우에는 파일 내용을 아래와 같이 수정하자.
    * `layout: none` 설정이 추가된 것을 확인 가능하다.

<script src="https://gist.github.com/AwayDay/bc8a128b4ea61e6357217db28affcd1f.js"></script>

1. `/sitemap.xml` 페이지가 정상적인 xml 형식으로 표시되는 것을 확인한다.
1. __Sitemaps__ 메뉴에서 __Sitemap 추가/테스트__ 항목을 사용하여 사이트맵이 올바른 포맷을 가지고 있는지 확인한다.
1. __제출__

## 그 이후

* 접수부터 처리 시작에는 3일쯤 걸린다.
* 한 번에 사이트맵의 모든 내용이 처리가 되는 것도 아니고, 첫 처리 이후부터 순차적으로 색인 수가 증가한다.
* 이왕 검색엔진에 붙인 것, [애널리틱스](https://analytics.google.com/analytics) 도 등록해보자.