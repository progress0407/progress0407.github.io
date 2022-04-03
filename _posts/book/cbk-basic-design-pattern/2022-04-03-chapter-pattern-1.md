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

    private static enum State {NO_COIN, SELECTABLE, SOLD_OUT}

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

            case SOLD_OUT:
                returnCoin(coin);
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

            case SOLD_OUT:
                break;
        }
    }

    public int returnCoin(int coin) {
        switch (state) {
            case NO_COIN:
            case SELECTABLE:
            case SOLD_OUT:
                return coin;
        }
        return coin;
    }

    private void provideProduct(int productId) {

    }

    private void decreaseCoin() {

    }
}

```



## 템플릿 패턴
> 템플릿이라는 틀 위에서 구현체마다 다른 동작을 할 때

## 널 객체 패턴
> 비어있는 상태를 널 객체를 넣음으로써 NPE 로부터 안전하게끔 구조를 변경