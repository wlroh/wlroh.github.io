---
layout: post
title: "RejectedExecutionException과 Handler"
date: 2023-12-23
excerpt: "RejectedExecutionException과 Handler에 대해 알아보자"
categories: Spring Java
---

비동기를 사용하기 위해 ThreadPool 을 사용하다보면 `RejectedExecutionException` 가 종종 발생해서 로그 혹은 모니터링 알림을 받는 케이스가 있을 겁니다. 비동기 실행이 거절되었을 때 발생하는 예외인데 왜 발생하고, 후처리를 할 수 있는 방안이 없는지 확인해보려고 합니다.

<br/>

## `RejectedExecutionException` 는 왜 발생할까?

비동기로 구현하기 위해 ThreadPoolTaskExecutor 를 사용해 ThreadPool 을 구성했을 때 다음과 같은 메커니즘으로 작동합니다.

> 현재 CorePool을 모두 사용 -> Queue가 가득 차있음 -> 현재 MaxPool 모두 사용 -> RejectedExecutionHandler 수행

즉, 설정한 가용 리소스가 모두 사용중일 때 더이상 비동기 수행이 불가능하기에 설정된 handler 에 따라 로직이 수행되는데, `ThreadPoolExecutor` 와 `ScheduledThreadPoolExecutor` 의 Default Handler는 `AbortPolicy` 를 사용하는데 해당 handler 는 `RejectedExecutionException` 를 항상 던지고 있습니다.

```java
public static class AbortPolicy implements RejectedExecutionHandler {

    public AbortPolicy() { }

    /**
     * Always throws RejectedExecutionException.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     * @throws RejectedExecutionException always
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        throw new RejectedExecutionException("Task " + r.toString() +
                                              " rejected from " +
                                              e.toString());
    }
}
```

만약, 별도의 RejectedExecutionHandler를 설정하지 않았다면 Default Handler 인 AbortPolicy를 이용하게 되고, RejectedExecutionException 이 발생하고 있는 구조로 보시면 됩니다.

<br/>

## 그렇다면 `RejectedExecutionHandler` 다른 구현체는 없을까?

Java 에서는 `RejectedExecutionHandler` 의 구현체로 4개가 있습니다. 어떤 구현체들이 있고 어떤 기능이 있는지 확인해보면 다음과 같습니다.

### AbortPolicy

위에서 언급한대로 Default Handler로 항상 `RejectedExecutionException`을 던지는 Handler 입니다.

### CallerRunsPolicy

`CallerRunsPolicy` 의 코드를 보면 다음과 같습니다.

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {
    
    public CallerRunsPolicy() { }

    /**
     * Executes task r in the caller's thread, unless the executor
     * has been shut down, in which case the task is discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            r.run();
        }
    }
}
```

RejectedExecutionException 예외를 던지지 않고, 비동기를 요청한 Thread가 종료되지 않았으면 요청한 쓰레드에서 로직을 수행하게 됩니다.<br/>
예외가 발생하지 않고 정상로직 수행을 하겠지만, 비동기가 아닌 동기방식으로 바뀌기 때문에 성능이슈가 발생할 수 있습니다. 

### DiscardPolicy

`DiscardPolicy` 의 코드를 보면 다음과 같습니다.

```java
public static class DiscardPolicy implements RejectedExecutionHandler {

    public DiscardPolicy() { }

    /**
     * Does nothing, which has the effect of discarding task r.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    }
}
```

이때는 RejectedExecutionException 예외를 던지지 않고, 요청한 Task 를 버리는 정책입니다.<br/>
개인적으로, DiscardPolicy 정책을 사용하면 모든 가용리소스가 사용중일때 버리게 되는 것이라 추후 이슈 대응이 어려워진다는 점에서 실무에서 사용하기 어렵다고 생각됩니다.

### DiscardOldestPolicy

`DiscardOldestPolicy` 의 코드를 보면 다음과 같습니다.

```java
public static class DiscardOldestPolicy implements RejectedExecutionHandler {

    public DiscardOldestPolicy() { }

    /**
     * Obtains and ignores the next task that the executor
     * would otherwise execute, if one is immediately available,
     * and then retries execution of task r, unless the executor
     * is shut down, in which case task r is instead discarded.
     *
     * @param r the runnable task requested to be executed
     * @param e the executor attempting to execute this task
     */
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        if (!e.isShutdown()) {
            e.getQueue().poll();
            e.execute(r);
        }
    }
}
```

이때는 `DiscardPolicy` 과 비슷하게 Task 를 버리는 방식은 동일한데, 요청한 Task가 아닌 큐에 등록된 가장 오래된 큐를 버리고 이번에 요청들어온 Task 를 큐에 등록하는 정책입니다.

> Queue는 FIFO 방식으로 선입선출 자료구조입니다. 그렇기 때문에 가장 오래된 큐는 가장 빠르게 등록된 큐로 보시면 됩니다.

<br/>

## 4개의 정책 모두 충족이 안 될 경우 다른 방안은 없을까?

Java 에서 가장 강력한 능력 중 하나인 `다형성` 이 있기 때문에, 직접 커스텀 Handler 를 구현해주셔도 됩니다.

`RejectedExecutionHandler` 인터페이스를 이용해 구현체를 만들어주면, 현재 모든 가용리소스가 사용중일 때 어떤 식으로 후처리할지 직접 업무 상황에 맞게 처리가 가능합니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/spring-async/rejectedexecutionhandler/image.png"/>
</div>

<br/>

## 결론

실무를 하다보면 비동기를 구현을 위해 쓰레드풀을 생성하는 케이스가 많은데 적정 쓰레드풀 산정이 어려움이 많습니다. 그래서 `RejectedExecutionException` 이 발생하는 케이스도 존재합니다.

물론, K6 혹은 nGrider 등 툴을 이용해 부하테스트를 하여 적정 쓰레드 풀을 만들어도 예상치 특정 프로모션, 이벤트로 인해 트래픽이 몰리며 `RejectedExecutionException` 발생할 수 있는데 Handler 를 커스텀하거나 적절한 Handler 세팅을 통해 도움이 되셨을면 좋겠습니다.

감사합니다. :)

<br/>

## 레퍼런스

- [ThreadPoolExecutor Docs](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadPoolExecutor.html)
- [학습 및 테스트한 Repository](https://github.com/wlroh/Spring-Async/blob/main/src/test/java/com/wlroh/async/threadtaskpoolexecutor/RejectedExceptionTest.java)
