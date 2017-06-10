---
layout: post
title: "Heroku에 Flask 앱 배포하기"
description: "heroku는 다 간편한데 설정이 귀찮고 설명이 불친절하다(지극히 주관적)"
tags: [heroku, python, flask]
---

# 들어가기 전에

## 크고 아름다운 삽질
* 주말동안 할 것이 없어서, Flask를 만져봄
    * 플라스크는 처음이고, 그 전에는 Django 썼음
* 그러다, 문득 [Heroku](https://www.heroku.com/home) 에 배포해볼까 해서 이것 저것 만져봄
* ...
* __그리고 두 시간을 설정에 씀__
    * 작년엔 3일쯤 걸렸으니 그 때보단 진보함(아마)

## 그래서
* 손쉽게 재활용 가능한 뼈다귀를 만들었다
    * [깃허브 레포지터리](https://github.com/AwayDay/heroku-flask-prototype)

# Heroku

## 헤로쿠 허로쿠 히어로쿠
* 읽는법은 잘 모르겠고

## 뭐 하는 서비스?
* 기적의 PaaS 서비스
    * 코드랑 설정파일만 만들어서 `git push heroku master` 하면 빌드 디플로이 런 다 해줌
    * 무료로 제공하는 서비스가 의외로 강해서 이것 저것 굴려보기 좋음(이라고 생각)
* 그러나 지금 이쪽 시장은 Docker와 그의 일당들이 다 먹었지롱
    * 불쌍

## 주요 서비스
* dyno
    * Docker 컨테이너와 비슷한 어떤 것이라 생각하면 이해는 빨리 됨
    * 웹 앱이 굴러가는 작고 가벼운 플랫폼
    * 그런데 웹 서버로는 뭘 쓰는지 아직도 모름
        * 사실 관심이 없음
* DB
    * Postgres : 포스구레의 존재를 여기서 처음 알게 되었음
    * Redis
* 기타 등등 수많은 개발/관리 기능 및 에드온

# Python in Heroku
* 기본적으로는 [Getting Started on Heroku with Python](https://devcenter.heroku.com/articles/getting-started-with-python#introduction) 문서를 참고하여 따라 하면 됨
* 문제는 이게 Django라는 것.

## 파이썬을 돌리자!
* 헤로쿠는 원래, Ruby만 돌아가던 서비스였는데, 확장하다보니 Python, Node를 넘어 Java도 굴러감
* 어쨌든 이번 

### 플라스크 프로젝트 폴더 구성
* 이런 형식을 많이들 쓰는지 안 쓰는지는 잘 모름
    * 이 중 헤로쿠 설정에 필요한 파일은 `Procfile` , `requirements.txt` , `runtime.txt`

```
PROJECT_DIR
    PROJECT_NAME
        static
            css
            js
        templates
        __init__.py
    app_start.py
    Procfile
    requirements.txt
    runtime.txt
```

## 파이썬 실행 버전 설정
* 현재 파이썬 2.7.x 와 3.6.1 지원
    * 설정 파일 없으면 기본값 2.7이니 주의합시다.

### runtime.txt
* 파이썬 버전 설정을 위한 파일 이름
* 아래처럼 써주면 됩니다.

```
python-3.6.1
```

## gunicorn
* 헤로쿠 공식 가이드에서는 gunicorn 위에 앱을 얹는 형태 권장
* __gunicorn__ : WSGI HTTP 서버
    * 정말 간단하게 요약 : Tomcat 같은 친구입니다.
* 플라스크 앱을 gunicorn 위에 얹기 위한 설정 파일 필요

### Procfile
* __애플리케이션을 어떻게 굴릴 것인가__ 에 대한 설정 파일 이름
    * 확장자 없음, (스펠링) 프로파일 아님.
    * 이 근본 없는 파일 이름 때문에 시간 좀 많이 쓴 것은 비밀.
* `web: + (실행 명령어)` 형태로 작성
    * 예시 : gunicorn 실행 명령어를 넣어주었다.

```
web: gunicorn prototype:app
```

* 물론 트롤 하고자 하면 `web: python app_start.py` 도 가능할 것

## 의존성
* venv를 써도 안 써도 상관은 없지만, 역시 쓰는 것이 낫다.
* 여하간, 의존성 문서 생성을 위해서는 freeze를 사용

```
pip freeze > requirements.txt
```

# 앱(헤로쿠 프로젝트) 생성 및 배포

## Heroku CLI 사용 시
* (주의) 이 앞에 귀찮음 있음

### 앱 생성
* 웹 대시보드에서도 앱 생성은 가능한데, 다만 그 경우 프로젝트에 git 물리는 작업이 추가로 필요

```bash
$ cd $PROJECT_DIR
$ heroku create
$ heroku rename $PROJECT_NAME
```

### 배포

```bash
$ git push heroku master
```

* 끝?
* 끝.

## GitHub 연동 시
* 앱 생성은 어디서 해도 상관 없음
* 깃허브 계정을 연동시켜두면, 특정 프로젝트(+특정 브랜치)에 소스가 푸시될 때마다 자동으로 빌드/배포하게 할 수 있음
    * (프로젝트) > Deploy