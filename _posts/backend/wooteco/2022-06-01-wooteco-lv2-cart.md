---
layout: post
title: "우테코 레벨2 장바구니 미션"
subtitle: "..."
date: 2022-06-01 11:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

# 알게된 것

## AOP, 인터셉터를 적용하는 장소

![image](https://user-images.githubusercontent.com/66164361/171313328-45799380-4071-43fc-8a17-a0a0d4dbfb5f.png)

## 테스트 코드에 Nested 사용시...

`@MethodSource` 를 적용할 수 없다

inner class의 경우 static 메서드를 통해서 argument를 불러올수없다.. ㅠㅠ

그렇다고 밖에 있어도 접근이 안됀다.

## h2 db의 로컬 설정과 서버 설정

영한님 강의를 듣다보면 h2 설정을 한 번 한 이후로

`jdbc:h2:tcp` 로 종종 url을 바꾸는 것을 확인할 수 있다.

이렇게 해야 실제로 DB를 독립적으로 설치해서 여러 곳에서 접근할 수 있게 하기 위함으로 보인다

> 참고  
> https://atoz-develop.tistory.com/entry/H2-Database-%EC%84%A4%EC%B9%98-%EC%84%9C%EB%B2%84-%EC%8B%A4%ED%96%89-%EC%A0%91%EC%86%8D-%EB%B0%A9%EB%B2%95  
> https://lob-dev.tistory.com/entry/H2%EC%9D%98-LocalIn-Memory-%EC%99%80-ServerTCP-%EB%AA%A8%EB%93%9C

## 삭제 API 설계: DELETE vs POST

`DELETE` 는 기본적으로 `HTTP body`를 담아서 보내지 않게끔 설계되어있다.

특정 톰캣 버전, 스프링 버전에서는 아예 허용을 하지 않는 경우도 있다.

따라서 정보를 삭제할 때 body가 필요한 상황이면 POST 등의 메서드를 활용하자.

예로,

```
POST /api/users/myaccount/delete
```

처럼 설계하는 것이다.

자원 표현 방식의 controller 모델은 동사를 허용한다.

> 참고 kakao 도 post로 삭제한다!  
> ![image](https://user-images.githubusercontent.com/66164361/173305731-bddefffe-0705-4e5c-8352-9849953e7c0b.png)

> 참고  
> https://humblego.tistory.com/18  
> https://server-engineer.tistory.com/886  
> https://restfulapi.net/resource-naming/

> 카카오 API Reference  
> https://developers.kakao.com/docs/latest/ko/reference/rest-api-reference  
> 구굴 API Reference  
> https://developers.google.com/drive/api/v3/reference/permissions#resource

## JUnit와 SpringBootTest

JUnit이 생성자 DI를 먼저 사용하려기 떄문에

Spring의 생성자 DI를 받으려는 경우 문제가 생기게 된다, 따라서

Spring 으로 DI 받으려면

필드 주입을 받으면 된다

> 참고  
> https://pinokio0702.tistory.com/189?category=414017

## 헷갈리는 용어 정리

99 ~ 2014) Entity
2014 이후) Representation (메타 데이터 + 데이터)

# 참고 URL

> `NoClassDefFoundError`  
> https://yangbox.tistory.com/117  
> https://stackoverflow.com/questions/34413/why-am-i-getting-a-noclassdeffounderror-in-java

> `Origin`  
> https://velog.io/@kmdngmn/Spring-Postman-Rest-API-CORS-%ED%85%8C%EC%8A%A4%ED%8A%B8  
> https://www.baeldung.com/rest-assured-tutorial

> `RestAssured`  
> https://www.baeldung.com/rest-assured-tutorial

> `nohup`  
> https://jangseongwoo.github.io/linux/linux_signal/

> 영한님 DTO 답변  
>  https://www.inflearn.com/questions/33616  
> https://www.inflearn.com/questions/139564

> 영한님의 PUT, PATCH 답변  
> https://www.inflearn.com/questions/110644

> DTO에서의 eq & hs 오버라이딩  
> https://stackoverflow.com/questions/7252731/overriding-equals-method-in-dtos

> VM 옵션 넣기 - jojoldu  
> https://jojoldu.tistory.com/547

> CORS  
> https://stackoverflow.com/questions/42588692/testing-cors-in-springboottest

> https://stackoverflow.com/questions/299628/is-an-entity-body-allowed-for-an-http-delete-request

> **용어의 범위**  
> https://d2.naver.com/news/3435170

> google api reference  
> https://developers.google.com/drive/api/v3/reference
