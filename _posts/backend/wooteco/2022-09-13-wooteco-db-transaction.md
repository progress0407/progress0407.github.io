---
layout: post
title: "[wooteco] DB 트랜잭션"
subtitle: "..."
date: 2022-09-13 22:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

## 생각...

과연 도메인 로직을 DB에 의존적이지 코드를 작성할 수 수 있을까...?


## 트랜잭션

- 논리적 작업 단위
  - Transactions are atomic units of work that can be committed or rolled back.
  - A transaction can be defined as a logical unit of work on the database (w3school)
  - A transaction is a single unit of logic or work, sometimes made up of multiple operations. (wikipedia)

## ACID 속성

- 원자성(Atomicity): 트랜잭션과 관련된 작업은 모두 실행되거나 모두 실패함
- 일관성(Consistency): 트랜잭션은 참조 무결성 등의 제약 조건을 위반하지 않고 유효한 상태에서 또 다른 유효한 상태로 변경함
- 격리성(Isolation): 트랜잭션은 다른 트랜잭션이 존재하지 않는 것처럼 서로 간섭 없이 수행되어야 함
- 지속성(Durability): 트랜잭션 커밋 후에는 시스템이 중단되거나 장애가 발생해도 데이터가 그대로 유지되어야 함


## 격리 수준
다른 트랜잭션이 변경(Insert, Update)한 내용에 대해 어디까지 볼 것인지에 대한 설정

- Repeatable Read
  - 진행중인 트랜잭션내에서 항상 동일한 데이터 조회 (insert 된 데이터 제외)
- Serializable
  - 진행중인 트랜잭션에 락을 걸어서 다른 트랜잭션가 못읽게 한다

## `innoDB` 에서는 `Repeatable Read` 수준에서 `Phantom Read`가 발생하지 않는다
- 트랜잭션 별로 snapshot을 찍어서 관리해서요!
- 실제 데이터가 아닌 언두로그에 있는 데이터를 읽는다
- 언두 백업 데이터에 고유번호가 있어요
- Consistent read 트랜잭션 내부에서 non-locking read(기본 SELECT 구문) 실행할 때, 동시에 실행중인 다른 트랜잭션에서 데이터를 변경하더라도 특정 시점의 스냅샷(snapshot)을 이용하여 기존과 동일한 결과를 리턴할 수 있도록 해주는 기능.
- `MVCC` 로 검색해보자~~

# 인덱스

- B트리: 하나의 노드에 n개의 key를 가지고 있고, 자식 노드를 가지고 있고 균형을 가지고 있다
- innoDB는 B+ 트리를 사용한다
- 노드랑 페이지를 같은 의미로 사용하기도 한다
- InnoDB의 PK는 clustered index를 사용한다

## hash 테이블을 기준으로 인덱싱하지 않는 이유

- 성능은 b-tree보다 훨씬 빠르다 !  

B-Tree가 더 나은이유  
- 범위 조건 검색이 된다
- 정렬되어 있으니까.. index 기준으로 정렬이 빠르다(order by)  
- like 검색이 가능하다  

> 참고  
> https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html#btree-index-characteristics  

## (self) 인덱스를 사용하면 UPDATE가 느린 이유

- UPDATE는 DELETE한 이후에 INSERT를 하게 된다


## 인덱스 구분
- primary vs secondary
- unique vs non-unique
- clustered index
- InnoDB의 경우 아래 컬럼을 clustered index 로 사용
  - primary key
  - unique & not null column
  - 내부적으로 생성한 GEN_CLUST_INDEX (6byte)

### clustered index
- 데이터를 저장할 때 인덱스 기준으로 정렬한 순서대로 저장

### Cardinality
- 값이 얼마나 다양하게 저장되어있는지 척도


### 인덱스를 사용할 수 없는 경우
- <>, NOT IN, NOT BETWEEN, IS NOT NULL
  - 기 정렬된 순서와 다른 방식으로 서치
- LIKE '%??'
  - 와일드카드가 앞에 오면 문자열의 앞부터 읽을 수 없기 때문에 인덱스 타지 못하는 것 + B Tree
- SUBSTRING(index, 1, 1), DAYOFMONTH(index)
  - 이미 저장된 구조를 가지고 가공을 해서
- WHERE char_index = 10, WHERE int_index = '10'
  - 타입 때문에 인덱스를 가공한다
- INDEX (first_column, second_column) 일 때 WHERE second_column = some_value
  - 첫번째 컬럼 -> 두번째 컬럼인데... 첫번째 컬럼이 없어서 인덱스 검색 x
  - WHERE first_collumn in ('a', 'b') and second_column = some_value


## NginX

- NIO의 셀렉터랑 비슷해보인다..

한 쓰레드가 IO가 발생할 때까지 계속 기다리는 것이 아니라, Queue에 담겨서 이벤트가 발생했을 때만 일을 한다 !


## 퀴즈 문제에서

int_index = '10';
위의 경우는 인덱스를 탄다고 하는데 우측항의 문자가 숫자로 형변환되어서 그런 것일까??
인덱스 사용시 자동 형변환이 일어나는 것 같다
숫자형이 문자형보다 우선순위가 높아서 숫자형으로 형변환되는 것으로 보임

> 인덱스 사용시 묵시적 형변환  
> https://imnkj.tistory.com/54  
> https://findmypiece.tistory.com/227  
