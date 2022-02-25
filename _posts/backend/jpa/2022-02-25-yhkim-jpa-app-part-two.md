---
layout: post
title: "[김영한 - JPA 실전 활용 2편] 웹 애플리케이션 개발"
subtitle: "..."
date: 2022-02-25 18:00 +0900
categories: backend
tags: jpa
comments: true
---

영한님은 `API`를 사용하는 컨트롤러랑 (`@RestController`)

`화면`을 위해 존재하는 컨트롤러 (`@Controller`) 가 있는 패키지를 아예 분리하신다고 한다

왜냐하면 각각의 컨트롤러에서 공통처리 자체를 다르게 해주기 때문에!

예를들어 `API` 컨트롤러는 예외가 나왔을때 Json으로 예외 메세지 처리를 하는데

`화면` 컨트롤러는 오류 페이지를 보여준다

![image](https://user-images.githubusercontent.com/66164361/155687599-382b5a3a-9bf2-4d68-9da3-6cb5368f1d92.png)

1. 화면 벨리데이션이 모델ㅔ 있다

2. 화면마다 API 호출방식 다를수잇다
