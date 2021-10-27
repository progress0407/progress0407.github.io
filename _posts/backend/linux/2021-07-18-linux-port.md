---
layout: post
title: "윈도우, Linux : 포트 확인 및 중지"
subtitle: "..."
categories: backend
tags: linux
comments: true
published: true
---

> 개발을 하다 보면.. 특정 서비스를 중지하고 싶을 때가 있다. 그럴때 유용하다 !

---

### 포트 확인 및 중지

- ### Window

  - `netstat -ano` 포트 확인
  - `netstat -ano | findstr {특정포트 넘버}` 특정 포트 확인
  - `taskkill /f /pid {PID 넘버}` pid를 key로 하여 중지

- ### Linux
  - `netstat -tulpn | grep java` 포트 확인 (요게 더 편하다 !)
  - `netstat -nap | grep 8080` 포트 확인
  - `lsof -i TCP:80`
  - `kill (-9) {PID 넘버}` -9를 입력시 강제종료이다
