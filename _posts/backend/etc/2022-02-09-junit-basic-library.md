---
layout: post
title: "[TDD] JUnit 기초 문법들 정리 모음"
subtitle: "..."
date: 2022-02-09 15:30:00 +0900
categories: backend
tags: etc
comments: true
---

> `JUnit5` 기준으로 작성되었습니다 !

## JUnit ?

---

- 자바 개발자 93%가 사용하는 단위 테스트
- JUnit5는 2017에 공개
- 부트 2.2 버전부터 기본 제공

- Jupitor: JUnit5 구현체
- Vintage: Junit3~4 구현체

> 둘다 TestEngine API 구현체이다

## 항상 햇갈리는 `assertThat`의 클래스

---

자주 사용하는 API로 `assertThat`가 있는데

보통 static import로 사용할 떄가 많다

이때

```
org.assertj.core.api
org.junit.jupitor.api
```

위 두 `api`가 있는데

**assertj** 를 선택하면 된다 !

```java
import static org.assertj.core.api.Assertions.*;
```

`assertThat` 뒤에는 `actual` 실제로 나온 값이 오고

그 뒤에 `.`을 찍어서 오는 메서드의 인자는 `expected` 기대하는 값이다

### 기본적인 사용법

---

```java
assertThat(replaceString).isEqualTo("1,2");
assertThat(splitString).containsExactly("1", "2");
```

보면 바로 이해가 될 것이다

### 생명주기 어노테이션

---

| 항목          | 설명                                                     |
| ------------- | -------------------------------------------------------- |
| `@BeforeAll`  | 클래스 내 모든 테스트 메서드 실행 전에 **딱 한 번** 실행 |
| `@AfterEach`  | 클래스 내 모든 테스트 메서드 실행 후에 **딱 한 번** 실행 |
| `@BeforeEach` | 클래스 내 모든 테스트 메서드 실행 전에 **각각** 실행     |
| `@AfterEach`  | 클래스 내 모든 테스트 메서드 실행 후에 **각각** 실행     |

### 예외 상황에서

---

```java
assertThatThrownBy(() -> testString.charAt(3))
			.isInstanceOf(StringIndexOutOfBoundsException.class);
```

```java
assertThatExceptionOfType(StringIndexOutOfBoundsException.class)
			.isThrownBy(() -> testString.charAt(3))
			.withMessageMatching("String index out of range: \\d+");
```

### 하지 않을 테스트

---

파이썬의 `pass` 문법처럼 당장하고 싶지 않은 처리에 대해서 넘기고 싶을 때가 있다

그럴때 사용하는 기능이다

```java
@Disabled
@Test
@DisplayName("하지 않고자 하는 테스트 👻")
public void testDisable() {
    assertThat(1).isEqualTo(2);
}
```

### 반복 테스트

---

```java
@RepeatedTest(10)
	@DisplayName("반복 테스트")
	public void repeatedTest() {
		System.out.println("(++a) = " + (++a));
	}
```

```java
@RepeatedTest(value = 10, name = "{displayName} 중 {currentRepetition} of {totalRepetitions}")
@DisplayName("이름이 설정된 반복 테스트")
public void repeatedTestWithName() {
    System.out.println("(++b) = " + (++b));
}
```

### 매개변수 사용법

---

조금더 어려운 사용법이다 !

#### `@ValueSource`

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3})
@DisplayName("매개변수 : ValueSource 테스트")
public void parameterizedTest1int(int number2) {
    System.out.println("number2 = " + number2);
}
```

#### `@CsvSource`

처음 봤을 때 문법이 이해가 가지 않았다

`,` 등 구분자에 대해 나뉜 값들이 메서드의 인자로 들어간다

```java
@ParameterizedTest
@CsvSource({"test,TEST", "tEst,TEST", "Java,JAVA"})
@DisplayName("매개변수 : CsvSource 테스트")
void toUpperCase_ShouldGenerateTheExpectedUppercaseValue(String input, String expected) {
    System.out.printf("[input] %s  :: [expected] %s \n ", input, expected);
    String actualValue = input.toUpperCase();

    assertThat(actualValue).isEqualTo(expected);
}
```

```java
@ParameterizedTest
@CsvSource(value = {"test:test", "tEst:test", "Java:java"}, delimiter = ':')
@DisplayName("매개변수 : CsvSource with delimiter 테스트")
void toLowerCase_ShouldGenerateTheExpectedLowercaseValue(String input, String expected) {
    String actualValue = input.toLowerCase();
    System.out.printf("[input] %s  :: [expected] %s \n ", input, expected);

    assertThat(actualValue).isEqualTo(expected);
}
```

### @Nested

### 참고

> 바다의 JUnit5 사용법 https://youtu.be/EwI3E9Natcw
> ParameterizedTest 사용법 https://gmlwjd9405.github.io/2019/11/27/junit5-guide-parameterized-test.html

### assertThat(..).isEqualTo(..) 가 비교하는 것은 동등성일까 동일성일까?

---

위 사항이 궁금하여 아래의 코드로 테스트를 돌려보았다

결과는 동등성을 비교하였다

![image](https://user-images.githubusercontent.com/66164361/153705856-f825ae43-a43d-4531-a7c8-9b08d3b45e4f.png)

또한 객체를 따로 생성하여서 아래와 같이 동등성을 비교해보았다

![image](https://user-images.githubusercontent.com/66164361/153706073-0548b8c4-0ecf-43e9-aa2a-000e69366d1f.png)

`euals`를 오버라이드 하기 전에는 fail이었지만 한 후에는 pass가 되었다

만일 동일성을 비교하는 것이었으면 오버라이드 유무에 관계 없이 모두 fail이었을 것이다

따라서 동등성이 기준이었음을 확인하였다
