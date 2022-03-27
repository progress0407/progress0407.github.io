---
layout: post
title: "[김영한 - JPA 시리즈] 스프링 데이터 JPA"
subtitle: "..."
date: 2022-03-20 05:00 +0900
categories: backend
tags: jpa
comments: true
---

- 로깅 SLF4J : 인터페이스
- LogBack : 구현체

- Jpa Repository 를 설계할 때 굳이 Update는 넣지 않아도 된다
  - 하이버네이트가 자바 JCF에서 수정하는 것과 동일한 환경을 제공하기 때문이다
  - 더티체킹(변경감지)을 통해서 update 쿼리를 발생시킨다

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

구현체를 만들어서 주입해준다 !!

`@Repository` 생략 가능

- 스프링 컴포넌트 스캔
- JPA의 예외를 스프링에서 공통적으로 처리할 수 있는 예외로 변환하는 기능

- spring.data
  - 몽고, 레디스, JPA 등

`JpaRepository` 는 `spring-data-jpa` 패키지에 있고
부모 인터페이스인 `PagingAndSortingRepository`는 `spring-data` 패키지에 있다. jar는 `spring-commons`

공통 인터페이스인 `PagingAndSortingRepository`의 기능은 몽고나 NoSQL등으로 DB를 옮겨도 사용할 수 있다

그러나 현실적으로 이러한 행위가 어렵기 때문에 이러한 DB이관 이점보다는 유사한 인터페이스로 편하게 개발할 수 있다에 장점이 있다

#### 쿼리메서드 기능

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(String username);
}
```

구현하지 않아도 메서드가 동작한다

만일 프로퍼티명과 쿼리메서드 명이 다를 경우 동작하지 않는다 !

`JpaCreaAtion`
