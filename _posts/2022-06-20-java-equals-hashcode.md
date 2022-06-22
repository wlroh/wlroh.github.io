---
layout: post
title: "equals와 hashCode"
date: 2022-06-20
excerpt: "equals와 hashCode는 왜 같이 재정의해야할까?"
categories: Java
---

## `equals` 와 `hashCode` 는 같이 재정의해야할까?

넥스트스텝 교육을 듣던 중에 equals 메서드와 hashCode 메서드를 같이 재정의해야한다는 이야기가 있었습니다. equals 와 hashCode 가 각각 어떤 메서드인지는 알았지만 왜 같이 사용하는지에 대해서는 의문이 들면서 정리해보았습니다.

equals 와 hashCode 관련해서는 `이펙티브 자바` 책에서도 "equals를 재정의하려거든 hashCode도 재정의하라"라는 내용이 있습니다. 또한 대부분의 IDE Generate 기능에서도 equals 와 HashCode를 같이 재정의 해주며 lombok에서도 EqualsAndHashcode 어노테이션으로 같이 재정의해주고 있습니다.

우선 위의 내용만 봐도 "아... 다 이렇게 강조하는 것이면 무엇인가 이유가 있구나..." 생각이 듭니다. 우선 equals와 hashCode를 같이 쓰는 이유를 알아보기 이전에 각각의 메서드를 재정의 하는 이유에 대해 먼저 알아봅시다.

<br/>

## `equals` 재정의하는 이유는 무엇일까?

자바에서는 두 객체를 **동등 비교** 할 때 대부분 equals 메서드를 사용합니다. 재정의 없이 바로 사용할 경우 객체들의 최상위 클래스 `Object`에 정의되어 있는 `equals` 메서드를 사용하게 되는데, 주소값을 비교하는 로직으로 정의되어 있어 그대로 사용하게 되면 **동등한지 비교(동등성)가 아닌 동일한지 비교(동일성)를 하기 때문에 재정의가 필요** 합니다.

> 동등성: 두개의 객체가 같은 정보를 갖고 있는 경우를 의미한다. 객체의 주소가 서로 다르더라도 내용만 같으면 두 객체는 동등하다고 의미한다.
>
> 동일성: 두 객의 객체가 완전히 같은 경우를 의미한다. 주소 값이 같기 때문에 두 개의 객체는 동일한 객체로 볼 수 있다.

```java
public class Object {

    public boolean equals(Object obj) {
        return (this == obj);
    }

    ...

}
```

최상위 클래스인 `Object` 의 `equals` 메서드를 보면 비교연산자(==)을 통해 객체 비교를 하고 있고 비교연산자는 객체의 주소값을 동일한지 비교(동일성)하고 있는 것을 알 수 있습니다.

