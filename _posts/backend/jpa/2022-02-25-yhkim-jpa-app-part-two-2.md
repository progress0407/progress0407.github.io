---
layout: post
title: "[김영한 - JPA 실전 활용 2편] API 개발과 성능 최적화"
subtitle: "..."
date: 2022-03-01 20:00 +0900
categories: backend
tags: jpa
comments: true
---

# (Hibernate) 성능 최적화

#### JPA에서는 그림과 같이 id가 같으면 동일한 객체로 취급된다

![image](https://user-images.githubusercontent.com/66164361/156551814-b0f62a5e-a14b-4bdb-989e-f3eb0005e3b7.png)

#### 페이징 처리를 DB가 아닌 WAS에서 처리한다

![image](https://user-images.githubusercontent.com/66164361/156553666-fe4f4851-a4ef-46a6-8da7-f743b683bc5f.png)

자칫 잘못하면 **out of memory** 가 발생할 수 있다

#### 조인을 두 번 연달아 하면 .. 1 + N + N\*M

![image](https://user-images.githubusercontent.com/66164361/156557890-79efb3d7-6535-44ab-9f4b-5d360be70e86.png)

ㅎㅎㅎ

나비효과마냥 `Order` 1개로 이루어진 것이  
Order 하나에 대해 `OrderItem` 2개,  
각각의 `OrderItem`에 대해 2개씩 총 `Item` 4개의 쿼리가 발생한다

> 총 1 + 2 + 4개의 쿼리 발생

### `fetch join` vs `batch size`

1. `fetch join`

v3 방식에서 사용한 것  
한번에 조인해서 모두 가져오지만 컬럼 로우수가 비대해져서 데이터가 커진다

2. `BatchSize`

v4 에서 사용한 것  
한번에 가져오지는 못하지만 최적의 컬럼수를 가질 수 있어 데이터가 작다  
대신 조인된 엔티티를 가져와야 하므로 엔티티 만큼의 쿼리는 추가적으로 나간다 (대신 `IN` 절로 나간다)  
페이징이 가능하다 !

### 결론

`--ToOne`은 페치조인  
`--ToMany`(컬렉션)는 전역 batch size 설정으로 해결하자 (`hibernate.default_batch_fetch_size`)

### BatchSize

보통은 전역으로

```
jpa.properties.hibernate.default_batch_fetch_size: 100
```

이렇게 적는게 좋다 좀 더 구체적으로 접근하고자 한다면...

1. Item : OrderItem 이 일대다 단방향 관계라고할 때

`Item` 위에 `@BatchSize` 위에 걸어주면 된다

2. Order : OrderItem 이 일대다 양방향 관계일때

Order 클래스의 프로퍼티 `OrderItem` 위에 어노테이션을 걸어주면 된다

#### batch size 크기

맥시멈은 `1000`개 미만으로 잡자, 천개가 넘어가면 오류가 나는 DB들이 있다고 한다

### 컬렉션 조인 최적화

- order 조회 → orderItem을 `IN`절로 조회

### 플랫 데이터 최적화

- JOIN 결과를 그대로 조회한 후 애플리케이션에서 원하는 모양으로 직접 변환

## (컬렉션 조회) 실무 권장 순서

- 엔티티 조회 방식으로 우선 접근 (v1 ~ v3.1)

  - 페치 조인으로 쿼리수 최적화 (v3, v3.1)
  - 페이징이 필요할 경우 `batch size` 이용

- 위에서 해결이 안 될 경우 DTO 조회 방식 이용 (v4~v6)
- 그걸로도 해결이 안 되면 NativeSQL, 스프링 JdbcTemplate 이용

## OSIV

- Open Session In View

- 엔티티의 영속 상태를 어디까지 살려둘 것인지

트랜잭션이 시작되면 `Entitiy Manager`가 `영속성 컨텍스트`를 열면서 `DB Connection` 을 가져온다  
(그러면서 `setAutoCommit(false)` 를 같이 보낸다! )
이 때 OSIV가 켜져있으면 `Presentation Layer` 까지 커넥션을 물고 있게 된다.  
하지만 이 덕분에 컨트롤러에서 지연 로딩하는 것이 가능하다

### OSIV ON

- 컨트롤러와 화면단까지 영속 상태가 살아 있다
  - 따라서 해당 레이어에서 지연 로딩 가능
  - 굳이 Query를 나누지 않아도 된다.
- DB 커넥션 자원을 빨리 반환하지 못한다

### OSIV OFF

- 트랜잭션 종료시 영속 상태를 종료
- 컨트롤러 레이어, 뷰 등의 지연 로딩을 다 서비스 안쪽으로 넣어야 한다
- 더 이상 컨트롤러에서 프록시 초기화가 안 된다 !
  - `org.hibernate.LazyInitializationException: could not initialize proxy [jpa.app.shop.domain.Member#1] - no Session` 발생

### 커맨드와 쿼리 분리

- OSIV를 끈 상태로 App을 동작시켜주게끔 보조해준다
- 복잡한 쿼리는 Command와 Query를 분리한다

- OrderService

  - 쿼리가 단순하다
  - 하지만 핵심 비즈니스에 영향이 있다 (생성, 수정, 삭제)

- OrderQueryService
  - 핵심 비즈니스에 큰 영향을 주지 않는다
  - 그렇지만 쿼리가 복잡하기 때문에 성능 최적화가 중요하다
