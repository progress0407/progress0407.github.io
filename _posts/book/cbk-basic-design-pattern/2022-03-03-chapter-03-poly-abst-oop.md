---
layout: post
title: "[최범균님 디자인 패턴] 객체지향과 디자인 패턴 : 다형성과 추상타입"
subtitle: "..."
date: 2022-03-03 23:00 +0900
categories: book
tags: cbk-basic-design-pattern
comments: true
published: true
---

유연함을 얻는 방법

- 캡슐화 - 다른 코드에 영향을 최소화하면서 객체의 내부 구현을 변경
- 상속 - 이 장에서 배운다 !

> 책에서 소개한 예시

인터페이스 `ByteSource`의 구현체는 `FileDataReader`, `SocketDateReader` 들이 있다

인터페이스 `LogController`의 구현체는 `FtpLogFileDownloader`, `SocketLogReader`, `DbTableLogGateway` 들이 있다

## 상속 개요 (`Inheritance`)

---

- 한 타입을 그대로 사용하면서 구현을 추가할 수 있도록 해주는 방법

- 언어마다 다르지만 대게 `private`이 아닌 메서드나 필드를 물려받을 수 있다

- 하위 클래스에서 상위 클래스의 메서드를 재정의(`OverRiding`)할 수 있다
  - `OverLoading`은 다중 정의이다! (메서드 시그니처 중 메서드명이 다른 것들)

## 다형성(`Polymorphism`)과 상속(`Inheritance`)

---

한 객체(`poly`)가 여러가지 모습(`morph`)을 갖는다

- 여러 타입을 갖는다

```java
class SomeClass implements TypeA, TypeB { .. }
```

SomeClass는 TypeA, TypeB 라는 여러 타입을 갖는다

TypeA 로도 사용될 수 있고 TypeB 로도 사용할 수 있다

---

### 인터페이스 상속과 구현 상속

- 인터페이스 상속: 타입 정의만 상속 받음, 다형을 갖는다

- 구현 상속: 상위 클래스의 정의된 기능을 재상속

## 추상 타입과 유연함

---

- `추상화(abstraction)` : 데이터나 프로세스 등을 의미가 비슷한 개념이나 표현으로 정의하는 과정 (`...`)

1. 예) 여러 클래스에서 로그 수집 기능을 뽑아냄

- 구체적인 (국소적인) 정의: 공통의 기능을 뽑아내는 것
  - 이 정의는 추상화의 일부분이다 !!

2. 예) sum += mark;

- 메모리에서 값 읽고 변경하고 다시 할당  
  그러나 상세 구현에 대한 내용은 없다... 이런 과정을 개념적으로 추상화하고 있을 뿐!!

3. 예) 급여를 제공하는 것을 Employee.play() 로 `모델링`

- 이것도 하나의 추상화이다 !

### 추상 타입과 실제 구현의 연결

`Concrete Class` (구현 클래스)는 추상 타입(`Interface`)를 구현한다

```java
// LogController는 인터페이스
LogController collector = new SocketLogReader();
```

### 추상 타입을 이용한 구현 교체의 유연함

인터페이스를 뽑아냄으로써 `API`를 사용하는 클라이언트의 코드를 교체하지 않게끔할 수 있다

방법은 아래와 같이 2가지를 소개하였다

- ByteSource 타입의 객체를 생성하는 기능을 별도로 분리하여 위임

  - `ByteSourceFactory`

- 생성자 혹은 Setter를 이용해서 사용할 `ByteSource` 주입받기

> 재사용
> 상대적으로 저수준 로직보다는 고수준이 재사용될 수 있도록 설계하는 것이 더 중요하다
> 특정한 구현 로직은 변경되더라도 큰 흐름을 제어하는 코드의 전체적인 흐름은 바뀌지 않고 재사용될 수 있어야 한다

### 변화되는 부분을 추상화하기

```java
FileDataReader reader = new FileDataReader(); // SocketDateReader 로 변경 가능
```

위 처럼 사용하는 것이 아닌 아래처럼 사용하자

```java
ByteSource reader = ByteSourceFactory.getInstance().craete();
```

### 인터페이스에 대고 프로그래밍하기

> 영한님이 얘기한 인터페이스로 발라낸다

- 추상화를 통한 유연함을 얻기 위한 규칙

- 최초 설계에서 바로 도출되기 보다는 요구 사항의 변화와 함께 점진적으로 도출된다

- 유연함을 얻는 과정에서 추상 타입이 증가하고 구도고 복잡해진다
  - 때문에 모든 곳에서 사용해서는 안 된다
  - 변화 가능성이 높은 경우에 한해서 사용해야 한다

### 인터페이스는 인터페이스 사용자 입장에서 만들기

`FileDataReader`와 `SocketFileDataReader`가 있을 떄

둘을 추상화한 인터페이스명을 `FileDataReaderIF`로 하면 파일에서만 데이터를 읽는다고 생각하기 쉽다

`ByteSource`라고 지으면 클라이언트를 사용하는 입장에서 "데이터를 읽어온다"는 의미가 더 포괄적으로 잘 전달될 것이다

### 인터페이스와 테스트

테스트를 하기 위한 `Mock` 객체를 만들기 쉽다

이 객체를 통해서 프로덕션 코드의 완성을 기다릴 필요 없이 내가 만든 코드를 빠르게 테스트 할 수 있도록 해준다 !

```java
class MockByteSource implements ByteSource {
  public byte[] reade() { ... }
}
```

```java
public voic testProcess() {
  ByteSource mockSource = new MockByteSource();
  FlowController fc = new FlowController(mockByteSource);

  ...
}
```
