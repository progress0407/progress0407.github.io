---
layout: post
title: "자바 EnumMap vs HashMap 성능 비교"
subtitle: ""
date: 2022-02-26 14:30 +0900
categories: backend
tags: java
comments: true
published: true
---

## 자바 EnumMap vs HashMap 성능 비교

이번 우테코 로또 미션 진행중에 EnumMap을 사용하게 되어서 성능을 테스트 해보았다

`EnumMap`이 `HashMap`에 비해서 굉장히 빨랐다

또 혼자서 한번 벤치마크 테스터기를 만들어서 간단히 실행을 진행해보았다

> 대략적인 설계도

![image](https://user-images.githubusercontent.com/66164361/155831688-9bfb4ef0-baf0-4cb5-b15b-60d508f1e70d.png)

> 정확한 `UML` 이 절대로 아니다 !! 특히 화살표!

아이디어는 `프록시 패턴`에서 얻었다 다만 내부적인 `캐싱`은 하고 있지 않다 (그저 생성자로 `DI`만 함)

```java
public class Main {
    public static void main(final String... args) {
        MyBenchMark benchMark = new EnumMapClass();
//        MyBenchMark benchMark = new HashMapClass();

        MyBenchMarkRunner benchMarkRunner = new MyBenchMarkRunner(benchMark);
        benchMarkRunner.run();
    }
}
```

```java
public class MyBenchMarkRunner {

    private final MyBenchMark benchMark;

    private static final int NUMBER_OF_REPETITIONS = 10;

    public MyBenchMarkRunner(MyBenchMark benchMark) {
        this.benchMark = benchMark;
    }

    public void run() {
        long resultSum = 0L;
        try {
            for (int i = 0; i < NUMBER_OF_REPETITIONS; i++) {
                long start = System.currentTimeMillis();
                benchMark.test();
                long elapsedTime = System.currentTimeMillis() - start;
                out.printf("elapsedTime :  %d ms \n", elapsedTime);
                resultSum += elapsedTime;
            }
        } catch (RuntimeException e) {
            out.println(e.getMessage());
        }
        double avgTime = (double) resultSum / NUMBER_OF_REPETITIONS;
        out.printf("\n %d회 반복 평균 시간 :  %.2f ms \n", NUMBER_OF_REPETITIONS, avgTime);
    }

}
```

```java
public interface MyBenchMark {
    int NUMBER_OF_TRIALS = 10_000_000;
    void test() throws RuntimeException;
}
```

```java
public class EnumMapClass implements MyBenchMark {

    private EnumMap<MyEnum, Integer> map;

    public EnumMapClass() {
        map = new EnumMap<>(MyEnum.class);
        for (MyEnum value : MyEnum.values()) {
            map.put(value, value.num);
        }
    }

    @Override
    public void test() throws RuntimeException {
        final int myEnumSize = MyEnum.values().length;
        for (int i = 0; i < NUMBER_OF_TRIALS / myEnumSize; i++) {
            for (MyEnum value : MyEnum.values()) {
                map.get(value);
            }
        }
    }

    @RequiredArgsConstructor
    private enum MyEnum {
        N1(1), N2(2), N3(3), N4(4), N5(5), N6(6), N7(7), N8(8), N9(9), N10(10);
        private final int num;
    }
}
```

```java
public class HashMapClass implements MyBenchMark {

    private Map<Integer, Integer> map;

    public HashMapClass() {
        map = new HashMap<>();
        IntStream.rangeClosed(1, NUMBER_OF_TRIALS).forEach(value -> map.put(value, value));
    }

    @Override
    public void test() {
        map();
    }

    private void map() {
        IntStream.rangeClosed(1, NUMBER_OF_TRIALS).forEach(value -> map.get(value));
    }
}
```

> ## 실험 개요

`HashMap` 에는 천만 개의 숫자를 넣고 1부터 천만까지 순회하면서 찾았다

`EnumMap`은 우선 열 개의 인스턴스를 갖는 `Enum`을 내부 클래스에 정의하여 백만회 순회하면서 검색을 하였다

그러니까 각각 모두 천만번 검색을 한 것!

> ## 실험결과

> 1. HashMap

![image](https://user-images.githubusercontent.com/66164361/155831338-4715ff5f-fa8b-4b72-b8b2-7b954a844f22.png)

> 2. EnumMap

![image](https://user-images.githubusercontent.com/66164361/155831353-99fd78c7-43b2-4afc-b8d2-08c1cf07e376.png)

---

### 결론

_`EnumMap`이 10.9 배 빨랐다_

내부적으로 해싱을 하지 않고 ordinal을 이용 + 배열로 구현되어 있어서 그런지 굉장히 빨랐다
