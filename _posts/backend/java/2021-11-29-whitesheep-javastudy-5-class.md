---
layout: post
title: "[스터디할래] whitesheep #5 클래스"
subtitle: ""
date: 2021-11-29 00:10:00 +0900
categories: backend
tags: java
comments: true
---

## 이 스터디를 참여하는 이유

---

평상시 자바에 대한 기본기가 약하다고 느끼고 있던 참이었다..

그러나 이 스터디는 내년 정도에나 하려던 생각이었으며

올해는 간단한 코딩테스트 준비와 JPA 공부를 하려고 마음먹고 있었따

그러나 우테코 1차 코테에 붙으면서 플랜이 바뀌었다.

2차는 OOP & 클린코드 & 테스트 코드 라는 커다란 틀 안에서 코드를 작성해야 헀다.

이러기 위해서는 자연스레 자바에 대한 기초적인 객체지향 문법의 이해가 되어야만 했다.

오랜만에 OOP 코딩을 하는 만큼 이 부분이 약하다고 생각되었다.

회사가 ... 어쩌다 코로나로 쉬게 된 이상

백기선님의 스터디할래 수업 중에서 우테코에 준비하면서 필요한 것들을 위주로 공부하기로 하였다

내가 공부하기로 생각 한 것은 클래스, 상속, 인터페이스, Enum 이다

# 클래스

---

- 클래스 초기화 블럭, 인스턴스 초기화 블럭
  - 필드 변수를 초기화 할 때 if 문등의 문장을 가지는 조건문을 넣을 수 있다
- 접근 제어자
  - `final`이 클래스의 접근제어자로 올 수 있으며 이 경우 상속될 수 없다

```java
public class Init {
	private int number;

	static { // 스태틱 블럭
		System.out.println("static block");
	}

	{ // 인스턴스 블럭
		System.out.println("instance block");
		this.number = 20;
	}

	public Init() {
		System.out.println("init()");
		this.number = 100;
	}

	public Init(int number) {
		System.out.println("init(int)");
		this.number = number;
	}

	public static void main(String[] args) {
		Init init = new Init(50);
		System.out.println(init.number);
	}
}
```

실행결과는 아래와 같다

```
static block
instance block
init(int)
init.number = 50
```

본인은 저 `static block` 부분이 다소 의아했다..

저건 클래스 레벨의 실행이기 떄문에 단 한번만 되지 않을까..?

아닌가 다를까 예측이 맞았다

main함수를 아래처럼 수정했을 경우를 보도록 하자

```java
Init init = new Init(50);
System.out.println("init.number = " + init.number);

System.out.println();

Init init2 = new Init();
System.out.println("init2.number = " + init2.number);
```

이때 상속을 받을 경우 부모의 기본 생성자가 같이 실행된다

```java
public class ParentInit {
	public ParentInit() {
		System.out.println("ParentInit.ParentInit");
	}
}
```

```java
public class Init extends ParentInit {
    ...
}
```

```
static block
ParentInit.ParentInit
instance block
init(int)
init.number = 50
```

### 클래스 앞에..

- 인스턴스의 생성을 막을 때는 `abstract`를 쓰고
- 상속을 막을 때는 `final`을 쓴다
  - 언제 막는가? `String` 등이 하는 일로 상수풀에서 값을 가져오는 등의 일을 하는데
    Overloading을 해서 이 객체의 행위가 망가지는 것을 막기 위하여 `final`을 사용한다고 볼 수 있다.

### 퀴즈!

- 기선님 유튭에 나온 어느 분이 올린 퀴즈(?) 이다

```java
public class ClassQuiz {
	int x;
	int y;
	int z;

	{
		System.out.println("static block");
		if (x > 1) {
			y = 3;
		}
	}

	public ClassQuiz(int x) {
		System.out.println("constructor");
		this.x = x;
	}

	public static void main(String[] args) {
		ClassQuiz classQuiz = new ClassQuiz(3);
		System.out.println("classQuiz.y = " + classQuiz.y);
	}
}
```

참고로 실행 결과는

```
static block
constructor
classQuiz.y = 0
```

이다.

static이 실행되는 시점에 x는 기본값(0)으로 취급되어 if문을 통과하지 못한 것이다

### Local Class 란 것도 있다

한 메서드 안에 class를 정의하여 사용할 수 있다..

```java
public class LocalClassMain {
	public static void main(String[] args) {
		class LocalClass {
			private String name;

			public LocalClass(String name) {
				this.name = name;
			}

			public String getName() {
				return name;
			}
		}

		LocalClass localClass = new LocalClass("foo");
		System.out.println("localClass.getName() = " + localClass.getName());
	}
}
```

```java
public class InceptionClass {
	int a;
	class Inner {
		int b;
		class InnerInner {
			int c;
		}
	}
}
```

위와 같은 클래스를 compile해보았다

그러면 아래와 같이 클래스들이 생긴다

```
'InceptionClass$Inner$InnerInner.class'  'InceptionClass$Inner.class'   InceptionClass.class
```

위와 같다

### 저장되는 영역

|     종류      |          생성시기           |       소멸시기       | 저장메모리  |
| :-----------: | :-------------------------: | :------------------: | :---------: |
|  클래스 변수  | 클래스가 메모리에 올라갈 때 | 프로그램이 종료될 때 | method 영역 |
| 인스턴스 변수 |    인스턴스가 생성될 때     | 인스턴스가 소멸할 떄 |  heap 영역  |
|   지역 변수   |  블록 내 변수를 선언할 때   |   블록을 벗어날 때   | stack 영역  |

### 과제가 이진탐색..?

---

보니까 매주마다 과제가 있는 것 같다

클래스 개념을 배워서 그런데 이번회차의 과제는 `BinarySearch`의 자료구조를 구현해보란 것 같다...

Insert하는 로직은 오래걸릴 것 같고,

정렬이 되어 있다는 전제하에서 로직을 설계해 보자

### 참고한 곳

> 메모리 구조에 대해서도 설명: https://ahnyezi.github.io/java/javastudy-5/
>
> - 블로그 내용이 강의 내용과 다소 다르다
> - https://www.youtube.com/watch?v=5u_YnIeyYAU&list=PLfI752FpVCS96fSsQe2E3HzYTgdmbz6LU&index=14
> - 위 주소의 1:20:00 부터 확인해 보자
>   (Lob님) 이진탐색 참고: https://lob-dev.tistory.com/entry/Live-StudyWeek-05-%ED%81%B4%EB%9E%98%EC%8A%A4
