---
layout: post
title: "우테코 레이싱카 후기"
subtitle: "..."
date: 2022-02-17 20:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 레이싱 카를 만들면서 겪게된 후기를 작성합니다
> 깃허브 PR 주소 : https://github.com/woowacourse/java-racingcar/pull/331

## 자동차 경주를 하면서 했었던 고민들

---

### 자바의 정규식 Pattern, Matcher.. 주의사항!

`matcher.group()` 전에 최소한 `find()`를 한번은 호출해야 한다 !!

그렇지 않으면 예측하지 못한 런타임 에러가 나온다

### 정규식 컴파일한 것을 캐싱

---

`문자열.matches(정규식)` 이

매번 `Pattern` 를 생성 비용이 많이 든다 때문에 캐싱하는 방식으로 변경하였다

> CarName.java

```java
public static final Pattern COMPILED_CAR_NAME_PATTERN = Pattern.compile(NOT_ALLOWED_REGEXP_STRING);

private static boolean doesNotMatchCarName(String carName) {
    return COMPILED_CAR_NAME_PATTERN.matcher(carName).matches();
}
```

### Cars 객체를 불변으로 변경

---

> Cars.java

```java
private final List<Car> cars;

...

public List<Car> getCars() {
    return Collections.unmodifiableList(cars);
}
```

### private 메서드 테스트 ?

---

비즈니스에 연관된... 실제로 테스트하는 것이 중요하다고 판단되는 메서드에 대해서만 진행하면 된다 !

오죽하면 `should i test private method` 란 사이트도 있겠는가/

### 도메인 구조...

---

도메인 구조가 실전에서는

1.

```
- 도메인1
  - 컨트롤러
  - 서비스
- 도메인2
```

2.

```
- 컨트롤러
- 서비스
- 도메인
```

등 어떻게 되는지 궁금했는데 아래처럼 대답해주셨다

```
- controller
- service
   - lineService
   - stationService
- domain
   - line
   - station
```

### void -> void 인터페이스가 없어서 `Voider` 란 인터페이스를 만들었는데... 알고보니 있었다 !

---

내 결론은.. 아직도 결론이 안났다는 것이다!!

`Runnable`을 사용 하는 것은 편하지만,

그걸 사용하면 쓰레드에서 사용하는 함수를 만드는 것 같은 이 느낌.. 애매하다.

> 희봉의 답변

