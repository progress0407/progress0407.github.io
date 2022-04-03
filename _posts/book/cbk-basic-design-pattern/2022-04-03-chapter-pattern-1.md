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

> 행동을 추상화했다 : 구현체마다 각자 다른 행동을 해야할 때


## 상태 패턴

> 상태를 추상화 했다 : 컨텍스트나 상태에서 상태를 바꿀 때

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


## 템플릿 패턴
> 템플릿이라는 틀 위에서 구현체마다 다른 동작을 할 때

## 널 객체 패턴
> 비어있는 상태를 널 객체를 넣음으로써 NPE 로부터 안전하게끔 구조를 변경