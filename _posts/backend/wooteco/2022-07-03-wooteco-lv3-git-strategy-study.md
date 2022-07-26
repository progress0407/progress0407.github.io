---
layout: post
title: "우테코 레벨3 깃 전략 조사"
subtitle: "..."
date: 2022-07-03 19:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 깃 전략 조사하는 과정 중에 알게된 내용을 정리

### `git reset --hard HEAD` 를 해도 untracked files가 지워지지 않는 경우

`git clean -fd --dry-run`

## 로컬의 특정 브랜치를 원격 브랜치 내용으로 덮어 씌우기

### 1. 원격 브랜치 모두 최신화

`git remote update`

- 원격 브랜치 -> 로컬에 있는 remote tracking 브랜치로 최신화

### 1-2 원격 브랜치 하나만 최신화

`git fetch {브랜치명}`

### 1-3. 현재 브랜치 확인

`git branch -a`

- 브랜치 업데이트가 잘 되어있는지 확인

### 2. 로컬 브랜치 삭제

`git branch -d {브랜치명}`

### 3. 원격 브랜치 최신화

`git checkout -t origin/main`

### 내 현재 로컬의 refs 를 최신화

`git fetch --prune`

내 현재 로컬의 remote 브랜치 추적 정보를 최신화한다 !
