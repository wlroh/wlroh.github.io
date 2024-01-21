---
layout: post
title: "[Spring] 비동기와 ThreadPoolTaskExecutor"
date: 2023-12-09
excerpt: "Spring 비동기와 ThreadPoolTaskExecutor 에 대해 알아보자"
categories: Spring
---

서비스를 운영하다보면 성능 향상을 위해, 비동기를 통해 구현이 필요한 경우도 있습니다. <br/>
그렇다면 비동기로 구현하기 위해서는 어떻게 하는지 알아보겠습니다!

<br/>

## 비동기로 구현하고 싶으면 어떻게 해야할까?

스프링에서는 비동기로 구현하기 위해서는 `@EnableAsync`와 비동기로 수행할 로직에 `@Async` 설정해주면 됩니다! <br/>
끝! 그럼 오늘 포스팅은 이만.. 물러가기에는 이야기할 부분이 많죠..! 

제목에서 이야기한 것과 같이 비동기를 구현하기 위해서는 스프링에서 제공해주는 TaskExetutor 구현체인 ThreadPoolTaskExecutor 에 대해서도 학습이 필요합니다.

우선, 비동기 구현하는 것은 위에서 이야기한 대로 어노테이션만 추가해주면 비동기로 로직을 수행할 수 있습니다.
하지만, 서비스를 운영하다보면 갑자기 많은 트래픽이 몰릴 수 있으며, **비동기를 수행하는 TaskExecutor 가 어떤 구현체를 사용** 함에 따라 서버가 터지는 상황도 발생할 수 있습니다.

