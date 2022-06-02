---
layout: post
title: "우테코 레벨2 API 예외처리 핸들링"
subtitle: "..."
date: 2022-05-30 15:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 예외처리 과정 중에 정리하게 된 내용이 길기 때문에 아래에 적습니다... ㅠ

# ExceptionResolver 처리하기...

> `DefaultHandlerExceptionResolver` 와 `ExceptionHandlerExceptionResolver` 사이의 어딘가에...

문제의 발단... 분명히 `405` 에러가 나야할 것이 `500` 에러로 클라이언트에 던져진 것... 왜그런가 했더니

ExceptionHandler 에서 모두다 500으로 분기처리하는 것이었다

> 문제의 코드  
> ![image](https://user-images.githubusercontent.com/66164361/169856695-eacdff93-6715-47d3-a787-e45b78d4ff1f.png)

![image](https://user-images.githubusercontent.com/66164361/169856546-a81caf52-f2fd-4e39-82d7-6aa6ebb401fd.png)

만일 기존의 Exception 객체를 파라미터로 받는 메서드를 주석처리하면 아래와 같이 API 응답을 얻게 된다

![image](https://user-images.githubusercontent.com/66164361/169853559-5308b7d2-d231-49cd-8552-3803cc265528.png)

응답 코드 자체는 원하는 것이지만 메세지가 고객에게 던져주기에는 굉장히 개발자 친화적이다...

`DefaultHandlerExceptionResolver` 로 발생한 에러를 `ExceptionHandlerExceptionResolver`를 통해 처리할 수 없는 것일까...

힌트를 알아보기 위해... 우선 `DefaultHandlerExceptionResolver`를 상속받아서 사용해보기로 하였다

> 우선 상속을 받는다  
> ![image](https://user-images.githubusercontent.com/66164361/169917653-5164d6e0-8e18-437b-af6d-d761908cbff9.png)

> 이곳이 response의 상태코드가 `200`에서 `405`로 바뀌는 순간이다
> ![image](https://user-images.githubusercontent.com/66164361/169917664-63d865b2-6f11-4c8c-b0ae-d804f55ae30a.png)

> `DefaultHandlerExceptionResolver`의 아래 메서드로 이동한다
> ![image](https://user-images.githubusercontent.com/66164361/169917907-92e02f98-7778-4d02-8ad1-d3e80987ae30.png)  
> ![image](https://user-images.githubusercontent.com/66164361/169918038-d1dbf5a9-23bf-46c9-8427-27dafae43865.png)

> `ResponseFacade`  
> ![image](https://user-images.githubusercontent.com/66164361/169918238-266bc410-2917-4610-9ff7-45ede77481ab.png)

> `catalina.connector.Response`  
> ![image](https://user-images.githubusercontent.com/66164361/169918241-1be155da-7d22-46ad-a4ff-b1766ef2de6f.png)

> `coyote.Response`  
> ![image](https://user-images.githubusercontent.com/66164361/169918472-a888e1db-833b-4c6b-b655-200b2cb88751.png)

문제는 위 과정을 통하게 되면 이후에 어떤 처리를 하든  
앞서 실행된 `DefaultHandlerExceptionResolver.doResolveException` 메서드에 의해  
예외 메세지가 고정되어서 나간다.  
즉 코드값은 원하는대로 나가지만 전체적인 포맷이 우리가 원하던 바와 다르게 나간다..

## 해결아닌 해결...

결국 개발자에게 가장 금기시 되는 정성의 노가다 기법으로 해결하였다...

> ![image](https://user-images.githubusercontent.com/66164361/169920446-e74676d8-2619-44ba-8c02-96ffb56a3183.png)  
> ![image](https://user-images.githubusercontent.com/66164361/169920468-5324bcc4-9de5-4b8f-8652-683b8aaa5029.png)

[스택오버플로의 답변](https://stackoverflow.com/questions/29193190/can-a-generic-exceptionhandler-and-defaulthandlerexceptionresolver-play-nice)을 참고해서 아래처럼 작성해보아도... 원하는 형태의 예외메시지가 담기지는 않는다

![image](https://user-images.githubusercontent.com/66164361/170932155-622db3ae-6bc5-4d35-a8ec-1cd47e840db0.png)

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    ...
}
```

# 마침내 해결되었습니다...

## 요약

> `ResponseEntityExceptionHandler` 클래스의 `protected handleExceptionInternal` 메서드를 상속합니다 ... ㅎㅎ

![image](https://user-images.githubusercontent.com/66164361/170935752-dc35e060-ffb3-4810-bfb6-06db4c2a6f51.png)

> 에러 응답 성공  
> ![image](https://user-images.githubusercontent.com/66164361/170935782-8be4cab0-bd91-4b47-9dcc-d0032d887cc7.png)

## 동작 원리

`ResponseEntityExceptionHandler` 클래스는 내부적으로 `handleException`란 메서드를 가지고 있는데

이 메서드는 30개 정도 되는 분기처리에 의해 (아래 그림 참고) 어느 한 메서드를 호출하게 되고

![image](https://user-images.githubusercontent.com/66164361/170936097-76cc38f4-d113-444f-9893-ff3be0097b61.png)

결국 아래의 메서드를 호출합니다

![image](https://user-images.githubusercontent.com/66164361/170936171-c6e475fb-de1c-4928-b303-591975dcad70.png)

이제 스프링 내부의 분기처리 때문에 핸들링해주었던 아래 코드들은 필요 없게 되었네요...ㅎㅎ

![image](https://user-images.githubusercontent.com/66164361/170936605-de9d8c14-33e4-44c7-9880-c42fda624f5d.png)

## `Ambiguous @ExceptionHandler method mapped` 에러

`ResponseEntityExceptionHandler` 를 상속받을 때 발생한 에러인데

`ControllerAdvice` 와 `ResponseEntityExceptionHandler`에서 각각 선언한 `@ExceptionHandler` 가 겹쳐서 발생한 에러이다.

겹치는 부분은 따로 **오버라이드**해서 사용하면 된다.

예를들어 `MethodArgumentNotValidException`를 처리하는 부분이 그럴 수 있다

## 참고

> ![image](https://user-images.githubusercontent.com/66164361/169860359-306a296f-5ca3-4b9b-b87e-bfa11d0ef152.png)  
> 참고로 저렇게 하면 `request`, `response` 객체를 받아올 수 있다...

## 참고 URL

> 스프링 기본 예외처리를 거치게 하고 싶은 경우  
> https://stackoverflow.com/questions/29193190/can-a-generic-exceptionhandler-and-defaulthandlerexceptionresolver-play-nice  
> https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers  
> https://docs.spring.io/spring-framework/docs/5.3.20/javadoc-api/org/springframework/web/servlet/mvc/support/DefaultHandlerExceptionResolver.html  
> `스프링에서 제공하는 예외를 커스텀하기`  
> https://docs.spring.io/spring-framework/docs/5.3.20/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/ResponseEntityExceptionHandler.html

> `Ambiguous @ExceptionHandler method mapped` 에러  
> https://stackoverflow.com/questions/51991992/getting-ambiguous-exceptionhandler-method-mapped-for-methodargumentnotvalidexce  
> https://velog.io/@litsynp/Getting-Ambiguous-ExceptionHandler-method-mapped-for-XXXException
