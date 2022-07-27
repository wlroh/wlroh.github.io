---
layout: post
title: "equals와 비교연산자"
date: 2022-02-27 00:22:00 +0530
excerpt: "equals 메소드와 비교연산자에 대해 알아보자"
categories: Java
tags: equals
---

### equals 메소드란?

```java
public class Object {
  public boolean equeals(Object obj) {
    return (this == obj);
  }
}
```

`equals` 메소드는 `Object` 클래스의 메소드이다. `Object` 클래스 내부에 정의된 `equals` 메소드를 보면 위와 같다. 내용은 `매개변수(parameter)` 로 객체의 `참조 변수`를 가지고 비교한다.

위에서 보면 `==` 연산자가 나오는데 이것의 의미부터 정리하고 가자.

### `==` 연산자란?

비교연산자 `==` 는 `int`, `boolean` 과 같은 `원시타입(primitive type)`에 대해서는 값을 비교한다.
하지만 `참조타입(reference type)`에 대해서는 `주소값`을 비교해서 같으면 `true`, 다르면 `false` 를 반환해주는 연산자이다. 예를들어 아래의 예시를 봐보자

```java
public class Test {

  public static void main(String[] args) {
    int a = 3;
    int b = 3;

    if (a == b) {
      System.out.println("3 == 3 으로 같습니다.");
    } else {
      System.out.println("3 != 3 으로 다릅니다.");
    }

    String str1 = "test";
    String str2 = "test";

    if (str1 == str2) {
      System.out.println("test == test 으로 같습니다.");
    } else {
      System.out.println("test != test 으로 다릅니다.");
    }
  }
}

// 3 == 3 으로 같습니다.
// test == test 으로 같습니다.
```

`int`는 `원시타입(primitive type)`이기 때문에 `값`을 비교해서 나오는 것을 알 수 있다. `String` 클래스는 `new` 연산자를 이용한 것이 아니라 `객체 리터럴`을 이용해서 `상수풀(String Constant Pool)`의 값을 참조하기 때문에 같은 주소 값을 참조하고 있기 때문에 `true`를 반환한다.

> 상수풀(String Constant Pool)
>
> 상수풀의 위치는 `java 7`부터 `Perm` 영역에서 `Heap` 영역으로 옮겨졌다. Perm 영역은 실행 시간(Runtime)에 가변적으로 변경할 수 없는 고정도니 사이즈이기 때문에 intern 메소드의 호출은 저장할 공간이 부족하게 만들 수 있었다. 즉 OOM(Out Of Memory) 문제가 발생할 수 있는 위험이 있었던 것이다.
>
> Heap 영역으로 변경된 이후에는 상수풀에 들어간 문자열도 Garbage Collection 대상이 된다.

```java
public class Test {

  public static void main(String[] args) {
    Test test1 = new Test();
    Test test2 = new Test();

    if (test1 == test2) {
      System.out.println("true");
    } else {
      System.out.println("false");
    }
  }
}
```

따라서 위와 같이 `new` 연산자를 이용해서 `Heap` 영역에 객체를 만들었을 때는 각자의 메모리 영역이 존재하기 때문에 주소값이 달라 `false` 를 반환하게 된다.

### 그러면 이제 다시 `equals` 메소드에 대해 얘기해보자.

```java
public class Object {
  public boolean equals(Object obj) {
    return (this == obj);
  }
}
```

다시 `Object` 클래스 내부의 `equals` 메소드를 보면 `==` 연산자를 이용해서 `주소값`을 비교하는 코드인 것을 알 수 있다. 또한 가장 최상위 클래스 `Object`에 정의되어 있기 때문에 하위클래서에서 `오버라이딩`해서 사용할 수 있다.

```java
public class Test {
  int a;

  public Test(int a) {
    this.a = a;
  }

  public static void main(String[] args) {
    Test test1 = new Test(10);
    Test test2 = new Test(10);

    if (test1.equals(test2)) {
      System.out.println("test1과 test2는 같다.");
    } else {
      System.out.println("test1과 test2는 다르다.");
    }
  }
}

// test1과 test2는 다르다.
```

따라서 위의 코드를 보면 `test1`과 `test2`는 `주소값`이 다르기 때문에 `else`문 안에 있는 결과가 나오게 된다.

### 그러면 아래 코드의 결과는 어떻게 나올까?

```java
public class Test {
  public static void main(String[] args) {
    String str1 = new String("abc");
    String str2 = new String("abc");

    if (str1.equals(str2)) {
      System.out.println("같습니다.");
    } else {
      System.out.println("다릅니다.");
    }
  }
}

// 같습니다.
```

결과는 `같습니다`가 나오게 된다. 아까는 `주소값을 비교한다 했는데 주소값이 다른데 왜지?`라고 생각할 수 있다. 다르게 나오는 이유는 `String` 클래스에서 `equals` 메소드를 `오버라이딩` 했기 때문에 `같습니다.`가 나오는 것이다.

```java
public class String {

  ...

  public boolean equals(Object anObject) {
    if (this == anObject) {
      return true;
    }
    if (anObject instanceof String) {
      String aString = (String)anObject;
      if (!COMPACT_STRINGS || this.coder == aString.coder) {
        return StringLatin1.equals(value, aString.value);
      }
    }
    return false;
  }

  ...

}
```

위와 같이 `String` 클래스에서 `equals` 메소드를 `오버라이딩` 했기 때문에 `주소값`이 아니라 `내용 값`으로 비교하여 `true`, `false`를 반환하게 되는 것이다.

> Java에서 `equals` 메소드는 기본적으로 각 API 클래스마다 자체적으로 `오버라이딩`을 통해 재정의되어 있습니다.

```java
public class Test {
  Long id;

  public Test(long id) {
    this.id = id;
  }

  public boolean equals(Object obj) {
    if (this == obj) {
      return true;
    }
    if (obj instanceof Test) {
      return id == ((Test)obj).id;
    }
    return false;
  }

  public static void main(String[] args) {
    Test test1 = new Test(10L);
    Test test2 = new Test(10L);

    if (test1 == test2) {
      System.out.println("test1과 test2는 주소가 같다.");
    } else {
      System.out.println("test1과 test2는 주소가 다르다.");
    }

    if (test1.equals(test2)) {
      System.out.println("test1과 test2의 내용은 같다.");
    } else {
      System.out.println("test1과 test2의 내용은 다르다.");
    }
  }
}

// test1과 test2는 주소가 다르다.
// test1과 test2의 내용은 같다.
```

위와 같이 `equals` 메소드를 `주소`가 아니라 `내용`으로 비교하게 직접 `오버라이딩` 해서 사용할 수도 있다.
