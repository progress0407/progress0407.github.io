---
layout: post
title: "우테코 자주 사용되는 GIT 명령어들"
subtitle: "..."
date: 2022-02-17 14:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

## 페어 친구 짝 헤어지구 난 후 git 사용법

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

### 다른 크루의 코드를 clone하고 싶다면 ?

---

```
git clone -b {브랜치명} {git 저장소 URL}
```

> 예) `git clone -b step1 https://github.com/progress0407/java-racingcar.git`

### 1단계 -> 2단계로 넘어가기

---

1. 기존 브랜치 삭제 (하지만 나는 삭제하지 않는다 !)

> git checkout progress0407
> git branch -D step1

나는 위를 아래로 대체한다 !

> git checkout progress0407

2. 저장소 별칭 포함 추가

- 보통 원본 저장소는 `upstream` 이라고 칭한다

> git remote add upstream https://github.com/woowacourse/java-racingcar.git

확인

> git remote -v

3. 방금 추가한 저장소에서 자기 브랜치 가져오기 !

> git fetch upstream progress0407

확인

> git branch -a

4. rebase로 동기화

> git rebase upstream/progress0407

5. step2 브랜치를 생성 후 체크아웃

> git checkout -b step2
