---
layout: post
title: "우테코 프리코스 3차 - 자판기"
subtitle: "..."
date: 2021-12-12 02:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

# 우테코 프리코스 3차 과제

## 자판기 Vending Machine

---

## 이번주차에는 전혀 예상하지 못한 과제를 만나 보았다 !!

사실 이전에 경험해보았던 문제들과 달리 전혀 예상치 못한 문제를 만나게 되었습니다

지하철 노선도가 나올 것이라 기대를 하였는데 자판기가 나오게 되었을 줄은 상상도 못하였네요.. ㅎㅎ...

처음에는 쉽다고 생각하여 주말에 공부를 소홀히 했는데..

울며 겨자먹기로 열코딩하였습니다 ㅠㅠ

## 배운점

### 정규식을 통한 값의 검증

정규식을 통해서 문자열을 검증하였을때 편한점이 있다는 것을 파악하고 터득할 수 있는 기회가 되었습니다
정규식을 검증해볼 떄는 **SublimeText** 같은 메모장에 우선 연습을 해본 후에 표현식으로 옮기는게 수월하단 것을 알았습니다.
이때 주의할 것은 자바와 SublimeText 의 정규식 내부적인 처리 규칙이 다르다!
SublimeText에선

```
[\w]
```

위 식으로 한글까지 커버가 가능하지만 자바에선 그렇지 않습니다 !!

### Repository는 static 하게

객체로써 활용하지 않고 클래스레벨로 활용하자

왜냐하면 해당 Repository는 활용하는 곳이 많기 때문에 하나만 놓고 해당 클래스를 사용하는 곳이 좋다고 생각한다

예를들어 view와 validator 모두 BeverageRepository를 의존하고 있는데 이때마다 생성자 DI로 넣어주면 껄끄럽다 ㅠㅠ

### 역할과 분할

`getter`를 더욱더 줄여보기로 하고 `SRP`를 생각하면서 객체에 메세지를 보내는 방식의 설계를 추구하였습니다

### Validator 클래스 분리

검증 로직 자체는 비즈니스 로직상에 포함된다고 간주하여 Repository 클래스에 보관한 체로 별도로 분리하지 않았습니다

(Repository보다는 Service에 있어야 맞겠지만 현재 Repository가 Service의 역할도 같이 한다고 가정하였습니다)

그러나 검증 로직 또한 어떤 검증 로직 일부가 중복되는 등의 경우가 있었습니다

이에 따라 검증 로직을 별도로 상속구조를 가진 클래스로 분리하여 설계하였습니다

또 검증로직을 view단에서 (`InputView`) 에서 `Repository` 부분으로 변경하고 싶었지만

요구사항 중에 **예외** 가 발생하였을 경우 이전 입력부분부터 다시 시작하는 부분으로 변경하지 못했습니다

### 같은 메세지가 있다면 Message 클래스로 생성

간혹가다 어느 도메인에 속하지 않은 공통의 `message`가 있습니다

이럴경우는 `enum`으로 정의하여 공통 `message`로 처리할 경우 중복을 제거할 수 있습니다

그러나 이런 경우는 많지 않고 굳이 겹치는 message를 위해서 enum 클래스로 관리하는 것은

개인적인 소견으로 그다지 효율적인 방법은 아닌 것 같습니다

### 자료구조의 이해 `TreeSet`

이 과제를 하면서 자료구조에 대해 고민해볼 일이 많았습니다

예를들어 어떤 경우에 `Set`, `List`, `Map`을 사용할 지에 대해서입니다.

각각에서 순서가 보장되어야 하는 지에 따라서도 나뉘게 됩니다

예를 들어 중복은 없으며 순서가 보장되기를 원한다면 `TreeSet`을 사용하면 됩니다

이번 과제에서는 `TreeSet`을 많이 이용하였습니다.

### Enum에 대한 이해 및 활용

`Enum`에도 데이터를 저장하는 것이 가능하단 것을 알았습니다

그래서 중간에 일급컬렉션 등의 제 2의 클래스를 만드는 것이 아닌 `Enum` 자체의 클래스를 이용하였습니다.

`Coin` 클래스에서는 동전의 개수를 저장할 수 있어야 하는데

약간의 구조 변경으로 class 처럼 저장하는 역할로 변경하여 사용하였습니다.

이때 해당 클래스를 사용하면서 편했던 점으로는 전역전으로 어디에서든지 참조가 가능하며

해당 동전 리스트를 불러와서 사용할 수 있습니다

단점으로는 어디서든지 `Coin`에 접근 가능하기 때문에 결합도를 신경쓰지 않고 이곳저곳에서 무분별하게 사용했다는 점입니다.

`Enum`자체가 이런 용도로 사용하는 것이 맞는 건지는 좀 더 공부가 필요할 것 같습니다.

### OOP에 대한 이해

객체지향 개념에 대해 공부하게 된 계기가 되었다

클래스를 어떤 기준으로 분리하고 `getter`를 쓰지 않고 메세지를 보낼 때 어떤 이익이 있는가 (`SRP`)

객체를 여러번 생성하지 않고 싱글톤으로 만들어서 단일하게 생성하는 방법을 공부하는 등

전반적으로 설계에 대한 공부를 하게 되었습니다

자연스럽게 객체 지향에 대해 습관적으로 생각해보게 되었습니다

### 다른 사람들의 PR

미션이 깃허브에 public으로 오픈되어 있기 때문에 다른 사람들의 미션을 살펴볼 수 있습니다
사실 개인적으로 공부한 것도 그렇지만 가장 단시간에 많이 배운 것은 다른 사람들의 PR이었습니다
Validatort 분리, 오류 메세지를 별도로 분리하여 관리하는 등  
여러가지 구현할때 겪었던 어려움을 해소하는데 많은 도움이 되었습니다.

