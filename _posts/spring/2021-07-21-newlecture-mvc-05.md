---
layout: post
title: "newlecture: spring MVC (5) : 프론트로 출력하기"
subtitle: "..."
date: 2021-07-21 21:30:00 +0900
categories: backend
tags: spring
comments: true
---

#### 문서 출력 방법 4가지
---

- `서블릿 객체`를 얻어서 문자열 출력
- `@ResponseBody` 설정을 통한 문자열 출력
- `ResourceViewResolver`를 이용한 문서 출력
- `TilesViewResolver`를 이용한 문서 출력

#### 서블릿 객체를 얻어서 문자열 출력
---
```java
    @RequestMapping("index")
    public void index(HttpServletResponse response) {
    PrintWriter out = null;
    try {
        out = response.getWriter();
    } catch (IOException e) {
        e.printStackTrace();
    }

    out.println("Welcome");
}
```

#### @ResponseBody를 통한 문자열 출력
---

```java
    @RequestMapping("index")
    @ResponseBody
    public String index(HttpServletResponse response) {
	return "hello";
    }
```

#### @RestController 를 이용한 데이터 출력

---

@RestController은 문서가 아닌 _(RestFull한)_ 데이터를 제공하는 컨트롤러를 만들 때 사용합니다

`controller.api` 패키지 밑에 아래의 컨트롤러를 추가했습니다

```java
@RestController
@RequestMapping("/api/notice/")
public class NoticeController {

    @RequestMapping("list")
    @ResponseBody
    public String list( ) {

	return "notice list 문자열";
    }
}
```

그러면 bean 이름이 기존의 NoticeController와 충돌이 나면서 에러가 발생합니다.

```
(자바) NoticeController noticeController = new NoticeController();
(XML) <bean id="noticeController" class="... .NoticeController" />
```

위의 객체명/id가 키값이 되기 때문에 그런 것으로 보입니다. 이제 아래처럼 수정합니다.

```java
@RestController("apiNoticeController")
```

위처럼 명시적으로 이름을 부여하면 묵시적으로 부여된 기존 객체와 충돌나지 않습니다.

출력 결과는 아래와 같습니다.

![image](https://user-images.githubusercontent.com/66164361/126494221-7406f000-771f-47e1-9e44-2169ff62b8af.png)

한글이 깨졌는데 출력할 때 인코딩을 바꾸어 주면 됩니다.

기존의 `servlet-context.xml` 파일을 열어서

```xml
<mvc:annotation-driven>
```

위 내용을 아래와 같이 수정합니다.

```xml
<mvc:annotation-driven>
		<mvc:message-converters> <!-- @ResponseBody로 문자열 한글 처리 -->
			<bean class="org.springframework.http.converter.StringHttpMessageConverter">
				<property name="supportedMediaTypes">
					<list>
						<value>text/html;charset=UTF-8</value>
					</list>
				</property>
			</bean>
		</mvc:message-converters>
</mvc:annotation-driven>
```

아래와 같이 정상 출력됨을 확인할 수 있습니다.

![image](https://user-images.githubusercontent.com/66164361/126496649-2ff33bf4-f244-4bbe-84a9-e5159690910b.png)

### Json 데이터 출력
---
![image](https://user-images.githubusercontent.com/66164361/126497529-b9aa6e32-1a32-424e-a0f9-4e76a2332aa8.png)

데이터는 크게 위처럼 3가지 형태로 구분합니다.

이때 id, title 등은 데이터를 위한 데이터.. 즉 메타 데이터를 의미합니다

JSON은 본디 js를 위해 만들어진 객체 표기법입니다

그러나 단순하고 편리하여 거의 모든 곳에서 사용되는 표기이기도 합니다

js를 사용하는 클라이언트단에 전송하는 데이터는 JSON이 제격입니다

그 전에 `POM.xml`에 아래의 내용을 꼭 추가해주어야 합니다

```
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.11.2</version>
</dependency>
```

위 내용을 추가하지 않아서..

저는 1시간 가까이 방황했습니다 ㅠㅠ 여러분은 그러지 않길..


아래와 같이 메서드를 작성합니다

```java
    @Autowired
    private NoticeService service;

    @RequestMapping("list")
    public List<Notice> list( ) {

        List<Notice> list = service.getList(1, "TITLE", "");

        return list;
    }
```

위 메서드의 응답은 아래와 같습니다

![image](https://user-images.githubusercontent.com/66164361/126505658-4d430c50-b88b-4383-824e-b2c884ded07e.png)

또한 단일 건은 아래와 같습니다

```java
    @RequestMapping("justOne")
    public Notice justOne( ) {

	Notice jstOne = service.getList(1, "TITLE", "").get(0);

	return jstOne;
    }
```
응답 :

![image](https://user-images.githubusercontent.com/66164361/126506490-3ec225b9-25fc-439e-a5f5-d120695ff7cf.png)
