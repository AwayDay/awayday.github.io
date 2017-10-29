---
layout: post
title: "Git의 개념과 Git-flow의 흐름"
description: "형상관리의 묘"
tags: [git]
---

# 들어가기 전에
* 원본 포스팅은 1월 17일에 쓴 `형상관리의 묘 - git와 github` 였는데, 글을 워낙 갈겨 쓴 관계로 수정하여 재 업로드 하였다.

# 형상관리의 계보

## 형상관리는?

개발을 하다보면 소스코드가 자꾸 변경되기 마련이다. 사람이 모든 상황에서 완전할 수는 없는 것이라, 변경된 소스코드가 잘 안 돌아갈 수도 있고, `왜 변경했더라?` 기억이 안 날 수도 있고, 동료가 트럴이라(혹은 __내가 트럴이라__) 전체 소스 코드의 품질이 낮아질 수도 있다.

> 그렇다면 소스코드의 상태를 저장할 수 있으면 되지 않냐능?

그래서 사람들은 소스코드의 변화를 추적하고 관리하는 __형상관리__ 의 필요성을 느끼게 되었다. 일반적으로, 형상관리(형상관리를 하는 그 자체)의 이점은 아래와 같다.

### 형상관리의 특징 및 장점
* 파일의 변화를 시간에 따라 기록하여 특정 시점의 버전을 다시 불러올 수 있음
* 개별 파일 또는 프로젝트 전체를 이전 상태로 되돌릴 수 있음
* 시간에 따른 변경 사항을 기록 검토
    * 변경 시 변경 내역을 기록 가능
* 문제가 발생한 시점과 발생 원인(=사람)을 빠르게 확인 가능
    * 각 변경마다 __누가__ (커미터) / __언제__ (커밋 시간) / __무엇을__ (수정 위치) / __왜__ (커밋 로그) 를 기록한다.
* 파일이 누락되거나 잘못된 경우에도 쉽게 복구 가능

## 비욘드-형상관리 시스템
* 폴더 등 최소최저의 기능만을 이용하여 버전 관리

### 예시

```
* Project Folder
    * v0.1(yyyymmdd)
        * 변경내역.txt
        * 파일1
        * 파일2
    * v0.2(yyyymmdd)
        * 변경내역.txt
        * 파일1
        * 파일2
        * 파일3
        * 파일4
    * (중략)
    * v0.9
        * 변경내역.txt
        * src
        * resources
        * ...
    * v0.99
        * 변경내역.txt
        * ...
    * v0.9999(final)
    * v0.9999(final-release)
    * v0.9999999(FINAL)
```

(사족 : 해본 적 있음)

### 문제점

* 버전 관리를 하기 위해 디렉터리나 파일 등을 복사
* 인력 관리 = 실수할 가능성

### 해결법과 협업 문제

> 수정 이력을 내 컴퓨터의 디비에 넣으면 되겠네!

인력으로 파일을 복붙하던 것에 고통받던 고대의 선각자들은 변경 이력을 자동으로 관리할 필요를 느끼게 되었고, 이러한 내역을 디비 등으로 관리하려는 시도를 하였다. 이러한 시도는 나 혼자 소스를 수정할 때는 확실히 유용하였다.

> 이거 재밌겠네! 나도 끼워줘라!

문제가 있다면, 웬만하면 프로젝트를 혼자 하는 일이 드물다는 것. 로컬 디비에 내역을 저장하는 경우, 협업이 이루어지면 대략 망하는(...) 특징이 있었다. 여러 사람이 같은 파일을 수정할 경우에 대해서는 어떻게 해야 할까?

## 형상 관리 시스템의 등장 - 중앙 집중식 버전 관리 시스템

> 소스를 한 곳에 전부 모아서, 여기서 이력도 관리하고 충돌도 협의하고 하면 되지 않을까?

대충 이런 발상으로 나온 것이 중앙 집중식 버전 관리 시스템이고, SVN을 대표로 세울 수 있다.

* 파일을 저장하는 하나의 서버 존재
* 중앙에서 파일을 가져오는 각각의 클라이언트(=개발자)

### 장점
* 누구나 다른 사람이 무엇을 하고 있는지 알 수 있음
* 난이도
    * 수많은 클라이언트의 로컬 디비 관리 >>>>>>>>>>(넘사벽)>>>>>>>>>> 중앙 디비 관리

### 단점
* 중앙집중식 = 중앙이 죽으면....?
    * 예시 : 불이 났다.
    * 아 망했어요
