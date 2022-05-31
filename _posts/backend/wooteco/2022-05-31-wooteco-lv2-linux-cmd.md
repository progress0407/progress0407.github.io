---
layout: post
title: "우테코 레벨2 리눅스 명령어"
subtitle: "..."
date: 2022-05-31 14:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

> 토미의 리눅스 수업내용을 정리합니다

- 파일을 로컬에서 서버로 전송한다

```sh
scp -i {key-file-name}.pem access_log.txt ubuntu@{ec2-ip}:/home/ubuntu
```

### sort

- `sort` 하고 `ctrl d` 하면 정렬이 된다

```sh
sort < input.txt > result.txt
```

### vi

```
vi input.txt
```

- i를 통해 입력 모드로 전환
- 숫자 목록 입력
- ESC를 통해 입력 모드 종료
- :wq!를 통해 저장 및 파일 닫기

1. 읽기 Reading - 4
2. 쓰기 Writing - 2
3. 실행 Executing - 1 ,, rwxr -xr-x 775..

## access_log.txt 에서 접속 URL 목록만 출력하기

```sh
cat access_log.txt | cut -d " " -f 3 | sort | uniq
```

연속된 문자열에 대해서만 중복제거함(uniq)
즉 아래의 경우는 문자열 제거가 안됨

```sh
cat access_log.txt | cut -d " " -f 3 | uniq | sort
```

```sh
history | grep 'docker'
```

```sh
cd /home/ubuntu/repository/jwp-shopping-cart/
```

# ./gradlew bootJar #jar 파일 생성

```sh
cd build/libs
java -jar jwp-shopping-cart-0.0.1-SNAPSHOT.jar &
```

포어 그라운드
터미널을 점유중인 작업

<-> 백그라운드

jobs: 백그라운드에서 작업 중인 항목을 볼 수 있음
fg %1 저것 중에 첫 번째

sighup

### kill 명령어

| a   | b       | c                                                          |
| --- | ------- | ---------------------------------------------------------- |
| -9  | SIGKILL | 무조건 종료, 강제 종료시키는 시그널                        |
| -15 | SIGTERM | Terminate의 약자로 가능한 정상 종료시키는 시그널 (default) |

## 퀴즈 풀이

### 배포하기

```sh
#!/bin/bash
git clone # 클론하기
git clone -b {브랜치명} --single-branch {원격 저장소명} . # 원격 저장소 받아오기
cd {로컬 저장소} # 로컬 저장소로 이동
./gradlew bootJar # jar 파일 만들기
cd {jar 파일 위치한 디렉터리} # hint: /build/libs
nohup java -jar {자르파일} >> application.log 2>&1 & # 실행
```

### URL 추출 및 중복 제거

```sh
cat access_log.txt | cut -d ' ' -f 3 | sort | uniq
```

### 역순 정렬

```sh
cat access_log.txt | cut -d ' ' -f 3 | sort -r | uniq
```

# 그 외 크루들에게 공유 받은 자료

`shell script`에서 현재 프로그램 pid 찾아서 kill 하는 법

```sh
pid=$(pgrep -f jwp-shopping-cart)

if [ -n "${pid}" ]
then
        kill -9 ${pid}
        echo kill process ${pid}
else
        echo no process
fi
```
