---
layout: post
title: "Command Obj."
description: "많은 사람들이 이것의 존재를 경험적으로 안다, 이름 빼고."
tags: [java, spring, oop]
---

# 커맨드 오브젝트
* 컨트롤러의 책임을 획기적으로 줄여봅시다.

## 컨트롤러에 들러붙은 책임들
* URL에 접근하는 사용자의 유효성 판단 등 사전 (검증 / 연산) 로직 등
    * → 인터셉터로
* 주요 비즈니스 로직 등
    * → 서비스 / BO / DAO로
* __★☆HTTP 응답☆★__
    * 이게 컨트롤러의 진짜 책임
* HTTP 요청
    * __엉?__
    * 분명 위랑 같은 HTTP 이긴 한데....

### HTTP 요청 파라미터
* 컨트롤러가 길어지는 주요 원인(적어도 내 경험상으로는)
    * 파라미터를 하나하나 적어줄 (내)책임과(귀찮다) 이를 검증하고 관련 데이터를 설정하는 (지금은 컨트롤러가 가진)책임을(보기 안 좋다) 어딘가에 몰아넣고 싶다.

```java
@RequestMapping(value = "/signup", method = RequestMethod.POST)
public @ResponseBody String upload(Model model, 
        @RequestParam(value = "email", required = true) String email,
        @RequestParam(value = "name", required = true) String name,
        @RequestParam(value = "password", required = true) String password,
        @RequestParam(value = "profile", required = false) String profile,
        @RequestParam(value = "birthday", required = false) String birthday,
        @RequestParam(value = "homepage", required = false) String homepage,
        @RequestParam(value = "profileImg", required = false) MultipartFile profileImg,
        @RequestParam(value = "protected", required = false, defaultValue = "false") boolean protect,
        HttpServletRequest request) throws BinaryException, NotEnoughParameterException {
        
    // ...이메일 포맷 확인...
    // ...패스워드 검증...
    // ...혹은 패스워드 해싱...
    // ...생일 -> Date...
    // ...프로필 이미지 사이즈 확인...
    // 등등등

    long userId = userService.signup(email, name, password, profile, birthday, homepage, profileImg, protect);
    // 뭐 이렇게 길어

    return result;
}
```

## Command Object
* 요청 파라미터를 다 묶어서 객체로 받을 수 있으면 좋지 않을까?
    * OOP : 데이터를 가진 쪽에서 관련된 로직을 처리하게 하라

```java
@RequestMapping(value = "/signup", method = RequestMethod.POST)
public @ResponseBody String upload(Model model, 
    UserSignupForm form, // 이 친구 
    HttpServletRequest request) throws BinaryException, NotEnoughParameterException {

    long userId = userService.signup(form);

    return result;
}
```

* UserSignupForm : __Command Object__
    * 이 좋은 것을 이제야 알았다니.

### 선언
* POJO : 너무 간단해서 할 말이 없다.
    * 인자에 맞춰 멤버 변수를 생성하고, getter, setter를 생성하자

```java
public class UserSignupForm {
    private String email;
    private String name;
    private String password;
    private String profile;
    private String birthday;
    private String homepage;
    private MultipartFile profileImg;
    private boolean protect;

    // getter & setter
}
```

### in spring

스프링은 커맨드 오브젝트를 아래와 같은 순서로 생성한다.

* 인자 없는 생성자 → `public void setter(param)`

스프링은 많은 데이터 타입을 지원한다

* 아래 내용은 `@RequestParam` 에서도 지원하는 내용이다.
* String
* 정수형은 setter 메서드의 인자를 정수로 받을 수도 있지만, String으로 받아 직접 파싱할 수도 있다
    * 예외 검출이 필요하거나, 기타 특수한 목적이 있는 경우 고려 가능
* boolean 같은 경우에는 문자열 `true`, `false` 를 적절한 불리언 값으로 변환한다
    * `yes` , `no` 등도 변환할 수 있다
* List<>
    * 동일한 이름으로 들어오는 같은 타입의 여러 인자를 담을 수 있다
