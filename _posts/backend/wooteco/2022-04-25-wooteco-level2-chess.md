---
layout: post
title: "우테코 레벨2 - 체스"
subtitle: "..."
date: 2022-04-25 5:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

현재 시각 새벽 5시 ... 체스로 DB 전환이 이렇게 힘들었을 줄은.. 생각지도 못하였다..

난 얼마 안 걸릴 줄 알았으나.. 오산이었다.. ^^;

## 깨달은 것들

> JQuery의 Ajax 요청이 생각보다 까다롭다

일반적인 js의 호출과 다르게 jquery 문법은 다소 다른게 있다

예를들어

- `dataType` : `Accept` 이다. (서버 -> 클라이언트)
- `contentType` : `Content-Type` 이다 (클라이언트 -> 서버)

그리고 `dataType`의 옵션 값으로

```js
dataType: "json";
```

이 들어가는데

contentType은

```js
contentType: "application/json";
```

이렇게 들어간다 (...)

또한 서버로 json을 보낼 때는

```js
data: JSON.stringify(gameData);
```

위 처럼 보내주어야 한다 (...)

즉 .. 정리하면 아래와 같다

```js
const gameData = {
  currentTeam: $("#current-team").text(),
  pieces: getPieces(),
};

$.ajax({
  url: "/save-game",
  method: "POST",
  data: JSON.stringify(gameData),
  contentType: "application/json",
})
  .done(function (data) {
    alert("저장 완료 !");
  })
  .fail(function (xhr, status, errorThrown) {
    console.log(xhr);
  });
```

그리고 서버에서 받을떄는.. 놀랍게도 사람들이 아는 것과는 다르게 `Setter`가 필요하다..

분명 리플렉션 등의 `Util`이 있기 때문에 필요 없었던 것으로 아는데.. 필요하다 !!

### Exception Api 와 Interceptor 호출 순서

ExceptionResolver -> Interceptor.afterCom~ 순으로 실행이 된다.
즉 앞에서 예외를 잡아버리면 인터셉터에서 Exception Param 은 null 이 된다

안그러면 `null` 값이 들어온다... ㅠ

### Session Store

쓰레드 로컬은 현재 쓰레드에만 해당되는 scope로 bean이 관리된다!!
즉, request 스코프에 대해서만 해당되므로 이 스코프가 아닌 웹 브라우저를 닫기 전의 Session 스코프가 적용되어 있어야 한다 !

그러나 스프링에서 관리하는 세션 스코프에도 한계가 있었다...

스프링에서 세션을 관리하다보니 이것의 초기화를 찾기 못해서 (즉 해당 세션에 대한 Bean을 강제 초기화해야한다)
그러나 이것을 하지 못해서 자바에서 강제로 SessionMap을 구현해서 이 방식으로 해당 세션에 대해 재 요청이 있을시 강제로 초기화하는 방법을 선택하였다!

```java
@Repository
public class SessionToChessRepository {

    private static final int SESSION_REMOVE_MINUTE = 1;
    private Map<HttpSession, ChessBoard> sessionToChessBoard = new ConcurrentHashMap<>();

    public void add(HttpSession session, ChessBoard chessBoard) {
        sessionToChessBoard.put(session, chessBoard);
    }


    public ChessBoard get(HttpSession session) {
        return sessionToChessBoard.get(session);
    }

    @Scheduled(cron = "*/10 * * * * *")
    public void checkAndRemoveSessions() {
        for (HttpSession session : sessionToChessBoard.keySet()) {
            checkAndRemoveSession(session);
        }
    }

    private void checkAndRemoveSession(HttpSession session) {
        long currentTimeMillis = System.currentTimeMillis();
        long lastAccessedTime = session.getLastAccessedTime();

        long elapsedMinutes = Duration.ofMillis(currentTimeMillis - lastAccessedTime).toMinutes();

        if (SESSION_REMOVE_MINUTE == elapsedMinutes) {
            sessionToChessBoard.remove(session);
            System.out.println("remove it " + session);
        }
    }
}
```

그러나 위 스케쥴링 기능은 Spring Boot에서 지원하는 글로벌 설정으로 바꾸게 된다.

