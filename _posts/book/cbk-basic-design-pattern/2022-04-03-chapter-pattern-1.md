---
layout: post
title: "[최범균님] 객체지향과 디자인 패턴 : 전략, 상태, 템플릿, 널"
subtitle: "..."
date: 2022-04-03 23:00 +0900
categories: book
tags: cbk-basic-design-pattern
comments: true
published: true
---

## 전략 패턴

---

> 행동을 추상화했다 : 구현체마다 각자 다른 행동을 해야할 때

지금까지.. 너무나도 많이 다루어 왔기 때문에 생략합니다.. ㅠㅠ

## 상태 패턴

---

> 상태를 추상화 했다 : 컨텍스트나 상태에서 상태를 바꿀 때

컨텍스트에서 상태를 내부적으로 갖고 있다

> 상태 패턴을 사용하지 않은 경우

```java
class VendingMachine {

    private static enum State {NO_COIN, SELECTABLE}

    private State state;

    public void insertCoin(int coin) {
        switch (state) {
            case NO_COIN:
                insertCoin(coin);
                state = State.SELECTABLE;
                break;

            case SELECTABLE:
                insertCoin(coin);
                break;
        }
    }

    public void select(int productId) {
        switch (state) {
            case NO_COIN:
                break;

            case SELECTABLE:
                provideProduct(productId);
                decreaseCoin();
                break;
        }
    }

    private void provideProduct(int productId) {
      ...
    }

    private void decreaseCoin() {
      ...
    }
}
```

처음에 가지는 동작이나 상태들이 몇 가지 없을 때엔 큰 문제가 되지 않는다.. 그러나 요구사항이 추가되면 될 수록 상황을 달라진다

예를들어 코인을 반환할 수 있는 기능이 있어야 한다면 

`returnCoin`이라는 메서드를 추가한다

문제는 그뿐만 아니라 아래와 같은 코인들도 덩달아 추가된다

```java
public int returnCoin(int coin) {
    switch (state) {
        case NO_COIN:
        case SELECTABLE:
            return coin;
    }
    return coin;
}
```

(물론 위 코드는 `return coin` 코드 한줄이면 되지만.. 대게의 경우 여러 상태도 고려하게 된다)

자, 이게 끝이 아니라 `SOLD_OUT`이라는 상태도 추가된다고 한다면... 

각 메서드는 아래와 같은 양상을 보이게 된다

```java
public void insertCoin(int coin) {
    switch (state) {
        case NO_COIN:
            insertCoin(coin);
            state = State.SELECTABLE;
            break;

        case SELECTABLE:
            insertCoin(coin);
            break;

        case SOLD_OUT:
            returnCoin(coin);
            break;
    }
}
```

중요한 내용은 아니지만

위와 같이 `2`개의 `상태`, `2`개의 `주 기능`에서

1개의 기능이 추가될 때 `2`개의 `상태`를 고려해야 했고

1개의 상태가 추가될 때 `3`개의 기능에 `상태`의 경우를 추가해야 했다

만일 10개의 상태와 10개의 주 기능에서 

각각 1개씩 추가하게 될때 10+11 개의 고려사항이 생긴다

### 상태 변경을 사용한 코드

> 컨텍스트
> 이 경우 자판기가 컨텍스트가 된다 (상태를 갖는 객체)

```java
class VendingMachine {

    private State state;

    private int coin = 0;

    public void insertCoin(int coin) {

    }

    public void select(int productId) {

    }

    void changeState(State state) {
        this.state = state;
    }

    public void increaseCoin(int coin) {
        this.coin += coin;
    }

    public void decreaseCoin(int productId) {
        int productCoin = getCoinByProductId(productId);
        this.coin -= productCoin;
    }

    public boolean hasNoCoin() {
        return this.coin == 0;
    }

    public void provideProduct(int productId) {

    }

    private int getCoinByProductId(int productId) {
        return 0;
    }
}
```

