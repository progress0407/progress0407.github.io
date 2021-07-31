---
layout: post
title: "IntelliJ 팁"
subtitle: "Thread Sync ..."
date: 2021-07-26 22:15:00 +0900
categories: etc
tags: tip
comments: true
---

### 터미널에서 한글이 깨질때

![image](https://user-images.githubusercontent.com/66164361/126995355-750969de-8013-49f4-a66b-0cfc0d688226.png)
위와 같이 환경 변수를 편집합니다

![image](https://user-images.githubusercontent.com/66164361/126995221-3b8736d3-dafb-44d1-a8ed-c9aef01e1885.png)

위와 같이 설장합니다
Shell Path를 `cmd`에서 `git bash` 로 바꾸면 `Linux Shell`이 열립니다

![image](https://user-images.githubusercontent.com/66164361/126995518-cf651f50-fa76-495b-af04-4d5bab17d617.png)

위와 같이 한글이 정상적으로 동작합니다.

### 윈도우 cmd 꺠짐 char 수정

#### 일시적용

cmd에 `chcp 65001` : Change CodePage

- `65001` : UTF-8
- `949` : EUC-KR (한국어 전용)

#### 영구적용

- regedit
  - HKEY_CURRENT_USER
    - Console
      - %SystemRoot%\_system32_cmd.exe 키생성
        - CodePage 키 생성
          - 10진수로 65001 입력

![image](https://user-images.githubusercontent.com/66164361/127558391-940a277c-df24-47d6-81b6-0fb3d11f5d20.png)