```yml
server:
  servlet:
    session:
      timeout: 600
```

### 실험

---

1. setter를 넣는다

![image](https://user-images.githubusercontent.com/66164361/164994277-844adb0b-f696-4184-86cc-f20902aa9e27.png)

2. 호출한다

![image](https://user-images.githubusercontent.com/66164361/164994303-0812a416-757b-4be1-bff2-a057ce99839b.png)

3. 결과

![image](https://user-images.githubusercontent.com/66164361/164994310-552deb50-56b1-420c-85c7-4624afd6f1eb.png)

... 소름이다 Setter를 호출한다...

## 그 외

---

jQuery에서 method `POST` 지정하고 `contentType` 을 지정하지 않으면 기본적으로 `x-xxx` 로 날라가더라구요..
`application/json` 으로 보내야 하는데...

---

본래 의도 ! gameId 를 검색해서 가져오려고 했었다! 그러나...
계속 클라이언트가 favicon.ico를 호출한다 !

@GetMapping("/{gameId}")
public 메서드명 (@PathVariable String gameId) {

}

> localhost:8080/favicon.ico
> 이게 브라우저에 표시되는 아이콘

@GetMapping("/game-id/{gameId}")

---

queryForObject 이친구..

없거나 2개 이상이면 예외 터트림..

하나가 아니면 예외 터트림...!!

---

js 비동기를 몰라서

오찌/스컬이 30분 가까이 트러블 슈팅...

비동기...

---

`jdbcTemplate`과 관련해서

`queryList()` 는 컬럼이 하나인 곳에서만 사용할 수 있는 리스트 자료형이다

`query()` + `rowMapper` 를 사용해야 한다

### 그외 들은 것들

> 제이슨이 알려주었던 것

아래와 같은 TEST FW가 있다

restAssured..
web-test-client
test-rest-template...
mock-mvc...

이런 류의 테스트는 건 옵션 사항이라고 한다 !

connection 확인 및 안 끊기게 끔 요청을 할 떄 EXIST 쿼리를 사용한다 !

## 참고한 것들

> getter를 사용해야 `HttpMessageConverter` 가 변환이 가능한 것으로 보인다  
> https://thinkground.studio/httpmessagenotwritableexception-no-converter-for-vo-class-with-preset-content-type-null/

> string to boolean in js  
> https://stackoverflow.com/questions/263965/how-can-i-convert-a-string-to-boolean-in-javascript

> `query` vs `queryForList`  
> https://stackoverflow.com/questions/33008205/incorrectresultsetcolumncountexception-incorrect-column-count-expected-1-actu

> `try-with-resources`  
> https://hyoj.github.io/blog/java/basic/jdbc/jdbc-try-catch-tip.html#java-7-%EB%B6%80%ED%84%B0%EB%8A%94-try-with-resources-%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%9E%90

> HTTP 설계
> https://kyleyj.tistory.com/46

> 혜원님의 스프링 고분분투 테스트 ~!
> https://hyewoncc.github.io/2022/04/25/spring-study-1.html

> AJAX Content Type In JQuery  
> https://mchch.tistory.com/181

> JQuery 공식문서  
> https://api.jquery.com/jquery.ajax/

> JQuery 옵션들  
> https://cofs.tistory.com/404

> https://stackoverflow.com/questions/2253453/how-should-i-name-a-java-util-map
> map 이름 짓기
> `keyToValue` 처럼 읽는 순서대로 짓는 것이 좋다는 의견

> https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers
> HTTP URL 길이

> https://rastalion.me/mysql-8-0-1-%EB%B2%84%EC%A0%84%EB%B6%80%ED%84%B0-%EC%B1%84%ED%83%9D%EB%90%9C-utf8mb4_0900_ai_ci%EC%9D%98-%ED%95%9C%EA%B8%80-%EC%82%AC%EC%9A%A9%EC%97%90-%EB%8C%80%ED%95%9C-%EB%AC%B8%EC%A0%9C%EC%A0%90/
> utf8mb4_0900_ai_ci의 한글 사용에 대한 문제점
