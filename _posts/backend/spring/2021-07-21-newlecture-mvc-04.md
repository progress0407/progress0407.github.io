---
layout: post
title: "newlecture: spring MVC (4) : XML to Annotaion"
subtitle: "..."
date: 2021-07-21 21:20:00 +0900
categories: backend
tags: spring
comments: true
---

### XML을 Annotaion 으로

---

```xml
<beans ...
	xmlns:context="http://www.springframework.org/schema/context"
    ...
    xsi:schemaLocation="...
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
">

	<context:annotation-config />

    ...
```

위 문구를 추가하고 property 태그를 지운다

그러면 바인딩이 된다

서비스 객체 자체를 IoC 컨테이너에 관리를 하게끔 등록을 하려면

아래의 xml을 등록한다

```xml
<context:component-scan base-package="com.newlecture.web.service" />
```

위 태그는 `<context:annotation-config />`의 기능을 포함하므로 이전 태그는 지우도록 하자

다만 스캔 범위가 너무 많아지면 많은 탐색시간을 갖게 된다..
controller 와 service만 필요하다면 필요한 범위만 스캔하게끔 하자

Controller도 똑같이 하자
다만 컨트롤러는 urlMapping을 하기 위해서 아래와 같은 태그를 넣어야 한다

```xml
<mvc:annotation-driven />
```

이제 코드를 바꿔보자

```java
public class ListController implements Controller {
	...
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		...
        	return mv;
    }
}
```

위 코드는

```java
@Controller
public class ListController {

    ...

    @RequestMapping("/notice/list")
    public void 메서드명() {
		...
    }

}

```

이렇게 바꿀 수 있다.

이전에는 오버라이드된 handleRequest 메서드 하나에 하나의 URL만 매핑할 수 있기 때문에 Controller 당 하나의 URL만 매핑할 수 있었는데

이제는 여러개의 메서드를 가질 수 있으므로 여러 URL 매핑이 가능하다

다시 기존 컨트롤러들을 바꾸도록 하자

`IndexController` -> `HomeController`

```java
@Controller
@RequestMapping("/")
public class HomeController {

    @RequestMapping("index")
    public String index() {
	return "root.index";
    }

    @RequestMapping("help")
    public String help() {
	return "";
    }

}
```

`ListController, DetailController` -> `NoticeController`

```java
@Controller
@RequestMapping("/customer/notice/")
public class NoticeController {

    @Autowired
    private NoticeService noticeService;

    @RequestMapping("list")
    public String list() {

	List<Notice> list = noticeService.getList(1, "TITLE", "");

	return "notice.list";
    }

    @RequestMapping("detail")
    public String detail() {
	return "notice.detail";
    }
}
```

### 다시 정리 (by 뉴렉쳐 센세)

---

- 기존 방식

![image](https://user-images.githubusercontent.com/66164361/126488390-b1c77f5c-48d2-419a-ae96-82d115a44f13.png)

- 현재 방식
  ![image (1)](https://user-images.githubusercontent.com/66164361/126488387-5b82d146-83de-4cbf-a61c-2167f18a6689.png)

Controller의 메서드를 대신 호출해준다

![image (2)](https://user-images.githubusercontent.com/66164361/126488392-621da32b-0602-4574-83d9-86f5a8f04ee0.png)
