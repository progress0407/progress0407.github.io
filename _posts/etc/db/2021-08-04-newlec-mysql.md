---
layout: post
title: newlec MySQL
subtitle: "..."
categories: etc
tags: db
comments: true
published: true
---

## MySQL

- 터미널접속하기 : 시작 > mysql > `MySQL 8.0 Command Line Client`
- `show databases` DB목록 조회
- 설치후 service.msc에서 MySQL80 (default) 로 서비스목록에 있는 것을 확인할 수 있다
- 마스터db는 `mysql`이다. (사용자계정 등)
- sakila, world 는 sample db (공부용)
- `use {DB명}` 해당 db 사용
- `show tables` 생략
- `create database[또는 schema] {db명} default character set utf8`
- `create user '{사용자id}' identified by '{비번}'` %를 생략하면 %로 됨 (외부 접근 허용)
- `grant all privileges on {테이블명}.* to {유저명}`
- 테이블만들기 > database탭 > reverse engineer
- 큰 자료형 : `TEXT`

---

## 그외 메모한 것

- `select @@autocommit` autocommit 여부 조회 (1 사용, 0 미사용)
- `set autocommit = {0|1}` 셋팅

- mysql 재시작

  - 종료 : `mysqladmin -uroot -p shutdown `
  - 시작: `mysqld`
  - 위 명령어보다는 서비스 > MySQL80 을 재시작하자

- mysql 터미널 접속 : `mysql -u root -p`

  - 이후에 비번 입력할 것

- 윈도우에 DB설치시 my.ini 위치 확인하기

  - 리눅스에서는 my.conf, 윈도우에서는 my.conf 이다
  - `show variables where variable_name like "%dir";`
  - `datadir` 항목 확인
    - 필자의 경우 `C:\ProgramData\MySQL\MySQL Server 8.0\Data\` 에 위치하였다

- 외부에서 접근 가능하게 하기

  - `my.ini` 파일을 열어서 `bind-address={private ip address}` 로 기입하자
    - 루프백 주소는 접근이 안된다 !
    - 수정후 MySQL을 재시작해야 한다

- 위 디렉터리는 보안관련 접근이 제한된다

  - 윈도 관리자 권한 수정 `netplwiz`
  - 위 방법으로도 안되기에.. 바탕화면에 파일을 복사하여 수정하고 원 디렉터리에 넣는 방식으로 해결하였다

- 외부에서 접근하기

  - `mysql -h 192.168.56.1 -u root -p`
  - `ERROR 1045 (28000): Access denied for user 'root'@'DESKTOP-6N1GO2A' (using password: YES)`
    여전히 안된다..

    ![image](https://user-images.githubusercontent.com/66164361/128830752-4ac2642e-53c7-4d2b-83f0-a8005a0196f5.png)

!! 드디어 되었다  
원인은 루트 계정이 `'%'` 가 아니었기 때문...

- 마지막 관문이 완성되었다
  - 리눅스 배포 + 게스트 호스트 mysql 접근 확인 + 스케쥴링 확인
    - 게스트 : vmware의 내부 os
    - 호스트 : vmware 밖의 내 os

> 참고: https://idchowto.com/?p=11068

## 테이블 생성

> 본 강의 내용은 굉장히 텍스트로 옮기기 힘들기에... 강의 링크만 붙이겠습니다.

ReverseEngineering을 통해서 뒷단에서 작업후  
Forward-를 통해서 앞으로 가져올 수 있다

Notice 테이블 생성 스크립트

```sql
CREATE TABLE IF NOT EXISTS `newlecture`.`notice` (
  `id` INT NOT NULL,
  `title` VARCHAR(45) NOT NULL,
  `content` TEXT NULL,
  `regdate` DATETIME NULL,
  `hit` INT NULL DEFAULT 0,
  `pub` BIT NULL DEFAULT FALSE,
  PRIMARY KEY (`id`))
ENGINE = InnoDB;
```

![image](https://user-images.githubusercontent.com/66164361/129566325-780351cd-79b7-4d95-a403-a6e421068dcc.png)

테이블 이름을 아무리 부분 대문자로 바꾸려 해도 되지 않음

- lower_case_table_names=1 을 바꾸어도 마찬가지이다
- 이유는 유닉스기반 파일시스템과 달리 윈도우에서는 대소문자를 구분하지 않기 때문
