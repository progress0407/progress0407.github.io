---
layout: post
title: "Integer는 Immutable"
subtitle: ""
date: 2021-10-09 11:30:00 +0900
categories: backend
tags: java
comments: true
---

## 래퍼클래스는 불변타입이기에 Call By Reference가 불가능

코딩 테스트 문제를 풀고 있었는데..

코드를 리펙터링 하는 와중에 신기한 것을 발견헀다

아래는 코드의 핵심을 단순화해서 작성하였다

```java
public static void main(String[] args) {

        Integer a = 100;
        Integer b = 50;

        out.printf("a, b = %d, %d \n", a, b);
        adding(a, b);

        out.printf("a, b = %d, %d \n", a, b);
    }

    private static void adding(Integer a, Integer b) {
        a = a + 10;
        b = b + 5;
    }

}
```

`call by reference` 라면 아래의 결과가 출력되었을 것이다

```
a, b = 110, 55
```

그러나 `call by value`처럼 동작한 결과가 나왔다

```
a, b = 100, 50
```

> 위 내용에 대한 글
> https://stackoverflow.com/questions/3330864/how-can-i-pass-an-integer-class-correctly-by-reference

답변글을 살펴보니 래퍼클래스 Integer는 `Immutable` 불변이기에 `call by value` 로 작동한다는 것이다

꼭 `int`나 `Integer`를 `call by reference`처럼 동작시키고 싶다면

커스텀 클래스를 사용해서 구현하면 된다

```java
public static void main(String[] args) {

        WrapInt wrapInt = new WrapInt();

        wrapInt.a =  100;
        out.println("wrapInt = " + wrapInt);

        adding3(wrapInt);
        out.println("wrapInt = " + wrapInt);
    }

    public static void adding3(WrapInt wrapInt) {
        wrapInt.a = wrapInt.a + 100;
    }

    static class WrapInt {
        int a;

        @Override
        public String toString() {
            return "WrapInt{" +
                    "a=" + a +
                    '}';
        }
    }
```

결과

```
wrapInt = WrapInt{a=100}
wrapInt = WrapInt{a=200}
```
