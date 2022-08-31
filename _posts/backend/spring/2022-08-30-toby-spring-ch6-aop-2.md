---
layout: post
title: "토비 스프링 AOP (2)"
subtitle: "..."
date: 2022-08-30 14:00:00 +0900
categories: backend
tags: spring
comments: true
published: true
---

# 스프링 AOP

## 자동 프록시 생성

한 번에 여러 개의 빈에 프록시를 적용할 만한 방법이 필요하다

## 빈 후처리기를 이용한 자동 프록시 생성기

![image](https://user-images.githubusercontent.com/66164361/187453214-b5240cb6-f32d-4792-8cef-c38aab05176b.png)

![image](https://user-images.githubusercontent.com/66164361/187451645-1fd6b9dd-10b0-4600-b202-fc6d0b67e5f4.png)

BeanPostProcessor 인터페이스를 구현해서 만든 빈 후처리기 !

이것을 이용하면 Bean을 프록시로 바꿔칠 수 있다!

```java
public class BeanPostProcessorTest {

    // Bean Post Processor
    @Slf4j
    static class AToBPostProcessor implements BeanPostProcessor {
        
        @Override
        public Object postProcessAfterInitialization(Object bean, String
                beanName) throws BeansException {
            log.info("beanName={} bean={}", beanName, bean);
            if (bean instanceof A) {
                return new B();
            }
            return bean;
        }
    }

    // Configuration
    @Slf4j
    @Configuration
    static class BeanPostProcessorConfig {

        @Bean(name = "beanA")
        public A a() {
            return new A();
        }

        @Bean
        public AToBPostProcessor helloPostProcessor() {
            return new AToBPostProcessor();
        }
    }

    @Slf4j
    static class A {
        public void helloA() {
            log.info("hello A");
        }
    }

    @Slf4j
    static class B {
        public void helloB() {
            log.info("hello B");
        }
    }
    
    @Test
    void postProcessor() {
        ApplicationContext applicationContext = new
                AnnotationConfigApplicationContext(BeanPostProcessorConfig.class);
        //beanA 이름으로 B 객체가 빈으로 등록된다.
        B b = applicationContext.getBean("beanA", B.class);
        b.helloB();
        //A는 빈으로 등록되지 않는다. Assertions.assertThrows(NoSuchBeanDefinitionException.class,
        () -> applicationContext.getBean(A.class));
    }
}
```

## 확장된 포인트컷

```java
public interface Pointcut {
    ClassFilter getClassFilter();
    MethodMatcher getMethodMatcher();
}
```

프록시를 적용할 클래스인지 판단하고 나서, 적용 대상이라면 부가 기능을 적용할 메서드인지를 판단한다

## 포인트컷 표현식

`AspectJExpressionPointcut`에서 제공하는 포인트컷 표현식을 사용하면 복잡한 선정 조건을 쉽게 지정할 수 있다.

## 포인트컷 표현식 문법

포인트컷 지시자 중에서 가장 많이 사용되는 것은 `execution()` 이다

```
execution([접근제한자] 반환타입 [패키지+클래스.]메서드이름 (파라미터))
```

```java
@Test
public test() {
    AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
    pointcut.setExpression("execution(public int wooteco.chess.Pawn.attack(Row, Column))");

    assertThat(pointcut.getClassFilter().matches(Pawn.class)).isTrue();
    assertThat(pointcut.getMathodMatcher().matches(Pawn.class.getMethod("attack", Row.class, Column.class), null)).isTrue();
}
```

## 포인트컷 표현식 테스트

```c
execution(int minus(int, int))
```

반환타입: int
클래스명: anything
메서드명: minus
파라미터: int, int

```c
execution(* minus(int, int))
```

> `변한 부분만 작성!`
반환타입: anything


```c
execution(* minus(..))
```

파라미터: anything

```c
execution(* *(..))
```

메서드명: anything

## 포인트컷 표현식을 이용하는 포인트컷 적용

`@Transactional` 이라는 어노테이션만을 아래와 같이 선정할 수 있다

```java
@annotation(org.springframework.transaction.annotation.Transactional)
```

# AOP란 무엇인가

## AOP: 관점 지향 프로그래밍

공통 관심사를 분리함으로써 핵심기능을 설계하고 구현할 때 OOP의 가치를 지킬 수 있도록 도와주는 것

# [AOP 적용기술]

# 프록시를 이용한 AOP

- 자바의 기술들 대거 조합해서 AOP 지원
  - Spring IoC
  - Dynamic Proxy
  - CGLib
  - Design Pattern (Decorator / Proxy) 
  - PointCut
  - Bean Post Processor

핵심은 `프록시`를 이용했다는 것 !

# [김영한님] AOP 적용기술

## 컴파일 시점

- .java 파일을 .class 파일(`바이트코드`)로 만들 때 적용 
- 가장 SOLID한 방식이다, 파일에 직접 부가 기능을 넣는 것

![image](https://user-images.githubusercontent.com/66164361/187591642-16b89705-02e5-4c33-89a8-7de175a5ffd3.png)

### 단점
- 특별한 컴파일러가 필요하고 복잡하다

## 클래스 로딩 시점

- .class 파일을 JVM에 로딩할 때에 적용
- .class 파일 자체는 온전하나 JVM에 적재될 때 부가 기능을 넣는다

![image](https://user-images.githubusercontent.com/66164361/187591659-12efeaea-7661-41e3-aa6d-574ceb735215.png)

### 단점

- 로드 타임 위빙은 자바를 실행할 때 특별한 옵션( java -javaagent )을 통해 클래스 로더 조작기를
지정해야 하는데, 이 부분이 번거롭고 운영하기 어렵다.

- 자바 실행시 java -javaagent을 통해 클래스 로더 조작기를 지정해야하는데, 이 부분이 번거롭다 !

![image](https://user-images.githubusercontent.com/66164361/187592602-08665849-02b2-45ae-bb1c-a9881cfef4f0.png)

(저런 모습이 아닌가 생각한다!)

## 런타임 시점

- JVM로딩 된 후에 Spring IoC에 적재될 때 적용
- 종합아트예술

![image](https://user-images.githubusercontent.com/66164361/187591683-5539a583-7839-4482-a9aa-6758b6d85a84.png)

# 바이트코드 생성과 조작을 통한 AOP

- AjpectJ: 대표적인 AOP 기술
- 직접 타깃을 뜯어고쳐서 부가 기능을 넣는 방법을 사용한다
- 컴파일 된 타깃 .class 파일을 수정하거나 JVM 로딩되는 시점에 가로채서 바이트 코드(`.class`)를 조작한다
- 

## AOP의 용어

- 타깃
  - 부가기능을 부여할 대상이다. 경우에 따라서 또 다른 부가기능을 담은 프록시 오브젝트일 수도 있다.

- 어드바이스
  - 타깃에 적용할 부가기능을 담은 모듈(것)

- 포인트컷
  - 부가기능을 적용할 클래스와 메서드를 선별하는 모듈

- 어드바이저
  - 포인트컷 + 어드바이스

- 애스펙트
  - 어드바이저가 싱글톤 형태로 존재하는 것

# [트랜잭션 속성]

# 트랜잭션 정의

더 이상 쪼갤 수 없는 최소 단위의 작업 (`원자성`)

TransactionDefinition 인터페이스에는 트랜잭션의 동작방식에 영향을 줄 수 있는 네 가지 속성이 정의되어 있다

- 전파
- 격리수준
- 제한시간
- 읽기전용

## 트랜잭션 전파

- PROPAGATION_REQUIRED
  - 기본설정이다!
  - 없으면 합류하고 있으면 합류하지 않는다

## 트랜잭션 전파 - 실험

BizService -> InnerService가 있을때

BizService
- execute();
- innerExecute();

InnerService
- execute();

### Q. BizService에서

![image](https://user-images.githubusercontent.com/66164361/187586610-43d924c1-431c-4de4-a136-71ff46f8a369.png)

위와 같은 상황에서 예외가 터지는가?

(두괄식으로 진행할 것)

### Q. BizService에서  @T (X) execute -> @T private innerExcute

Tx가 걸리는가?  X

### Q. 위에서 innerExcute가 public일 경우 

X

### BizService @T -> InnerService @T

O

### 격리수준

- 격리수준은 기본적으로 DB에 설정되어 있지만 JDBC 드라이버나 DataSource 등에서 재설정할 수 있다. 

### 제한시간

- Tx 수행 제한시간을 설정할 수 있다
  - 기본 설정은 제한 시간이 없다


### 읽기 전용

- Tx내에서 데이터를 조작하는 시도 막아줄 수 있다

- DB 접근 기술에 따라서 성능이 향상될 수 있다
  - JPA - 스냅샷

## TransactionInterceptor

- 디버거 수행시 `TransactionInterceptor`.`invoke` -> `TransactionAspectSupport`.`invokeWithinTransaction` 라는 곳으로 이동한다

- 기본적으로 `런타임`(`언체크`) 예외에 대해서만 롤백을 수행한다

- `체크` 예외에 대해서는 롤백 수행을 하지 않는다
  - 일부 체크 예외는 정상 작업 흐름으로 간주해야하는 경우가 있기 때문
  - 사실 이부분은 읽을 때마다 이해가 잘 가지 않는다...

## 프록시 방식 AOP는 내부 메서드 호출시 적용되지 않는다

- 주의사항 !!

![image](https://user-images.githubusercontent.com/66164361/187586220-f910b1da-6b27-4997-a1de-05102f31e69b.png)

- 위와 같은 흐름이다

클라이언트에서 호출한 메서드에서만 적용이 된다

## 트랜잭션 어노테이션

- 트랜잭션 적용을 선언적으로 사용할 수 있다

## 포인트컷과 어드바이스

- TransactionAttributeSourcePointcut
- TxInterceptor와 AnnotationTransactionAttributeSource
