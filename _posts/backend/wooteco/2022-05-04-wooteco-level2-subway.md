---
layout: post
title: "우테코 레벨2 지하철 미션"
subtitle: "..."
date: 2022-05-04 23:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

베루스와 페어가 되면서 많은 것을 배웠다

# 배운 것들

## RestAssured는 테스트 프레임워크를 Hamcret를 쓴다

> 참고
> https://www.baeldung.com/java-junit-hamcrest-guide

참고로 `Hamcret` 는 `Matchers` 의 애너그램이다 !

## `@DirtiesContext` 는 비용이 비싸다

그것도 매우 비싸다...

이 어노테이션 하나만 없어도 실행속도가 4배는 빨라지는 것을 목격했다

약 25초 -> 6초

> 참고 (4.4를 보면 된다 !)
> https://www.baeldung.com/spring-dirtiescontext

### 지인에게 배운것

> resources 에 있는 JSON 파일을 통해서 Domain 객체 생성
> https://stackoverflow.com/questions/60402338/get-json-file-from-resources-folder

# 알게된 것

## HTTP 상태코드

주소가 없는 것 뿐만 아니라 자원이 존재하지 않으면 `404` 를 반환한다.
https://velog.io/@jch9537/HTTP-%EC%9D%91%EB%8B%B5%EC%BD%94%EB%93%9C-http-%EC%9D%91%EB%8B%B5%EC%BD%94%EB%93%9C%EC%9D%98-%EC%84%A0%ED%83%9D

## ControllerAdvice는 Controller가 아닌 예외를 잡지 못한다...

```java
@RestControllerAdvice(annotations = {Service.class, Repository.class})
```

위와 같은 어노테이션은 Repository에서 발생한 예외를 핸들링해주지 않는다

컨트롤러에서 잡아야 한다 !

## RestAssured에서 객체 변환

- 기본생성자가 있어야 한다
  - 접근제어자가 `private`이어도 된다 !
  - 리플랙션으로 통과하는 것으로 추측됨

## 슈퍼타입 토큰

> https://sungminhong.github.io/spring/superTypeToken/

## RestAssured 를 통한 List of 커스텀 객체 변환

> 이것때문에.. 주말이 날라갔다...ㅎ  
> 아래 내용은 모두 되는 내용이다..
> 복잡한 것부터 심플한 것 순서대로 나열하였다

```java
ParameterizedTypeReference<List<LineResponse>> typeReference = new ParameterizedTypeReference<>() {};
List<LineResponse> responses = (List<LineResponse>) get(LINE).as(typeReference.getType());
```

```java
List<LineResponse> lineResponses = (List<LineResponse>) get(LINE).as(ResolvableType.forInstance(new ArrayList<LineResponse>()).getType());
```

```java
List<LineResponse> lineResponses = response.jsonPath().getList(".", LineResponse.class);
```

## Change Siganture - Default Value

항상 궁금했는데 크루 로마 덕분에 지레짐작할 수 있었다

