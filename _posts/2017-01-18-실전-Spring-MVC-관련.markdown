---
layout: post
title: "[작성 중]실전 Spring MVC 관련"
description: "자잘 자잘한 것들 잊지 않도록."
tags: [java, spring]
---

# 들어가기 전에
* DB 연동하는 코드는 따로 포스트 있으니 그 쪽을 보시오!

# Spring 관련

## `@Autowired` 이게 뭐에여?
* 아래에서 생성하는 객체가 Bean으로 등록되어 있다면, 스프링에서 알아서 만들어줌.
* 당연히 의존성도 알아서 해결해준다(적절한 설정 필요)

# JSP 관련

## 한글이 나오게 합시다
* `<%@ page pageEncoding="UTF-8" contentType="text/html; charset=UTF-8"%>`

## js나 css 쓰고 싶음 ㅎ
* `<script src="/resources/js/{이름}.js"></script>`
* `<link href="/resources/css/{이름}.css" rel="stylesheet">`

## 제어문

### 반복문

```jsp
<c:forEach var="{임시}" items="${배열, 리스트 등}">
    <!-- 알아서 반복한다. -->
</c:forEach>
```

### 조건문

```jsp
<c:choose>
    <c:when test="${조건문 1}">
        <!-- if -->
    </c:when>
    <c:when test="${조건문 2}">
        <!-- if else -->
    </c:when>
    <!-- ... -->
    <c:otherwise>
        <!-- else -->
    </c:otherwise>
</c:choose>
```