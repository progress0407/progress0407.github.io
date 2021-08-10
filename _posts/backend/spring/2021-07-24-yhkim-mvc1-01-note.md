---
layout: post
title: "영한킴: spring MVC 팁 및 부연설명"
subtitle: "..."
date: 2021-07-24 16:00:00 +0900
categories: backend
tags: spring
comments: true
---

> 이 포스트는 강의 내용 중 알게 된 것이나
> 필자에게 필요한 부분을 입력한 내용을 기술했습니다

---

### 프로젝트 오픈

- spring boot 프로젝트를 열 때는 build.gradle을 open 한다

### HttpServlet

- HttpServletRequest, HttpServletResponse는 interfaece 이다. 톰캣이나 제티 등 WAS들은 이 표준 서블릿 스펙을 구현한다. service등 메서드를 통해 구현체를 확인할 수 있다.

### http 관련

- http 헤더에 셋팅

`response.setContentType("text/plain")`  
`response.setCharacterEncoding("utf-8")`

- http 메세지 바디에 문자열 셋팅

`response.getWriter().write("...")`

- `request.getParamter()`는 GET/POST 2가지 모두 지원한다  
  동명의 파라미터가 복수개일때는 `.getParameterValues()` 를 사용하면 된다.

### Lombok Setting (플러그인에서 설치 후)

- Lombok Setting (플러그인에서 설치 후)  
  `ctrl+alt+S` (Settings) > `Build, Execution, Deployment` > `Compiler` > `Annotaion Processors` > `Enable Annotaion processing` 체크

### with non-zero ...

- 오류관련: WebServlet에 name 인자가 중복되었을 경우 굉장히 비 직관적은 로그가 뜬다  
  `finished with non-zero exit value 1`  
  찾느라 애먹었다.. 구글링으로도, 해외글에서도 찾아볼 수 없었다..
  꼭 검색 뿐만 아니라.. 이쯤에서 잘못되지 않았을까 하고 직관적인 의심도 할 줄 알아야겠다..
  길게 코딩하지 말고 코딩 중간중간 흐름이 끊겨도 결과를 확인하는 습관을 들이자..

### 캐시된 것 표시