* 서버가 다운되면 복구 때까지 협업은 커녕 개인 작업도 못 함

## 분산 관리 시스템으로의 진보

> 서버의 복제본을 너, 나, 우리가 가진다.

* Git 등
* 서버의 복제를 각 클라이언트에서 가지고 있음
* 서버가 죽어도 어느 누군가의 복제를 서버에 다시 올리면 됨

# Git

## Git의 태동

Git은 리눅스로 일약 아이돌이 된 천재 개발자 리누스 토발즈가 개발하였다. 토발즈는 당시 프로젝트에 비트키퍼라는 상용 형상관리 툴을 사용하고 있었는데(개발자들 작명법은 세월의 흐름에 영향을 받지 않는 것 같다.), 비트키퍼의 개발이 중단된다는 소식을 듣고 급하게 하나 만든 것이 Git이다. 과연 천재는 다르다.

Git은 빠른 속도, 단순한 구조, 비선형 개발, 완전한 분산, 대형 프로젝트에서도 유용할 것을 목표로 하였는데, 토발즈가 어떤 프로젝트를 했는가를 다시 한 번 떠올려보면, 음, 과연.

## Git의 특징

Git은 다른 형상관리 시스템과 비교하여, 몇 가지 두드러지는 특징을 가진다.

### 스냅샷
* Git의 변경 이력 저장 방식
    * SVN은 델타로 저장
* 특징 및 장점
    * 변경된 파일이 있으면 통째로 저장
    * 특정 시점으로 체크아웃 하는 속도가 매우 빠름 : 스냅샷을 받아오면 장땡이기 때문

### 로컬 환경
* 거의 대부분의 명령이 로컬에서 실행
    * 물론 push / pull 제외
    * 형상관리에 있어 핵심적인, 대부분의 동작이 로컬에서 돌아감
        * 원격 서버가 죽어도 각 개발자 ~~개미~~ 는 계속해서 작업 진행 가능
            * 물론 push / pull 제외(2)

### 무결성
* 깃은 어떻게 데이터를 온전히 저장하는가?
* 해시와 체크섬
    * 파일에 대한 체크섬을 구하여 무결성을 검사
* .git 디렉터리
    * 어떤 브랜치 이름을 가지는 폴더 > .. > 아무 파일 : 체크섬 달랑 있음

## Git이 관리하는 상태

### 커미티드
* 어떤 파일이 깃 저장소에 저장된 상태
    * git에 의해 버전이 관리되는 상태

### 모디파이드
* 커미티드 된 파일에 수정이 발생한 상태
    * 개발자가 수정하는 중

### 스테이지드
* 모디파이드 된 파일을 커밋하기 위한 준비 단계
    * 수정한 파일을 커밋하기 이전 스테이징 한 상태

### 상태로 보는 작업 플로우
* 워킹 디렉터리에서 파일 수정(모디파이드)
* 스테이지에 올리기(스테이지드)
* 커밋(커미티드)

### 파일 수정과 상태 변경
* 워킹 디렉터리 내용의 파일은 트랙드와 언트랙드로 분류됨
    * 트랙드 : 버전 관리 잘 되는 파일
        * 이미 스냅샷에 포함
        * 언모디파이드 : 커밋 이후 수정된 것이 없는 상태 
        * 모디파이드 : 짜잔! 수정하였습니다.
        * 스테이지드 : 스테이지에 올라간 상태
    * 언트랙드 : 버전 관리에서 제외되어 있는 파일
        * 파일이 생성된 상태
        * 파일 추가가 필요함.
* 언트랙드 -> 스테이지드 -> (커밋) -> 언모디파이드 -> 모디파이드

## 원격 저장소 : Remote

Git은 로컬 저장소 환경에서만 사용할 수 있지만, 원격 저장소(레포지터리)를 추가하여 힘세고 강한 협업을 가능하게 할 수 있다.

대표적인 원격 저장소 서비스로는

* Github
* Bitbucket

이 있고, 위의 두 서비스가 마음에 들지 않는다면 자신의 서버에 Git 및 관련 서비스를 추가하는 것으로 수제 레포지터리를 만들어낼 수도 있다. 혹은 __GitLab__ 을 설치하여 사용하는 방법도 있다. 개인적으로는 UI가 취향이라 좋아함.

기본 원격 저장소의 이름은 __origin__ 으로 설정되는데, 원한다면 다른 것으로 해도 무방하다. 어차피 push는 명령을 보고 수행한다.

