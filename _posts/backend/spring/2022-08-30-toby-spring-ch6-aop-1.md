---
layout: post
title: "토비 스프링 AOP (1)"
subtitle: "..."
date: 2022-08-30 14:00:00 +0900
categories: backend
tags: spring
comments: true
published: true
---

# AOP

## 출처

- 토비의 스프링 1권  
  - https://search.shopping.naver.com/book/catalog/32473603051?cat_id=50010881&frm=PBOKPRO&query=%ED%86%A0%EB%B9%84%EC%9D%98+%EC%8A%A4%ED%94%84%EB%A7%81&NaPm=ct%3Dl7frdrnc%7Cci%3Daf5605ee59c05aa685d862f580c74d4d36f818fb%7Ctr%3Dboknx%7Csn%3D95694%7Chk%3D23b05a9bc36f6af69892f8e6587e8dc03b05e677
  
- 김영한님 스프링 고급편 
  - https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard 

# 트랜잭션 코드의 분리

`트랜잭션 경계설정`과 `비즈니스 로직`이 공존한다

```java
public void upgradeLevels() throws Exception {
    TransactionStatus status = this.transactionManager
            .getTransaction(new DefaultTransactionDefinition()); // 트랜잭션 경계설정 START
    try {
        List<User> users = userDao.getAll(); // 비즈니스 로직
        for (User user : users) {
            if (canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }

        this.transactionManager.commit(status); // 트랜잭션 경계설정 END
    } catch (Exception e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}
```

메서드를 분리하면 아래와 같다

```java
public void upgradeLevels() throws Exception {
    TransactionStatus status = this.transactionManager
            .getTransaction(new DefaultTransactionDefinition()); // 트랜잭션 경계설정 START
    try {
       upgradeLevelsInternal(); 
        this.transactionManager.commit(status); // 트랜잭션 경계설정 END
    } catch (Exception e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}

public void upgradeLevelsInternal() {
    List<User> users = userDao.getAll(); // 비즈니스 로직
    for (User user : users) {
        if (canUpgradeLevel(user)) {
            upgradeLevel(user);
        }
    }
}
```

그러나 여전히 트랜잭션 코드가 존재한다

# DI 적용을 이용한 트랜잭션 분리

UserService 인터페이스 밑에

비즈니스로직을 담당하는 UserServiceImpl과  
트랜잭션을 담당하는 UserServiceTx를 놓자

![image](https://user-images.githubusercontent.com/66164361/187352006-cd6c2af2-2dd2-42e0-aba7-bc0ba0e10514.png)

```java
public interface UserService {
    void add(User user);
    void upgradeLevels();
}
```



```java
// 트랜잭션 경계 설정
@Setters
public class UserServiceTx implements UserService {
    UserService userService;
    PlatformTransactionManager transactionManager;

    ...

    public void upgradeLevels() {
        TransactionStatus status = this.transactionManager
                .getTransaction(new DefaultTransactionDefinition());
        try {
            userService.upgradeLevels(); // 비즈니스 로직은 진짜 객체에 위임

            this.transactionManager.commit(status);
        } catch (RuntimeException e) {
            this.transactionManager.rollback(status);
            throw e;
        }
    }
}
```

```java
// 비즈니스 로직
public class UserServiceImpl implements UserService {
    UserDao userDao;

    public void upgradeLevels() {
        List<User> users = userDao.getAll();
        for (User user : users) {
            if (canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }
    }
}
```

이렇게 되면 이제 비즈니스 로직에는 더 이상 기술적인 내용의 코드를 볼 수 없다(서버환경, 트랜잭션 등), 즉 비즈니스 로직만 보인다.

# 트랜잭션 적용을 위한 DI 설정

![image](https://user-images.githubusercontent.com/66164361/187352592-0818ba68-bd5d-47c9-a62c-cd0c0e1060a4.png)

위와 같은 XML 설정을 통해서 진행할 수 있다

-> Java 로 가능하다!

# 트랜잭션 분리에 따른 테스트 수정

- 생략

# 단위 테스트와 통합 테스트

### 단위 테스트

정하기 나름이다, 중요한 것은 하나의 단위체 초점을 맞춘 테스트라는 것  

- 사용자 관리 기능 전체
- 하나의 클래스
- 하나의 메소드

여기서는 테스트 대상 클래스를 목 등의 대역을 통해서 의존 대상이나 외부 리소스를 사용하지 않도록 고립시켜서 테스트하는 것.

### 통합 테스트

- 두 개 이상의 오브젝트가 연동하도록 만든 테스트

- 외부의 DB나 파일(File I/O), 서비스(PG사 결제 연동)가 참여하는 테스트

# 테스트 가이드

- 항상 단위 테스트를 먼저 고려한다.

- 단위 테스트는 실행 속도가 빠르다

- 외부 리소스를 사용하는 테스트는 통합 테스트로 만든다

- DAO같은 경우는 단위 테스트로 만들기 어렵다. 고립된 테스트로 만들기 힘들며 만들어도 가치가 없는 경우가 대부분이다. DAO는 DB까지 연동하는 테스트로 만드는 편이 효과적.

- DAO 테스트는 DB라는 외부 리소스를 사용하기 때문에 통합 테스트로 분류된다. 하지만 하나의 기능 단위를 테스트하는 것이기도 하다.

- 단위 테스트를 만들기 복잡하다고 판단되는 코드는 처음부터 통합 테스트를 고려해본다. (`서비스 테스트`를 말하는 것 같기도 하다)

- `SpringBootTest` 등 스프링 테스트 컨텍스트 프레임워크를 사용하는 테스트는 통합테스트다.

# 목 프레임워크

생략

# 프록시, 프록시 패턴, 데코레이터 패턴

## 🥝 프록시 

실제 객체 대신에 클라이언트의 요청을 대신 받아주는 것을 proxy라 부른다.

영한님의 정의로는

프록시를 통해서 할 수 있는 일은 크게 2가지로 구분할 수 있다.
- 접근 제어
- 권한에 따른 접근 차단
- 캐싱
- 지연 로딩
부가 기능 추가
원래 서버가 제공하는 기능에 더해서 부가 기능을 수행한다.
예) 요청 값이나, 응답 값을 중간에 변형한다.
예) 실행 시간을 측정해서 추가 로그를 남긴다.

