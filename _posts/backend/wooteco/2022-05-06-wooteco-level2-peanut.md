---
layout: post
title: "우테코 레벨2 땅콩 박스 만들기"
subtitle: "..."
date: 2022-05-65 00:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 지하철 미션 도중 만들어본 땅콩 박스 만들기를 기록한다 !

## 만들게 된 계기

우스갯 소리이긴 하였지만... 레벨2 강의를 듣는 도중..

지하철 미션 1 구현 도중 IoC 컨테이너가 없으면 어떤 점에서 불편할 지 경험해보란 뜻에서 `Spring Container` 사용을 금지하였다 (`@Autowired 등`)

# 현재 구현된 기능

## 콩(`Bean`)과 땅콩(`Peanut`)의 차이

| 항목                      | 스프링 콩                                                | 땅콩                        |
| ------------------------- | -------------------------------------------------------- | --------------------------- |
| 객체 저장소               | 빈(Bean) 컨테이너                                        | PeanutContext               |
| 빈을 자동 등록 어노테이션 | `@Component`                                             | `@Peanut`                   |
| 빈을 주입하는 기능        | `T annotationConfigApplicationContext.getBean(Class<T>)` | `T peanutContext(Class<T>)` |
| 빈을 주입받는 어노테이션  | `@Autowired`                                             | `@GiveMePeanut`             |
| 객체 생애주기             | `@Scope`                                                 | `@Peanut(scope)`            |

## 땅콩이 지원하는 가능한 생애주기

| 항목      | 설명                                           |
| --------- | ---------------------------------------------- |
| singleton | 애플리케이션의 종료시점까지 단일한 객체를 가짐 |
| prototype | 항상 객체를 생성                               |
| request   | HTTP Request 의존하는 생애주기                 |
| session   | 브라우저 등의 세션에 의존하는 생애주기         |

# 아쉬운 점...

- 오로지 필드 주입만 가능

  - 생성자 주입 불가
  - 수정자 주입 부락

- 아직도 의존하는 `ApplicationContext`
  - 정작 기존의 Bean Container 를 대체하고자 했는데.. 기존의 것을 사용한다 !
