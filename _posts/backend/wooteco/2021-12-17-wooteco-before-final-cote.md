---
layout: post
title: "우테코 최종 코딩테스트 준비"
subtitle: "..."
date: 2021-12-17 20:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

# 시험 준비를 위한 전략을 모았다 !

---

## 약한점 보강해야할 점 (시험을 위한 목적)

---

- 자바 API 정규식 작성 능력
  자바의 정규식 작성능력이 다소 떨어진다고 느꼈습니다. 좀 더 연습이 필요할 것 같습니다 !

- 도메인과 Repository의 역할 분할

- 마지막 자판기 잔돈 반환 로직
  필자가 논리력이 약해서 잘 수행하지 못했었던 부분이다 ㅠ
  연습이 필요할 것 같다 !1

### (고찰) 과연 `controller` 가 필요할까?

내가 아는 컨트롤러는 서비스와 달리 비즈니스 로직을 구현하지 않는다

지인분의 말을 빌리면 *서비스 도데인 결과 전달과 HTTP 인터페이스에 맞는 변환 코드 정도만 갖는게 맞다*고 한다

그렇다면 순수 자바 로직에서는 굳이 Controller는 필요하지 않은 것 같다..

따라서 시험에서는 controller 를 굳이 구현하지 않고 service 만으로 해결해 나가자 ! ㅠ

### 완벽한 예외처리..?

최소한의 지켜야할 코딩 컨벤션과 이 정도는 지켜야 그나마 OOP를 하려고 노력했구나 하는 구조가 있다

그런데 모든 예외 처리를 하면서 시험을 치룬다...? 논노데스다.. 그럴 수는 없다 현실적인 목표를 두자

## 그래서 준비할 것은?

---

### 초기 세팅

- java8인지 확인

  - 터미널에 `java -version`

- build.gradle UTF-8로 변경

> build(build.gradle): UTF-8 인코딩 네이버 핵데이 컨벤션 적용

```
compileJava.options.encoding = 'UTF-8'
compileTestJava.options.encoding = 'UTF-8'
```

- git bash로 테스트 코드가 빌드 되는지 확인

  - 터미널은 한글이 깨지는데 아직까지 이걸 고치지 못했다 ㅠㅠ

- docs(`README`): 요구사항 작성

### 현실적인 코딩 순서

1. 우선 기존의 방식대로 구현
2. 리펙터링 (재구조화)
3. 예외 케이스 Validater 구현
4. 테스트 케이스 작성

### 코딩 팁

앵간하면 자료형은 성능은 신경쓰지말구
순서가 있는 자료형으로 사용하자

- Set: TreeSet, LinkedHashSet

  - 이때 TreeSet 은 정렬할 때 `Comparable` 에러가 나기도 한다

  ```
  vendingmachine.domain.Item cannot be cast to java.lang.Comparable
  ```

  조치법은 `Comparable`을 상속받아서 `compareTo` 메서드를 구현하면 된다
  (추측) `TreeSet` 이 내부적으로 자료를 정렬할때 `Comparable` 인터페이스의 `compareTo`를 사용하는 것으로 보인다

- Map: LinkedHashMap
- List: LinkedList

구현은 InputView 중심으로 하도록 하자

이렇게 해야 무엇을 구현해야할 지 상기가 잘 된다 !

### `Subway`를 다시 풀어 보니...

순수 기능 구현만 해보니 약 3시간 40분 정도가 걸렸다

`InputView`에서 다른 구현 화면으로 넘기는 것이 쉽지 않았다

그리도 넘어간 화면에서 현재 화면으로 `DTO`를 넘겨줘야 하는 부분을 떠올리는 것도 오래 걸렸다

map에 넣은 값을 지연평가하는 것도 넣어줘야 한다

그렇지 않으면 `if` 문으로 둘러 쌓인 코드들로 범벅이 될 것이다