> Spring에서 `@Async` 만 설정해주었을 때 어떤 TaskExecutor 를 사용할지 궁금하지면 [Spring Default TaskExecutor는 무엇일까?](https://wlroh.github.io/spring/2023/02/11/spring-async-default.html) 포스팅을 참고해주세요. :)


<br/>

## ThreadPoolTaskExecutor 란 무엇일까?

`ThreadPoolTaskExecutor` 에 대해 학습하기 이전에 우선 `ThreadPoolTaskExecutor` 가 무엇인지에 대해 chatGPT 선생님께 물어보니 다음과 같은 답변을 받았습니다.

> ThreadPoolTaskExecutor는 TaskExecutor 인터페이스를 구현하고 여러 작업이 실행기에 의해 관리되는 스레드 풀 내에서 동시에 실행될 수 있도록 하는 스레드 풀링 구현을 제공하는 Spring Framework의 Java 클래스입니다. 스레드를 재사용하고 각 작업에 대해 새 스레드를 생성하는 오버헤드를 방지하여 수명이 짧은 많은 작업의 수명 주기를 효율적으로 실행하고 관리하는 데 도움이 됩니다.

정리하면, `ThreadPoolTaskExecutor` 는 쓰레드 풀을 관리하는 역할이라고 볼 수 있습니다. 그렇다면 쓰레드 풀을 관리하기 위해 어떠한 기능을 제공하고 있는지 확인해보겠습니다.

그렇다면 어떤 메커니즘으로 수행되는지부터 확인하면 다음과 같습니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/threadPoolTaskExecutor.png"/>
</div>

크게 흐름은 사용가능한 threadPool 에서 사용가능한 Thread 를 찾고 없으면 Queue 에 등록합니다. 설정한 Queue 가 가득 찰 경우 새로 들어오는 요청건에 대해 설정한 maxPoolSize 까지 thread 를 생성해서 로직을 수행합니다.

ThreadPoolTaskExecutor 에 큰 흐름을 파악했으면 자주 사용하는 제공하는 기능을 살펴보면 다음과 같습니다.

### setCorePoolSize

CorePoolSize 조절하기 위해서는 `setCorePoolSize()` 메서드를 이용해 지정할 수 있습니다. CorePoolSize 는 별도 세팅을 안하면 사용하지 않을 때는 종료없이 Pool 에 유지됩니다.

기본값은 1 입니다.

### setQueueCapacity

Queue 개수를 지정하는 기능으로 `setCorePoolSize()` 메서드를 이용해 지정할 수 있습니다. 기본값은 `Integer.MAX_VALUE` 로 **적정 개수를 설정해주지 않으면 많은 요청이 왔을 때 큐가 계속 쌓이면서 위험성**이 있으니 적정 개수 설정이 필요합니다.

### setMaxPoolSize

MaxPoolSize를 조절하기 위해서는 `setMaxPoolSize()` 메서드를 이용해 지정할 수 있습니다. 기본값은 `Integer.MAX_VALUE` 로 **적정 개수를 설정해주지 않으면 많은 요청이 왔을 때 쓰레드가 계속 생성되어 위험성**이 있으니 적정 개수 설정이 필요합니다.

### setKeepAliveSeconds

큐가 가득차면서 코어 풀 사이즈보다 더 많은 쓰레드가 생성되었을 때 사용안하고 있을 경우 설정한 코어 풀 사이즈만큼 쓰레드를 정리합니다. 이 정리하는 시간을 지정해줄 때 사용하는 메서드입니다.

기본값은 60초이고, 더 빠르게 코어 풀 사이즈만큼 정리하고 싶을 때 사용합니다.

### setAllowCoreThreadTimeOut

코어 쓰레드의 경우 사용하지 않아도 종료하지 않고 풀에 유지된다고 말씀드렸는데, `setAllowCoreThreadTimeOut()` 메서드를 이용하면 코어 쓰레드도 정리할 수 있습니다.

기본값은 `false` 이고, `true` 로 설정 시 코어쓰레드 또한 정리해줍니다. Pool 을 사용하는 이유가 새 스레드를 생성하는 오버헤드를 방지하는 이유도 있기때문에 개인적으로 자주 사용하지는 않습니다.

### setPrestartAllCoreThreads

ThreadPoolTaskExecutor 의 경우 기본적으로 lazy loading을 하기 때문에 처음 어플리케이션 실행 후 쓰레드 풀에는 쓰레드가 0개입니다. <br/>
그렇기 때문에 코어쓰레드 개수만큼 생성되기 전까지는 요청이 올때 마다 신규 쓰레드를 생성해주기 때문에 처음 어플리케이션 실행 후 비동기 로직을 수행할 때 성능이 원하는 만큼 안나올 수 있습니다.

이때 미리 코어쓰레드를 생성하고자 할 때 사용하는 기능으로 기본값은 false 입니다.

### setTaskDecorator

요청한 로직을 수행하기 전에 각각의 쓰레드마다 Decorator 를 설정해줄 수 있습니다. 해당 내용은 별도의 포스팅으로 다루겠습니다.

### setRejectedExecutionHandler

큐도 가득차고 쓰레드 개수가 최대 쓰레드 개수만큼 생성되어 있을 때 요청이 들어오면 기본적으로는 `RejectedExecutionException` 예외가 발생됩니다. <br/>
이때 후처리를 어떻게 할지 핸들러를 통해 설정해줄 수 있습니다. 관련 내용에 대해 자세히 보시려면 [RejectedExecutionException과 Handler](https://wlroh.github.io/spring/java/2023/12/23/spring-rejected-exception-handler.html) 포스팅을 참고바랍니다.

<br/>

## 결론

ThreadPoolTaskExecutor 의 메커니즘과 자주 사용하는 기능이 무엇이 있는지 정리를 했습니다. 과거 개발자 선배님들께서 우리를 위해 비즈니스 로직에 집중할 수 있도록 작업해주셨지만, 어떤 메커니즘으로 작동되고 어떤 기능이 알고 사용해야 추후 이슈 대응이 가능하다 생각하여 공식문서와 테스트 코드 기반 학습하여 정리해보았습니다.

비동기의 경우, 기존 로직 수정을 최소화하며 성능개선이 가능한 방법이지만 잘못 사용하면 리스크 또한 상승하는 양날의 검으로 생각합니다. <br/>
그렇기에 위의 기능을 활용해 각각의 서비스마다 적절한 쓰레드 풀을 생성해서 성능 개선할 수 있었으면 합니다.

<br/>

## 레퍼런스

- [ThreadPoolTaskExecutor Docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html)
- [학습 및 테스트한 Repository](https://github.com/wlroh/Spring-Async/blob/main/src/test/java/com/wlroh/async/threadtaskpoolexecutor/ThreadTaskPoolSizeTest.java)
