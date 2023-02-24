---
layout: post
title: "Static 키워드"
date: 2022-03-01
excerpt: "Static은 무엇일까?"
categories: Java
tags: static 메모리
---

Java에서 Static 키워드를 사용한다는 것은 메모리에 한번 할당되어 프로그램이 종료될 때 해제되는 것을 의미합니다. 이를 정확히 이해하기 위해서는 메모리 영역에 대한 이해가 필요합니다.

- ### 메모리 구조(Static, Heap)

  - 일반적으로 우리가 만든 **Class는 Static area에 생성**되고, **new 연산을 통해 생성한 객체는 Heap area에 생성**됩니다.
  - Heap 영역의 메모리는 **Garbage Collector를 통해 수시로 관리**를 받습니다.
  - Static 키워드를 통해 **Static 영역에 할당된 메모리는 모든 객체가 공유하는 메모리**라는 장점을 지니지만, Garbage Collector의 관리 영역 밖에 존재하므로 Static을 자주 사용하면 **프로그램의 종료시까지 메모리가 할당된 채로 존재**하므로 자주 사용하게 되면 시스템의 퍼포먼스에 악영향을 주게 됩니다.

- ### 특징

  - Static 변수는 클래스 변수이다.
    > 변수에 관한 자세한 내용은 [변수의 종류](https://wlroh.github.io/java/2022/02/20/java-variables.html) 내용 참조
  - 객체를 생성하지 않고도 Static 자원에 접근이 가능하다.

  Static 변수와 Static 메소드는 Static 메로리 영역에 존재하므로 객체가 생성되지 이전에 이미 할당이 되어 있습니다. 그렇기 때문에 객체의 생성없이 바로 접근할 수 있습니다.

  ```java
  public class MyCalculator {
    public static String appName = "MyCalculator";

    public static int add(int x, int y) {
      return x + y;
    }

    public int min(int x, int y) {
      return x - y;
    }
  }

  MyCalculator.add(1,2);  // o static 메소드이므로 객체 생성 없이 사용가능
  MyCalculator.min(1,2);  // x static 메소드가 아니므로 객체 생성후에 사용가능

  MyCalculator cal = new MyCalculator();
  cal.add(1,2); // o 가능은 하지만 권장하지 않는 방법
  cal.min(1,2); // o
  ```

- ### Static 변수 (정적 변수)

  - 메모리에 고정적으로 할당되어 프로그램이 종료될 때 해제되는 변수로, **메모리에 한번 할당되면 여러 객체가 해당 메모리를 공유**하게 됩니다.
  - 예를들어, 세상 모든 사람의 이름이 'Pual'인 세상에 살고있다고 가정을 하겠습니다. 이럴때면 우리는 아래와 같이 객체를 만들 수 있습니다.

    ```java
    public class Person {
      private String name = "Paul";

      public void printName() {
        System.out.println(this.name);
      }
    }
    ```

  - 하지만 위와 같은 클래스를 통해 100명의 Person 객체를 생성하면, "Pual" 라는 값을 갖는 메모리가 100개나 중복해서 생성되게 됩니다.
  - 이러한 경우에 **static을 사용하여 여러 객체가 하나의 메모리를 참조**하도록 하면 메모리 효율이 더욱 높아질 것입니다.
  - 또한, "Pual"라는 이름은 결코 변하지 않는 값이므로 final 키워드를 붙여주며, 일반적으로 Static은 상수의 값을 갖는 경우가 많으므로 public으로 선언을 하여 사용합니다.

    ```java
    public class Person {
      public static final String name = "Paul";

      public void printName() {
        System.out.println(this.name);
      }
    }
    ```

- ### Static 메소드 (정적 메소드)

  - Static 메소드는 객체의 생성 없이 호출이 가능하며, 객체에서는 호출이 가능은 하지만 지양하고 있습니다.
  - 일반적으로 유틸리티 관련 함수들은 여러번 사용되므로 static 메소드로 구현을 하는 것이 적합한데, static 메소드를 사용하는 대표적인 Util Class로는 java.util.Math가 있습니다.
    ```java
    public class Test {
      public void printMax(int x, int y) {
        System.out.println(Math.max(x, y));
      }
    }
    ```
  - Static 메소드에서는 **static이 선언되지 않은 변수에 접근이 불가능**합니다. 아래 예시 코드를 보면,

    ```java
    public class Test {
      private String name1 = "Paul";
      private static String name2 = "Paul";

      public static void printName(){
        // System.out.println(name1); 불가능한 호출
        System.out.println(name2);
      }
    }
    ```

  - 우리가 Test.printName()을 사용하려고 하는데, name1은 new 연산을 통해 객체가 생성된 후에 메모리가 할당됩니다.
  - 하지만 static 메소드는 객체의 생성 없이 접근하는 함수이므로, 할당되지 않은 메모리 영역에 접근을 하므로 문제가 발생하게 됩니다.
  - 그러므로 static 메소드에서 접근하기 위한 변수는 static 변수 혹은 지역변수, 매개변수만 가능합니다.

- ### 실제 Static 변수와 Static 메소드의 활용

  - static 변수

    - 일반적으로 상수들만 모아서 사용하며 상수의 변수명은 대문자와 \_를 조합하여 만든다. 또한 상속을 방지하기 위해 final class로 선언한다.

      ```java
      public final class AppConstants {

        public static final String APP_NAME = "myApp";
        public static final String APP_VERSION = "ver.1.2.2";

      }
      ```

  - Static 메소드

    - 마찬가지로 상속을 방지하기 위해 final class로 선언을 하고, 유틸 관련된 함수들을 모아둔다.

      ```java
      public final class CommonUtils {

        public static String getCurrentDate() {

          Date date = new Date();
          SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
          return sdf,format(date);

        }

      }
      ```
