---
layout: post
title: "Spring - MyBatis 연동하기"
description: "지난 프로젝트에서, 디비 접근은 잘 되는데, 테스트 하려니까 웬 희안한 에러들이 떠서 정석대로 재시도."
tags: [Spring, MyBatis, Java]
---

# 무엇을 할 것인가
* 스프링 프로젝트 생성
* 메이븐 디펜던시 추가
* 마이바티스 설정
* 데이터베이스 연동
* 테스트 작성, 실행

# 스프링 프로젝트 생성
* STS 기준
* 레가시 스프링 프로젝트 생성
* 스프링 버전, 자바 버전 수정

# 메이븐 디펜던시 추가
* (__MySQL__ 기준) 아마 이 정도면 될 것이다

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>RELEASE</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.31</version>
</dependency>
<!-- JUnit 테스트용 -->
<!-- @RunWith(SpringJUnit4ClassRunner.class) -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
```

# 마이바티스 설정

## SQL 연결 설정
* 일단 mybatis-context.xml 파일을 생성했다.
* 일단 스프링에서 제공해주는 데이터소스 객체를 활용해서 커넥션을 얻어보자

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://{호스트}:{포트}/{스키마}" />
    <property name="username" value="{아이디}" />
    <property name="password" value="{패스워드}" />
</bean>
```

* 다음엔 마이바티스 세션 팩토리를 얻어야 한다

```xml
<!-- MyBatis -->
<!-- 세션 팩토리는 위의 데이터소스를 받아서 생성한다. 매퍼 위치는 알아서 설정하자. -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <!-- 매퍼로케이션의 값은 매퍼.xml이 있는 경로와 파일 이름. 보통 *.xml이더라. -->
    <property name="mapperLocations" value="classpath*:{어딘가}" />
</bean>
```

* 세션 팩토리를 얻었다면, SqlSession을 얻을 수 있다.

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

* SqlSession(사실은 SqlSessionTemplate)은 DAO에 바로 쏴줄 수 있다.
* 일단 DAO에 주입할 수 있도록 제반작업을 하자

```Java
private SqlSession sqlSession;

public void setSqlSession(SqlSession sqlSession) {
    this.sqlSession = sqlSession;
}
```

* 그리고 컨텍스트로 넘어와서

```xml
<!-- MyDao를 Bean으로 등록 -->
<!-- sqlSession을 디펜던시로 등록하였다. -->
<bean id="myDao" class="../../../MyDao">
    <property name="sqlSession" ref="sqlSession" />
</bean>
```

* 이제 MyDao 상에서 sqlSession 을 휘둘러 SQL 쿼리를 던질 준비는 어느 정도 끝났다.

## SQL 쿼리 매핑
* 작성 중

## JUnit 테스트 환경 작성

```java
@RunWith(SpringJUnit4ClassRunner.class)
// 컨텍스트 파일을 잡는 중.
@ContextConfiguration(locations="file:~~~/mybatis-context.xml")
public class MyTestDAOTest {
    // @Autowired이 선언된 객체는 해당 선언에서 가능한 Bean을 스프링이 알아서 생성해준다.
    @Autowired
	private MyDAO md;
    // ...
}
```

# 참고사항

## 경로에 대하여
* __classpath:__ 는 /resources 폴더 하위를 가리킨다.
* __file:__ 은 프로젝트 폴더 최상위부터 시작한다.

## 트러블슈팅
* org.mybatis.spring.transaction.SpringManagedTransaction.getTimeout()Ljava/lang/Integer;
    * 보통 마이바티스 버전 문제.
    * 아래와 같이 수정하면 문제가 해결된다.

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>RELEASE</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>RELEASE</version>
</dependency>
```