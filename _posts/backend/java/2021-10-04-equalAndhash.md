---
layout: post
title: "Java 지금이야 말로 equan and HashCode를 재정의 할 때"
subtitle: ""
date: 2021-10-04 11:30:00 +0900
categories: backend
tags: java
comments: true
---

## String 의 초기화

```java
String a = "aa";
String a = new String("aa");
```

위 두개는 비슷해 보이지만 서로 다른 초기화를 한다
첫번째는 문자열 리터럴로 할당한다

```java
String a = "aa";
String a2 = "aa";
```

위와 같은 경우 이미 존재하는 리터럴을 재사용한다

그러나 new 연산은 새로이 만들어낸다

따라서 `System.identityHashCode(Object o)` 를 통해서 확인해보면 서로 다름을 알 수 있다

참고로 String의 경우 `hashCode()`를 오버라이딩했기 때문에 `identityhahCode` 가 아닌 `HashCode`는 같다

오버라이딩 하지 않는 한 ``hashCode()`는 서로 다른 값을 가진다

> 참고: 자바의 정석
> https://johngrib.github.io/wiki/Object-hashCode/

## 원시타입과 레퍼런스 타입의 Equals

레퍼 클래스의 경우 값을 비교할때 주의를 해야 한다 !

```java
private static void PrimitiveReferenceTypeTest() {
    Integer a = 123;
    Integer b = 123;

    out.println("a==b = " + (a == b));
    out.println("a.equals(b) = " + a.equals(b));
    out.println("Objects.equals(a, b) = " + Objects.equals(a, b));
}
```

예를 들어 이경우 모두 참이다

```
a==b = true
a.equals(b) = true
Objects.equals(a, b) = true
```

그러나 아래와 같은 경우 참이 아니다

```java
    Integer a = 123;
    Integer b = 123;
```

```
a==b = false
a.equals(b) = true
Objects.equals(a, b) = true
```

이유는 내부적으로 `Integer.valueOf()` (int형) 메서드가 실행되는데.. 이곳에서 128 범위 이내의 값만
원시타입의 비교가 가능한 연산을 수행하게끔 하는 것으로 보인다... 구현내용은 아래에 적어 놓았다

```java
    @HotSpotIntrinsicCandidate
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

> 참고 : https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=thdansgur&logNo=220910298503

## 키가 같은 객체는 저장하고 싶지 않다 !

![image](https://user-images.githubusercontent.com/66164361/135786426-38f11f80-3e5a-4bc2-a8e6-654c41dbe10c.png)

Map 인터페이스 put 설명

```
Associates the specified value with the specified key in this map (optional operation). If the map previously contained a mapping for the key, the old value is replaced by the specified value. (A map m is said to contain a mapping for a key k if and only if m.containsKey(k) would return true.)
```

중요부분을 해석하면

```
맵에 이전에 키에 대한 매핑이 포함된 경우 이전 값이 지정된 값으로 대체됩니다.
```

예시 코드는 아래와 같다

```java
// 동기화된 HashMap
Map<String, Integer> map = Collections.synchronizedMap(new HashMap<>());
map.put("a", 1);
map.put("b", 2);
map.put("a", 10); // 키가 동일할 경우 치환된다

out.println("map.get(\"a\") = " + map.get("a"));

Set<String> keySet = map.keySet();
out.println("keySet = " + keySet);

out.println("===== all entries print");
Set<Map.Entry<String, Integer>> entries = map.entrySet();
entries.forEach(out::println);

out.println("map.size() = " + map.size());
```

실행결과

```
map.get("a") = 10
keySet = [a, b]
===== all entries print
a=10
b=2
map.size() = 2
```

당연한 얘기지만, 구현체인 HashMap이 Map의 put의 약속된 설명에 따라 잘 움직이고 있다 ! (LSP 원칙)

문제는 기존의 자바에서 지원하는 String 타입의 키뿐만 아니라 우리가 정의한 VO 형식의 객체에서도 Key임을 인지시켜주어야 한다.

즉... 아래와 같은 코드에서도 수행되어야 한다

```java
private static void test3() {
    Map<InnerNested, Integer> itemStore = new HashMap<>();

    itemStore.put(new InnerNested(101L, "바나나"), 12_000);
    itemStore.put(new InnerNested(102L, "사과"), 15_000);
    itemStore.put(new InnerNested(103L, "파인애플"), 9_000);

    itemStore.put(new InnerNested(101L, "바나나"), 6_000); // 요게 저장되면 안된다 !
    itemStore.put(new InnerNested(101L, "바나 나"), 8_000); // 이것도 저장되지 않기를 원한다

    for (Map.Entry<InnerNested, Integer> entry : itemStore.entrySet()) {
        InnerNested nested = entry.getKey();
        out.printf("%s : %s : %s \n", nested.getCode(), nested.getName(), entry.getValue());
    }
}

static class InnerNested { // Nested Static 클래스

    private Long code;
    private String name;

    public InnerNested(Long code, String name) {
        this.code = code;
        this.name = name;
    }

    public Long getCode() {
        return code;
    }

    public String getName() {
        return name;
    }
}
```

위 문제를 조~금 더 단순화 해보자

```java
 private static void CustomObjectTest() {
    Item apple = new Item(1_100_101L, "사과");
    Item apple2 = new Item(1_100_101L, "사 과");

    out.println("Objects.equals(apple, apple2) = " + Objects.equals(apple, apple2));
}

private static class Item {
    private final Long code;
    private final String name;

    public Item(final Long code, final String name) {
        this.code = code;
        this.name = name;
    }
}
```

위와 같은 상황에서 객체의 동등성을 만들어주면 되는 것이다

이때다 ! `equals` 메서드를 오버라이딩 하자

```java
@Override
public boolean equals(Object obj) {
    if (!(obj instanceof Item)) return false;
    Item item = (Item) obj;
    return Objects.equals(this.code, item.code);
}
```

그러면 이제 같다고 결과가 나온다

```
Objects.equals(apple, apple2) = true
```

그러나 아직 해시코드가 같지 않다

```
apple = test.grammer.clazz.EqualAndHashMain$Item@21bcffb5
apple2 = test.grammer.clazz.EqualAndHashMain$Item@380fb434

apple.hashCode() = 566034357
apple2.hashCode() = 940553268
```

자 이제 해시코드도 오버라이딩 해보자

```java
@Override
public int hashCode() {
    return Objects.hash(code);
}
```

결과는 아래와 같다

```
apple = test.grammer.clazz.EqualAndHashMain$Item@10c964
apple2 = test.grammer.clazz.EqualAndHashMain$Item@10c964

apple.hashCode() = 1100132
apple2.hashCode() = 1100132
```

하지만 주의 해야한다 ! 해시코드가 같다고 해서 `==` 연산이 유효하지 않다

```
out.println("(apple == apple2) = " + (apple == apple2));
```

실행 결과

```
(apple == apple2) = false
```

왜냐하면 `System.identityHashCode(Object x)` 가 같지 않기 때문이다

```
System.identityHashCode(apple) = 940060004
System.identityHashCode(apple2) = 234698513
```

> 참고
> 정말 많은 도움되었다 ㄷㄷ...
> https://velog.io/@sonypark/Java-equals-hascode-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%96%B8%EC%A0%9C-%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C  
> https://codechacha.com/ko/java-objects-equals/