![image](https://user-images.githubusercontent.com/66164361/154469842-36cfe32b-8de0-4921-87ae-080e713133c2.png)

### 앵귤러 커밋 컨밴션에서 `scope` 란?

---

> 희봉의 답변

![image](https://user-images.githubusercontent.com/66164361/154470487-7d5cfae2-084f-4cdd-9951-b709219f0ccb.png)

### 🥝람다를 이용한 입력할 때마다 보이는 👻보일러 플레이트 제거 !

---

#### 문제점

예를들어 우리는 입력받을 때마다 아래와 같은 보일러 플레이트(`중복`) 코드를 발견한다

~~있어보이려고 저런 단어를 선택하였다 미안하다. 어쩔거야?~~

아래와 같은 코드다 `T.T`

```java
while (true) {
	try {
		// (1)
		return;
	} catch (Exception e) {
		System.out.println(e.getMessage());
	}
}


```

저 수많은 코드 중에서

결국 필요한 입력 코드는 `(1)` 코드에 불과하다

이걸 매입력마다 받는건 에바다

#### 해결법

---

자바`8`의 `람다`를 이용하여 해결해볼 수 있다!

JS는 함수를 인자로 넘겨서 지지고 볶는데 능통한 언어인데 자바도 이게 가능하다

자! 조필즈의 코드를 보도록 하자 (?!)

```java
public static void 무언가_중복되는_입력_코드(Runnable runnable) {
	while (true) {
		try {
			runnable.run();
			return;
		} catch (Exception e) {
			System.out.println(e.getMessage());
		}
	}
}
```

저렇게 하면 함수를 인자로 넘겨서 오로지 저 부분만 실행할 수 있다 !!

저 함수를 실행하는 예시 코드는 아래와 같다

```java
Runnable runnable = () -> {
	int randomNum = new Random().nextInt(100);
	if (randomNum % 2 == 0) {
		throw new RuntimeException("짝수일 경우 이 예외를 발생합니다.");
	}
	System.out.println("날 실행해줘 !");
};

for (int i = 0; i < 10; i++) {
	무언가_중복되는_입력_코드(runnable);
}
```

#### 적용한 실제 코드

---

```java
public void initGame() {
  initCars();
  initRoundNumbers();
}

private void initCars() {
  Voider voider = () -> {
    String carNames = inputView.inputCarNames();
    Cars cars = new Cars(carNames);
    List<Car> carList = cars.getCars();
    carRepository.addCars(carList);
  };
  commonInputProcess(voider);
}

private void initRoundNumbers() {
  Voider voider = () -> {
    String input = inputView.inputRoundNumber();
    TryRoundNumber tryRoundNumber = new TryRoundNumber(input);
    roundNumber = tryRoundNumber.get();
  };
  commonInputProcess(voider);
}

private void commonInputProcess(final Voider inputFunction) {
  while (true) {
    try {
      inputFunction.execute();
      return;
    } catch (Exception exception) {
      inputView.printErrorMessage(exception);
    }
  }
}
```

### 🏭빌더와 🚢전략 패턴을 이용한 자동차 움직임 제어

---

> ### 리펙터링 히스토리

---

1. 전략 패턴을 생성자 주입

2. 전략 패턴을 외부로 드러내지 않는 정적 팩터리 메서드로 변경

3. 전략 패턴을 다시 외부로 드러내고 빌더 패턴으로 변경

`1.` 에서 `2.` 로 변경하게 된 이유와 같다

![image](https://user-images.githubusercontent.com/66164361/154472050-e253fcd6-62dc-4dc7-a14d-4fe78f05104d.png)

---

#### 문제점

---

자동차 움직임을 구현하면서

테스트에서 필요한 자동차와 운영코드에서 필요한 자동차의 움직임이 각각 다르단 것을 깨달았다

또한 자동차는 경우에 따라 서로 다른 생성(`new`) 스타일을 가지고 있었다

#### 해결법

---

빌더 패턴과 전략 패턴을 이용해서 자동차의 생성을 구현하였다

빌더 패턴을 이용한다면 여러가지 스타일의 생성자에 대한 커버가 가능하다 !!

> Car.java

```java

public class Car {

	private CarName name;
	private MovingStrategy movingStrategy;

	/**
	 * 다른 스타일의 생성자를 허용하지 않는다
	 */
	private Car() {
	}

	private Car(Builder builder) {
		this.name = builder.name;
		this.position = builder.position;
		this.movingStrategy = builder.movingStrategy;
	}

	public String getName() {
		return name.get();
	}

	public static class Builder {
		private CarName name;
		private int position = 0;
		private MovingStrategy movingStrategy;

		public Builder name(String name) {
			this.name = new CarName(name);
			return this;
		}

		public Builder position(int position) {
			this.position = position;
			return this;
		}

		public Builder movingStrategy(MovingStrategy movingStrategy) {
			this.movingStrategy = movingStrategy;
			return this;
		}

		public Car build() {
			return new Car(this);
		}
	}

	public static Builder builder() {
		return new Builder();
	}
	...
```

> 사용부

```java
final Car car = Car.builder()
	.name(carName)
	.movingStrategy(new RandomMovingStrategy())
	.build();
```

### 중복된 차가 있을시에 예외처리

---

아래처럼 처리하였다

```java
private boolean isDuplicated(final List<Car> cars) {
  return cars.size() != new HashSet<>(cars).size();
}
```

### 그래 아주 그냥 ! 멀티 쓰레드까지 고려하는 거야 !! 1. Random 대신 ThreadLocalRandom 그리고 2. Scanner 대신 BufferedReader(new InputStreamReader(System.in))

---

위 `1.` 과 `2.` 모두 멀티 쓰레드에서 성능이 좋은 API이다.

따라서 변경하였고 `Scanner` 는 `InputView` 에서 사용하고 있던 `API` 인데 유틸리티 클래스에서 `new`로 인스턴스화 가능한 클래스로 변경하였다

### System.**setIn** 을 통한 Scanner의 입력값 교체하기

---

이건 다른 분의 블로그를 통해서 알게 되었다..

신기하군 !

> https://sakjung.tistory.com/33 > https://steadyjay.tistory.com/10

그러나 `Scanner` 는 `string`도 input으로 받을 수 있는 것 같다.. 이걸 어떻게 잘 활용하면 다른 방법으로도 풀어낼 수 있지 않을까??

### toString 의 바람직한 용도

---

로깅/디버깅 용도로 사용되지 운영 코드로는 사용되지 않는 다는 것..

InputView로 도메인이나 DTO를 넘겨서 이곳에서 view를 위한 정보를 만들어 내는 것이 좀 더 용례 맞을 지도 모른다고 생각이 들기도 한다..

> 참고: https://hudi.blog/java-correct-purpose-of-tostring/

그러나 이부분도 의견이 엇갈린다.. 아래는 스택오버플로 답변

![image](https://user-images.githubusercontent.com/66164361/154497968-b2e1f858-320e-4f1a-84f7-f16cc4f178fc.png)

> 출처: https://stackoverflow.com/questions/563676/is-tostring-only-useful-for-debugging

흠, 일단은 안쓰는 쪽으로 가보려고 한다.

뷰는 콘솔에서 스윙이나 HTML등으로 바뀔 수 있으니..

toString은 좀 더 온전한 의미를 담기로 하고, 시각적 요소를 없애는 방향으로 가자

### TDD: RED-GREEN-REFACTOR

내가 아는 `TDD`는 테스트 코드를 작성하는 행위 하나로 알고 있었는데 알고보니

어떤 순서를 가지고 있는 일련의 행위를 가지고 있었다

바로 `RED-GREEN-REFACTOR` 인데

이 `cycle`을 돌림으로써 테스트코드가 개발을 주도하게끔 하는 것이 TDD였다
