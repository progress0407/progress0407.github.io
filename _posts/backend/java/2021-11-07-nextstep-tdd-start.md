---
layout: post
title: "플레이그라운드, 넥스트 스탭 강의 정리"
subtitle: ""
date: 2021-11-07 15:30:00 +0900
categories: backend
tags: java
comments: true
---

## 플레이그라운드 자바 - 인프런

---

- `if`, `else` 예약어 뒤의 중괄호는 안써도 되는 상황(한줄)에서도 쓰는게 좋다

## 넥스트 스탭 박재성님 - 객체지향 & TDD

---

### TDD Cycle

- 1. Test Fails
  - 먼저 실패하는 테스트 케이스를 만든다
- 2. Test Success
  - (첫번째일 경우) 해당 테스트를 성공시킬 정도의 간단한 프로덕션 코드를 만든다
  - (그 이후)
- 3. Refactor
  - 프로덕션 코드를 리펙터링한다

### 그외 메모

---

- 너무 자주 인터페이스를 만들지 말아라

  - 정리 되지 않은 인터페이스들이 남게 된다

- 필드에 유효성 검사 등이 필요시 객체로 나눌 것

  - 앞단에서 유효성 체크를 한다고 해서 안쪽까지 체크되는지 모른다
  - 코드의 중복을 막을 수 있다
  - SRP (단일 책임 원칙)와 관련되어 있다
    - 변경지점이 한곳으로만 갈 수 있게끔 설계하자

- 테스트를 위해서 생성자 추가하는 것도 나쁘지 않다

- `containsExactly`는 **순서**까지 체크한다 !!

### 문법적인 사항

---

- Shape이 조상 Line이 자손이라 했을 때 조상에서

```java
Shape(List<Point> points) {
  this.points = points;
}
```

와 같이 생성자 주입 받는 것을 자손객체가 해당 기능을 물려받지 않는다

혹은 자손에서

```java
Line(List<Point> points) {
  super();
}
```

라고 하여도 조상에만 반영이 된다.

항상 하듯이 아래처럼 해주어야 한다.

```java
Line(List<Point> points) {
  this.points = points;
}
```

### 자바 컨벤션

- 한 클래스 내에서 순서
  - 상수 (`static final`)
  - 클래스 변수 (`static`)
  - 인스턴스 변수 (객체)
  - 생성자

```java
public class SmartPhone {
  private static final int FIRST_RELEASE = 2006; // 상수

  private static int autoPriceIncrease = 10000; // 클래스 변수

  private String name; // 인스턴스 변수
  private int price;
  private int batteryCapacity;

  public SmartPhone(int price) { // 생성자
    this.price = price;
  }

  // 일반 메서드
  public void turnOn() {
    ...
  }

  public void call() {
    ...
  }


  // equals, hashCode, toString

}
```

> 핵데이 Java 컨벤션
> https://naver.github.io/hackday-conventions-java/

### 2.13. 임시 변수 외에는 1 글자 이름 사용 금지

---

> [avoid-1-char-var]

메서드 블럭 범위 이상의 생명 주기를 가지는 변수에는 1글자로 된 이름을 쓰지 않는다. 반복문의 인덱스나 람다 표현식의 파라미터 등 짧은 범위의 임시 변수에는 관례적으로 1글자 변수명을 사용할 수 있다.

---

위 사항에 따르면 이제 람다식 안에

```java
...stream().filter(e -> e.getName() ... ) ...
```

등으로 한글자 표기를 안심하고 사용해도 된다 !!