- 캐시된 것은 개발자도구에서 Status가 회색 글씨로 나온다 !  
  ![image](https://user-images.githubusercontent.com/66164361/126886435-13d66bf9-4e3b-41a6-bb74-111f7164267c.png)  
  다시 새로 고침하면  
  정상 표시 된다.  
  ![image](https://user-images.githubusercontent.com/66164361/126886454-8c2d1240-7bac-4f75-8170-69fcb062272d.png)  
  이렇게 되면서 현재의 갱신된 자원이 보임  
  상세하게 보면 아래와 같다.  
  ![image](https://user-images.githubusercontent.com/66164361/126886813-653a0f77-751b-4024-a415-e14f2520a02a.png)  
  ![image](https://user-images.githubusercontent.com/66164361/126886822-0eb58334-4a19-4144-b9d8-9c9a4f7eccf7.png)

### Forward & Redirect

- `forward`: 호출이 서버 내부에서 일어남  
  `redirect`: 클라이언트가 호출하게 함  
  `forward`는 서버 내부에서 이전 request, reponse을 갖고 내부에서 재호출을 한다. 그렇기에 브라우저 url은 그대로 이다 . 이런 이유로 클라이언트는 전혀 인지하지 못한다.  
  `redirect`는 말 그대로 클라이언트가 재호출을 하기 때문에 url이 바뀐다. 새로운 요청이 들어오는 것이므로 이전 request, reponse은 없고 새로 만들어진다.  
  클라이언트가 Status 값 302를 보고 Redirect가 일어났단 것을 알 수 있다

### 코드를 개선할 때는

- 코드를 개선하다 보면.. 구조나 디테일을 개선하고 싶을때가 만드시 생긴다고 한다  
  이때 조급함을 참아야 한다고 한다. 같은 레벨의 것을 먼저 개선 후 그 다음 다른 레벨의 것을 개선한다
  예를 들어 구조를 개선할 떄는 디테일을 개선하지 아니한다
  구조 개선 후 테스트가 제대로 동작하면 그 다음 디테일을 개선

### 우연히 보게 된.. Void 클래스

![image](https://user-images.githubusercontent.com/66164361/127171137-0e1e40e0-cc3f-43c2-9540-c5a04187bc2f.png)
신기하다..

### BeanNamedUrlHandlerMapping & Adapter

- 컴포넌트 이름으로 url 등록을 해서 해당 bean을 호출할 수 있다

```java
  @Component("/springmvc/old-controller")
  public class OldController implements Controller {
      @Override
      public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
          System.out.println("OldController.handleRequest");
          return null;
      }
  }
```

- HandllerMapping의 우선순위

  - #0 RequestMappingHandlerMapping : 우리가 아는 @RequestMapping
  - #1 BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러 찾기

_HandlerAdpater는 생략_

### 뷰 리졸버 셋팅 비교

- Bean으로 등록

```java
  @Bean
	ViewResolver internalResourceViewResolver() {
		return new InternalResourceViewResolver("/WEB-INF/views/", ".jsp");
	}
```

- 부트의 properties 등록

```properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

### 강의에서 작성한 view와 InternalResourceView 비교

- `model` -> `request` 로 변환해주는 부분

- MyView.java

```java
  private void modelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
    model.forEach((key, value) -> {
      request.setAttribute(key, value);
    });
  }
```

- InternalResourceViewReolver 클래스 > InternalResourceView 클래스 > renderMergedOutputModel메서드 > exposeModelAsRequestAttributes 메서드

```java
protected void exposeModelAsRequestAttributes(Map<String, Object> model, HttpServletRequest request) throws Exception {
		model.forEach((name, value) -> {
			if (value != null) {
				request.setAttribute(name, value);
			}
			else {
				request.removeAttribute(name);
			}
		});
```

- 타임리프의 view & reolver 도 별도로 있다

- JSP는 forward() 로 렌더링이 되고 타임리프는 자체적으로 자바코드로 바로 렌더링된다

### @Controller

- Component 스캔의 대상 (자체적으로 `@Component` 를 포함)
- 핸들러 매핑의 대상이 됨
  - `@RequestMapping`, `@Controller`가 클래스 레벨에 붙어있어야 `RequestMappingHandlerMapping` 이 인식해서 url정보로 bean을 인식한다
- 즉, `@Controller` = `@Component` + `@RequestMapping`

- RequestMappingHandlerMapping > isHandler

```java
	@Override
	protected boolean isHandler(Class<?> beanType) {
		return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
				AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
	}
```

해석: 클래스 레벨에 Controller나 RequestMapping이 붙어있으면 RequestMappingHandlerMapping이 인식 가능한 Handler이다

- 혹은 아래와 같은 방법으로 Controller 등록이 가능하다

```java
@RequestMapping
public class SpringMemberFormControllerV1 { ... }
```

```java
@ServletComponentScan
@SpringBootApplication
public class ServletApplication {
	public static void main(String[] args) { ... );

	@Bean
	SpringMemberFormControllerV1 springMemberFormControllerV1() {
		return new SpringMemberFormControllerV1();
	}
  ...
```

## ReuestMapping 과 GET, POST

- `@ReuestMapping` 은 기본적으로 `GET`, `POST`를 둘다 수용한다
  - 무언가 하나만으로 타겟하고 싶다면 method 인자를 추가하면 된다

```java
@RequestMapping(value = "new-form", method = RequestMethod.GET)
```

아래 POST 요청을 않음을 알 수 있다
![image](https://user-images.githubusercontent.com/66164361/128630727-60fda425-d0fe-4273-ab2b-d84722cc1962.png)

- 이젠 이것마저도.. `@GetMapping`, `@PostMapping`으로 대체 가능

### War & Jar

- `War`를 사용할 때는
  - `JSP`가 포함될떄
  - 부트의 내장 톰캣이 아닌 별도의 톰캣을 외부에 두고 배포할 떄
- `Jar`이어도 될 떄는
  - `JSP`가 없을때
  - 부트의 내장 톰캣을 이용할 때
  - 최근에는 주로 이 방식 !

### 캐시를 사용하지 않을 때는

- Cache-Control : no-store
  > - https://www.inflearn.com/questions/112647
  > - https://stackoverflow.com/questions/49547/how-do-we-control-web-page-caching-across-all-browsers
