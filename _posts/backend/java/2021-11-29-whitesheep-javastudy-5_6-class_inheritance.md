---
layout: post
title: "[스터디할래] whitesheep #5 클래스 ~ #6 상속"
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

## 클래스 앞에..

- 인스턴스의 생성을 막을 때는 `abstract`를 쓰고
- 상속을 막을 때는 `final`을 쓴다
  - 언제 막는가? `String` 등이 하는 일로 상수풀에서 값을 가져오는 등의 일을 하는데
    Overloading을 해서 이 객체의 행위가 망가지는 것을 막기 위하여 `final`을 사용한다고 볼 수 있다.

## 퀴즈!

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

## Local Class 란 것도 있다

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

## 각 변수의 종류에 따라 저장되는 영역

|     종류      |          생성시기           |       소멸시기       | 저장메모리  |
| :-----------: | :-------------------------: | :------------------: | :---------: |
|  클래스 변수  | 클래스가 메모리에 올라갈 때 | 프로그램이 종료될 때 | method 영역 |
| 인스턴스 변수 |    인스턴스가 생성될 때     | 인스턴스가 소멸할 떄 |  heap 영역  |
|   지역 변수   |  블록 내 변수를 선언할 때   |   블록을 벗어날 때   | stack 영역  |

## 리플렉션 소개

다른 클래스의 정보를 가져올 수 있다

(mysql 등 )드라이버 로딩할 때 사용하던 것도 리플렉션이다 ! ㅎ

```java
Field[] declaredFields = Class.forName("test.grammer.reflection.RefObject").getDeclaredFields();
for (Field declaredField : declaredFields) {
	out.println(declaredField);
}
```

메타 정보를 이용해서 정보를 가져오므로 우리가 자주사용하는 문장들보다는 느리다

그러나 성능에 크리티컬한 정도는 아니다 !

> Relfection을 이용한 엑셀 개발  
> https://techblog.woowahan.com/2698/

## 추상클래스와 인터페이스는 어떨 때 쓰면 좋을까?

8버전 이후 `default` 메서드 등의 구현이 가능하기 때문에 추상클래스의 입지는 약해졌다고 볼 수 있다

추상클래스가 필요 없는 경우가 많다..

스프링에서도 추상클래스에서 인터페이스로 옮긴 코드가 많다고 하셨다

## IntelliJ 상속(오버라이드), 구현 표시

- 상속

