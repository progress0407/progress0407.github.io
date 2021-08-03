---
layout: post
title: newlec MySQL
subtitle: "..."
categories: etc
tags: db
comments: true
published: true
---

### MySQL

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
- 큰 자료형 : TEXT
