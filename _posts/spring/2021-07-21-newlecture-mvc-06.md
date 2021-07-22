---
layout: post
title: "newlecture: spring MVC (6) : 프론트로부터 입력받기"
subtitle: "..."
date: 2021-07-21 21:40:00 +0900
categories: backend
tags: spring
comments: true
---

### Controller로 입력받기

---

브라우저에서 전달받을 수 있는 데이터 형태는 다음과 같습니다.

- `QueryString` URL로 전달되는 문자열
- `POST` 사용자의 입력으로 전달되는 문자열 (Multipart도 포함)
- `Path` 경로로 전달되는 문자열
- `Cookie` 브라우저에 보관하고 있던 문자열
- `Header` 요청 헤더로 전달되는 문자열

### QueryString 입력 (GET 요청)

---

URI로 `/list?p=1` 을 요청해서 받아보겠습니다

java 코드입니다.

```java
    @RequestMapping("list")
    public String list(HttpServletRequest request) {
      String p = request.getParameter("p");
      System.out.println(">>>>>>>" + p);
      ...
    }
```

혹은 아래와 같이 작성해도 됩니다

```java
    public String list(String p) {
       ...
    }
```

굉장히 심플합니다 이게 가능한 이유는

Front Controller가 호출하는 컨트롤러가 어떤 매개인자, 반환타입 지니는지 체크 하기 때문입니다

![image](https://user-images.githubusercontent.com/66164361/126517277-67a5b9f2-70ba-4452-b453-aa4aa5ac9977.png)

HTTP GET요청은 클라이언트의 종류에 따라 다르지만

IE의 경우는 2KB가 MAX라 합니다.

URL 요청 받을 때는 p 등의 짧은 문자로 받고

제약이 없는 java에서는 page라는 의미있는 긴 문자로 받으면 좋겠죠?

```java
public String list(@RequestParam("p") String page) {...}
```

짠! 위와 같이 받을 수 있습니다

다만 `list` 등 처음 요청시엔 p값이 없어서 예외가 발생합니다

```java
public String list(@RequestParam(name="p", defaultValue="1") int page) {...}
```

java에서는 제공하지 않지만 스프링에서 위와 같은 옵션을 제공해 줍니다

참고로 매개값은 모두 문자열이지만

스프링은 int로 바꿔도 동작합니다 ! (갓프링!!)

구글에 "requestParam spring io" 라고 검색하면 아래의 page가 상단에 옵니다

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html

그러면 우리가 아는 인자 말고도 추가로 2개의 인자가 더 있음을 알 수 있습니다

- name
- defaultValue
- required
- value

required는 필수여부라 받아들이면 될 것 같습니다 기본값은 true입니다

false일 경우 GET요청시 인자가 없어도 null이 들어옵니다

대신 int 등의 Primitive Type 변수는 못받겠죠?

```java
@RequestParam(name="p", required = false) Integer page
```

value 속성은 name과 같습니다

> 추가로...
> @RequestMapping("문자열1")에 있는 문자열1은 패턴 문자열입니다
> 예를들어 문자열1은 reg이고 reg.html이란 요청이 올 경우
> 패턴에 일치하기 때문에 요청을 가로채서 정적컨텐츠를 제공하지 않습니다

### POST 요청

---

GET과 별반 다르지 않습니다. 아래와 같은 등록 화면이 있다고 가정을 해봅니다

```html
<form action="reg" method="post">
  ...

  <input type="text" name="title" />
  ...
  <textarea class="content" name="content"></textarea>
</form>
```

응답은 아래와 같이 합니다

```java
@RequestMapping("reg")
    @ResponseBody
    public String reg(String title, String content) {

	return String.format("title : %s <br>  content : %s <br>", title, content);
}
```

역시 잘 작동합니다

#### select box

카테고리도 추가해 봅니다.

```html
<th>카테고리</th>
<td class="text-align-left text-indent text-strong text-orange" colspan="3">
  <select name="category">
    <option>카테고리1</option>
    <option>카테고리2</option>
    <option>카테고리3</option>
    <option>카테고리4</option>
  </select>
</td>
```

문제는 위와 같이 하면.. 한글값이 넘어가기 때문에

```html
<option value="1">카테고리1</option>
```

값을 넘기게끔 바꾸어 줍니다.


#### checkbox
```html
<tr>
    <th>좋아하는 등 운동 : select box</th>
    <td class="text-align-left text-indent text-strong text-orange" colspan="3">
        <input type="checkbox" name="back-exercises" value="1" id="ch1"><label for="ch1">풀업(턱걸이)</label>
        <input type="checkbox" name="back-exercises" value="2" id="ch2"><label for="ch2">랫 풀 다운</label>
        <input type="checkbox" name="back-exercises" value="3" id="ch3"><label for="ch3">원 암 덤벨 로우</label>
        <input type="checkbox" name="back-exercises" value="4" id="ch4"><label for="ch4">케이블 로우</label>
        <input type="checkbox" name="back-exercises" value="5" id="ch5"><label for="ch5">인버티드 로우</label>
        <input type="checkbox" name="back-exercises" value="6" id="ch6"><label for="ch6">시티드 로우</label>
    </td>
</tr>
```

checkbox, radio 모두 name태그로 그룹임을 식별합니다

``` java
public String reg(... @RequestParam("back-exercises") String[] backExercises) {
    for (String backExercise : backExercises) {
        System.out.println(backExercise);
    }
}
```

#### radio box
```
<tr>
    <th>좋아하는 등 운동 : select box</th>
    <td class="text-align-left text-indent text-strong text-orange" colspan="3">
        <input type="radio" name="rd-back-exercise" value="1" id="rd1"><label for="rd1">풀업(턱걸이)</label>
        <input type="radio" name="rd-back-exercise" value="2" id="rd2"><label for="rd2">랫 풀 다운</label>
        <input type="radio" name="rd-back-exercise" value="3" id="rd3"><label for="rd3">원 암 덤벨 로우</label>
        <input type="radio" name="rd-back-exercise" value="4" id="rd4"><label for="rd4">케이블 로우</label>
    </td>
</tr>
```

```java
public String reg( .. @RequestParam("rd-back-exercise") String rdBackExercise) {..}
```

### 한글 설정
---
![image](https://user-images.githubusercontent.com/66164361/126653164-e28d5ab6-eafa-432c-93fe-9bf1c506f679.png)
브라우저는 UTF-8로 설정돼 있는 반면 톰캣의 내장 인코딩으로 ISO-8859-1 를 사용합니다

다음과 같은 방법들을 생각해볼 수 있습니다.

#### 톰캣 인코딩 설정
```
<Connector port="8080"
           protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443"
           URIEncoding="UTF-8" />
```
단 이 방법은 톰캣 전역 설정이기에.. 해당 톰캣을 사용하는 다른 프로젝트에 영향을 미칩니다

#### 서블릿 인코딩 설정
```java
request.setCharacterEncoding("UTF-8");
```
이 방법 또한 모든 서블릿마다 설정해주어야 하기 때문에 비효율적입니다.

#### 필터 인코딩 설정
한 프로젝트안에서만 유효하며 모든 서블릿마다 설정해줄 필요 없습니다
요청/등답이 있을 때마다 실행됩니다
