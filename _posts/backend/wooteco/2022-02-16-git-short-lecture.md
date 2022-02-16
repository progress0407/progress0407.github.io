---
layout: post
title: "우테코 git 특강 하면서 작성한 회고"
subtitle: "..."
date: 2022-02-16 12:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 진행하면서 느겼던 사소한 것들을 옮겨보았어요

---

## 해맸던 포인트

---

> Git 특강 주소  
> https://github.com/woowacourse/retrospective/discussions/4

`Request changes` 를 못찾아서 굉장히 해맸었다..

알고보니 아래와 같은 화면에 숨겨져 있었던 것임...

![image](https://user-images.githubusercontent.com/66164361/154189910-3515352a-2c74-406f-bc2f-ee7053dfc617.png)

그리고 아래 블로그에 설명이 아주 잘되어있다

> http://blog.cowkite.com/blog/2003062358/

## 작성한 리뷰 내용

---

### 오늘 작업한 원본 저장소와, fork한 저장소, 로컬 컴퓨터의 관계와 필요성을 이해한만큼 작성해주세요.

- remote와 local만 upstream, downstream 관계가 형성되는 줄 알고 있었는데, 원본 저장소와 fork 저장소 또한 그 관계가 성립된다는 걸 알게되었어요

### 페어간 작업한 커밋 기록을 어떻게 가져왔나요? 문제를 해결한 방법을 공유해주세요.

- 저장소를 추가한 다음에 pull을 땡긴걸로 알고 있어요
  - 저는 관리자이기 때문에 어떻게 해결을 하셨는지 자세한 내역은 ㅎㅎ..

### 혹시 conflict 상황을 만났다면 어떻게 해결했나요? 충돌을 해결하고 나서 commit 기록은 어떻게 변화했나요?

- conflict가 나지 않아서 일부러 해당 상황을 연출했어요
  - 페어1, 2가 "각자" 다른 변경내역 커밋을 푸시, 그리고 pull request를 한다
- 먼저 페어1님 것은 머지가 되니까 그대로 진행하고 페어2님의 것은 request changes 요청을 별도로 드려서 conflict resolve 요청 드렸어요
- 그리고 페어 2님이 병합 후에 풀 요청하면 merge 가능 상태가 되니까 관리자 입장에서 merge를 수행했어요

### 오늘 미션을 진행하면서 느낀 점을 자유롭게 작성해주세요.

- 마지막 미션 재미있게 했어요 ㅎ
- 잠깐 해맨 부분이 있는데 같이 해결해서 즐거웠었거든요 하하

### 오늘 Git 특강에 대한 피드백을 부탁해요. 매운맛도 환영입니다!

- ㅎㅎ 재미있었습니다. 조금 더 어려운 주제로 해도 재밌을 거 같아요
- 오늘 사실 Objects? Tree 이런 부분이 흥미로웠었거든요 ㅠㅠ ㅎㅎ 감사합니다
