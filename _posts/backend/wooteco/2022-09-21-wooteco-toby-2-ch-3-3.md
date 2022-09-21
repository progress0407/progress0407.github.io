---
layout: post
title: "[toby] 스프링 웹 기술과 Spring"
subtitle: "..."
date: 2022-09-13 22:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

# 3.3 컨트롤러

MVC  세 컴포넌트 중에서 가장 많은 책임을 지고 있다.  
- 클라이언트 호스트, 포트, URI, 쿼리스트링, 폼 파라미터, 쿠키, 헤더, 세션을 비롯해서 서블릿 컨테이너가 요청 애트리뷰트로 전달해주는 것 등을 고려해서 처리를 한다.
- 사용자가 바르게 요청을 보냈는지 검증.  (프레젠테이션 예외처리)
- 사용자에게 적절한 응답값을 보내준다.  
- 뷰를 선택할 지 JSON 등의 데이터를 선택할 지.  
- URI Mapping  

# 컨트롤러 어댑터  

핸들러 매핑으로부터 전달 받은 핸들러를 알맞은 핸들러 어댑터를 반환해준다. 이 어댑터는 각 종류의 컨트롤러와 호환이 되게 하는 역할을 지닌다.  

컨트롤러의 종류는 몇 가지 있다.  

- SimpleSeevlet Handler  
- HttpRequestHandler와 HttpRequestAdapter  
- Controller와 SimpleControllerAdapter  
- AnnotationMethodAdapter  

## 왜 스프링은 어댑터 패턴을 사용해서 여러 가지의 컨트롤러를 지원하게 했을까...

```
...
이런 모든 작업을 단순하게 컨트롤러 메소드 하나에 모두 담아두는 건 비효율적이며 객체지향적이라고 보기도 힘들다. 애플리케이션 성격상 컨트롤러의 역할이 크다면 책임의 성격과 특징, 변경 사유 등을 기준으로 세분화해줄 필요가 있다.  

스프링 MVC가 컨트롤러 모델을 미리 제한하지 않고 어댑터 패턴을 사용해서라도 컨트롤러의 종류를 필요에 따라 확장할 수 있도록 만든 이유가 바로 이 때문이다. 스프링 MVC의 컨트롤러는 다양한 방식으로 진화하고 발전하고 있다. DispatcherServlet의 전략 패턴을 통한 유연함의 가치가 가장 잘 드러나는 영역이 바로 컨트롤러다.
...
```

토비님의 생각과 더불어 개인적인 생각으로는 기존의 컨트롤러 매핑 방식을 지원하면서 새로운 지원 방법도 같이 적용하기 위해서 어댑터 방식으로 버전 업그레이드를 해온 것이 아닌가 싶다 ! (`하위 호환성`)  

## SimpleServletHandlerAdapter

- 표준 서블릿  
- 표준 서블릿 인토페이스인 javax.sevlet.Servlet을 구현했다

```java
@Component("/hello")
 class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp){
        String name=req.getParameter("name");
        resp.getWriter().print("Hello" +name);
    }
}
```

## HttpRequestHandler와 HttpRequestAdapter  

- 인터페이스로 정의된 컨트로ㅓㄹ러 타입

```java
public interface HttpRequestHandler {
    void handleRequest(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException;
}
```

## Controller와 SimpleControllerAdapter  

- 스프링 3.0의 애노테이션 관례를 이용한 컨트롤러가 등장하기 전의 스프링 MVC의 가장 대표적인 컨트롤러 타입.
- 3.0 이전에 MVC의 컨트롤러라고 하면 이 컨트롤러를 뜻하였다

```java
public interface Controller {
    ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```


## AnnotationMethodAdapter

- 스프링 MVC에서 가장 인기 있는 컨트롤러 작성 방법
- 하나 이상의 URL에 매핑 될 수 있다
- 기능은 강력하며 코드는 단순하다
- 대신 어노테이션을 많이 붙여야 한다

```java 
@Controler 
class HelloController{

  @RequestMapping("/hello")
  public String hello(@RequestParam("name") String name, ModelMap map) {
    map.put("message", "Hello" + name);
    return "/WEB-INF/view/hello.jsp";
  }
}
```

# 핸들러 매핑

## BeanNameUrlHandlerMapping

```xml
<bean name="/s*" class"hello...Controller">  
```

