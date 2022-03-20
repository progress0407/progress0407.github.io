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
