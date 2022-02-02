---
layout: post
title: "[스터디할래] whitesheep #14 제네릭"
subtitle: ""
date: 2022-02-02 01:00 +0900
categories: backend
tags: java
comments: true
---

### 제네릭(Generics) 란?

---

- since JDK 1.`5`
- 컴파일시 타입을 체크해 주는 기능
  - `ClassCastException` 으로부터 자유롭다
  - `compile`-time `type` check
  - `타입 안정성` 제공
- 타입을 객체 생성시 지정할 수 있기 때문에 값을 가져올 때  
  `형 변환이 생략 가능` 하다

### 개발자의 실수를 미리 막을 수 있어

---

`RuntimeException` 은 프로그래머의 실수로 발생하는 에러라 할 수 있어

이 에러를 런타임이 아닌 컴파일 타임 때 잡아버리자 !

### 런타임 예외 예시) 예를 들어..

---

```java
String str1 = null; // (1)
String str2 = ""; // (2)
```

`(1)` 보다는 `(2)`가 더 좋은 코드다

왜냐하면 NPE로부터 안전하기 때문이다 아래의 코드를 보면 알 수 있다

```java
str1.length();  // NPE !
str2.length();
```

str1은 NPE 가 발생한다 !

아래의 경우도 마찬가지이다

```java
Object[] objArr1 = null;
Object[] objArr2 = new Object[0]; // 길이가 0인 배열
Object[] objArr2 = {};
```

### 타입 변수

---

- 클래스를 작성할 때 Object 타입 대신 타입 변수 (E)를 선언해서 사용

### 타입 변수에 대입하기

---

- 객체를 생성시, 타입 변수(E) 대신 실제 타입(Tv 등)을 대입

```java
// 참조 변수
ArrayList<Tv> list = new ArrayList<Tv>();
//                        생성자
```

### 지네릭스 용어

---

| 항목   | 설명                                          |
| ------ | --------------------------------------------- |
| Box<T> | 지네릭 클래스. T의 Box 혹은 T Box 라고 읽는다 |
| T      | 타입 변수 혹은 타입 매개변수                  |
| Box    | 원시 타입 (raw type), 일반 클래스             |

### 지네릭 타입과 다형성

1. 참조 변수에 대입된 타입과 생성자의 대입된 타입은 일치해야 한다

```java
ArrayList<Tv> list = new ArrayList<SamsungTv>();
// 허용하지 않음
```

2. `1`번을 만족할때 지네릭 클래스간의 다형성은 성립

```java
List<Tv> list = new ArrayList<Tv>();
```

3. 매개변수의 다형성도 성립

```java
List<Tv> list = new ArrayList<Tv>();
list.add(new SamsungTv());
list.add(new LgTv());
```

이게 성립하는 이유는

```java
public boolean add(E e) {
  add(size(), e);
  return true;
}
```

위 코드에서 `E`가 Tv로 변경이 되었기 때문에 우리가 아는 다형성이 성립한다

하지만 get 과 같은 경우는 아래와 같은 불편함이 생긴다

```java
SamsungTv tv = (SamsungTv) list.get(0);
```

왜냐하면 이 역시 타입변수가 조상타입인 Tv로 대체되었기 때문

### 제한된 지네릭 클래스

---

- `extends`나 `super`로 대입할 수 있는 타입을 제한

```java
class FruitBox <T extends Fruit> { // Fruit과 그 자손 타입만 지정 가능
  List<T> = new ArrayList<>();
}
```

- 인터페이스도 가능하다

```java
interface Eatable { ... }

class FruitBox <T extends Eatable> { ... }
```

- And 조건으로 제한 가능

```java
class FruitBox <T extends Fruit & Eatable> { ... }
```

위와 같은 코드가 있을 떄 대입이 가능한 타입 변수는 아래와 같이 선언이 되어 있어야 한다

```java
static class Apple extends Fruit implements Eatable { ... }
```

### 잠시 보는 필자의 코드 !

---

여러개 제한 가능과 관련해서 필자가 작성한 코드를 같이 보자 !

```java
static class FruitBox <T extends Fruit & Eatable>
```

