---
layout: post
title: "우테코 레이싱카 후기 & 포스팅"
subtitle: "..."
date: 2022-02-10 02:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 레이싱 카를 만들면서 겪게된 후기를 작성합니다

### 자바의 정규식 Pattern, Matcher.. 주의사항!

`matcher.group()` 전에 최소한 `find()`를 한번은 호출해야 한다 !!

### 페어 친구 짝 헤어지구 난 후 git 사용법

---

로컬 저장소는 하나, 원격 저장소는 두 개인 경우다

먼저 페어의 원격 저장소를 추가한다

```
git remote add [페어의 베이스 브랜치] [페어의 원격 저장소 URL]
```

> 예) `git remote add woong7 https://github.com/woong7/java-racingcar.git`

그리고 페어의 remote로부터 branch를 가져온다

```
git pull [페어의 베이스 브랜치]
```

> 예) `git pull woong7`

### 다른 크루님의 브랜치 염탐하는 git 커맨드 !

---

```
git clone -b {브랜치명} {git 저장소 URL}
```

> 예) `git clone -b step1 https://github.com/progress0407/java-racingcar.git`
