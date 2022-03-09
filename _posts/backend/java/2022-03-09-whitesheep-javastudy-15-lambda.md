---
layout: post
title: "[스터디할래] whitesheep #15 Lambda Expression"
subtitle: ""
date: 2022-03-10 01:00 +0900
categories: backend
tags: java
comments: true
---

## java.util.function 패키지

| 함수형 인터페이스  | 시그니처          |
| ------------------ | ----------------- |
| java.lang.Runnable | void run()        |
| Supplier\<T\>      | T get()           |
| Consumer\<T\>      | void accept()     |
| Function\<T, R\>   | R apply(T t)      |
| Predicate\<T\>     | boolean test(T t) |

### UnaryOperator, BinartOperator

`Funciton`의 변형이다

| 항목                | 시그니처          |
| ------------------- | ----------------- |
| UnaryOperator\<T\>  | T apply(T t)      |
| BinartOperator\<T\> | T apply(T t, T t) |

### Function의 합성과

예시로 작성하는 것이 더 나을 것 같다 !

```java
Function<String, Integer> f = (String x) -> Integer.parseInt(x, 16); // f("FF") -> 255
Function<Integer, String> g = (Integer x) -> Integer.toBinaryString(x); // g(255) -> "11111111"

Function<String, String> h = f.andThen(g); // h("FF") -> 11111111

```

`compose(T t)`은 `andThen(T t)` 의 반대 방향 연산이다

즉 `g.compose(f)`와 `f.andThen(g)`는 같다

### Predicate의 결합

```java
Predicate<Integer> p = x -> x < 100;
Predicate<Integer> q = x -> x < 200;
Predicate<Integer> r = x -> x % 2 == 0;
Predicate<Integer> notP = p.negate();

Predicate<Integer> all = notP.and(q.or(r));
System.out.println(all.test(150));
```

### 메서드 참조

```java
BiPredicate<String, String> f = (a, b) -> a.equals(b);  // (1)
BiPredicate<String, String> f2 = String::equals;

BiPredicate<String, String> g = (a, b) -> b.equals(a);  // (2)
```

`(1)` 은 되지만 `(2)` 는 안 되는 것으로 보인다

| 종류                           | 람다식                     | 메서드 참조           |
| ------------------------------ | -------------------------- | --------------------- |
| static 메서드 참조             | (x) -> ClassName.method(x) | ClassName::methodName |
| 인스턴스 메서드 참조           | (obj, x) -> obj.method(x)  | ClassName::methodName |
| 특정 객체 인스턴스 메서드 참조 | (x) -> obj.method(x)       | obj::method           |

### 람다의 스코프(?)

람다식의 중괄호는 밖의 스코프를 가리지를 못한다

예를 들어 아래와 같은 경우는 컴파일 에러가 뜬다.

```java
int a = 100;
Runnable somethingDo = () -> {
    int a = 20; //  에러  Variable 'a' is already defined in the scope
};
```

## 람다 캡처링

`Variable Capturing` 이라고도 한다

여기서 `Capture`란 사진을 찰칵 찍듯이 가지고 와서 저장한다는 뜻으로 보인다

람다는 일종의 익명 객체로 구현되어 런타임시에 작동하는데

이때 익명 객체의 메서드가 실행되면서 자신만의 콜 스택이 생긴다

예를들어

```java
main() {
  // 람다 식
  Runnable somethingDo = () -> out.println("풉ㅋ");
  somethingDo.run();
}
```

`main` 의 콜스택이 있고 `somethingDo` 만의 스택이 또 생기는 것이다.

이때 `somethingDo`만의 독립적인 스택이 있기 때문에 원본 스택에 접근은 불가능하다

(로컬 변수는 해당 스택 내에서만 접근이 가능하다)

```java
int a = 100;
Runnable somethingDo = () -> {
    a = 20; // 에러  Variable used in lambda expression should be final or effectively final
};
```

예를 들어 이런 코드에서는 어짜피 람다 식은 실행되는 컨텍스트 (스택)이 다르기 때문에 외부 변수는 접근이 불가능 한 것이다

이때 람다식을 사용하는 사용자 입장에서는 "어? 분명 변경했는데 a가 왜 안바꼈지?" 라고 오인할 수 있으므로

위와 같은 컴파일 예외를 보여주는 것으로 보인다

> 하지만 반대로 !

람다식은 힙에는 접근이 가능하다

무슨 말이냐면,

```java
private int b = 100;

@Test
void test7() {
    Runnable somethingDo = () -> {
        b = 20;
    };
}
```

인스턴스 변수은 b에는 접근이 가능하다.

왜냐하면 스택이 바뀌더라도, 힙에는 접근이 가능하기 때문이다

(쓰레드가 힙에 접근이 가능하다는 것을 떠올리면 될 것 같다, 그렇기에 자원 동시 접근(`동시성 이슈`)을 조심하는 것이기도 하고)