```java
import static basic.GenericSampleClass.*;

public class GenericMain {
	public static void main(String[] args) {
		test1();
	}

	private static void test1() {
		Apple apple = new Apple(1L);
		FruitBox<Apple> fruitBox = new FruitBox<>();

		long addedAppleId = fruitBox.add(apple);
		Fruit findApple = fruitBox.findOne(addedAppleId);

		System.out.println("findApple = " + findApple);
	}
}
```

```java
class GenericSampleClass {

	@Getter
	@ToString
	@AllArgsConstructor
	static class Fruit {
		long id;
	}

	interface Eatable {
	}

	@ToString(callSuper = true)
	static class Apple extends Fruit implements Eatable {
		public Apple(long id) {
			super(id);
		}
	}

	static class Banana extends Fruit {
		public Banana(long id) {
			super(id);
		}
	}

	static class FruitBox <T extends Fruit & Eatable> {
		private List<T> fruits = new ArrayList<>();

		public long add(T fruit) {
			fruits.add(fruit);
			return fruit.getId();
		}

		public Fruit findOne(long id) {
			return fruits.stream()
				.filter(fruit -> fruit.getId() == id)
				.findAny()
				.get();
		}
	}
}
```

### 지네릭스의 제약

---

1. 타입 변수에 대입은 인스턴스 별로 다르게 가능

```java
Box<Apple> appleBox = new Box<Banana>(); // 예외, Apple만 가능
```

2. static 필드에 타입 변숴 사용 불가

```java
class Box<T> {
  static T item; // 에러
  static int compare(T t1, T t2) {...} // 에러
}
```

> 이 부분에 대해선 어떤 분이 잘 고민해놓은 생각이 있다

애당초 지네릭은 new 생성된 객체들에 대한 타입 지정인데 static은 그렇지 않다 ! (새로 생성되는 것이 아님)

때문에 클래스레벨에서 무언가를 처음부터 고정시키는 것은 불가능, 그렇게 되면 각 인스턴스마다 서로 다른 타입을 지정하는게 불가능해진다

3. 배열 생성할 떄 타입 변수 사용불가

- 타입 변수로 배열 선언은 가능

`new` 연산자 뒤에 타입 변수가 오는 것이 불가능 하다 (new는 동적인인데 타입 변수 자체는 확정적이지 않음)

```java
static class FruitBox <T extends Fruit & Eatable> {

  private static final int DEAFULT_CAPACITY = 100;

  // T[] arr = new T[DEAFULT_CAPACITY]; // 에러
  private T[] arr = (T[]) new Object[DEAFULT_CAPACITY];
}
```

### 와일드 카드 `<?>`

---

> 타입 변수에 대해서도 `다형성`과 같은 사상을 가능하게 해준다

- 참조 변수의 타입이 다른 타입을 참조 가능하게 한다

```java
ArrayList<Tv> list = new ArrayList<SamsungTv>(); // 예외
```

```java
ArrayList<? extends Tv> list = new ArrayList<SamsungTv>(); // 가능
```

| 항목           | 설명                                                      |
| -------------- | --------------------------------------------------------- |
| <? extends Tv> | 와일드 카드의 상한 제한, T와 그 자손들만 가능             |
| <? super Tv>   | 하한 제한, T와 그 조상들만 가능                           |
| <?>            | 제한 없음. 모든 타입이 가능. `<? extends Object>` 와 동일 |

#### 메서드의 매개 변수에 와일드 카드 사용

> 구현부

```java
static Juice makeJuice(FruitBox <Fruit> box) {...}
```

만일 위와 같이 메서드를 정의했다면

> 사용부

```java
out.println(Juicer.makeJuice(new FruitBox<Fruit>)); // (1)
out.println(Juicer.makeJuice(new FruitBox<Apple>)); // (2)
```

사용부의 (2) 는 적용이 불가능하다

```java
static Juice makeJuice(FruitBox <? extends Fruit> box) {...}
```

그래서 위와 같이 와일드 카드를 적용 해주어야 한다 !

### 지네릭 메서드

---

클래스 레벨이 아닌 **메서드 레벨**에서 적용된 지네릭 !

```java
FruitBox.printSomething(new Banana(101L));
```

```java
class FruitBox <T extends Fruit & Eatable> {
  public static <S> void printSomething(S something) {
    System.out.println("something class name = " + something.getClass().getSimpleName());
  }
}
```