> 상태

```java
public interface State {
    void increase(int coin, VendingMachine vendingMachine);
    void select(int productId, VendingMachine vendingMachine);
}
```

```java
public class NoCoinState implements State {

    private static final NoCoinState INSTANCE = new NoCoinState();

    private NoCoinState() {
    }

    public static State getInstance() {
        return INSTANCE;
    }

    @Override
    public void increase(int coin, VendingMachine vendingMachine) {
        vendingMachine.changeState(SelectableState.getInstance());
        vendingMachine.increaseCoin(coin);
    }

    @Override
    public void select(int productId, VendingMachine vendingMachine) {
        SoundUtil.beep();
    }
}
```

```java
public class SelectableState implements State {

    private static final State INSTANCE = new SelectableState();

    public static State getInstance() {
        return INSTANCE;
    }

    @Override
    public void increase(int coin, VendingMachine vendingMachine) {
        vendingMachine.insertCoin(coin);
    }

    @Override
    public void select(int productId, VendingMachine vendingMachine) {
        vendingMachine.provideProduct(productId);
        vendingMachine.decreaseCoin(productId);

        if (vendingMachine.hasNoCoin()) {
            vendingMachine.changeState(NoCoinState.getInstance());
        }
    }
}
```

### 상태변경을 컨텍스트에서 할 경우
비교적 상태 개수가 적고 상태 변경 규칙이 거의 바뀌지 않는 경우에 유리하다.  
왜냐면 상태 종류가 지속적으로 변경되거나 상태 규칙이 자주 바뀔 경우 콘텍스트의 상태 변경 코드가 복잡해질 가능성이 높기 때문이다.  
상태 변경 처리 코드가 복잡해질 수록 상태 변경의 유연함이 떨어지게 된다.

### 상태 변경을 각각 상태에서 할 경우

> 장점  
컨텍스트에 영향을 주지 않으면서 상태를 추가하거나 상태 변경 규칙을 바꿀 수 있게 된다

> 단점
1. 상태 변경 규칙이 여러 클래스에 분산되어 있기 때문에, 상태 구현 클래스가 많아질 수록 상태 변경 규칙을 파악하기가 어려워지는 단점이 있다
2. 한 상태 클래스에서 다른 상태 클래스에 대한 의존이 발생한다


## 템플릿 패턴

---

> 템플릿이라는 틀 위에서 구현체마다 다른 동작을 할 때

> _프로그램을 구현하다 보면 오나전히 동일한 절차를 가진 코드를 ㅈ가성하게 될 때가 있다._
> _심지어 이 코드들은 절차 중 일부 과정의 구현만 다를 뿐 나머지 구현은 똑같을 때도 있다._
> (책에서)

사실 이 책 내용 하나만으로 설명이 아주 잘 되기 때문에.. 자세한 코드는 생략합니다.. 

## 널 객체 패턴

---


> 비어있는 상태를 널 객체를 넣음으로써 `NPE` 로부터 안전하게끔 구조를 변경

아래와 같은 수많은 `null` 체크 로직을 피하기 위하여 만들어진 패턴이다

```java
if(어떤_객체 != null) {
  어떤_객체.행동한다();
}
```

구체적인 코드를 통해서 알아보자

```java
class SpecialDiscountFactory {

    public SpecialDiscount create(Customer customer) {
        if (checkNewCustomer(customer)) {
            return new NewCustomerDiscount();
        }

        // 특별 할인 혜택이 없을 때 null 대신할 객체를 리턴한다
        return new NullSpecialDiscount();
    }

    private boolean checkNewCustomer(Customer customer) { ... }
}
```

> 널 객체 패턴의 꽃인 널 객체 ~~(재귀인가..?)~~
```java
class NullSpecialDiscount extends SpecialDiscount {
    @Override
    public void addDetailTo(Bill bill) {
        // 그 어떤 것도 하지 않는다
    }
}
```

