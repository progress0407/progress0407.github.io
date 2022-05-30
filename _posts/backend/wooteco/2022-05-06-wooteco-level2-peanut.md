---
layout: post
title: "우테코 레벨2 땅콩 박스 만들기"
subtitle: "..."
date: 2022-05-06 00:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 지하철 미션 도중 만들어본 땅콩 박스 만들기를 기록한다 !

## 소스 코드 위치

> https://github.com/progress0407/atdd-subway-map/tree/di_alone > `step1` 브랜치가 아닌 `di_alone` 브랜치에 있어요!

![image](https://user-images.githubusercontent.com/66164361/167452991-cc526468-1ca2-4e00-93c9-ba87c645aef0.png)

## 만들게 된 계기

우스갯 소리이긴 하였지만... 레벨2 강의를 듣는 도중..

지하철 미션 1 구현 도중 IoC 컨테이너가 없으면 어떤 점에서 불편할 지 경험해보란 뜻에서 `Spring Container` 사용을 금지하였다 (`@Autowired 등`)

그래서... 직딩일 때 종종 필자를 괴롭혀왔던 이 IoC에 대해 한 번 만들어보기로 생각했다 !

# 구경하기 `^-^`

### 맛 좋은 땅콩 등록하기

<img src="https://user-images.githubusercontent.com/66164361/167449570-b8f6caa9-9960-4d93-aeb9-1896f443c8f2.png" width="60%">

### 땅콩 로켓 배달 받기 !

<img src="https://user-images.githubusercontent.com/66164361/167449890-12d5a449-6260-4844-a443-fb642f93b25f.png" width="60%">

### 땅콩 어노테이션

<img src="https://user-images.githubusercontent.com/66164361/167449158-337c15e3-85d2-473f-b210-dedcb733abf5.png" width="60%">

### 땅콩의 유통 기한

<img src="https://user-images.githubusercontent.com/66164361/167451140-7003db40-d914-4314-b716-d423956dedbe.png" width="80%">

### 테스트 코드 작성

<img src="https://user-images.githubusercontent.com/66164361/167451597-8a169770-51cd-4333-8f75-9ae2bad71d4a.png" width="80%">

### 땅콩 박스의 생김새

> 스킵 추천 !

<img src="https://user-images.githubusercontent.com/66164361/167459481-bb330138-893a-4cd1-bf15-16682a773f00.png" width="75%">

# 전체적인 흐름

- Spring IoC가 로딩된다

- 빈이 초기화 된 이후 `ApplicationListener` 구현체의 메서드 `onApplicationEvent` 가 오직 한 번 실행됩니다

  - 이곳에서 `땅콩 박스`를 실행시킵니다
  - 이 떄 이곳에서 모든 클래스파일을 스캔하여 `@Peanut` 어노테이션이 있는 클래스를 유통기한과 함께 등록합니다

- 땅콩 박스는 자체적으로 싱글톤인데... 생성과정이 다소 복잡하므로 `@Configuration` 을 통해 쉽게 Bean 주입받을 수 있도록 등록합니다.

- 땅콩이 필요한 곳에서는 `@GiveMePeanut` 어노테이션을 작성하면  
  앞서 동작했던 `ApplicationListener` 구현체의 `onApplicationEvent` 메서드에 의해 땅콩박스가 `run`할 떄 땅콩을 배달해줍니다
  - 해당 어노테이션 뿐만 아니라 `peanutContext.getPeanut` 을 통해서도 주입이 가능합니다
  - 땅콩이 존재하지 않으면 `NoSuchPeanutDefinitionException` (땅콩의 정의를 찾을 수 없음 예외)가 발생합니다

# 현재 구현된 기능

## 콩(`Bean`)과 땅콩(`Peanut`)의 차이

| 항목                      | 스프링 콩                                                | 땅콩                        |
| ------------------------- | -------------------------------------------------------- | --------------------------- |
| 객체 저장소               | 빈(Bean) 컨테이너                                        | PeanutContext               |
| 빈을 자동 등록 어노테이션 | `@Component`                                             | `@Peanut`                   |
| 빈을 주입하는 기능        | `T annotationConfigApplicationContext.getBean(Class<T>)` | `T peanutContext(Class<T>)` |
| 빈을 주입받는 어노테이션  | `@Autowired`                                             | `@GiveMePeanut`             |
| 객체 생애주기             | `@Scope`                                                 | `@Peanut(scope)`            |

## 땅콩이 지원하는 가능한 생애주기

| 항목      | 설명                                           |
| --------- | ---------------------------------------------- |
| singleton | 애플리케이션의 종료시점까지 단일한 객체를 가짐 |
| prototype | 항상 객체를 생성                               |
| request   | HTTP Request 의존하는 생애주기                 |
| session   | 브라우저 등의 세션에 의존하는 생애주기         |

# 아쉬운 점...

- 오로지 필드 주입만 가능

  - 생성자 주입 불가
  - 수정자 주입 부락

- 아직도 의존하는 `ApplicationContext`

  - 정작 기존의 Bean Container 를 대체하고자 했는데.. 기존의 것을 사용한다 !

- 복잡한 의존관계를 가지는 객체를 주입

- 예를 들어 A 가 B1, B2 의존, B1 이 C의존할 때 주입할 수 없다 !

  > 예시의 상황  
  > <img src="https://user-images.githubusercontent.com/32123302/165883364-64c3b6fa-de07-4c3d-95ef-db834afc96c6.png" width="30%">

- 이미 스프링 빈이 관리하는 클래스에만 등록할 수 있다

  - 예) 컨트롤러 !
  - 사실 `땅콩 박스` 는 명시적으로만 스프링 컨테이너에 의존을 하지 않을 뿐 의미상으로는 의존하고 있다...

- `request` 스코프 별도 처리
  - 알고 있던 내용과 다르게 실상 컨트롤러로 넘어오는 request객체는 거의 매번 같은 객체가 도착해서 별도 처리를 해주었다

본래라면 PenutLifeCycle 클래스의 enum 분기문은 아래처럼 처리되었을 것이다

```java
REQUEST((request, previousRequest) -> request != previousRequest) // 이전과 현재가 다르면 객체에 넣는다 !
```

# 앞으로 구현해보고 싶은 기능

- proxyMode
  - `ObejctProvider<T>` 를 통해서 객체를 꺼내지 않아도 기 등록한 `Bean Scope`로 `LifeCycle`을 관리할 수 있는 기능
  - 구현한다면 스프링에서 `인터페이스` 기반(JKD Dynamic Proxy)과 `상속` 기반 (CGLIB) 을 추상화한 ProxyFactory를 써서 구현할 것으로 생각한다

# 알게 된 점

### 톰캣의 HttpServletRequest 캐싱

Servlet 을 준비하는 페어 루키에게 알게된 내용인데...
`request` 스코프를 지원하기 위해 디버거를 찍었는데 계속 같은 `HttpServletRequest` 객체가 찍히는 것이었다.
응답을 받기 전에 여러번 요청을 주면 그제서야 다른 `HttpServletRequest` 가 찍힌다...

stackoverflow 에서는 알고보니 톰캣이 내부적으로 캐싱등의 최적화를 거친다고 전해 들었다

# 참고 URL

> https://stackoverflow.com/questions/3437897/how-do-i-get-a-class-instance-of-generic-type-t
