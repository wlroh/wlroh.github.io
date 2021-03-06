---
layout: post
title: "객체지향 생활 체조 원칙"
date: 2022-06-17
excerpt: "객체지향 생활 체조 원칙에 대해 알아보자"
categories: Java
---

이번 NextStep의 TDD, Clean Code with Java 14기 교육을 받으며 맨 처음 알게 된 것이 객체지향 생활 체조 원칙입니다.

## 객체지향 생활 체조 원칙은 무엇일까?

객체지향 생활 체조 원칙은 `소트웍스 앤솔러지` 책에서 다루고 있는 내용으로 책게지향 프로그래밍을 잘하기 위한 9가지 원칙을 제시하고 있습니다. 이 책에서 주장하는 9가지 원칙은 다음과 같습니다.

1. 한 메서드에 오직 한 단계의 들여쓰기(indent)만 한다.
2. else 예약어를 쓰지 않는다.
3. 모든 원시값과 문자열을 포장한다.
4. 한 줄에 점을 하나만 찍는다.
5. 줄여쓰지 않는다.(축약 금지)
6. 모든 엔티티를 작게 유지한다.
7. 3개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.
8. 일급 콜렉션을 쓴다.
9. 게터/세터/프로퍼티를 쓰지 않는다.

객체지향 체조 원칙은 객체 지향 설계와 구현과 관련해서 구체적인 가이드를 제시하고 있습니다. 객체지향 설계 혹은 객체지향 프로그래밍을 할 때 처음 하시는 분들의 경우 어디서 부터 접근을 해야될지 감이 안오는 경우가 많이 있습니다.

이런 경우 객체지향 체조원칙을 바탕으로 리팩토링이 필요한 곳들을 쉽게 찾을 수 있고, 수정하면서 객체지향 프로그래밍에 다가갈 수 있다는 점에는 객체지향 체조 원칙이 정말 중요하다고 생각이 되어집니다.

### 원칙 1. 한 메서드에 오직 한 단계의 들여쓰기(indent)만 한다.

클린코드 책에서 메서드를 만드는 첫 번째 규칙은 `작게` 라고 말합니다. 그리고 두번째 규칙은 `더 작게` 입니다. 또한 메서드는 `한 가지 일만 해야한다` 라고 말합니다.

**메서드는 맡은 일이 적을수록 재사용성이 높고 디버깅이 용이합니다.** 그래서 들여쓰기가 많이 있는 경우 한가지 이상의 일을 맡고 있을 확률이 높고 여러가지 일을 하다보니 재사용성이 떨어지고 유지보수를 하기 어려워집니다. 이렇기 때문에 오직 한 단계의 들여쓰기만 하도록 리팩토링하다보면 자연스럽게 메소드가 작아지며 한가지 일만 하도록 하여 재사용성을 높이고 가독성 또한 좋아져 유지보수성이 용이해집니다.

그럼 오직 한 단계의 들여쓰기를 하기 위한 방법으로는 `메서드 분리` 혹은 `클래스 분리` 이 있으며, 위의 방법으로 리팩토링을 하면 한 단계의 들여쓰기를 유지하면서 더 좋은 클린코드를 만들 수 있습니다.

### 원칙 2. else 예약어를 쓰지 않는다.

`else` 예약어를 쓰게되면 기능을 구현할 때 빠르게 할 수 있습니다. 하지만 달면 몸에 안좋다는 얘기처럼 `else` 예약어를 쓰게되면 유지보수 측면에서 단점들이 많이 있습니다.

우선, 코드를 보면서 얘기를 하면

```java
public class Order {

    private static final String REQUEST = "주문 요청";
    private static final String COMPLETE = "주문 완료";

    private Boolean isComplete;

    ...

    public String status() {
        String orderStatus = "";
        if(isComplete) {
            orderStatus = COMPLETE;
        } else {
            orderStatus = REQUEST;
        }
        return orderStatus;
    }
}
```

위와 같은 오더의 상태를 확인하는 메서드가 있을 때 `orderStatus` 로컬변수가 전체적으로 퍼지게 되어 `side effect` 가 나타날 확률이 높아집니다. 또한 코드의 의도를 읽기 쉽지 않고 `가독성` 이 떨어져 유지보수 측면에서도 안좋아 집니다.

```java
public class Order {

    private static final String REQUEST = "주문 요청";
    private static final String COMPLETE = "주문 완료";

    private Boolean isComplete;

    ...

    public String status() {
        if(isComplete) {
            return COMPLETE;
        }
        return REQUEST;
    }
}
```

초기 코드를 `early return` 방식을 이용해 리팩토링을 해보았는데 코드 라인도 줄어들고, 가독성 또한 좋아지며, 로컬 변수 또한 필요없어져서 `side effect` 리스크도 줄일 수 있습니다.

`else` 예약어를 안쓰기 위해 **인터페이스, early return** 등 다양한 방법이 있고, 이 방법을 이용해 개발을 하면 `else` 를 사용하는 방식 고려해야 되는 것들이 더 많아 시간은 다소 걸리겠지만 **장기적인 관점에서 유지보수 측면과 추후 확장성 등을 고려했을 때 else 예약어를 쓰지 않는 것이 좋습니다.**