![image](https://user-images.githubusercontent.com/66164361/144050529-890268d1-93e7-49a8-b717-e35d95d774bc.png)

- 구현

![image](https://user-images.githubusercontent.com/66164361/144050780-6609d9f1-59b2-4337-8b73-b1d9bfa7fbb0.png)

## 더블 디스 패치 패턴을 이용한 것

- DomParser
- SaxParser (Simple)

## Clonable을 통해 살펴보는 얕은 복사, 깊은 복사

얕복과 깊복을 알아보자!!

객체의 기본적 개념을 안다면 알겠지만 !! 아래와 같은 경우는 얕은 복사이다

예제 구조

```java
public class CloneableMain {

	public static void main(String[] args) {...};

	static class Car {...}
}
```

```java
static class Car {
	private String name;

	public Car(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	@Override
	public String toString() {
		return "Car{" +
			"name='" + name + '\'' +
			'}';
	}
}
```

```java
public static void main(String[] args) {
	Car car1 = new Car("aa");
	System.out.println("car1 = " + car1);

	Car car2 = car1;
	car2.setName("aa_2");
	System.out.println("car1 = " + car1);
	System.out.println("car2 = " + car2);
	System.out.println("car1 = " + car1.hashCode());
	System.out.println("car2 = " + car2.hashCode());
}
```

위와 같이 실행하면 결과는 아래와 같다

```
car1 = Car{name='aa'}
car1 = Car{name='aa_2'}
car2 = Car{name='aa_2'}
car1 = 1121172875
car2 = 1121172875
```

그리고 깊은복사를 하고 싶으면 우선 **`Clonealbe`** 을 상속받아야 한다

안그러면 `CloneNotSupportedException` 이 터진다

그리고 `clone` 메서드를 인텔리제이의 도움을 받아 오버라이딩하자

```java
static class Car implements Cloneable{
	@Override
		protected Object clone() throws CloneNotSupportedException {
			return super.clone();
		}
}
```

실행부

```java
	public static void main(String[] args) {
		Car car1 = new Car("aa");
		System.out.println("car1 = " + car1);

		Car car3 = null;
		try {
			car3 = (Car)car1.clone();
		} catch (CloneNotSupportedException e) {
			e.printStackTrace();
		}
		car3.setName("aa3");
		System.out.println("car1 = " + car1);
		System.out.println("car3 = " + car3);
		System.out.println("car1.hashCode() = " + car1.hashCode());
		System.out.println("car3.hashCode() = " + car3.hashCode());
	}
```

실행결과

```
car1 = Car{name='aa'}
car1 = Car{name='aa'}
car3 = Car{name='aa3'}
car1.hashCode() = 649734728
car3.hashCode() = 610984013
```

## 다이나믹 메소드 디스패치

필자는 다형성의 핵심 원리라고 생각했다..

인터페이스의 메서드를 어떤 구현체의 메서드가 담당할 지를 결정한다고 볼 수 있다

런타임시에 결정된다고 볼 수 있다

## 과제가 이진탐색 ?!

### 그래서 "탐색"만 구현해 보았다

---

보니까 매주마다 과제가 있는 것 같다

클래스 개념을 배워서 그런데 이번회차의 과제는 `BinarySearch`의 자료구조를 구현해보란 것 같다...

Insert하는 로직은 오래걸릴 것 같고,

정렬이 되어 있다는 전제하에서 로직을 설계해 보자

```java
public class BinaryTree {

	static class Node {
		private int value;
		private Node leftNode;
		private Node rightNode;

		public Node(int value) {
			this.value = value;
		}

		public Node(int value, Node leftNode, Node rightNode) {
			this.value = value;
			this.leftNode = leftNode;
			this.rightNode = rightNode;
		}

		public int getValue() {

			return value;
		}

		public Node getLeftNode() {
			return leftNode;
		}

		public Node getRightNode() {
			return rightNode;
		}
	}

	public static void main(String[] args) {
		Node root = getNode();
		System.out.println("DFS: ");
		BinaryTree.searchByDfs(root);
		System.out.println("\n\nBFS: ");
		BinaryTree.searchByBfs(root);
		System.out.println();
	}

	private static void searchByBfs(Node node) {
		Queue<Node> queue = new LinkedList<>();
		queue.offer(node);
		do {
			Node poll = queue.poll();
			if (poll == null) {
				continue;
			}
			System.out.print(poll.getValue() + " ");
			Node leftNode = poll.getLeftNode();

			if (leftNode != null) {
				queue.offer(leftNode);
			}

			Node rightNode = poll.getRightNode();
			if (rightNode != null) {
				queue.offer(rightNode);
			}
		} while (!queue.isEmpty());
	}

	private static Node getNode() {
		Node leftOfLeftOfLeftNode = new Node(1);
		Node leftOffLeftNode = new Node(2, leftOfLeftOfLeftNode, null);
		Node rightOfLeftNode = new Node(4);
		Node leftNode = new Node(3, leftOffLeftNode, rightOfLeftNode);

		Node rightOfLeftOfRightNode = new Node(9);
		Node leftOfRightNode = new Node(8, null, rightOfLeftOfRightNode);
		Node rightOfRightNode = new Node(10, null, null);
		Node rightNode = new Node(7, leftOfRightNode, rightOfRightNode);

		Node root = new Node(5, leftNode, rightNode);
		return root;
	}

	private static void searchByDfs(Node node) {
		System.out.print(node.getValue() + " ");
		Node leftNode = node.getLeftNode();
		Node rightNode = node.getRightNode();
		if (leftNode != null) {
			searchByDfs(leftNode);
		}
		if (rightNode != null) {
			searchByDfs(rightNode);
		}
	}
}
```

실행결과는 아래와 같다

```
DFS:
5 3 2 1 4 7 8 9 10

BFS:
5 3 7 2 4 8 10 1 9
```

### 참고한 곳

> 메모리 구조에 대해서도 설명: https://ahnyezi.github.io/java/javastudy-5/
>
> - 블로그 내용이 강의 내용과 다소 다르다
> - https://www.youtube.com/watch?v=5u_YnIeyYAU&list=PLfI752FpVCS96fSsQe2E3HzYTgdmbz6LU&index=14
> - 위 주소의 1:20:00 부터 확인해 보자
>   (Lob님) 이진탐색 참고: https://lob-dev.tistory.com/entry/Live-StudyWeek-05-%ED%81%B4%EB%9E%98%EC%8A%A4
