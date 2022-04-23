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

## 다른 크루의 코드를 clone하고 싶다면 ?

---

```
git clone -b {브랜치명} {저장소 URL}
ex) `git clone -b step1 https://github.com/progress0407/java-racingcar.git`
```

> 만일 브랜치 하나만을 가져오고 싶다면

```
git clone -b {브랜치명} {저장소 URL} .
ex) git clone -b step2 --single-branch https://github.com/jinyoungchoi95/java-chess.git .
```

# 미션 진행 요약

---

> 코드리뷰 과정은 아래에 있다
> https://github.com/woowacourse/woowacourse-docs/tree/master/maincourse

## 1단계

---

1. 미션 시작

2. woowacourse Fork

3. 브랜치 가져오기

```
git clone -b {본인_아이디} --single-branch https://github.com/{본인_아이디}/{저장소 아이디}
ex) git clone -b javajigi --single-branch https://github.com/javajigi/java-baseball.git
```

4. 기능 구현 브랜치 생서

```
git checkout -b 브랜치이름
ex) git checkout -b step1
```

5. 기능 구현 add, commit

```
git status // 변경된 파일 확인
git add -A(또는 .) // 변경된 전체 파일을 한번에 반영
git commit -m "메시지" // 작업한 내용을 메시지에 기록
```

6. 자기 원격 저장소에 올리기

```
git push origin 브랜치이름
ex) git push origin step1
```

## 1단계 -> 2단계로 넘어가기

---

1. 기존 브랜치 삭제 (하지만 나는 삭제하지 않는다 !)

```
git checkout progress0407
git branch -D step1
```

나는 위를 아래로 대체한다 !

```
git checkout progress0407
```

2. 저장소 별칭 포함 추가

- 보통 원본 저장소는 `upstream` 이라고 칭한다

```
git remote add upstream {우아코스 깃 URL}
ex) git remote add upstream https://github.com/woowacourse/java-racingcar.git

git remote -v // 확인
```

3. 방금 추가한 저장소에서 자기 브랜치 가져오기 !

```
git fetch upstream progress0407
git branch -a // 확인
```

4. rebase로 동기화

```
git rebase upstream/progress0407
```

5. step2 브랜치를 생성 후 체크아웃

```
git checkout -b step2
```