- 클래스에 선언된 제네릭과 메서드에 선언된 제네릭은 서로 별개이다 !

  - 설령 같은 변수명 `T`를 가질지라도 !

> 위 내용을 증명하는 코드

```java
	private static void test3() {
		FruitBox<Apple> fruitBox = new FruitBox<>();
		Apple apple = new Apple(1L);
		fruitBox.add(apple);

		fruitBox.printSomething(new Banana(101L));
	}
```

```java
class FruitBox <T extends Fruit & Eatable> {
		public <T> void printSomething(T something) {
			System.out.println("something class name = " + something.getClass().getSimpleName());
		}
```

클래스와 메서드가 동일한 변수 T를 가지지만

클래스는 Apple을 타입으로 메서드는 Banana로 타입을 받아서 사용하기 때문에 클래스와 메서드 선언부의 동일 타입 변수 T가 별개임을 알 수 있다

메서드 레벨의 `<T>` 를 제거한다면 클래스 레벨의 타입변수가 적용되기 때문에 클라이언트 코드에 컴파일 에러가 난다

iv와 lv의 관계와 같다 (적용 범위가 구체적이고 협소할 수록 우선시됨)

- 또한 메서드에 새로이 정의된 제네릭은 로컬 변수 영역에서 타입 변수가 사용되기 떄문에  
  `static` 메서드에서도 사용이 가능하다 !!

- 본래 메서드를 호출할 떄마다 타입을 대입해야 한다
  - 하지만 대부분 생략이 가능!

```java
out.println(Juicer.<Fruit>makeJuice(fruitBox));
out.println(Juicer.<Apple>makeJuice(appleBox);
```

```java
static Juice <T extends Fruit> makeJuice(FruitBox <T> box) {...}
```

참고로 위 제네릭 코드는 아래 와일드카드를 파라미터로 갖는 메서드로 바꿀 수 있다

```java
static Juice makeJuice(FruitBox <? extends Fruit> box) {...}
```

### 지네릭 타입의 형변환

---

- 지네릭 타입과 원시 타입간의 형 변환은 바람직 하지 않다
  - 경고 발생
- 타입이 다른 것은 컴파일 에러

```java
Box<Object> objBox = null;
Box rawtypeBox = (Box) objBox;  // 되지만 경고
objBox = (Box<Object> box);     // 되지만 경고

Box<String> strBox = (Box<Object> strBox) // 에러
```

- 와일드 카드가 사용된 지네릭 타입으로는 형변환 가능

```java
Box<? extends Object> wBox = (Box<? extends Object>) new Box<String>(); // 된다 #1
Box<? extends Object> wBox = new Box<String>(); // #2
```

사실 #1은 우리가 친숙하게 보던 #2와 같다

하지만 사실은 #1은 #2의 축양형 ! (문법적으로 생략이 가능하게끔 해준 것)

### 지네릭 타입의 제거

---

- 컴파일러는 지네릭 타입을 제거하고, 필요한 곳에 형변환을 넣는다 !

  - 때문에 런타임 시점에는 지네릭이 없다 !

- 아래와 같은 과정들을 거친다 (자세한 건 생략)

1. 지네릭 타입 변수 제거

2. 타입 제거 후 타입이 불일치 하면 형변환을 추가

3. 생략...

> 이외의 이야기

위와 같은 C#은 거치지 않고 런타임 시점에도 지네릭이 남아있다고 한다 !

실제로 그렇게 하는 편이 성능상에도 좋고 런타임 시점에 해당 타입을 받아올 수 있기 때문에 언어적인 측면에서도 유리하다고 한다

그렇다면 왜.. 자바는 그렇게 하지 않았을까?

**하위 호환성** 때문이다..

ㅠㅠ 너무 많은 곳에서 사용하고 있기 때문에 이전 버전에 대한 배려로 삥 둘러가서 이런 방식으로라도 이전 버전에 대한 호환성을 지켜주는 것이다

자바가 몰라서 안했던 것이 아니라.. 배려적인 측면에서 안했던 것.. (C#은 속도를 선택했던 것)

이런 이유 떄문에 자바의 발전 속도가 좀 더뎠던 적도 있었다고 한다...

### JpaRepo <CRUD>

---

### Erasable

---

### 런타임 시점에 클래스 가져오기 !

---
