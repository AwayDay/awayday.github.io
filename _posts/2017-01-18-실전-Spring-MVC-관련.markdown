---
layout: post
title: "[작성 중]실전 Spring MVC 관련"
description: "자잘 자잘한 것들 잊지 않도록."
tags: [java, spring]
---

# 들어가기 전에
* DB 연동하는 코드는 따로 포스트 있으니 그 쪽을 보시오!
* 작성 중임
    * __점점 자잘해져가고 있다.(관심사의 관점에서)__

# Spring 관련

## `@Autowired` 이게 뭐에여?
* 아래에서 생성하는 객체가 Bean으로 등록되어 있다면, 스프링에서 알아서 만들어줌.
* 당연히 의존성도 알아서 해결해준다(적절한 설정 필요)

## 리다이렉션

### 보통의 리다이렉션
* Ajax가 활개를 치는 요즘 시대에 리다이렉트가 그렇게 많이 필요할까 싶긴 한데
* 필요하더라.
* `return "redirect:/주소";`

### 파워풀 실황 예외 리다이렉션
* 특정 예외에 대해 특정 페이지로 보내고 싶을 때가 분명 있다.

```java
@ExceptionHandler(XxException.class)
public String exceptionControl(){
    return "{에러 페이지}";
}
```

## 컨트롤러가 받는 인자에 대하여

### 경로 관련 인자
* `@PathVariable` 를 활용해보자.

```java
@RequestMapping(value = "/asd/{value}", method = RequestMethod.METHOD)
public String moimInformationPage(Model model, @PathVariable int value){
    model.addAttribute("yourValue", value);
    return "page";
}
```

## 로그인 관리
* 세션으로 하는 것 같은데, 과연 내가 구글링으로 찾는 것이 현업에서도 먹힐지, 원.

### 요청 관련 인자
* POST면 `@RequestParam` 으로 받을 수 있음

```java
@RequestMapping(value = "/moim/{value}", method = RequestMethod.POST)
public String controlJoin(Model model, @PathVariable int value, @RequestParam(value="id", required=true) int id) {
    model.addAttribute("yourValue", value);
    model.addAttribute("yourId", id);
    return "page";
}
```

## Spring REST
* REST 많이들 쓰고, 나도 좋아함.
    * 나같은 웹성애자들에게 아주 안성맞춤이다. HTTP가 세계를 지배할 것 으흥흥.
* REST 엔드포인트 만드는 법과, 스프링에서 REST 요청하는 법 정도를 알아두면 두고두고 쓸모가 있을 것.

### REST 엔드포인트 만들기.
* 작성 중

### 내부에서 REST 요청 하기 : RestTemplate
* __RestTemplate__ 객체를 사용한다.
* 백문이 불여일견, 포스트를 봅시다.

```java
public ResponseObj postForPost(requestObj ro) {
    RestTemplate restTemplate = new RestTemplate();
    return restTemplate.postForObject("url", ro, ResponseObj.class);
}
```

* 당연히, 포스트 말고도 쓸만한 메서드가 많이 있다.
* 일단 메서드 별로 다 있다.
    * __GET__
        * 
    * __POST__
        * 
    * __PUT__
        * 
    * __DELETE__
        *

```java
public void restForREST(requestObj ro) {
    RestTemplate restTemplate = new RestTemplate();
}
```

## Multipart : 파일 전송
* 손 봐야 할 것이 조금 많다.

### 의존성 등록

```xml
<!-- MultipartHttpServletRequset -->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.0.1</version>
</dependency>
    
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.2.2</version>
</dependency>
```

### Bean 등록

```xml
<!-- multipart -->
<!-- maxUploadSize는 50MB가 상한. -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="100000000" />
</bean>
```

### MultipartHttpServletRequest 사용한다
* 컨트롤러 인자에 `MultipartHttpServletRequest request` 를 추가해보자.

### 파일 전송을 마음껏 누리십시오 여러분
* `request.getFile` 와 유사품들이 `MultipartFile` 리턴으로 당신을 반긴다!

# JSP 관련

## 한글이 나오게 합시다
* `<%@ page pageEncoding="UTF-8" contentType="text/html; charset=UTF-8"%>`

## js나 css 쓰고 싶음 ㅎ
* `<script src="/resources/js/{이름}.js"></script>`
* `<link href="/resources/css/{이름}.css" rel="stylesheet">`

## JSTL : 진보한-JSP 문법
* `<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>`

## 제어문

### 반복문

```
<c:forEach var="{임시}" items="${배열, 리스트 등}">
    <!-- 알아서 반복한다. -->
</c:forEach>
```

### 조건문

```
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