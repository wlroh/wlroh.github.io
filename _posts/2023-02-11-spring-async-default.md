---
layout: post
title: "[Spring] @Async 어노테이션 Default TaskExecutor는 무엇일까?"
date: 2023-02-11
excerpt: "Spring Default Task Executor에 대해 알아보자"
categories: Spring
---

Spring을 통해 비동기 처리를 하기 위해서 `@Async` 어노테이션을 통해 비동기 처리를 할 수 있습니다. `@Async` 어노테이션의 value 값을 지정해 어떤 `Executor` 혹은 `TaskExecutor` Bean을 사용할지 지정할 수 있는데, value 값 없이 실행했을 때 Default TaskExecutor가 무엇일지 궁금해 학습하게 되어 포스팅하게 되었습니다.

구글링 혹은 요즘 핫한 Chat GPT 조차 Default TaskExecutor에 대해 답변을 제대로 주지 못해 직접 테스트하면서 어떤 메커니즘으로 수행되는지 확인했습니다.

<br/>

## Default `TaskExecutor` 가 무엇일까?

구글링 혹은 Chat GPT의 답변은 크게 `ThreadPoolTaskExecutor` 와 `SimpleAsyncTaskExecutor` 두 가지의 의견이 분분하게 보였습니다.
우선 결론부터 말씀드리면 두가지 다 맞습니다.

스프링 공식 홈페이지에서는 `ThreadPoolTaskExecutor`로 설명을 해주고 있고, 제가 직접 디버깅을 통해 확인했을 때는 `SimpleAsyncTaskExecutor`를 이용하고 있어 왜 다를까하여 열심히 찾아보았지만 해당 내용을 쉽게 찾을 수 없어서 직접 테스트하면서 확인한 결과 스프링에서 Default `TaskExecutor` 아래와 같은 우선순위를 가지고 수행되는 것으로 보여졌습니다.

1. ThreadPoolTaskExecutor
2. SimpleAsyncTaskExecutor

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/default/image1.png"/>
</div>

Default TaskExecutor를 찾는 로직을 풀어서 설명해보면 다음과 같습니다.

Spring은 우선 ThreadPoolTaskExecutor Bean을 찾습니다. <br/>
해당하는 Bean이 없는 경우 스프링에서 ThreadPoolTaskExecutor를 Bean으로 등록한 뒤 해당 ThreadPoolTaskExecutor를 이용해 요청받은 Task를 수행합니다.

만약 ThreadPoolTaskExecutor Bean을 찾는 과정에 해당하는 Bean이 있을 때 어떤 Bean을 사용할지 정해져 있으면 해당 ThreadPoolTaskExecutor Bean을 이용해 요청받은 Task를 수행합니다. <br/>
어떤 Bean을 사용할지 정해져 있지 않는 경우 스프링에서는 다음 우선순위인 SimpleAsyncTaskExecutor을 이용해 요청한 Task를 수행합니다.

각각의 케이스별로 실제로 ThreadPoolTaskExecutor 혹은 SimpleAsyncTaskExecutor를 가져오는지 확인해보기 위해 아래와 같은 별도로 TaskExecutor를 지정이 안된 비동기 어노테이션을 걸어둔 메서드를 바탕으로 확인해보도록 하겠습니다.

``` java
@Async
public void doAsyncTask() {

    ... 

}
```

<br/>

### ThreadPoolTaskExecutor Bean이 없는 경우

``` java
@Configuration
public class ThreadPoolConfig {

}
```

위의 코드처럼 `ThreadPoolTaskExecutor` Bean을 등록하지 않은 상황에서 비동기 호출을 하게 되면, Spring에서 우선순위가 더 높은 `ThreadPoolTaskExecutor` 를 Bean 중에서 찾습니다. <br/>
사전에 `ThreadPoolTaskExecutor` 를 Bean으로 등록하지 않았기 때문에 당연히 찾을 수 없고 그때 Spring이 Default Pool Setting 값으로 `ThreadPoolTaskExecutor` 를 빈으로 등록 후 해당 Baen을 이용해 요청받은 Task를 수행합니다.

> Default Pool Setting 값은 아래 래퍼런스의 Spring Docs를 참조하시면 됩니다.

디버깅한 결과 executor의 구현체로 `ThreadPoolTaskExecutor` 를 사용하는 것을 확인할 수 있습니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/default/default-executor1.png"/>
</div>

<br/>

### ThreadPoolTaskExecutor Bean이 1개만 있는 경우

``` java
@Configuration
public class ThreadPoolConfig {

    @Bean
    public ThreadPoolTaskExecutor myThreadPoolTaskExecutor() {
        final ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        ...
        return taskExecutor;
    }
}
```

위의 코드처럼 myThreadPoolTaskExecutor 이름으로 `ThreadPoolTaskExecutor` Bean을 등록한 뒤 비동기 호출을 하게되면, 
Spring에서 우선순위가 더 높은 `ThreadPoolTaskExecutor` 를 Bean 중에서 찾습니다. <br/>
사전에 myThreadPoolTaskExecutor 이름으로 된 `ThreadPoolTaskExecutor` Bean을 등록했기 때문에 해당 Bean을 찾고, 해당 Bean 이용해 요청받은 Task를 수행합니다.

