---
layout: post
title: "Spring 4 비동기"
description: "너는 너의 일을 하거라"
tags: [java, spring, async]
---

# 가장 간단하게는 Async 어노테이션

## 참 쉽죠?

* 비동기 작업이 필요한 대상에 `@Async` 애노테이션을 추가한다.

```java
@Async
public void runAsync() {
    // Run run do run run Aikatsu
}
```

## 안 되는데요
뻥이다, 사실 이렇게만 하면 여전히 전통적인 방식으로 동작한다. 정말 다른 스레드에서 돌리고자 한다면 `EnableAsync` 설정을 추가로 해야 한다. 스프링 설정에 다음과 같이 `@EnableAsync` 애노테이션을 추가하자.

```java
@Configuration
@EnableAsync
public class AsyncConfiguration {
}
```

## 성능이 왜 이래요

비동기 작업의 실행에는 `Executor`가 관여하여 스레드를 가져오고 코드를 올리고 실행하고 버리고... 이런 일련의 작업을 도와준다. 그런데 스프링에서 기본으로 사용하는 심플 (중략) 익스큐터는 스레드를 쓰고 버리는 특징이 있다.

...어?

* 스레드는 대표적인 생성 비용이 큰 물건

그래서, 진짜 스레드풀 맛이 어떤 것인지를 똑똑히 보여줄 필요가 있다. 우리에게 필요한 `Executor`를 직접 생성하여 빈으로 등록해보자.

```java
@Configuration
@EnableAsync
public class AsyncConfiguration {
    private final int ASYNC_THREAD_POOL_SIZE = 5;

    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(ASYNC_THREAD_POOL_SIZE);
        taskExecutor.setMaxPoolSize(ASYNC_THREAD_POOL_SIZE);
        taskExecutor.setQueueCapacity(Integer.MAX_VALUE);
        taskExecutor.setThreadNamePrefix("threadPoolTaskExecutor-");
        taskExecutor.initialize();
        return taskExecutor;
    }
}
```

이렇게 생성한 `Executor`는 이렇게 힌트를 주어 사용할 수 있다.

```java
@Async("threadPoolTaskExecutor")
public void runAsync() {
    // Run run do run run Aikatsu
}
```

매번 설정하는 것이 귀찮다면 `AsyncConfigurer` 인터페이스를 구현해주자.

```java
@Configuration
@EnableAsync
public class AsyncConfiguration implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```

```java
@Async
public void runAsync() {
    // Run run do run run Aikatsu
}
```

# 리턴이 void면 정말 좋은데
정말 좋은데 살다보면 값을 반환해야 할 일이 반드시 생긴다. 이 때 비동기 작업의 플로우 관리는 `Feature`가 할 수 있다. 스프링은 `Feature`를 입맛에 맞게 수정한 `ListenableFuture`를 기본으로 사용하나, 자바 1.8 이상을 사용할 수 있다면 `CompletableFuture`을 선택할 수도 있다.

* Spring 5에선 설정에 따라 Mono와 Flux가 이를 대체하게 된다.

## 어노테이션 없는 1안

`threadPoolTaskExecutor` 는 위에서 생성한 그 `Executor`가 맞다. 이걸 설정 안 해주면 디폴트로 포크조인풀`ForkJoinPool`을 사용하는데, 얘는 코어를 노예처럼 굴리는게 특징이고, 그러다 여차하면 코어를 다 쳐묵고 뱉질 않는 경우가 발생할 수도 있는 문제가 있다. 이런 전차로 프로덕트에선 잘 안 쓴다 카더라.

```java
public CompletableFuture<T> runAndReturn() {
    // 이 CompletableFuture 는 비동기로 해줍니다.
    return CompletableFuture.supplyAsync(() -> {
        // Run run do run run Aikatsu
        return something;
    }, threadPoolTaskExecutor);
}
```

## 어노테이션 있는 2안

```java
@Async("threadPoolTaskExecutor")
public CompletableFuture<T> runAndReturn() {
    // 이 경우엔 메서드 자체가 비동기라서 supplyAsync 해줄 필요 없이, completedFuture 사용하면 된다.
    // Run run do run run Aikatsu
    return CompletableFuture.completedFuture(something);
}
```