### 원격 레포지터리와 로컬 레포지터리의 차이
* 오리진(리모트) 마스터와 로컬 마스터는 서로 다른 브랜치이다.
    * 더 당연한 것으로는, 동일한 프로젝트를 진행하는 내 마스터 브랜치와, 옆 사람의 마스터 브랜치가 다른 것이 있다.

## Git 명령어
* 대표적인 것 몇 개만
* 고급 사용자가 되면 체크아웃이나 리버트 같은 명령어가 더욱 유용한데, 나는 그것을 지금 당장 설명할 능력이 없기 때문에.

### add
* `git add ~`
* 모디파이드 상태의 파일을 스테이지드 파일로 만드는 명령어
    * 나와 같이 뒤도 안 돌아보고 `git add .` 하는 인간에겐 백약이 무효하다, 잘 확인하자.

### commit
* __커밋__ : 버전 관리 시스템에서 포인트를 만드는 것을 이름
    * __커밋 로그__ : 어떤 커밋에 관한(=포인트에 관한) 설명
* `git commit -m {"comment"}`

### fetch
* 원격 저장소의 변경 사항을 내려 받는 것.
    * fetch 명령어는 변경 사항만 내려받고, 로컬에 머지 하진 않는다.
    * 로컬 브랜치에 원격 저장소의 변경 사항을 반영하고 싶다면 수동으로 원격 브랜치를 로컬 브랜치로 머지해야 한다.

### pull
* = fetch + merge
* 변경사항 fetch 이후 로컬에 merge

### push
* 로컬 저장소의 변경 사항을 원격 저장소로 업로드.
* 사무실에 화재가 났을 때에는 재빨리 `commit` > `push` > __run__ 프로세스를 수행할 수 있어야 한다.

## Git의 브랜치
* 개발의 큰 줄기, 여러 워킹 디렉터리를 생성.
* 실제 개발 과정에서의 사용
    * 작업 중인 프로젝트
    * 새로운 이슈 처리를 위해 브랜치 생성
    * 새로 만든 브랜치에서 작업을 진행
* 다른 예시 : 문제 수정 중 중요한 문제가 생겨서 급하게 핫픽스를 해야 한다.
    * 새로운 이슈를 처리하기 위해 가장 최신의 운영 브랜치로 이동함.
    * 핫픽스 브랙치 생성
    * 수정한 핫픽스 테스트를 마치고 운영 브랜치로 머지함
    * 다시 작업하던 브랜치로 돌아가 이슈 해결 진행.
* 가상의 작업 공간.

### Git이 개별 브랜치의 데이터를 저장하는 방법
* 깃이 브랜치를 어떻게 다루는가?
* 깃 커밋 -> 커밋 객체 생성
    * 커밋 객체 : 스냅샷에 대한 포인터, 메타 데이터, 이전 커밋에 대한 포인터.
        * __이전 커밋이 무엇인지 저장__
* 브랜치는 커밋을 가리키는 포인터
    * 커밋은 커밋 객체를 가리키고, 커밋 객체는 스냅샷을 가리킨다.

## 기타 용어

### HEAD
* 현재 커밋 위치를 가리킴.

### Tag
* 출시 관리용
* 보통 버전 명으로 작성한다.
* 특정 커밋을 가리키는 속성이 있는데, 이 점에서 브랜치랑 비슷한 면을 보인다.

### checkout
* 워킹 디렉터리의 상태를 해당 브랜치의 워킹 디렉터리로 변경하는 것.
    * 특정 상태로의 천이
* Git에서는 뭔가 작업 비슷한 것을 하기 위해 뭔가 작업 비슷한 것을 하길 원하는 브랜치로 체크아웃을 해 주어야 한다.

### merge
* 특정 브랜치의 변경 내역을 대상 브랜치에 적용하는 것
* 머지가 되면 해당 커밋(머지-커밋)은 부모가 두 개
    * 이렇게 되면 개발 브랜치를 지우는 것이 보통

### 브랜치 삭제
* 다른 브랜치로 체크아웃 한 다음에 해야 함.
* 이 경우 브랜치(=포인터)는 삭제되나 해당 그래프의 커밋 내용이 실제로 삭제되는 것은 아니다.

### Fast-Forward
* 한 쪽에는 변화가 없고, 한 쪽은 커밋이 주르륵 올라간 상태
* 이 상태에서 뒤쳐진 쪽에 앞서간 쪽을 머지하면
* 뒤쳐진 쪽 포인터가 앞으로 파밧 이동함
* 따로 커밋도 필요 없음, 끝.

