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

# 해결되지 않은 의문

---

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

실제로 이 어노테이션을 처리하는 RequestMappingHandler

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
