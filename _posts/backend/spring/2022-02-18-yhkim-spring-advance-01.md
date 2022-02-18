---
layout: post
title: "영한님: spring 고급 (1)"
subtitle: "..."
date: 2022-02-18 14:00:00 +0900
categories: backend
tags: spring
comments: true
---

> 이 포스트는 강의 내용에 관한 노트나 필자가 코딩한 것을 기록했습니다

---

# 스프링 핵심원리 고급편 정리

---

## 예제 만들기

---

- 로그 추적기 만들기 !
  - 표현
    - 트랜잭션 ID : `http request` 쓰레드가 요청하는 처리 당 ID
    - 깊이 : 요청을 처리하는 깊이
  - `@Component`로 등록하여 스프링 빈으로 등록 !

## 쓰레드 로컬 - ThreadLocal

---

- 구조를 최대한 망가뜨리지 않고 TraceId를 `동기화` 하자 !

- 그러나 이 과정에 `동시성` 문제가 발생한다

- 강의에서 발생한 `동시성 문제` : 여러 쓰레드가 같은 인스턴스 필드에 접근할 때, 조회하려던 데이터가 수정되었음

- `동시성 문제`와 관련된 참고
  - 지역 변수에서 발생하지 않고 (쓰레드마다 각각 다른 메모리 영역)
  - 같은 인스턴스의 필드 (주로 `싱글톤`) 혹은 `static`같은 `공용` 필드 접근할때 발생
  - 값을 `읽기만` 하면 발생하지 않는데 어디선가 값을 `변경`하기 때문에 발생 !

> 이때 ThreadLocal을 소개한다 !

### 쓰레드 로컬

---

- 해당 쓰레드만 접근할 수 있는 저장소를 말한다

- 각 쓰레드마다 별도의 내부 저장소를 제공하기 때문에 같은 인스턴스 필드에 접근해도 문제 없다 !

```java
public class ThreadLocalService {

	private final ThreadLocal<String> nameStore = new ThreadLocal<>();

	... {
		nameStore.set(name); // 저장
    ...
		return nameStore.get(); // 조회
	}
  ...
}
```

| 항목    | API                  |
| ------- | -------------------- |
| 값 저장 | ThreadLocal.set(xxx) |
| 값 조회 | ThreadLocal.get()    |
| 값 제거 | ThreadLocal.remove() |

> 사용후 반드시 쓰레드 로컬의 값을 제거(`.remove`)해주어야 한다

- 스프링을 사용시 쓰레드 풀을 이용하게 되는데  
  이때 기본 정책이 한번 이용한 쓰레드는 파기하지 않는 것이다  
  사용후 남아 있는 쓰레드 로컬 저장소가 다음 사용자가 확인시  
  이전 사용자의 데이터가 남아있을 수 있는 것 !

## 템플릿 메서드와 콜백 패턴

---

- 핵심기능 vs 부가 기능
  - 핵심기능: 비즈니스 로직, 객체가 제공하는 고유의 기능
    - 예) 주문
  - 부가 기능: 핵심 기능을 보조하기 위해 제공되는 기능
    - 예) 로그 추적, 트랜잭션

> 문제점!

핵심기능이 있는 로직에 부가 기능을 넣었더니 부가기능 코드가 너무 많아!

> 변하는 것과 변하지 않는 부분을 분리 !

### 템플릿 메서드 패턴

---

템플릿이라는 거대한 틀에 변하지 않는 부분을 몰아두고 일부 변하는 부분은 별도로 호출

```java
@RequiredArgsConstructor
public abstract class AbstractTemplate<T> {

	private final LogTrace trace;

	public T execute(String message) {
		TraceStatus status = null;
		try {
			status = trace.begin(message);

			// 로직 호출
			T result = call();

			trace.end(status);

			return result;
		} catch (Exception e) {
			trace.exception(status, e);
			throw e;
		}
	}

	protected abstract T call();
}
```

### 영한쌤이 생각하는 `좋은 설계`란?

---

`변경`이 일어날 떄 자연스럽게 드러난다!
예를들어 현재 패턴을 적용 전과 후로 볼 때
로그 남기는 로직이 변경된다고 생각할 떄..
`전`과 같은 경우 모든 클래스를 찾아 고쳐야 하지만
`후`는 그렇지 않다

### 단일 책임 원칙 (SRP)

---

변경 지점을 하나로 모아서 변경에 쉽게 대처할 수 있는 구조로 바꾼 것
핵심 기능과 부가 기능을 분리

### GOF에서의 템플릿 메서드 정의

---

> "작업에서 알고리즘의 골격을 정의하고 일부 단계를 하위 클래스로 연기합니다. 템플릿 메서드를 사용하면
> 하위 클래스가 알고리즘의 구조를 변경하지 않고도 알고리즘의 특정 단계를 재정의할 수 있습니다." [GOF]

### 영한님이 다시 해석한 템플릿 메서드

---

부모 클래스에서 알고리즘의 골격인 템플릿으 ㄹ정의하고, 일부 변경되는 로직은 자식 클래스에 정의한다
알고리즘의 전체 구조를 변경하지 않으면서 특정 부분만 재정의 할 수 있다

> 상속 + 오버라이딩을 통한 다형성

