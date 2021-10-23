---
layout: post
title: "Java8 백기선님 강의 메모"
subtitle: ""
date: 2021-09-29 13:30:00 +0900
categories: backend
tags: java
comments: true
---

## 인터페이스

-- 인터페이스내 추상메서드가 오로지 하나면 `@FunctionalInterface`를 정의 가능하다

- `static`, `default` 메서드 선언 가능
  - 구현해야 한다

```java
RunSomething runSomething = new RunSomething() {
    // 익명 내부 클래스 annoymous inner class
    @Override
    public void doIt() {
        System.out.println("Foo.doIt");
    }
};
```

```java
RunSomething runSomething = () -> System.out.println("Foo.doIt");
```

```java
    RunSomething runSomething = () -> {
        System.out.println("Hello");
        System.out.println("Lambda");
    };
```

## FunctionalInterface와 입출력

|      인터페이스      | 입력과 출력  |
| :------------------: | :----------: |
|    Function<T, R>    |    T -> R    |
|   UnaryOperator<T>   |    T -> T    |
| BiFunction<T, U, R > |  T, U -> R   |
|     Consumer<T>      |  T -> void   |
|     Supplier<T>      |  void -> T   |
|     Predicate<T>     | T -> boolean |

> 참고
> https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html

> \*\* 생성자의 반환값은 해당 객체의 타입이다 !

## Stream

|   중개형 Operator    |   종료 Operator    |
| :------------------: | :----------------: |
|     Stream 반환      | Collection 등 반환 |
| 연산 수행 X. Lazy 함 | 비로소 연산을 수행 |

## 기타 팁

### args 넣기

```
java 클래스명 a b c d
```

와 결과가 같다

![image](https://user-images.githubusercontent.com/66164361/135746868-b7239387-bba5-462a-b06c-e3cfed76c3d5.png)