> equals 메서드와 비교연산자(==)에 대해 알아보고 싶으면 [equals와 비교연산자](https://wlroh.github.io/java/2022/02/26/java-equals.html) 를 참고하자!

그래서 `Person` 이라는 별도의 클래스를 만들었을때

```java
public class Person {

    private Stirng name;
    private int age;

    ...
}

@Test
void equals() {
    Person person = new Person("tom", 30);
    Person samePerson = new Person("tom", 30);

    assertThat(person.equals(samePerson)).isFalse();
}
```

`equals` 메서드를 재정의(override) 없이 바로 사용하면 두 개의 객체가 같은 정보를 가지고 있지만 주소값이 다르기 때문에 `false` 가 나오게 됩니다. 이러한 이유로 객체간의 동등 비교를 하기 위해 `equals` 메서드를 재정의합니다.

```java
public class Person {

    private Stirng name;
    private int age;

    ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
}

@Test
void equals() {
    Person person = new Person("tom", 30);
    Person samePerson = new Person("tom", 30);

    assertThat(person.equals(samePerson)).isTrue();
}
```

이처럼 `equals` 메서드를 재정의해주어 `객체간의 동등성 비교` 를 하기 위해서 재정의를 합니다.

<br/>

## `hashCode` 재정의하는 이유는 무엇일까?

`hashCode` 메서드는 **객체를 식별하는 하나의 정수값을 반환** 하기 위해 사용 되어집니다. 여기서도 hashCode 메서드를 재정의하지 않고 사용하면 최상위 클래스인 `Object`의 hashCode() 메서드를 사용하게 되는데 Object의 hashCode 메서드는 **객체의 메모리 주소값** 을 이용해서 정수값을 만들어 리턴하기 때문에 객체마다 다른 값을 가지고 있습니다.

hashCode 메서드의 의미는 이해했는데 여기서 왜 재정의를 해야되는지에 관해 설명드리기 위해서는 우선 자바의 `Collection(HashSet, HashMap, HashTable)` 과 같이 연관지어서 설명해야합니다.

Collection(HashSet, HashMap, HashTable)은 객체가 논리적으로 같은지(동등성 비교) 비교할 때 아래 그림과 같은 과정을 거칩니다.

<div align=center>
  <img src="https://raw.githubusercontent.com/wlroh/wlroh.github.io/main/assets/images/2022-06-20-java-equals-hashcode-1.png"/>
</div>

hashCode 메서드의 리턴 값이 우선 일치하고 equals 메서드의 리턴 값이 true 일 경우 `동등 객체` 로 판단합니다.

그러면, 위에서 사용한 `Person` 클래스를 다시보면 현재 `equals` 메서드만 재정의되어 있고 `hashCode` 메서드는 재정의 되어 있지 않습니다. 이때 HashSet 컬렉션에 넣었을 때 결과값을 보면 아래와 같습니다.

```java
public class Person {

    private Stirng name;
    private int age;

    ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
}

@Test
void equals() {
    Person person = new Person("tom", 30);
    Person samePerson = new Person("tom", 30);

    Set<Person> people = new HashSet<>();
    people.add(person);
    people.add(samePerson);

    assertThat(people.size()).isEqualTo(2);
}
```

`Set` 컬렉션의 경우 **객체를 중복해서 저장할 수 없고, 저장 순서가 보장되지 않는 특징** 을 가지고 있습니다. 그럼 위의 테스트 코드를 보면 개발자가 기대하는 것은 `person`과 `samePerson` 객체를 동등객체로 바라보고 있기 때문에 `Set` 컬렉션에 추가했을 때 중복이 제거되어 `people` 의 크기가 `1` 을 기대하지만, `people` 의 크기는 `2` 입니다.

그 이유는 바로 `hashCode` 가 재정의 되어 있지 않아 최상위 클래스 `Object` 의 `hashCode` 메서드를 호출하게 되는데 해당 메서드는 객체의 메모리 주소값을 이용해 정수값을 반환하는 메서드이기 때문입니다.

두 객체의 메모리 주소 값은 다르기 때문에 hashCode 메서드의 반환 값이 서로 다르고, 다르기 때문에 Set 컬렉션은 다른 객체로 인식해 중복 제거 없이 두 객체 모두 저장하여 크기가 2가 되는 것입니다.

그럼 hashCode 메서드를 재정의한 뒤 다시 테스트를 해보면,

```java
public class Person {

    private Stirng name;
    private int age;

    ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

@Test
void equals() {
    Set<Person> people = new HashSet<>();
    people.add(person);
    people.add(samePerson);

    assertThat(people.size()).isEqualTo(1);
}
```

해시코드를 메모리 주소값이 아닌 객체의 내부 값들을 이용해 정수값을 반환하도록 재정의되어 처음 기대했던 것처럼 두 객체를 동등객체로 판단하여 중복이 제거된 것을 확인할 수 있습니다.

이러한 이유로 해시값을 이용한 컬렉션을 사용할 때 **동등성 비교** 를 위해 재정의가 필요합니다.

<br/>

## 그럼 꼭 `equals` 와 `hashCode` 를 같이 재정의 해주어야할까?

결론만 말하면 저는 `Yes!` 라고 답변드리고 싶습니다.

해시값을 이용한 컬렉션을 이용하지 않은 경우 `equals` 메서드만 재정의해주어도 객체간의 동등비교 시 문제는 없습니다. 다만, 제가 만든 객체를 다른 개발자가 사용할 때 해시값을 이용한 컬렉션을 사용하거나 해시 값을 이용한 동등비교를 하게 되면 문제가 발생될 소지가 있어 `equals` 메서드와 `hashCode` 메서드를 둘 다 재정의 해주는 것이 좋다고 생각되어집니다.

또한 `equals` 와 `hashCode` 메서드는 둘다 **동등 비교를 위해 필요한 것인데 객체간의 동등 비교를 할 때 둘 중 하나만 재정의해주면 해당 객체는 동등 비교를 제대로 할 수 없다** 고 생각됩니다. 만약 equals 메서드만 재정의하고 hashCode 메서드는 재정의하지 않은 경우 해시값을 이용한 동등 비교가 메모리 주소값으로 하게되어 제대도 된 동등 비교를 할 수 없는 개체로 볼 수 있어 `equals` 와 `hashCode` 메서드 둘 다 재정의하는 것이 더 좋은 코드를 짜는 개발자이지 않을까 생각됩니다.

### 레퍼런스

- [동일성(identity)과 동등성(equlity)](https://steady-coding.tistory.com/534)
- [hashcode()와 equals() 메서드는 언제 사용하고 왜 사용할까?](https://jisooo.tistory.com/entry/java-hashcode%EC%99%80-equals-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B3%A0-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C)
- [equals와 hashCode는 왜 같이 재정의해야 할까?](https://tecoble.techcourse.co.kr/post/2020-07-29-equals-and-hashCode/)