**_그러나_**

상속을 사용하는 템플릿 패턴은 상속에서 오는 단점을 그대로 안고 간다!
부모 클래스에 강하게 의존, 즉 결합한다 !

> 상속보다는 위임(구성)을 !

## 전략패턴

---

변하지 않는 부분을 `Context`에 두고 변하는 부분을 `Strategy` 라는 인터페이스를 만들고 각 구현체들을 둔다

### GOF에서 정의한 전략 패턴

---

> 알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을
> 사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.

### 생성자 DI 전략 패턴 코드

---

> 스프링에서 의존관계 주입에서 사용하는 방식이 바로 전략 패턴이다 !

```java
public interface Strategy {
  void call();
}
```

```java
@Slf4j
public class StrategyLogic1 implements Strategy {
	@Override
	public void call() {
		log.info("비즈니스 로직1 실행" );
	}
}
```

```java
/**
 * 필드에 전략을 보관하는 방식
 */
@Slf4j
@RequiredArgsConstructor
public class ContextV1 {

	private final Strategy strategy;

	public void execute() {
		long startTime = System.currentTimeMillis();
		// 비즈니스 로직 실행
		strategy.call(); // 위임
		// 비즈니스 로직 종료
		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime;
		log.info("resultTime={}", resultTime);
	}
}
```

### 선조립, 후 실행

---

`Context`와 `Strategy`를 실행 전에 원하는 모양으로 조립해두고, 그 다음에 `Context`를 실행한다

> 스프링으로 애플리케이션을 개발할 때 애플리케이션 로딩 시점에 의존관계 주입을 통해 필요한 의존관계를
> 모두 맺어두고 난 다음에 실제 요청을 처리하는 것 과 같은 원리이다

조립을 미리 하지 않고 실행시마다 전략을 바꾸고자 한다면 다른 방법이 필요하다 !

### 메서드 주입 방식의 전략 패턴 코드

---

```java
/**
 * 필드에 파라미터로 전달 받는 방식
 */
@Slf4j
public class ContextV2 {

	public void execute(Strategy strategy) {
		long startTime = System.currentTimeMillis();
		// 비즈니스 로직 실행
		strategy.call(); // 위임
		// 비즈니스 로직 종료
		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime;
		log.info("resultTime={}", resultTime);
	}
}

```

### 템플릿 콜백 패턴

---

> - 스프링에서는 ContextV2 와 같은 방식의 전략 패턴을 템플릿 콜백 패턴이라 한다.
> - 참고로 템플릿 콜백 패턴은 GOF 패턴은 아니고, 스프링 내부에서 이런 방식을 자주 사용하기 때문에, 스프링 안에서만 이렇게 부른다.
> - 스프링에서 이름에 XxxTemplate 가 있다면 템플릿 콜백 패턴으로 만들어져 있다고 유추 가능 !

#### 콜백이란 ?

---

콜백(callback) 혹은 콜애프터 함수(call-after function)은 파라미터로 넘기는 실행 가능한 코드 조각 !

> `ContextV2` 예제의 `Strategy` !

### 언뜻 비슷해 보이는 디자인 패턴들

---

중요한 건 **의도** ! 다!

UML로 봐도 코드로 봐도 비슷비슷해 보여서 이 패턴을 적용할 때의 의도가 중요하다 !

### 이것만으로는 부족하다...

---

아직까지는 원본 코드에 `template.execute(비즈니스_로직)` 처럼 여전히 부가 기능의 코드가 있다 !

## 프록시 패턴과 데코레이터 패턴

---

### 잠시 설명하고 넘어가신 부분 @RequestParam

---

클래스와 다르게 인터페이스는 아래와 같이 명시하지 않으면 스프링이 빈으로 등록할때 해석하지 못하는 경우가 있다고 한다

```java
@RequestMapping // 스프링은 @Controller 또는 @RequestMapping 이 있어야 스프링 컨트롤러로 인식
@ResponseBody
public interface OrderControllerV1 {

	@GetMapping("/v1/request")
	String request(@RequestParam("itemId") String itemId);

	@GetMapping("/v1/no-log")
	String noLog();
}
```

`@RequestParam("itemId")` 이부분이다 !

> 참고로 `@RequestMapiing` 으로 등록을 해야 컴포넌트 스캔의 대상이 되지 않기 때문에 수동 등록이 가능하다!

### 수업 진행중 아래와 같이 어노테이션 설정한 이유

---

```java
@Import(AppV1Config.class) // #2
@SpringBootApplication(scanBasePackages = "hello.proxy.app") //주의 // #1
public class ProxyApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProxyApplication.class, args);
	}

}
```

`#1` 은 해당 패키지만 컴포넌트 스캔의 대상으로 삼겠다는 것이다

![image](https://user-images.githubusercontent.com/66164361/154673017-2f9d02d4-5218-4922-bf8a-92b3735a912c.png)

예를들어 위와 같은 그림에서 `Config`파일을 버전별로 관리하여 현재는 `V1` 버전만 적용하고 싶은 경우 유용하다

그대로 풀스캔을 태우면 `config` 내 모든 버전을 스캔할 것이다

그리고 스캔되지 않은 특정 `Config`파일은 `@Import` 로 별도로 내포한다 !