디버깅한 결과 executor의 구현체로 `ThreadPoolTaskExecutor` 로 사용하며, beanName이 myThreadPoolTaskExecutor 인 것을 확인할 수 있습니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/default/default-executor2.png"/>
</div>

<br/>

### ThreadPoolTaskExecutor Bean이 2개 이상이지만 어떤 Bean을 사용해야하는지 정해져있는 경우

``` java
@Configuration
public class ThreadPoolConfig {

    @Primary
    @Bean
    public ThreadPoolTaskExecutor myThreadPoolTaskExecutor() {
        final ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        ...
        return taskExecutor;
    }

    @Bean
    public ThreadPoolTaskExecutor myThreadPoolTaskExecutor2() {
        final ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        ...
        return taskExecutor;
    }
}
```

위의 코드처럼 myThreadPoolTaskExecutor 이름과 myThreadPoolTaskExecutor2 이름으로 `ThreadPoolTaskExecutor` Bean 등록한 뒤, 두 개의 빈 중에 myThreadPoolTaskExecutor 이름의 Bean에 우선순위를 주었습니다.

비동기를 호출하면 Spring에서 우선순위가 더 높은 `ThreadPoolTaskExecutor` 를 Bean 중에서 찾습니다. <br/>
현재 `ThreadPoolTaskExecutor` Bean이 2개 등록되어 있기 때문에 Spring은 두개의 Bean을 찾고, 2개의 빈 중 우선순위가 더 높은 myThreadPoolTaskExecutor 이름의 Bean을 이용해 요청받은 Task를 수행합니다.

디버깅 결과 executor의 구현체로 `ThreadPoolTaskExecutor` 로 사용하며, beanName이 myThreadPoolTaskExecutor 인 것을 확인할 수 있습니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/default/default-executor3.png"/>
</div>

<br/>

### ThreadPoolTaskExecutor Bean이 2개 이상이지만 어떤 Bean을 사용해야하는지 정해져있지 않은 경우

``` java
@Configuration
public class ThreadPoolConfig {

    @Bean
    public ThreadPoolTaskExecutor myThreadPoolTaskExecutor() {
        final ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        ...
        return taskExecutor;
    }

    @Bean
    public ThreadPoolTaskExecutor myThreadPoolTaskExecutor2() {
        final ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        ...
        return taskExecutor;
    }
}
```

위의 코드처럼 myThreadPoolTaskExecutor 이름과 myThreadPoolTaskExecutor2 이름으로 `ThreadPoolTaskExecutor` Bean 2개를 등록했습니다.

비동기를 호출하면 Spring에서 우선순위가 더 높은 `ThreadPoolTaskExecutor` 를 Bean 중에서 찾습니다.. <br/>
현재 `ThreadPoolTaskExecutor` Bean이 2개 등록되어 있기 때문에 Spring은 두개의 Bean을 찾고, 2개의 Bean 중 어떤 Bean을 사용할지 모르기 때문에 Spring은 다음 우선순위인 `SimpleAsyncTaskExecutor`를 이용해 요청받은 Task를 수행합니다. 

디버깅 결과 executor의 구현체로 `SimpleAsyncTaskExecutor` 로 사용하는 것을 확인할 수 있습니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/default/default-executor4.png"/>
</div>

<br/>

## 마무리

Spring에서 @Async 어노테이션만 명시했을 때 Default Executor를 어떤 것을 사용하는지 확인해보았습니다.

우선 별도의 **Executor 지정없이 @Async를 사용하는 것을 지양하는 것** 이 제일 좋습니다. 지정없이 사용하다보면 의도치않게 SimpleAsyncTaskExecutor를 사용하게 될 수 있어 리소스 관리 측면에 문제가 생길 수 있습니다.

> SimpleAsyncTaskExecutor를 이용하게 되면 매번 신규 Thread를 생성해 Task를 수행하고, Task 수행이 완료되면 해당 Thread를 Terminate하고 있습니다.

API 호출을 받고 내부 수행을 비동기로 하게 될 때 SimpleAsyncTaskExecutor를 사용하게 되면 요청 받는 만큼 Thread를 생성하기 때문에 리소스 관리에 주의가 필요합니다.

또한 ThreadPoolTaskExecutor 사용하게 될 때도 이미 등록된 해당 Pool이 관리하는 Thread 내에서만 Task가 수행되기 때문에 리소스 관리측면에서는 안전하지만 여러 메서드에서 동일한 Pool을 사용하게 됨으로써 Thread가 예상치 못하게 모두 점유되어 기대했던 성능이 안나오거나 Pool 설정에 따라 `RejectedExecutionException` 예외가 발생될 수 있습니다.

긴글 읽어주셔서 감사합니다. :)

<br/>

## 레퍼런스

- [Spring Docs](https://docs.spring.io/spring-boot/docs/3.0.x/reference/html/features.html#features.task-execution-and-scheduling)
- [학습 및 테스트한 Repository](https://github.com/wlroh/Spring-Async)