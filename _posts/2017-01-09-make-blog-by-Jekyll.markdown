---
layout: post
title: "Jekyll과 함께 블로그 만들기"
description: "Jekyll을 사용하여 GitHub Page에 블로그를 올려 보았다."
categories: 만들기
tags: [ruby, github, web, jekyll]
---

## 간략한 순서
1. Github 레포지터리 생성
2. 로컬 저장소에 클론
3. Jekyll 프로젝트 생성
4. 커밋 & 푸시

## Github 레포지터리 생성
1. Github에 __username__.github.io 라는 이름으로 레포지터리를 생성한다
2. 끝.

## 클론
* git clone {git 레포지터리 주소}

## Jekyll 프로젝트 생성
1. 루비를 설치한다.
2. bundler 설치 `gem install bundler`
3. Jekyll 설치 `gem install jekyll`
4. Jekyll 프로젝트 생성  

### 만약 당신이 현재 디렉터리에서 프로젝트를 생성하고 싶어 한다면 
* `jekyll new .`

## 커밋 & 푸시
1. `git add .`
2. `git commit -m "첫 커밋"`
3. `git push origin master`