영한님의 본 것 기준으로 `proxy`를 통해서 할 수 있는 일들은 아래와 같다.

- 접근 제어
  - 권한에 따라 접근 차단
- 캐싱
  - 지연 로딩
  - 성능 상의 이점을 가질 수 있다
- 부가 기능 추가
  - ex) 
    - 실행 시간을 측정해서 추가 로그를 남기낟.
    - 쿼리가 나간 횟수를 기록
    - 요청/응답 값을 중간에 변경

## 🥝 데코레이터 패턴 

객체에 기능을 동적으로 추가가 가능하다

> 데코레이터 패턴 예

```java
InputStream is = new BufferedInputStream(new FileInputStream("a.txt"));
```

## 🥝 프록시 패턴

접근 제어를 할 수 있다. 토비님 설명으로 타깃의 기능을 확장하거나 추가하지는 않는다고 한다... 

- 사용된 곳 
  - `Collections.unmodifiable();`
  - JPA의 지연로딩 시 사용되는 Proxy 객체

그러나 필자가 보기에는 프록시 패턴 또한 가능한 것으로 보인다.

왜냐하면 타깃을 호출하기 전후로 추가 기능을 넣을 수 있다.

```java
@RequiredArgsConstructor
class 프록시 extends Target {
    Target target;

    @Overide
    public execute() {
        // (1) 기능 수행
        target.execute();
        // (2) 기능 수행
    }
}
```

# 다이나믹 프록시 

위의 문제는 대상 클래스가 100개라면 100개 다 !,

클래스당 평균 메서드가 10개라면 10개 다 만들어야 한다는 것이다!

## 리플렉션 API

다이나믹 프록시는 리플렉션 기능을 이용해서 프록시를 동적으로 만든다.

리플렉션을 이용하면 클래스나 메서드의 메타 정보(접근 제한자, 매개변수, 반환 타입 등)를 동적으로 알아낼 수 있고, 호출할 수 있다

```java
interface Hello {
    String sayHello(String name);
    String sayHi(String name);
    String sayThankYou(String name);
}
```

위와 같은 인터페이스를 두고 아래의 실제 객체(이하 `타겟`)를 두었다

```java
public class HelloTarget implements Hello {
    public String sayHello(String name) {
        return "Hello " + name;
    }

    public String sayHi(String name) {
        return "Hi " + name;
    }

    public String sayThankYou(String name) {
        return "Thank You " + name;
    }
}
```

Hello 진짜 객체의 프록시 객체이다

```java
public class HelloUppercase implements Hello {

    private final Hello delegate;

    public HelloUppercase(Hello delegate) {
        this.delegate = delegate;
    }

    @Override
    public String sayHello(String name) {
        return delegate.sayHello(name).toUpperCase();
    }

    @Override
    public String sayHi(String name) {
        return delegate.sayHi(name).toUpperCase();
    }

    @Override
    public String sayThankYou(String name) {
        return delegate.sayThankYou(name).toUpperCase();
    }
}
```

프록시 팩토리를 통해서 런타임 시에 동적으로 실제 객체에 대한 다이나믹 프록시를 만든다.

다이나믹 프록시는 타겟의 인터페이스와 같은 타입으로 만들어진다.

