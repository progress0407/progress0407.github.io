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

- 다른 말 : 하나의 타입에 여러 기능이 섞여 있을 경우 한 기능의 변화로 인해 다른 기능이 영향을 받을 가능성이 높아진다.

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

변화되는 부분을 추상화하지 못하면 개방 폐쇠 원칙을 지킬 수 없게 되어 시간이 흐를 수록 기능 변경이나 확장을 어렵게 만든다.

- 변화되는 부분을 추상화해야 한다 !!

## 리스코프 치환 원칙 (Liskov Substitution Printiple)

상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.

### 리스코프 치환 원칙을 지키지 않을 때의 문제

- **정사각형-직사각형 문제**

개념적으로 상속 관계에 있는 것처럼 보일지라도 실제 구현에서는 상속 관계가 아닐 수도 있다는 것을 보여주고 있다.
개념상 상속받아 구현하는 것이 합리적으로 보일 수 있으나, 실제 프로그램 상에서는 상속 관계로 묶을 수 없다.

### 리스코프 치환 원칙은 계약과 확장에 대한 것

하위 타입은 상위 타입에서 정의한 명세를 벗어나지 않는 범위에서 구현해야 한다.

- 명세에서 벗어난 동작을 하게 되면 이 명세에 기반해서 구현한 코드는 비정상적으로 동작할 수 있다.

```java
class Coupon {
  public int calculateDiscountAmount(Item item) {
    if (item instanceof SpecialItem) {
      return 0;
    }
    return item.getPrice() * discountRate;
  }
}
```

- `instanceof` 연산자를 사용한다는 것은 **전형적인** **리스코프 치환 원칙**을 **위반**할 때 발생하는 증상이다 !

하위 타입이 상위 타입을 완전히 대체할 수 없다는 뜻

클라이언트가 특정 하위 타입을 신경 쓰면서 동작이 이루어진다..

이것은 완전히 상위 타입 안에 하위 타입이 캡슐화되지 못한 것 (`내 해석`)

> 해결

```java
class Item {
  public boolean isDiscountAvailable() {
    return true;
  }
}

class SpecialItem {
  @Override
  public boolean isDiscountAvailable() {
    return false;
  }
}
```

```java
class Coupon {
  public int calculateDiscountAmount(Item item) {
    if(!item.isDiscountAvailable()) {
      return 0;
    }
    return item.getPrice() * discountRate;
  }
}
```

## 인터페이스 분리 원칙

### 인터페이스 변경과 그 영향

~~cpp 예제가 이해가 안된다..~~

### 인터페이스 분리 원칙

클라이언트 입장에서 사용하는 기능만 제공하도록 인터페이스를 분리함으로써 한 기능에 대한 변경의 여파를 최소화할 수 있게 한다

### 인터페이스 분리 원칙은 클라이언트에 대한 것

각 클라이언트가 사용하는 기능을 중심으로 인터페이스를 분리함으로써,
클라이언트로부터 발생하는 인터페이스 변경의 여파가 다른 클라이언트에 미치는 영향을 최소화할 수 있게 된다.

## 의존 역전 원칙 (Dependency Inversion Principle)

> 책에 나온 정의를 나누어서 생각해보자 !

1. 고수준 모듈은 저수준 모듈의 **구현**에 의존해서는 안 된다.

2. 저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 한다.

> 1번

![image](https://user-images.githubusercontent.com/66164361/159169480-8fdd4873-f572-4a19-a617-ac256dbd171c.png)

> 2번

![image](https://user-images.githubusercontent.com/66164361/159169511-7abb3723-c35d-462a-9940-307b3d99f34c.png)

### 고수준 모듈이 저수준 모듈에 의존할 때의 문제

```java
class 가격계산_모듈 {
  public int calculate() {
    if(condition1) {
      쿠폰타입1 쿠폰1 = ...
    } else if(condition2) {
      쿠폰타입2 쿠폰2 = ...
    }
  }
}
```

기존 쿠폰이 변경되거나 새로운 쿠폰이 추가 될 때 마다,

이를 사용하는 가격계산 모듈이 수정된다

### 의존 역전 원칙을 통한 변경의 유연함 확보

고수준 모듈과 저수준 모듈이 추상화 타입을 의존하게 만든다면

고수준 모듈의 변경 없이 저수준 모듈을 변경할 수 있는 유연함을 얻게 된다.

> DIP 는 LSP 는 OCP 를 만들어주는 기반이 된다.

### 소스 코드 의존과 런타임 의존

추상화에 의존한다는 것은 소스코드 상에서 의존한다는 것이다! (컴파일 타임)

런타임때는 추상화가 아닌 구체 클래스에 의존하게 된다 (생각해보면 당연한 얘기다 !)

### 의존 역전 원칙과 패키지

클래스 수준 뿐만 아니라 패키지 수준까지 확장시켜 주는 디딤돌이 된다.

### SOLID 정리

- SRP, ISP: 객체가 커지지 않도록 막아 준다

  - 객체가 단일 책임을 갖게 하고 클라이언트마다 다른 인터페이스를 사용하게 함으로써 한 기능의 변경이 다른 곳에까지 미치는 영향을 최소화할 수 있다

- LSP, DIP는 OCP를 지원한다

  - `OCP`는 변화되는 부분을 추상화하고 다형성을 이용함으로써 기능확장을 하면서도 기존 코드를 수정하지 않는 것이 가능하다
  - `DIP`는 변화되는 부분을 추상화할 수 있도록 도와준다
  - `LSP`는 다형성을 도와준다

- `SOLID`는 사용자 입장에서의 `기능 사용을 중시`한다 / `설계를 지향`한다
  - `ISP`는 클라이언트 입장에서 인터페이스 분리
  - `DIP`는 저수준 모듈을 사용하는 고수준 모듈 입장에서 추상화 타입을 유도
  - `LSP`는 사용자에게 기능 명세를 제공하고 그에 따라 기능을 구현할 것을 약속
