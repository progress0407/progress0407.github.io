---
layout: post
title: "우테코 늦게 다시 해보는 도커 및 DB 연동"
subtitle: "..."
date: 2022-04-24 22:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

이 글을 작성하는 시기는 우테코 레벨 2를 한창하고 있는 시기로.. 상당히 늦은 기간에 이 블로그를 작성하게 됩니다.. ^^

제 게으름 때문이죠 ㅠㅠ 흑흑

---

## 도커 수업 정리한 내용

1. yml 파일을 `프로젝트 위치/docker` 에 작성

![image](https://user-images.githubusercontent.com/66164361/164979968-684ab61d-2b85-4f81-878c-6500ff8297b2.png)

MySql 8.0.17 ~ 8.0.19 은 `Member` 예약어 이슈 때문에 이 버전의 MySql은 피하는게 좋다

```yml
version: "3.9"
services:
  db:
    image: mysql:8.0.28 # 버전
    platform: linux/x86_64
    restart: always # 컴터 껐다 켜질때 자동으로 부팅되는가?
    ports:
      - "13306:3306" # 호스트 포트 : 컨테이너 포트
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: woowa-chess
      MYSQL_USER: woowa-chess
      MYSQL_PASSWORD: 1234
      TZ: Asia/Seoul
    volumes: # 도커를 종료하면 휘발이 됨. 영속화하기 위함
      - ./db/mysql/data:/var/lib/mysql
      - ./db/mysql/config:/etc/mysql/conf.d
      - ./db/mysql/init:/docker-entrypoint-initdb.d
```

2. 도커실행

> docker-compose -p chess up -d

`-p` : 프로젝트 이름

`up` : 실행하겠다

`-d` : daemon, 백그라운드로 실행하겠다

3. 실행 결과

![image](https://user-images.githubusercontent.com/66164361/164980068-5612ef3f-e75a-425b-9923-4abd2ab2bf22.png)

![image](https://user-images.githubusercontent.com/66164361/164980119-efcb7980-92c4-473a-ae7c-1deaee9609e0.png)

![image](https://user-images.githubusercontent.com/66164361/164981051-9ab8779c-f97f-44f5-8aa0-5202020780fa.png)

4. `.gitignore` 에 아래 항목을 추가

> 아래와 같은 내용 마저 리뷰어에게 검토를 받을 필요는 없다 !!

```
### Docker ###
docker/db/mysql/data/
docker/db/mysql/init/*
!docker/db/mysql/init/init.sql
```

---

제이슨은 DB Client Tool 을 `Sequal Ace`를 사용하는 것 같다

> docker ps
> 사용중인 컨테이너를 조회

> docker exec -it chess_db_1 bash  
> 셸 기반으로 대화한다

> Error: No such container: chess_db_1  
> 이름이 잘못 된 것! 이 이름이 아닐 수 있다

`_`를 `-` 로 바꾸어 보자

> docker exec -it chess-db-1 bash

> mysql -u root -proot
> mysql 접속
> 띄어쓰기를 하면 안됀다고 한다!

> show databases;

`Public Key Retrieval is not allowed` 라는 오류 메시지가 뜰 경우 아래와 같이 설정해보면.. 된다 (필자는 !)
![image](https://user-images.githubusercontent.com/66164361/164981401-55cc1fac-30cb-428c-b529-cb7bce235200.png)

---

`build.gradle` 에서 아래 의존성을 추가

```
runtimeOnly 'mysql:mysql-connector-java:8.0.28'
```

---

## 커넥션

> show processlist;

![image](https://user-images.githubusercontent.com/66164361/164982040-1d6c702f-9c0b-425c-a472-e5a74761885a.png)

하나는 데몬이고 다른 하나는 커넥션을 물고 있는 자신이다 !

참고로 필자는 아래와 같이 나온다 !

![image](https://user-images.githubusercontent.com/66164361/164982125-43896574-561b-4798-9015-67fdbb75e393.png)

![image](https://user-images.githubusercontent.com/66164361/164982235-667cf394-34cf-4f65-927b-171fc7721d0e.png)

db 커넥션을 물고 있으면 아래와 같이 하나 더 생기는 것을 확인 가능

![image](https://user-images.githubusercontent.com/66164361/164982224-92751338-26cd-48a0-8a31-a13af5b77362.png)

---

## DDL 쿼리

![image](https://user-images.githubusercontent.com/66164361/164982336-d2259d3e-808a-43ab-81e2-aa97fc99489f.png)

위 위치에 `SQL` 문을 만들 수 있다 !

![image](https://user-images.githubusercontent.com/66164361/164982475-ee53cd0c-70c7-4d40-85b8-0075ee601cb8.png)

> show create table member;
> 테이블 DDL 확인

---

![image](https://user-images.githubusercontent.com/66164361/164982876-9d9e925e-c266-4636-90a4-4d9cf2eb9ac6.png)

---

## 기타

> https://www.slipp.net/questions/276
> 포비의 jdbc driver 등록 추적