그리고 자극이 많이 되었습니다 학생분들의 경우 기말고사 기간이 겹쳐 있어서 많이 바쁜 것으로 알고 있는데
이렇게 열심히 하시는 분들이 많다는 것을 알고 더욱 분발하게 되었습니다.

### 객체지향 학습을 위해 공부한 자료들

#### 백기선님 Java 스터디 (멤버십)

> https://www.youtube.com/watch?v=v2BXL6SYaBc&list=PLfI752FpVCS96fSsQe2E3HzYTgdmbz6LU&index=16

#### 서적

> Java 객체 지향 디자인 패턴 - 한빛미디어

책의 내용이 잘 읽히지 않았지만 코드 중심으로 보았습니다.  
싱들톤이 멀티 쓰레드 환경에서 어떻게 구현을 하는 지에 대한 것을 잘 살펴 보았습니다.  
그 외에 전체적으로 객체 지향에 대한 필요한 개념들을 소개해 주어서 살펴보았습니다.

> 개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴 - 최범균

비교적 쉽게 읽혀서 앞에 있는 객체 지향에 대한 철학을 다시 한번 읽어 보면서 이해할 수 있었습니다

### 우테코 Custom Assertion 예러

아래와 같은 코드에서 우테코의 테스트코드를 통과하지 못하였다

```java
while (true) {
	try {
		System.out.println(INPUT_MACHINE_MONEY_MESSAGE);
		String input = Console.readLine().trim();
		CoinInputValidator.validateMachineMoneyInput(input);
		// System.out.println();
		return Integer.parseInt(input);
	} catch (Exception exception) {
		System.out.println(exception.getMessage());
		continue;
	}
}
```

아래와 같은 콘솔로그가 찍히면서 안 된다 ㅠ

![image](https://user-images.githubusercontent.com/66164361/145818774-e5ba5db3-a0a1-4c38-b795-c93f0ce90c31.png)

위의 `try` 문장이 포인트다 여기서 `input`변수를 해당 스코프 범위 밖으로 꺼내면 문제가 해결된다

### 사실은 CoinRepository는 필요 없다..?

Coin은 다른 자료구조와 달리 `enum`이다 !!

때문에 static하게 전역적으로 저장되며 어디서든 불러올 수 있다

때문에 굳이 일급컬렉션과 비슷한 용도로 사용하는 `CoinRepository`로 만들어서 관리 하지 않고

`Coin` 자체에서 관리를 하는 로직으로 변경하였다

다른 `Repository`는 해당 도메인 객체의 컬렉션을 지니고 있어서 `CRUD`등의 기능을 제공하는 게 맞다고 본다 !!

### 에러 메세지를 따로 상수로 관리할 떄는 enum보다는 class로 !

개인적으로 `enum`으로 관리할 때는

```
ERROR_PREFIX = "[ERROR] ";
INPUT_PREFIX = "입력값이 ";
ERROR_NOT_NUMBER = ERROR_PREFIX + INPUT_PREFIX + "숫자이어야 합니다.";
```

위와 같은 것을 할 수가 없다 (_내가 아는한_)

그래서 class로 만들어서 `public static final` 로 관리해주면 수월하다

### 사실은 음료만 있을 수 있는게 아니 잖아...?

어느샌가 존댓말에서 편한말로 옮겨졌다...

사실은 비단 음료수 뿐만 아니라 그 이외의 칙촉 등의 과자류, 콘돔(...) 작은 물건들이 올 수 있을 수 있는데

음료수(`Beverage`)로 제한을 둔 것 같다

`Product`는 운영환경을 떠올리는 딱딱한 단어인 것 같아서

조금 더 게임 인벤터리처럼 편하게 감기는 `Item`이란 용어를 차용했다

실제로 영한님 강의에서도 상품을 `Item`으로 사용하였다 !



## 개선해나갈 점 (장기적인 부분)

---

### 중복되는 코드의 제거

```java
		while (true) {
			try {
				System.out.printf(OUTPUT_INPUT_MONEY_TO_BUT, inputMoneyToBuy);
				System.out.println(INPUT_BEVERAGE_NAME);
				String input = Console.readLine();
				BeverageInputValidator.hasInBeverages(input);
				return input;
			} catch (Exception exception) {
				System.out.println(exception);
				continue;
			}
```

위와 같은 코드가 굉장히 반복적으로 들어난다

위 코드를 제거할 수 있으면 좋을 것 같다  
람다 식을 생각하고 있는데 input과 output이 다양하게 나올 수 있기 때문에  
그것을 고려한 함수식을 어떻게 설계할지가 관건일 것 같다

### 근본없는 결합도

`ItemRepository` 를

동일한 Level의 클래스인 `CoinRepository`를 참조하거나

view단인 `InputView` 참조하고 있다

#### 다른 사람들의 코드를 통한 회고

`Controller` 나 `Service` 계층을 통해서

`CoinRepository`, `ItemRepository`를 참조하는 방식으로 의미 없는 거미줄같은 의존 관계를 막을 수 있다 !!

### 깃 사용 방법 익히기

깃을 잘 다루지 못하여

커밋 메세지 하나를 수정하려다가 아래와 같은 불필요한 메세지를 남기게 되었다 ㅠㅠ

![image](https://user-images.githubusercontent.com/66164361/145912249-2b29946a-28cc-43f5-9319-a4ac37fb12b3.png)

원격에 이미 올린 커밋 메세지를 돌리지 못하여 생긴 문제인데

git에 대해 다시 공부해볼 필요가 있다

...
