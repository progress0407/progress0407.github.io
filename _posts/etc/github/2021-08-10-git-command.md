---
layout: post
title: Git 개념 & 명령어
subtitle: ""
categories: etc
tags: github
comments: true
published: true
---

# Git 개념 & 명령어

## Checkout

`git bran초`

- 브랜치를 생성함과 동시에 그 브랜치로 전환한다.
- 예) `git checkout -b number-to-alphabet`

## Push

저장소 만든 후 최초 push하기

- `git push -u origin master`
  - origin 저장소에 master 브랜치로 연결한다

## Pull

서버에서 당겨올 때 2가지 방식이 있다

- `git pull` = `git fetch; git merge FETCH_HEAD`
  - fetch는 서버에서 변화만 가져오고 내 로컬에는 반영하지 않음
  - FETCH_HEAD는 remote 저장소의 HEAD
  - 따라서 merge를 한다는 것은 방금 원격에서 가져온 변화와 병합하겠단 뜻

## Branch

브랜치 강제 이동

- `git branch -f {옮길 브랜치} {옮길 곳} `
- 예) ` git branch -f master HEAD~3`

> 출처: https://cornswrold.tistory.com/251

## Cherry-pick

- `git cherry-pick {타겟 버전}`
- 다른 브랜치의 특정 버전의 "변화" 만을 가져온다.
- 그동안의 스냅샷을 뜻하는게 아니다 !

## Rebase

- `git rebase {옮기고자 하는 위치}`
- base를 다시 설정한다
- 브랜치와 브랜치를 플랫하게 만들고 싶을때
- 한 브랜치의 끝을 다른 브랜치의 끝점에 연결한다

## Stash

- `git stash` 현재 변경 내용 임시 저장

- `git stash list` 임시 저장한 내용 목록 조회
- `git stash apply ` 가장 최근에 저장한 stash 불러오기
- `git stash clear ` stash 목록 모두 지우기
- `git stash apply stash@{특정번호} ` 특정 stash 가져오기

> 실수로 삭제한 statsh 복구하기 (터미널이 아직 열려있을 떄 사용 가능)  
> 출처: https://starkying.tistory.com/entry/restoring-git-stash

```
git fsck --no-reflog | awk '/dangling commit/ {print $3}' | xargs -L 1 git --no-pager show -s --format="%ci %H" | sort
```

hash를 찾은 이후 이후 아래 명령어 진행

`git stash apply {찾은 hash값}`

## git 상대경로 이동

- `git checkout HEAD {연산자}`

| 연산기호 | 설명                   |
| :------: | :--------------------- |
|    ^     | 직전으로 이동          |
|    ^^    | 전전으로 이동          |
|  ~[N번]  | [N번] 만큼 전으로 이동 |

## Reflog

- 명령어로 인해 실수로 hard reset 했을 때 등의 복구 수단
- git 명령어를 사용한 이력

# 기타 상황들

---

## 만일 push된 커밋내용을 --amend로 바꾸고자 한다면?

```bash
git commit --amend
```

로 커밋내용 변경후에 `push`를 하게 되면

![image](https://user-images.githubusercontent.com/66164361/140637406-83101919-3169-4e24-ab96-2136f18a2d47.png)

위와 같은 에러 메세지가 나온다

그래서 아래처럼 옵션을 붙여주자

```bash
git push -f
```

강제 수행이 된다

다만 이 명령어는 위험하니.. 최대한 커밋내역이 꼬일 일이 없는 혼자 쓰는 공간에서 사용하는게 좋을 것 같다
