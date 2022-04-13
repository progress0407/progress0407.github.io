---
layout: post
title: "우테코 긴급하게 학습하는 Spark"
subtitle: "..."
date: 2022-04-06 13:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

## 이 글을 작성한 이유

긴급하고 신속하게 스파크를 학습 및 정리하기 위해서 작성하였다

따라서 굉장히 짧을 것이다

## 주의사항

템플릿 엔진 뷰 코드가 깨진 것이 있다

루비 + 마크다운 특정 문법 상

깨지는 문법이 있는 것 같다 (주의해서 읽을 것 ~! )

## Spark란??

> 자바 진영에서 사용 가능한 경량 웹 프레임워크다
> 신속하고 빠르게 웹 환경을 구현할 수 있다

## 참고 레퍼런스

> http://sparkjava.com/documentation#exception-mapping  

- Routes
- Request
- StaticFiles
- View And Template


## build.gradle 에 추가할 내용

- 스파크 - 웹 프레임 워크
- HandleBars - 뷰 템플릿 엔진

```js
implementation 'com.sparkjava:spark-core:2.9.3'
implementation 'com.sparkjava:spark-template-handlebars:2.7.1'
```

## 초기 IP Port 주소

> http://localhost:4567

## 따라해보기

---

### 이름을 받아서 Http Message Body에 전달한다

> http://localhost:4567/hello/philz

```java
public static void main(String[] args) {
    get("/hello/:name", (req, res) -> "hello ~! " + req.params(":name"));
}
```


### 쿼리스트링으로 이름을 받아서 전달한다

> http://localhost:4567/hello2?name=philz

```java
get("/hello2", (req, res) -> {
    return "hello ~! " + req.queryParams("name");
});
```

### 여러가지 매개 변수를 입력받는다

```java
get("/hello3", (req, res) -> {
    return "hello ~! " + req.queryParams("name") + ", 나이는 " + req.queryParams("age");
});
```

### 모델, 뷰 전송 & template 가공

```java
post("/members", (req, res) -> {
    Map<String, Object> model = new HashMap<>();
    model.put("name", req.queryParams("name"));
    model.put("age", req.queryParams("age"));

    return render(model, "result.html");
});
```

```html
<h1>회원 가입 결과</h1>
이름 : {{name}}
<br />
<br />
나이 : {{age}}
```

### 뷰에 도메인 객체 전송

```java
post("/members", (req, res) -> {
    User user = new User(req.queryParams("name"), req.queryParams("age"));
    Map<String, Object> model = new HashMap<>();
    model.put("user", user);

    return render(model, "result.html");
});
```

```html
<h1>회원 가입 결과</h1>
이름 : {{user.name}}
<br />
<br />
나이 : {{user.age}}
```

> 혹은

```html
{{#user}}
    <h1>회원 가입 결과</h1>
    이름 : {{name}}
    <br />
    <br />
    나이 : {{age}}
{{/user}}
```

### 뷰에 도메인 리스트 전송

```java
List<User> users = new ArrayList<>();

post("/members", (req, res) -> {
    User user = new User(req.queryParams("name"), req.queryParams("age"));
    users.add(user);

    Map<String, Object> model = new HashMap<>();
    model.put("users", users);

    return render(model, "result.html");
});
```

```html
<h1>회원 가입 결과</h1>
{{#users}}
이름 : {{name}}
<br />
나이 : {{age}}
<br />
<br />
{{/users}}
```

> 참고로 데이터는 `this` 라는 임시 변수에 저장됨  
> `this.name` 이라고 치환해도 작동하는 것을 확일 할 수 있음
