---
layout: post
title: "[김영한 - JPA 실전 활용 2편] 웹 애플리케이션 개발"
subtitle: "..."
date: 2022-03-01 20:00 +0900
categories: backend
tags: jpa
comments: true
---

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
