---
layout: post
title: "플레이그라운드, 넥스트 스탭 강의 정리"
subtitle: ""
date: 2021-11-07 15:30:00 +0900
categories: backend
tags: java
comments: true
---

## 플레이그라운드 자바 - 인프런

---

- `if`, `else` 예약어 뒤의 중괄호는 안써도 되는 상황(한줄)에서도 쓰는게 좋다

## 넥스트 스탭 박재성님 - 객체지향 & TDD

---

### TDD Cycle

- 1. Test Fails
  - 먼저 실패하는 테스트 케이스를 만든다
- 2. Test Success
  - (첫번째일 경우) 해당 테스트를 성공시킬 정도의 간단한 프로덕션 코드를 만든다
  - (그 이후)
- 3. Refactor
  - 프로덕션 코드를 리펙터링한다

### 그외

---

- 너무 자주 인터페이스를 만들지 말아라
  - 너무 자주 만들 경우 정리 되지 않는 인터페이스들이 남게 된다

### 자동차 경주 게임 구현 도중 indent

---

![image](https://user-images.githubusercontent.com/66164361/142017242-341e5226-6a91-4315-8b24-3c20a387dd42.png)

위와 같이 indent를 더 이상 줄이기 힘들다고 판단되는 곳까지 왔다

사실 본인은 indent를 1로 만드는 것이 더이상 불가능하다고 생각했다

그러나.. 갓텔리제이가 힌트를 주기를...

![image](https://user-images.githubusercontent.com/66164361/142017625-640925cb-3b66-4edd-aaec-632c10181fe8.png)

이것을 `do while`문으로 바꿔보지 않겠는가 라고 계시를 내려주었다

![image](https://user-images.githubusercontent.com/66164361/142017793-fd9fc0d8-0e70-4c9d-b5d0-863372e675ed.png)

역시 갓텔리제이다
