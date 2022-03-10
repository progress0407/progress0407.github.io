---
layout: post
title: "[자바의 정석 위주] 긴급하게 정리해보는 Stream API"
subtitle: ""
date: 2022-03-10 02:00 +0900
categories: backend
tags: java
comments: true
---

> 갑자기 Stream API ??

최근에 Stream API를 사용하면서 해당 API의 활용 능력이 다소 약하다고 느껴져서 자바의 정석 책을 보면서 긴급하게 정리해 보았다

**약점 위주**로 정리하려고 한다 !

## 스트림의 특징

---

- 스트림은
  - 데이터 소스를 변경하지 않는다
    - 원본 데이터를 변경하지 않음
  - 일회용이다
  - 작업을 내부 반복으로 처리한다
  - 중간 연산과 최종 연산이 있다
  - 지연된 연산을 한다
    - 최종 연산이 수행되어야 실제로 연산을 한다.
  - IntStream 등 기본 primitive 타입 스트릠 제공 (Stream\<Integer\> 을 대신함)
    - `오토박싱&언박싱` 비용을 줄이기 위함
  - 병렬 스트림
    - 병렬 처리가 쉽다
    - 내부적으로 fork&join 프레임웍 이용
    -

### 지연된 연산의 예시

```java
@Test
void test() {
    List.of(1, 2, 3, 4).stream()
            .map(num -> num * 2)
            .peek(out::println) // 출력 된다
            .collect(Collectors.toList());
}

@Test
void test2() {
    List.of(1, 2, 3, 4).stream()
            .map(num -> num * 2)
            .peek(out::println); // 출력되지 않음
}
```

## 스트림 생성

```java
// #1
Arrays.asList(...);

// #2
List<T> list;
list.stream();

// #3
tream.of(T... ts)
Arrays.stream(T[] ts)

// #4
IntStream.rangeClosed(int begin, int end)

// #5
new Random().ints(); // 무한 스트림
new Random().ints().limit(5);

// #6  무한 스트림
Stream.iterate(0, n -> n + 2);
Stream.generate(() -> 1)

// #7
Files.line(Path path);

// #8
Stream.empty()

// #9
Stream.concat(Stream s1, Stream s2);
```

### 무한 스트림 확인법

```java
Stream<Integer> integerStream = Stream.iterate(0, n -> n + 2);
integerStream.peek(out::println).count();
```

## 스트림의 중간 연산

---

### 자르기

- skip()
- limit()

### 요소 걸러내기

- fileter()
- distinct()

### 정렬

- sorted()

### 조회

- peek()

### 변환

- map()

- 기본형으로 변환
  - mapToInt(), mapToLong(), mapToDouble()

### 변환 : flatMap()

`Stream<T[]>` 를 `Stream<T>` 로 변환

- 사용하고자 할 때 : 박스 포장을 벗기고 싶을 때 !

## 스트림의 최종 연산

- forEach()
- allMatch()
  - 모두 맞아야 참
- anyMatch()
  - 하나라도 맞으면 참
- noneMatch()
  - 어느 하나도 맞지 않아야 참
- findFirst()
  - 첫번째 것을 찾는다 (정렬 개념이 포함되어 있다)
- findAny()
  - 정렬을 포함하지 않고 하나를 찾는다
- count(), sum(), average(), max(), min()
- reduce()
- collect()