- URL에 `ANT 패턴`을 이용해서 기입할 수 있다.  

## ControllerBeanNameHandlerMapping

```xml
<bean id="hello" class="hello...Controller">
```

```java
@Component("hello")
public class MyController implements Controller {
  ...
}
```

## ControllerClassNameHandlerMapping

```java
public class HelloController implements Controller { ... }
```

- `"/hello"` 에 URL 매핑된다
- `HelloController` 에서 Controller를 제거한 후 모두 소문자로 변경한 것이 URL 매핑

## SimpleUrlHandlerMapping

```xml
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/hello">helloController</prop>
            <prop key="/sub/*">myController</prop>
        </props>
    </property>
</bean>
```

## DefaultAnnotationHandlerMapping

```java
@RequestMapping("/hello)
```

- 메서도 단위로 URL 매핑 가능


## 핸들러 인터셉터

우리가 너무 잘 알고 있는(wooteco) 내용이다.. 생략!


# 3.6 스프링 3.1의 MVC

- 플래시 맵

![image](https://user-images.githubusercontent.com/66164361/191414003-556f814d-2358-46f6-85ff-98916dcf4297.png)

![image](https://user-images.githubusercontent.com/66164361/191414082-c00ef7cd-c09b-4224-bc1e-644d38eb9d11.png)

- `POST` 요청후 REDIRECT로 인한 `GET`요청 까지 기억할 임시 데이터

```java
FlashMap fm = new FlashMap();
fm.put("temp", "임시데이터");
fm.setTargetRequestPath("/user/list");
fm.startExpirationPeriod(10);
```

![image](https://user-images.githubusercontent.com/66164361/191414464-642ccac5-e37d-4f1b-9acc-5f69f1169326.png)

옛날 기술 ㅠ 제네릭도 지원 안 된다...

# WebApplicationInitializer를 이용한 컨텍스트 등록

## mvc 미션

```java
public class AppWebApplicationInitializer implements WebApplicationInitializer {

    private static final Logger log = LoggerFactory.getLogger(AppWebApplicationInitializer.class);

    @Override
    public void onStartup(final ServletContext servletContext) {
        PeanutContainer.INSTANCE.init("com.techcourse.controller");

        final DispatcherServlet dispatcherServlet = initDispatcherServlet();
        final Dynamic dispatcher = servletContext.addServlet("dispatcher", dispatcherServlet);
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping("/");

        log.info("Start AppWebApplication Initializer");
    }

    private DispatcherServlet initDispatcherServlet() {
        final var dispatcherServlet = new DispatcherServlet();
        dispatcherServlet.addHandlerMapping(new ManualHandlerMapping());
        dispatcherServlet.addHandlerMapping(new AnnotationHandlerMapping("com.techcourse.controller"));
        dispatcherServlet.addHandlerAdapter(new ManualHandlerAdapter());
        dispatcherServlet.addHandlerAdapter(new AnnotationHandlerAdapter());
        return dispatcherServlet;
    }
}
```

---

(XML )`web.xml`에서 하던 설정을 (JAVA) `ServletContainerInitializer`으로 대신할 수 있다

```xml
<!-- /WEB-INF/applicationContext.xml 생성-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>    
</listener>
```

```java
public interface WebApplicationInitializer{
    void onStartup(ServletContext servletContext) throws ServletException;
}
```

```java
ServletContextListener listener = new ContextLoaderListener();
servletContext.addListener(listener);
```

## 서블릿 웹애플리케이션  컨텍스트 등록 (servlet-context.xml)

```xml
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <sevlet-name>appServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

```java
ServletRegistration.Dynamic dispatcher = servletContext.addServlet("appServlet", new DispatcherServlet());

dispatcher.setInitParameter("contextConfigLocation", "/WEB-INF/spring/appServlet/servlet-context.xml");
dispatcher.setLoadOnStartUp(1);
dispatcher.addMapping("/");
```

```java
AnnotationConfigWebApplicationContext sac = new AnnotationConfigWebApplicationContext();
sac.register(WebConfig.class);

ServletRegistration.Dynamic dispatcher = servletContext.addServlet("appServlet", new DispatcherServlet(sac));
dispatcher.setLoadOnStartUp(1);
dispatcher.addMapping("/");
```


# 참고
> 토비의 스프링 (2)
> https://jjoystory.tistory.com/m/36  