![image](https://user-images.githubusercontent.com/66164361/169940593-5e454646-1813-4dce-b0da-8ffe9e862311.png)

위처럼 default value 설정을 하면 해당 값으로 셋팅이 된다

![image](https://user-images.githubusercontent.com/66164361/169940880-64918dfc-abd9-4d29-9e53-7f1ce397fc47.png)

만일 설정하지 않으면 비어진 상태로 깨져있게 된다

![image](https://user-images.githubusercontent.com/66164361/169941025-ee6123db-4247-4191-9138-09f6c19a4a6e.png)

## enum 구현 방식을 추상클래스로

꼭 장점만 있지는 않지만

enum + 람다가 아닌 추상클래스 구현일때의 장점은 아래와 같다

- 항목이 많아져도 추가적인 메서드 네이밍을 하지 않아도 된다

- 람다식을 찾기 위해(메서드로 추출되어있다는 전제하에) 선언부/호출부를 들낙거리지 않아도 된다

> https://github.com/woowacourse/atdd-subway-path/pull/296#discussion_r884501518

# 해결되지 않은 의문

## `AliasFor`

라는 어노테이션이 있다

내가 생각한 이 어노테이션의 효능 중 하나는 `value`란 속성에 값을 넣어도 `다른_속성`으로 값이 대입되는 것이라고 생각했었다.

예를들어 `@RequestMapping`에는 `path`와 `value` 라는 속성이 있는데 실제로도 그 두개는 같은 값을 갖는 것으로 보인다

`RequestMappingHandlerMapping` 클래스에 디버깅을 찍어서 확인해볼 수 있다

![image](https://user-images.githubusercontent.com/66164361/167099489-8e6f3394-a22c-4fec-9a64-99cdafb08e67.png)

그러나 내가 만든 어노테이션인 @Peanut에서는 그런 것을 찾아볼 수 없었다

## `HttpServletRequest` 객체의 재생성이 되지 않는다

> 이하 `HttpServletRequest` 을 `request` 라고 하겠다

필자가 예측한 이 객체는 사용자가 서버에 요청을 할 때마다 객체가 생성되는 것으로 알고 있었다

그러나 실제 동작은 그렇지 않았다

> 동작 개요

- 사용자가 순차적으로 요청하고, 다시 재 요청한다
  - 동일한 `request`

마치 쓰레드풀에 담긴 것 마냥.. 재생성되지 않는다 !! 왜그런걸까..

API 툴을 이용해서 디버거를 찍고 테스트해보았지만... 예측한 결

```java
@RequestMapping("/users")
```

라고 쓰면 `value` 에 이 값이 들어가지만 이 값은 `path` 에도 공유될 것이라는 믿음이었다 !

실제로 이 어노테이션을 처리하는 RequestMappingHandler은 `value`와 `path`변수가 공유되고 있었다 !

## 시도해본 것

enum + static method 기반으로 정책 코드를 작성했었다

- 현재는 enum + abstract 메서드 기반으로 변경

```java
public enum DistanceFarePolicy {

    SHORT_DISTANCE_POLICY(DistanceFarePolicy::shortDistanceCondition, DistanceFarePolicy::calculateShortPolicy),
    MIDDLE_DISTANCE_POLICY(DistanceFarePolicy::middleDistanceCondition, DistanceFarePolicy::calculateMiddlePolicy),
    LONG_DISTANCE_POLICY(DistanceFarePolicy::longDistanceCondition, DistanceFarePolicy::calculateLongPolicy);

    private static final int BASIS_FARE = 1_250;
    private static final int FARE_AT_50KM = 2_050;
    private static final int FIRST_FARE_INCREASE_STANDARD = 10;
    private static final int LAST_FARE_INCREASE_STANDARD = 50;
    private static final int FIRST_FARE_INCREASE_STANDARD_UNIT = 5;
    private static final int LAST_FARE_INCREASE_STANDARD_UNIT = 8;
    private static final int INCREASE_RATE = 100;

    private final IntPredicate condition;
    private final IntUnaryOperator calculator;

    DistanceFarePolicy(IntPredicate condition, IntUnaryOperator calculator) {
        this.condition = condition;
        this.calculator = calculator;
    }

    private static boolean shortDistanceCondition(int distance) {
        return distance >= 0 && distance <= FIRST_FARE_INCREASE_STANDARD;
    }

    private static boolean middleDistanceCondition(int distance) {
        return distance > 0 && distance <= LAST_FARE_INCREASE_STANDARD;
    }

    private static boolean longDistanceCondition(int distance) {
        return distance > FIRST_FARE_INCREASE_STANDARD;
    }

    private static int calculateShortPolicy(int distance) {
        return BASIS_FARE;
    }

    private static int calculateMiddlePolicy(int distance) {
        return BASIS_FARE + INCREASE_RATE *
                (int) Math.ceil((double) (distance - FIRST_FARE_INCREASE_STANDARD) / FIRST_FARE_INCREASE_STANDARD_UNIT);
    }

    private static int calculateLongPolicy(int distance) {
        return FARE_AT_50KM + INCREASE_RATE *
                (int) Math.ceil((double) (distance - LAST_FARE_INCREASE_STANDARD) / LAST_FARE_INCREASE_STANDARD_UNIT);
    }

    public IntPredicate condition() {
        return condition;
    }

    public IntUnaryOperator calculator() {
        return calculator;
    }
}
```

```java
public enum AgeFarePolicy {

    CHILD_AGE_POLICY(AgeFarePolicy::childCondition, AgeFarePolicy::calculateChildPolicy),
    YOUTH_AGE_POLICY(AgeFarePolicy::youthCondition, AgeFarePolicy::calculateYouthPolicy),
    ADULT_AGE_POLICY(AgeFarePolicy::adultCondition, AgeFarePolicy::calculateAdultPolicy);

    private final IntPredicate condition;
    private final IntUnaryOperator calculator;

    AgeFarePolicy(IntPredicate condition, IntUnaryOperator calculator) {
        this.condition = condition;
        this.calculator = calculator;
    }

    private static boolean childCondition(int age) {
        return age >= 6 && age < 13;
    }

    private static boolean youthCondition(int age) {
        return age >= 13 && age < 19;
    }

    private static boolean adultCondition(int age) {
        return age >= 19 || (age >= 0 && age < 6);
    }

    private static int calculateChildPolicy(int amount) {
        return (int)((amount -350) * 0.5);
    }

    private static int calculateYouthPolicy(int amount) {
        return (int)((amount -350) * 0.8);
    }

    private static int calculateAdultPolicy(int amount) {
        return amount;
    }

    public IntPredicate condition() {
        return condition;
    }

    public IntUnaryOperator calculator() {
        return calculator;
    }
}
```

# 참고한 사이트

> 모든 클래스 정보 가져오기

- https://stackoverflow.com/questions/40540915/how-to-find-a-file-recursively-in-java

> 인터페이스 및 클래스명 명명규칙
> https://riehle.org/computer-science/programming/conventions/classes.html

> 잭슨 역직렬화 주의점
> https://findmypiece.tistory.com/m/104

> https://stackoverflow.com/questions/15531767/rest-assured-generic-list-deserialization

> `@RequestBody` 에 기본생성자가 필요한 이유
> https://velog.io/@conatuseus/RequestBody%EC%97%90-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EB%8A%94-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%80  
> https://velog.io/@conatuseus/RequestBody%EC%97%90-%EC%99%9C-%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%A0%95%EC%9E%90%EB%8A%94-%ED%95%84%EC%9A%94%ED%95%98%EA%B3%A0-Setter%EB%8A%94-%ED%95%84%EC%9A%94-%EC%97%86%EC%9D%84%EA%B9%8C-2-ejk5siejhh

> 변수 이름  
> https://chronic794.blogspot.com/2021/02/blog-post_22.html

> 옵셔널 바르게 쓰기  
> https://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/
