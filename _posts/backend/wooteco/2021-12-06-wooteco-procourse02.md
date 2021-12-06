---
layout: post
title: "우테코 프리코스 2차 - 레이싱 게임"
subtitle: "..."
date: 2021-12-06 02:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

# 우테코 프리코스 2차 과제

---

1주차 과제를 무사히 제출하고 나서 2차 과제인 레이싱 게임을 만나 보았다 !!

## 이것두 역시 사실은 미리 풀어보았지만..

---

사실 이것 역시도 미리 풀어보았던 과제 중 하나였다 ! ㅎ

다음에는 아마 지하철 노선도가 나올 것 같지만..

일단 그동안 배운 객체지향 개념을 가지고 부지런히 리펙터링을 하면서 기록해본다

# 배운 점

## inputView에 대한 Strategy 패턴 적용

Random 을 처리하는 테스트하기 어려운 부분 뿐만 아니라

InputView 또한 테스트하기 쉬운 구조로 리펙터링해보았습니다

- `InputStrategy`
  - `ConsoleInputStrategy` 실제 App에서 돌아가는 입력 전략입니다
  - `FixedInputStrategy` 테스트코드에서 돌아갑니다

## MVC 구조에 추가적으로 `DI` 주입 방식 채택

스프링 프레임워크가 `DI` 컨테이너를 사용하여 객체들을 관리하듯이

`InputView`, `OutputView`등의 객체를 외부에서 주입하는 방식으로 변경해보았습니다.

## 문자열이 긴 것은 StringBuffer에 !

StringBuffer는 1. 쓰레드 세이프 2. 문자열 추가 연산에 최적화

## 예외가 나왔을때의 반복문

예외가 나왔을 때의 반복문에 대한 처리에 대해 고민을 해보았습니다

함수가 작게 분할이 되어 있었기 때문에 `return`문을 활용하여 다시 반복할 수 있었습니다
