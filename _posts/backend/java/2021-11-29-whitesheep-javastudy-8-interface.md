---
layout: post
title: "[스터디할래] whitesheep #8 인터페이스"
subtitle: ""
date: 2021-11-29 00:10:00 +0900
categories: backend
tags: java
comments: true
---

# 인터페이스.. 아는 내용은 생략 내지 짧게 !

인터페이스는 OOP를 공부하면서 알게 된 내용들이 많다 !

java8 이상의 내용에 대해 보자

사실.. 보기만 해도 이해가 되기 때문에 굳이 설명을 첨언하지는 않았다 !!

## JAVA8 default와 static의 등장

java8부터 default 메서드와 static 메서드를 등장시켰다

따라서 interface 자체에서도 구현부를 동시에 지닐 수 있게된 것이다

```java
public interface Java8Interface {
	void method();

	default void defaultMethod() {
		System.out.println("this is default method");
	}

	static void staticMethod() {
		System.out.println("this is static method");
	}
}
```

```java
public class Java8Impl implements Java8Interface {
	@Override
	public void method() {
		System.out.println("this is method");
	}
}
```

```java
public class InterfaceMain {
	public static void main(String[] args) {
		new Java8Impl().method();
		new Java8Impl().defaultMethod();
		Java8Interface.staticMethod();
	}
}
```

다만 static 메서드는 구현부에서 호출할 수 없다

즉

```java
Java8Interface.staticMethod();
```

와 같은 호출이 안 되고

인터페이스에서 바로 호출해야 한다

```java
Java8Impl.staticMethod();
```

## 왜 저런 메서드가 생겼는가?

예를들어 `List`나 `Map`같은 자료형에 추가적인 기능을 필요로 한다고 하였을 때

어떤 한 메서드를 추가하게 되면 필수적으로 구현해야하기 때문에 모든 자손 구현체들이 컴파일 에러가 떨어진다..

인터페이스를 구현하고 있는 자손 클래스들에게 하위 호환성을 지키기 위하여 만들었다고 볼 수있다

![image](https://user-images.githubusercontent.com/66164361/144354480-401532ac-0906-49e6-a378-7454013cdbc3.png)

`List`의 `forEach` 메서드도 8버전 이후 `default`가 등장하면서 생겼다

## java9 private과 private static의 등장

```java
public interface Java9Interface {
	default void defaultMethod() {
		privateMethod();
		privateStaticMethod();
	}

	private void privateMethod() {
		System.out.println("this is private method");
	}

	private static void privateStaticMethod() {
		System.out.println("this is private static method");
	}
}
```

```java
public class Java9Impl implements Java9Interface {
}
```

```java
public class InterfaceMain {
	public static void main(String[] args) {
		new Java9Impl().defaultMethod();
	}
}
```

## 두 인터페이스를 상속받는다면

두 인터페이스 중 구현할 내용을 취사선택할 수 있다

실행부

```java
JoinImpl join = new JoinImpl();
join.preJoin();
join.afterJoin();
```

```java
public interface JoinGroup {
	default void preJoin() {
		System.out.println("그룹에 가입 전");
	}

	default void afterJoin() {
		System.out.println("그룹에 가입 후");
	}
}
```

```java
public interface JoinMember {
	default void preJoin() {
		System.out.println("회원 가입 전");
	}

	default void afterJoin() {
		System.out.println("회원 가입 후");
	}
}
```

```java
public class JoinImpl implements JoinGroup, JoinMember {
	@Override
	public void preJoin() {
		JoinGroup.super.preJoin();
		JoinMember.super.preJoin();
	}

	@Override
	public void afterJoin() {
		JoinGroup.super.afterJoin();
		JoinMember.super.afterJoin();
	}
}
```

## 인터페이스들끼리도 상속이 된다

```java
public interface Join {
	default void preJoin() {
		System.out.println("Join 클래스의 preJoin");
	}
}
```

```java
public interface JoinMember extends Join {
	default void preJoin() {
		System.out.println("회원 가입 전");
	}
	...
}
```

### 추상 클래스와 인터페이스

- 추상 클래스는 `IS-A` 관계이고
  - TV는 가전제품이다

```java
Tv extends ElectronicProduct { }
```

- 인터페이스는 `HAS-A` 관계이다
  - TV는 영상을 보여줄 수 있다
  - TV는 소리를 재생시킬 수 있다

```java
Tv implements Video, Audio { }
```

### Lambda 와 인터페이스를 이용한 Custom 계산기

> 수업중에 나온 프로그램인데 한번 모방해보았다
> (1:44) https://www.youtube.com/watch?v=v2BXL6SYaBc&list=PLfI752FpVCS96fSsQe2E3HzYTgdmbz6LU&index=16

```java
public interface CustomCalculator {

	default int addEvenNumbers(int... nums) {
		return add(x -> x % 2 == 0, nums);
	}

	default int addOddNumbers(int... nums) {
		return add(x -> x % 2 == 1, nums);
	}

	private int add(IntPredicate predicate, int... nums) {
		return IntStream.of(nums)
			.filter(predicate)
			.reduce((a, b) -> a + b)
			.getAsInt();
	}
}
```

```java
public class CustomCalculatorImpl implements CustomCalculator {
}
```

```java
private static void test3() {
	CustomCalculatorImpl calculator = new CustomCalculatorImpl();
	int[] nums = {1, 2, 3, 4};
	// odd sum = 4
	// even sum = 6
	int oddSum = calculator.addOddNumbers(nums);
	System.out.println("oddSum = " + oddSum);
	int evenSum = calculator.addEvenNumbers(nums);
	System.out.println("evenSum = " + evenSum);
}
```
