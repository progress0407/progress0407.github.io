---
layout: post
title: "영한님: spring 고급 (2) 동적 프록시 기술 ~ ..."
subtitle: "..."
date: 2022-02-21 17:30:00 +0900
categories: backend
tags: spring
comments: true
---

## 리플렉션

---

리플렉션을 사용하면 클래스와 메서드의 메타정보를 사용해서 애플리케이션을 동적으로 유연하게 만들 수 있다.
문자열을 인자로 넘겨서 클래스나 메서드 정보를 동적으로 변경할 수 있기 때문이다
하지만 리플렉션 기술은 런타임에 동작하기 때문에, 컴파일 시점에 오류를 잡을 수 없다.

> 소스 코드

```java
// 클래스의 메타정보 획득
Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

Hello target = new Hello();

// callA 메서드 정보 획득
Method methodCallA = classHello.getMethod("callA");
dynamicCall(methodCallA, target);

// callB 메서드 정보 획득
Method methodCallB = classHello.getMethod("callB");
dynamicCall(methodCallB, target);
```

## JDK 동적 프록시

---

지금까지는 적용 대상 클래스가 100개면 프록시 클래스도 100개를 만들어야 했었다.

그런데 정작 기본 공통 코드와 흐름은 거의 같았다. 적용 대상만이 다른 것일 뿐...

동적 프록시 기술을 사용하면 직접 프록시 클래스를 만들지 않고 동적으로 런타임에 만들어 준다.

> 핵심 구현

```java
@Slf4j
@RequiredArgsConstructor
public class TimeInvocationHandler implements InvocationHandler {

	private final Object target;

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		log.info("TimeProxy 실행");
		long startTime = System.currentTimeMillis();
		Object result = method.invoke(target, args);
		long endTime = System.currentTimeMillis();
		long resultTime = endTime - startTime;

		log.info("TimeProxy 종료 resultTime={}", resultTime);
		return result;
	}
}
```

> 실행 로직

```java
AInterface target = new AImpl();
InvocationHandler handler = new TimeInvocationHandler(target);
AInterface proxy = (AInterface) Proxy.newProxyInstance(
  AInterface.class.getClassLoader(),
  new Class[] {AInterface.class},
  handler);

proxy.call();

log.info("targetClass={}", target.getClass());
log.info("proxyClass={}", proxy.getClass());
```

> 한계

`JDK 동적 프록시`는 인터페이스가 필수다.
클래스만 있을 경우에 적용하려면 다른 방법이 필요하다

### CGLIB : Cde Generate Library

---

- 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리이다.
- 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 만들어낼 수 있다.
- 원래는 외부 라이브러리인데, 스프링 내부 소스 코드에 포함되었다.

> 핵심 구현

```java
@Slf4j
@RequiredArgsConstructor
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;

        log.info("TimeProxy 종료 resultTime={}", resultTime);
        return result;
    }
}
```

> 실행 로직

```java
ConcreteService target = new ConcreteService();

Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(ConcreteService.class);
enhancer.setCallback(new TimeMethodInterceptor(target));
ConcreteService proxy = (ConcreteService) enhancer.create();
log.info("targetClass={}", target.getClass());
log.info("proxyClass={}", proxy.getClass());

proxy.call();
```

> CGLIB 가 생성한 프록시 클래스 이름

- `ConcreteService$$EnhancerByCGLIB$$25d6b0e3`

### CGLIB의 제약

---

클래스 기반 프록시는 상속을 사용하기 때문에 몇가지 제약이 있다.

- `부모 클래스의 생성자`를 체크해야 한다. CGLIB는 자식 클래스를 동적으로 생성하기 때문에 `기본 생성자가 필요`하다.
- `클래스`에 `final` 키워드가 붙으면 `상속이 불가능`하다. CGLIB에서는 예외가 발생한다.
- `메서드`에 `final` 키워드가 붙으면 해당 메서드를 `오버라이딩 할 수 없`다. CGLIB에서는 프록시 로직이 동작하지 않는다.