* Date
    * `@DateTimeFormat` 을 사용하여 적절히 포매팅 됭 String 으로부터 Date 타입으로 바로 변환 가능하다

# 커맨드 오브젝트로 할 수 있는 것

## default value
* 생성자에서 값을 설정하는 것으로 가능

```java
public class UserSignupForm {
    private String email;
    private String name;
    private String password;
    private String profile;
    private String birthday;
    private String homepage;
    private MultipartFile profileImg;
    private boolean protect;

    public UserSignupForm() {
        protect = false;
    }

    // getter & setter
}
```

## 로직이 함께 하는 getter, setter

* 입력 데이터에서 수정이 필요할 때
    * 금칙어 필터 등 적용 시
* 특정 데이터에서 유도되는 다른 데이터를 사용하고 싶을 때
    * 사실 Date는 이럴 필요 없이, 바로 변환 가능하다. 어디까지나 예시.
* 등등등

```java
public class UserSignupForm {
    private String email;
    private String name;
    private String password;
    private String profile;
    private String birthday;
    private String homepage;
    private MultipartFile profileImg;
    private boolean protect;

    private Date birthdayDate;

    public UserSignupForm() {
        protect = false;
    }

    public setProfile(String profile) {
        this.profile = SomeFilter.doFilter(profile);
    }

    public void setBirthday(String birthday) {
        this.birthday = birthday;
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd");
        birthdayDate = format.parse(birthday);
    }

    public Date getBirthdayDate() {
        return birthdayDate;
    }
}
```

## @Valid

### Spring Validator
Spring에서 제공하는 Spring Validator를 사용하여 커맨드 오브젝트를 검증할 수 있다.

### 옵션

* 반드시 입력해야 하는 값, required 옵션이 없어 불안
    * @NotNull

```java
public class UserSignupForm {
    @NotNull
    private String email;

    @NotNull
    private String name;

    @NotNull
    private String password;

    private String profile;
    private String birthday;
    private String homepage;
    private MultipartFile profileImg;
    private boolean protect;

    private Date birthdayDate;

    public UserSignupForm() {
        protect = false;
    }

    public void setBirthday(String birthday) {
        this.birthday = birthday;
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd");
        birthdayDate = format.parse(birthday);
    }

    public Date getBirthdayDate() {
        return birthdayDate;
    }
}
```

* DB에 넣을 데이터라 길이제한이 필요해요
    * `@Size` , `@Min` , `@Max`

```java
public class UserSignupForm {
    @NotNull
    @Size(min = 5, max = 100)
    private String email;

    @NotNull
    @Max(50)
    private String name;

    @NotNull
    @Max(100)
    private String password;

    @Max(500)
    private String profile;

    private String birthday;

    @Max(50)
    private String homepage;

    private MultipartFile profileImg;

    private boolean protect;

    private Date birthdayDate;

    public UserSignupForm() {
        protect = false;
    }

    public void setBirthday(String birthday) {
        this.birthday = birthday;
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd");
        birthdayDate = format.parse(birthday);
    }

    public Date getBirthdayDate() {
        return birthdayDate;
    }
}
```

* 이메일 형식이 맞는지 확인하고 싶은데 가능?
    * @Pattern

```java
public class UserSignupForm {
    @NotNull
    @Size(min = 5, max = 100)
    @Pattern(regexp = "^[0-9a-zA-Z]([-_\.]?[0-9a-zA-Z])*@[0-9a-zA-Z]*\.[a-zA-Z]{2,3}$")
    private String email;

    @NotNull
    @Max(50)
    private String name;

    @NotNull
    @Max(100)
    private String password;

    @Max(500)
    private String profile;

    private String birthday;

    @Max(50)
    private String homepage;

    private MultipartFile profileImg;

    private boolean protect;

    private Date birthdayDate;

    public UserSignupForm() {
        protect = false;
    }

    public void setBirthday(String birthday) {
        this.birthday = birthday;
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd");
        birthdayDate = format.parse(birthday);
    }

    public Date getBirthdayDate() {
        return birthdayDate;
    }
}
```

* 그냥 옵션을 다 줘봐

