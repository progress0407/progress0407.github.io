---
layout: post
title: "[최범균님] 객체지향과 디자인 패턴 : SOLID"
subtitle: "..."
date: 2022-03-20 22:00 +0900
categories: book
tags: cbk-basic-design-pattern
comments: true
published: true
---

## 단일 책임 원칙 (Single Responsibility Principle)

---

클래스는 단 한 개의 책임을 가져야 한다.

- 클래스를 변경하는 이유는 단 한 개여야 한다
  - `이 멘트가 이해가 되지 않았다`

### 단일 책임 원칙 위반이 불러오는 문제점

책임의 개수가 많아질 수록 한 책임의 기능 변화가 다른 책임에 주는 영향은 비례해서 증가하게 된다

이는 결국 코드를 절차 지향적으로 만들어 변경을 어렵게 만든다

### 책임이란 변화에 대한 것

책임의 단위는 변화되는 부분과 관련된다는 의미가 된다.

예를 들어 `DataViewer` 클래스에 `데이터를 읽어 오는 기능`에 변화 발생했다는 것
-> 이 기능이 별도로 분리되어야 할 책임

> 또다 른 예

SRP O : 데이터를 읽어 오는 기능이 변경될 때 보여주는 기능은 변경되지 않는다.
SRP X : 데이터를 읽어오는 기능이 변경되었는데 보여주는 기능도 같이 변경 됨

#### SRP 원칙을 잘 지키는 요령

메서드를 실행하는 것이 누구인지(`Caller`) 확인해보는 것

그리고 Caller의 요구사항이 바뀔 떄 메서드 시그니쳐가 어떻게 바뀔 지 생각을 해보자

### 개방 폐쇠 원칙 (Open-Closed Principle)

확장에는 열려 있고 변경에는 닫혀 있어야 한다

- 기능을 변경하거나 확장할 수 있으면서
- 그 기능을 사용하는 (클라이언트의) 코드는 수정하지 않는다

![image](https://user-images.githubusercontent.com/66164361/159164534-159a15e1-e169-40fc-9943-d36bc88c1dbf.png)

#### OCP 가 깨질 때 주요 증상

![image](https://user-images.githubusercontent.com/66164361/159164821-a64c2a0e-9df1-420e-ba5d-5249a6a57686.png)

- 다운 캐스팅을 한다

```java
public void drawCharacter(Character character) {
  if(character instanceof Missile) {
    Missile missile = (Missile) character;
    missile.drawSpecific();
  } else {
    character.draw();
  }
}
```

`drawCharacter` 메서드는 `Character` 가 확장될 때 함께 수정되어 있다,

다시 말해 기능을 확장할 때 클라이언트 코드인 `drawCharacter` 메서드는 함꼐 수정된다

- 비슷한 `if-else` 블록이 존재한다

```java
public class Enemy extends Character {

  private int pathPattern;

  public Enemy(int pathPattern) {
    this.pathPattern = pathPattern;
  }

  public void draw() {
    if(pathPattern == 1) {
      x += 4;
    } else if(pathPattern == 2) {
      y += 10;
    } else if ...
  }
}
```

```java
public class Enemy extends Character {

  private PathPattern pathPattern;

  public Enemy(PathPattern pathPattern) { // 인터페이스 등의 추상화
    this.pathPattern = pathPattern;
  }

  public void draw() {
    int x = pathPattern.nextX();
    int y = pathPattern.nextY();
  }
}
```

#### 개방 폐쇄 원칙은 유연함에 대한 것

...

## 리스코프 치환 원칙 (Liskov Substitution Printiple)

상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.

리스코프 치환 원칙을 지키지 않을 때의 문제

### 리스코프 치환 원칙은 계약과 확장에 대한 것

## 인터페이스 분리 원칙

### 인터페이스 변경과 그 영향

### 인터페이스 분리 원칙

### 인터페이스 분리 원칙은 클라이언트에 대한 것

## 의존 역전 원칙 (Dependency Inversion Principle)

- `SRP`

### 고수준 모듈이 저수준 모듈에 의존할 때의 문제

### 의존 역전 원칙을 통한 변경의 유연함 확보