![image](https://user-images.githubusercontent.com/66164361/187361022-9ac9b74c-73f7-4809-80df-e869df134f43.png)
![image](https://user-images.githubusercontent.com/66164361/187361037-69317195-1254-4f5c-818d-12cce94db773.png)

```java
public class UppercaseHandler implements InvocationHandler {
    private final Object target;

    public UppercaseHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result =  method.invoke(target, args);
        if (result instanceof String) {
            return ((String) result).toUpperCase();
        }
        return result;
    }
}
```

이때 인터페이스가 없으면 프록시 생성이 되지 않는다!!

![image](https://user-images.githubusercontent.com/66164361/187363689-0c4203ee-0ce2-411a-8834-be07e3795846.png)


## 다이나믹 프록시를 이용한 트랜잭션 부가기능

```java
@Setter
public class TransactionHandler implements InvocationHandler {
    private Object target; // 실제 객체
    private PlatformTransactionManager transactionManager; // 트랜잭션 기능을 제공
    private String pattern; // 트랜잭션을 적용할 메소드 패턴 정의

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (method.getName().startsWith(pattern)) {
            return invokeInTransaction(method, args);
        }
        return method.invoke(target, args);
    }

    private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object ret = method.invoke(target, args); // 타깃 호출
            this.transactionManager.commit(status);
            return ret;
        } catch (InvocationTargetException e) {
            this.transactionManager.rollback(status);
            throw e.getTargetException();
        }
    }
}
```

```java
public void upgradeAllOrNothing() throws Exception {
    // ...
    TransactionHandler txHandler = new TransactionHandler();
    txHandler.setTarget(testUserService);
    txHandler.setTransactionManager(transactionManager);
    txHandler.setPattern("upgradeLevels");

    UserService txUserService = (UserService) Proxy.newProxyInstance(
        getClass().getClassLoader(), new Class[] { UserService.class }, txHandler);
    // ...
}
```

## CGLIB 방식

다이나믹 프록시의 경우 항상 인터페이스가 필요하나 CGLIB 방식은 인터페이스가 궅이 필요 없다 !

```java
@Setter
class TransactionHandler implements MethodInterceptor {

    private Object target;
    private PlatformTransactionManager transactionManager;
    private String pattern;

    @Override
    public Object intercept(final Object obj, final Method method, final Object[] args, final MethodProxy proxy)
            throws Throwable {
        if (method.getName().startsWith(pattern)) {
            return proxy.invoke(target, args);
        }
        return method.invoke(target, args);
    }

    private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            Object ret = method.invoke(target, args);
            this.transactionManager.commit(status);
            return ret;
        } catch (InvocationTargetException e) {
            this.transactionManager.rollback(status);
            throw e.getTargetException();
        }
    }
}
```

인터페이스 `MethodInterceptor`의 `inercept`를 구현하면 된다 !

```java
@Slf4j
class TransactionHandlerTest {

    @Test
    void cglib() {
        final HelloTarget target = new HelloTarget();

        final TransactionHandler handler = new TransactionHandler();
        handler.setTarget(target);

        final Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(HelloTarget.class);
        enhancer.setCallback(handler);

        HelloTarget proxy = (HelloTarget)enhancer.create();

        log.info("targetClass={}", target.getClass());
        log.info("proxyClass={}", proxy.getClass());
        final String result = proxy.sayHello("swho");
        log.info("result={}", result);
    }
}
```

## 🥝 다이나믹 프록시를 위한 팩토리 빈

내용이 너무 어렵다!

스프링 DI에 Bean을 등록하려면 클래스 이름이 필요하다

```java
// 예시
Date now = (Date) Class.forName("java.util.Date").newInstance();
```

다이나믹 프록시 방식의 경우 클래스 이름을 알 수 없다

![image](https://user-images.githubusercontent.com/66164361/187367471-c8674a81-61e9-4e8f-85f7-4309de119b8e.png)

![image](https://user-images.githubusercontent.com/66164361/187367552-2cfc7cee-18fe-4f08-9516-99b343313a5f.png)
![image](https://user-images.githubusercontent.com/66164361/187367591-9130381f-a28a-4012-be7c-12e04ef5c5e9.png)

## 🥝 팩토리 빈

크게 어려운 것 없다 !

위와 같은 문제로 IoC에 등록을 할 수 없으니

FactoryBean을 통해서 가져올 수 있다는 것이다

```java
public class HelloTargetFactoryBean<H> implements FactoryBean<HelloTarget> {

    @Override
    public HelloTarget getObject() throws Exception {
        return new HelloTarget();
    }

    @Override
    public Class<?> getObjectType() {
        return HelloTarget.class;
    }
}
```

```java
@SpringBootTest
class HelloTargetFactoryBeanTest {

    @Autowired
    private ApplicationContext context;

    @Test
    void test() {
        final HelloTarget helloPeanut = context.getBean("helloPeanut", HelloTarget.class);
        assertThat(helloPeanut.getClass()).isEqualTo(HelloTarget.class);
        assertThat(helloPeanut.sayHello("philz")).isEqualTo("Hello philz");
    }

    @TestConfiguration
    static class TestConfig {
        @Bean
        public HelloTargetFactoryBean<HelloTarget> helloPeanut() {
            return new HelloTargetFactoryBean<>();
        }
    }
}
```

위와 같이 스프링 컨테이너에 등록해서 사용할 수 있다 !

## 트랜잭션 프록시 팩토리 빈

위와 같은 과정을 트랜잭션 기능을 담고 있는 기능을 담아서 적용하면 된다

![image](https://user-images.githubusercontent.com/66164361/187372059-57860946-ecc4-4577-9322-2d7cba15729c.png)


```java
@Setter
public class TxProxyFactoryBean implements FactoryBean<Object> {
    Object target;
    PlatformTransactionManager transactionManager;
    String pattern;
    Class<?> serviceInterface;

    public Object getObject() throws Exception {
        TransactionHandler txHandler = new TransactionHandler();
        txHandler.setTarget(target);
        txHandler.setTransactionManager(transactionManager);
        txHandler.setPattern(pattern);

        return Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { serviceInterface }, txHandler);
    }

    public Class<?> getObjectType() {
        return serviceInterface; 
    }

    public boolean isSingleTon() {
        return false; 
    }
}
```

## 프록시 팩토리 빈의 장점

개발자가 작성해둔 핵심 비즈니스 로직의 코드 수정 없이도 다양한 클래스에 적용할 수 있다. 

1. 프록시 클래스를 일일이 만드는 번거로움을 해결

2. 공통 관심시가 비즈니스 메서드에 반복적으로 나타나는 부분을 해결

## 프록시 팩토리 빈의 단점

1. 설정에 생기는 비약적인 중복

책 기준으로는 XML이지만.. 자바 코드상에 아래와 같은 중복 로직이 대거 등장하게 된다!

```java
@Configuration
static class Config {

    @Bean
    public TxFactoryBean<UserService> userService() {
        return new TxFactoryBean<>();
    }

    @Bean
    public TxFactoryBean<CoreService> coreService() {
        return new TxFactoryBean<>();
    }

    @Bean
    public TxFactoryBean<ItemService> itemService() {
        return new TxFactoryBean<>();
    }

    // ...
}
```

2. 거의 같은 역할을 하는 객체가 생성

또 다른 문제점은 TransactionHandler, 즉 프록시 객체가 proxy factory bean 갯수 만큼 만들어진다.

모든 타깃에 적용 가능한 싱글톤 빈을 만들 수 있는가?

# 스프링의 프록시 팩토리 빈

```java
public class DynamicProxyTest {

    @Test
    public void proxyFactoryBean() {
        ProxyFactoryBean pfBean = new ProxyFactoryBean();
        pfBean.setTarget(new HelloTarget()); //타깃 설정
        pfBean.addAdvice(new UppercaseAdvice()); // 부가기능 추가
        Hello proxiedHello = (Hello) pfBean.getObject(); // FacotryBean이므로 생성된 프록시를 가져온다.

        assertThat(proxiedHello.sayHello("Philz")).isEqualTo("Hello Philz");
    }

    static class UppercaseAdvice implements MethodInterceptor {

        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            String ret = (String)invocation.proceed(); //타깃을 알고 있기에 타깃 오브젝트를 전달할 필요가 없다.
            return ret.toUpperCase(); //부가기능 적용
        }
    }
}
```

- `invocation.proceed()`를 통해서 템플릿 콜백으로 실행
- 부가기능을 `proxyFactoryBean.add` 를 통해 추가 가능하다
- 타깃의 인터페이스가 필요 없다.

## SpringFactory와 SpringFactoryBean의 차이

```
ProxyFactory is independently of Spring's IoC container. ProxyFactoryBean is combining Spring AOP with Spring's IoC container.

ProxyFactory는 Spring의 IoC 컨테이너와 독립적입니다. ProxyFactoryBean은 Spring AOP와 Spring의 IoC 컨테이너를 결합하고 있습니다.
```

SpringFactory(Bean)은 `Interface`(`구현`)와 `구체 클래스`(`상속`)의 프록시 생성을 추상화한 것이다.

## SpringFactory 참고 자료

![image](https://user-images.githubusercontent.com/66164361/187381204-7dcf05e9-9e76-4c2e-a1c8-4354ce77d333.png)
![image](https://user-images.githubusercontent.com/66164361/187381233-960f1ad5-2439-4504-8efc-ba13620e6402.png)


```java
assertThat(AopUtils.isAopProxy(proxy)).isTrue();
assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse();
assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
```

