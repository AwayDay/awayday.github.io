---
layout: post
title: "Flask 프로젝트에 Google OAuth 붙이기"
description: "구글은 기      쁘다. 역시 구글 연동        해야 한다"
tags: [python, flask, oauth, google]
---

# 들어가기 전에
* 이번 주는 별 것 없는 내용.

# Flask + Google
* 로그인은 믿을 수 있는(정말 믿을 수 있는지는 개인 판단이지만) 제 3의 기관에 맡기는 것이 가장 마음이 편하다.

## Google OAuth
  인스타그램, 페이스북이나 트위터도 있지만, 국내 국외를 막론하고 가장 범용적인 플랫폼이라 하면 역시 구글인데, 그 근거는 별 것 없고 안드로이드 폰 점유율 하나밖에 없다. 페이스북이나 인스타그램은 내가 별로 안 좋아하고, 트위터라면 좋아는 하는데 플랫폼이 오늘내일 하고 있어서.

  그런 고로, 오늘 해본 것은 플라스크 웹 애플리케이션에 구글 로그인을 붙이는 일, 생각보다 얼마 안 걸렸다.

## Google API Client Libraries > Python
* [oauth2client 가이드 링크](https://developers.google.com/api-client-library/python/guide/aaa_oauth)

  구글 개발자 페이지에서는 친절하게 파이썬을 위한 라이브러리와 가이드를 제공하고 있는데, 이 __oauth2client__ 라이브러리로도 웬만큼 빠르게 로그인 연동을 할 수 있다. 그런데 구글은, 아주 고맙게도, 장고와 플라스크를 위한 라이브러리를 추가로 준비하고 있다. 

### flask_util module
  플라스크 연동을 위해서는 아래의 가이드를 보자.

* [flask_util module 가이드 링크](http://oauth2client.readthedocs.io/en/latest/source/oauth2client.contrib.flask_util.html)

  위 링크를 보고 그대로 따라서 하면 된다. 로그인 페이지를 따로 만들 필요도 없다, 은혜로운 `flask_util module` 께서 다 해주신다. 그런데 내가 영어를 못 해서, 나중에 다시 써먹을 수 있게 조금 풀어 써 보았다.

### 사용 설정

  현재 시각이 자정을 넘긴 관계로 구글 API 사용은 생략, 바로 코드를 넣었다.

```python
from oauth2client.contrib.flask_util import UserOAuth2

app = Flask(__name__)

app.config['SECRET_KEY'] = '아무 문자열이나 넣으면 된다. 용도는 대강 알기 때문에 설명 못 함.'

# API 사용을 위해서 아래와 같이 관련 정보를 담은 json 파일(개발자 페이지에서 받을 수 있다)을 사용할 수 있고,
app.config['GOOGLE_OAUTH2_CLIENT_SECRETS_FILE'] = 'client_secrets.json'

# 이렇게 코드에 직접 넣을 수도 있다.
app.config['GOOGLE_OAUTH2_CLIENT_ID'] = 'your-client-id'
app.config['GOOGLE_OAUTH2_CLIENT_SECRET'] = 'your-client-secret'

oauth2 = UserOAuth2(app)
```

  설정은 플라스크 app 선안하는 부분 아래에, 구글 OAuth 사용을 위한 클라이언트 아이디, 시크릿을 추가로 입력하면 끝난다.

### 실전 사용

#### 로그인
  `flask_util module` 을 사용하면 로그인 페이지나 콜백 페이지를 따로 만들 필요가 없다. 플라스크 유틸 모듈에서 아래의 경로를 자동으로 생성한다.

* `/oauth2authorize`
* `/oauth2callback`

  로그인을 성공적으로 완료하기 위해서는 구글 API 관리 페이지에서 콜백 주소를 입력해두어야 하는 것을 잊지 말자, 안 그러면 에러 페이지가 우리를 환한 미소로 맞아준다. 로컬에서 테스트 한다면 아래와 같이 등록하는 것으로 오케이.

* `http://127.0.0.1:5000/oauth2callback`

  이제 `/oauth2authorize` 경로로 접근할 경우, 익숙한 구글의 로그인 페이지가 보일 것이다. 라이브러리 다운받고 로그인 페이지 띄우는데 5분쯤 걸린다.

#### 로그인이 필요한 자원에 접근

  로그인이 필요한 경로에 아래의 애노테이션을 추가하면, 로그인 하지 않은 유저의 접근 시 로그인 페이지로 던져줄 수 있다.

* `@oauth2.required`

  아래는 구글에서 제공하는 예제 코드이다.

```python
@app.route('/info')
@oauth2.required
def info():
    return "Hello, {} ({})".format(oauth2.email, oauth2.user_id)
```

  `/info` 에 접근했을 때, `@oauth2.required` 에노테이션이 로그인 여부를 판별하여 로그인 페이지로 던져주거나, 정상적으로 페이지를 띄워준다.

  로그인 여부가 선택적일 경우에는 아래와 같은 방법이 있다.

```python
@app.route('/optional')
def optional():
    if oauth2.has_credentials():
        return '로그인!'
    else:
        return '로그인 하세여!'
```

----

이제 나머지는 더 찾아봐야지.

----

# 기타 잡다한 신변잡기

## 서적 구매 현황
* [오쿠노 미키야, 원리부터 배우는 관계형 데이터베이스 실전 입문(2016), 위키북스](http://www.yes24.com/24/goods/29343536?scode=032&OzSrank=1) 구매.
    * 자고로 독서는 서재에 있는 도서를 대상으로 하는 것이라. 나는 지금 읽을 책이 밀리고 밀렸기 때문에 읽지 않았다.
    * 그리고, 역시 위키북스는 책등을 예쁘게 뽑는다.
* [와카키 타미키, 나노하 양과자점의 좋은 일 2(2016), 학산문학사](http://www.yes24.com/24/goods/34969536?scode=032&OzSrank=2) / [와카키 타미키, 나노하 양과자점의 좋은 일 3(2017), 학산문학사](http://www.yes24.com/24/goods/34969536?scode=032&OzSrank=2) 구매
    * 역시 타선생님, 이번에도 좋은 만화가 될 것 같다.