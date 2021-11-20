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

## 자동차 경주 게임 구현 도중

### indent

---

![image](https://user-images.githubusercontent.com/66164361/142017242-341e5226-6a91-4315-8b24-3c20a387dd42.png)

위와 같이 indent를 더 이상 줄이기 힘들다고 판단되는 곳까지 왔다

사실 본인은 indent를 1로 만드는 것이 더이상 불가능하다고 생각했다

그러나.. 갓텔리제이가 힌트를 주기를...

![image](https://user-images.githubusercontent.com/66164361/142017625-640925cb-3b66-4edd-aaec-632c10181fe8.png)

이것을 `do while`문으로 바꿔보지 않겠는가 라고 계시를 내려주었다

![image](https://user-images.githubusercontent.com/66164361/142017793-fd9fc0d8-0e70-4c9d-b5d0-863372e675ed.png)

역시 갓텔리제이다

## 스택프레임과 객체

---

아래와 같이 스프링 DI 처럼 외부에서 객체를 주입하고 싶었다

그래서 아래처럼 코드를 구현하였는데..

(구현부)  
![image](https://user-images.githubusercontent.com/66164361/142428308-35a2b099-f816-4a6b-a357-6b0556071f80.png)

(실행부)  
![image](https://user-images.githubusercontent.com/66164361/142428485-822b8a7e-2ff4-46a7-a363-969c11805929.png)

근데 이상한 건 항상 외부에서 객체를 꺼냈을 떄 null이 나오는 것이다..

분명히 필자는 객체는 `reference`타입이라서

어떤 곳에서든지 객체를 생성받아서 값을 set하더라도

항상 동일한 객체를 바라본다고 생각했었다

그런데 null이 나왔다..

![image](https://user-images.githubusercontent.com/66164361/142427722-3dfc963a-e444-48c5-b6cc-abcde1c2e4b6.png)

그래서 조사를 해보았다

처음 main 메서드를 실행할 때는 `@815`이다. 이때 DI처럼 Prompt에 생성자 주입을 한다

그리고 새로운 객체를 생성하는 순간 ! `@980` 객체가 들어온다 (아래 그림)

![image](https://user-images.githubusercontent.com/66164361/142427805-e3746cd8-583f-4957-9aa8-30a7e947f28f.png)

게임이 종료된 후 호출부인 main 메서드에서 확인해보니 `@815` 객체가 표시되었다..

호출한 곳는 여전히 과거의 객체를 보고 있었고

호출 된 곳은 새로이 생성된 객체를 바라보고 있었다..

![image](https://user-images.githubusercontent.com/66164361/142427938-b450b27a-d07c-4c58-94d3-6cd8f5a7e35f.png)

그래서.. 새롭게 생성된 객체를 꺼내고 싶으면

![image](https://user-images.githubusercontent.com/66164361/142429671-48f74265-481d-40aa-ba32-ad35d8057242.png)

prompt가 이 일을 한다는 것이 조금 이상하게 느껴지지만.. 새로운 객체를 꺼내는 것에는 성공하였다