### 충돌
* 보통 두 브랜치에서 동일한 라인을 변경하였을 경우 충돌 발생 
* __예시__
    * 같은 라인에 대해 다른 두 개 이상의 브랜치에서 수정 발생
    * 전부 풀 리퀘스트
    * 가장 먼저 머지 된 브랜치는 머지가 될 것이나, 다음부터는 충돌이 발생한다.
* __해결__
    * 충돌이 발생한 파일의 충돌 위치를 확인 후 적절히 수정
        * 팀원과 협의 하에 진행합시다.

# Git-Flow

Git-Flow는 Git의 브랜치 흐름을 효율적으로 관리하는 관리 모델이다. 깃 플로우는 개발 상황에 따라 서로 다른 브랜치를 생성하도록 유도하여, 병렬적이고 능률적인 개발을 달성하게 한다.

## 메인 브랜치

메인 브랜치는 어떤 경우에도 항상 존재하는 브랜치들이다. __master__ 와 __develop__ 브랜치 두 개로 구성된다.

### master

master 브랜치는 배포본을 의미한다. 배포시에는 master 브랜치의 커밋을 사용하며, master 브랜치의 가장 최근 커밋에는 최신 배포 버전이 자리잡고 있어야 한다.

### develop

develop 브랜치에선 다음 배포를 위한 소스 작업을 진행한다. 개발이 완료되면 develop 브랜치의 소스는 일련의 과정을 통해 master로 머지된다. 배포는 개발 환경에 대해서만 진행한다.

## 서포팅 브랜치

서포팅 브랜치는 개발 상황에 따라 있을 수도, 없을 수도 있는 브랜치들이다. 프로젝트 진행 중, 개발자의 필요에 의해 생성한다. 이러한 서포팅 브랜치들은 복수의 팀 멤버들이 병렬적으로 작업할 수 있도록 도와준다. __feature__ 브랜치와 __release__ 브랜치, 그리고 그다지 반갑지만은 않은 __hotfix__ 브랜치로 이루어진다.

### feature

특정 기능을 개발하기 위해 사용하는 feature 브랜치는 develop 브랜치에서 생성된다. feature 브랜치의 작명법은 팀마다 다르지만, 이슈 관리 툴에 등록된 이슈넘버와 연관되게 작명하는 경우와, 개발하는 기능(이슈)를 반영하여 작명하는 경우로 나눌 수 있다. 

각 개발자는 자신의 개발 변경을 feature 브랜치로 한정하여 협업하는 동료에게 영향이 가지 않게 하며, 이는 develop 브랜치에 대해서도 마찬가지이다. 이러한 방법을 통해 Git-Flow 모델은 병렬적인 개발을 달성한다.

feature 브랜치에서 개발이 완료된 변경사항은 PR 등의 과정을 거쳐 그리운 어머니 develop 브랜치로 머지한다. 이 때, 개발한 기능의 병합을 커밋으로 구분할 수 있게 하기 위해, 패스트 포워드를 사용하지 않는 경우가 일반적이다.

### release

release 브랜치는 배포 준비를 위해 develop 브랜치에서 분리되어 나온다. develop 브랜치의 개발 진척이 성과를 보여, 배포에 충분한 수준으로 성숙되었다고 판단하면, 배포 준비 과정에 들어가며 develop 브랜치에서 release 브랜치를 생성한다.

release 브랜치를 사용하여 QA를 진행하며, 배포에 필요한 메타 데이터를 설정하거나, 작은 버그 등(QA 지적사항)을 수정한다. 이러한 변경사항은 master 브랜치와 develop 브랜치 양쪽에 머지해야 이후의 배포와 개발에서 문제가 발생하지 않는다.

### hotfix

hotfix 브랜치는 master 브랜치의 운영 환경 배포 이후, 다음 배포까지 여유롭게 수정할 수 없는 위급한 이슈를 수정하기 위해 생성한다. hotfix 브랜치에서는 이런 다급한 버그 등을 수정하며, 수정 사항은 master 브랜치와 develop 브랜치에 동시에 머지해야 한다.


# 여담
* 범용 이미지-아바타 서비스인 [Gravatar](http://en.gravatar.com) 사이트에 이메일을 등록하여 프사를 업로드 하면 이것이 깃허브에서도 나오고 비트버킷에서도 나온다
    * 장작위키에서도 나옴