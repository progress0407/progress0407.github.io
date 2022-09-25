---
layout: post
title: "[stress-test] 우리 서버가 어디까지 버틸 수 있을까??"
subtitle: "..."
date: 2022-09-25 20:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

우리 서버는 어디까지 버틸 수 있을까?

현대의 tomcat은 BIO를 사용하던 당시와는 다른 방식으로 요청을 처리한다.

보다 I/O 처리에 유리한, 많은 요청을 처리하기 위해 보다 더 유리한 방식을 사용하기 위해 NIO를 사용한다.

다른 ec2에 비해 우리 서비스가 TPS가 잘 안나오는 것으로 생각되어 테스트를 좀 더 진행해 보았다.



## 우선 노트북의 어댑터를 끈 경우와 킨 경우에 대해 비교를 해 보았다

1. 어댑터 off :  500
2. 어댑터 on  :  680

어댑터를 꼈을 때가 36% 정도 성능이 up된 걸 볼 수 있었다.

(이때는 최적의 요청을 주지 않아서 생각보다 TPS가 많이 나오지 않았다)

## 로그가 영향으 주었는가? : 그렇지 않다

File I/O가 빈번히 일어나기 때문에, logback등의 로깅 라이브러리를 사용하더라도 과부하가 생길 것으로 생각했으나 그렇지 않았다.

-- 로컬 to 로컬 요청
1. no-log :  680
2. latest :  680

생각보다 로그가 영향을 주지는 않았다.

## 아무것도 없는 Spring-Boot와 비교

1. moragora + log :  680
2. 비어있는 spring-boot :  750

## 로컬에서 해본 가장 많은 TPS

실험을 하다 보니, 680이란 평균 숫자가 유독 많이 보이는 것을 확인했다. 
TPS를 보내는 방식이 `쓰레드(th) * 초당 요청횟수(req/s)`인데 th와 req/s 값에 따라 TPS가 다르게 나올 것 같다란 느낌이 들었다.

따라서 이것을 먼저 테스트 해보았다

본인 로컬에서 테스트를 한 것 기준으로는 가장 많은 요청을 보낼 수 있었던 TPS는 아래와 같다

![image](https://user-images.githubusercontent.com/66164361/192141008-0253e828-e58f-45c4-9cfa-28397ef6b5e9.png)

vUser를 500을 주었을 때 가장 높은 TPS를 가지는 요청을 보낼 수 있었다.

> 위의 결과를 토대로 다시 한 번 실험을 해보았다

# 재실험

## 로그 영향도

로컬 to 로컬

1. no-log version:  1000
2. log exist version:  1000

## 비어있는 spring-boot와 비교

1. log exist version :      1000
2. 비어있는 spring-boot:    1200~

의외로 놋북이 아닌 네트웤을 타더라도 TPS가 1000이 나온다!

---

# 재실험...

아래의 기준으로 다시 해보앗다

- 200 request/sec -> 800
- local to ec2 server
- no tunning:
  - min-thread: 100
  - max-thread: 200
  - acceptance-count: 8192

## WAS 직접 요청

- moragora의 /server-time 의 요청을 받은 경우
  - TPS: 400
- 비어있는 spring-project
  - TPS: 700
- moragora의 no-log 버전
  - TPS: 560
- ngin-x를 거쳐서 moragora를 간 경우!
  - TPS: peak를 500까지 찍었지만 fail이 뜨는 요청들이 많아지더니 서버가 down 되었다 ㅠ

![image](https://user-images.githubusercontent.com/66164361/192143221-b6c5981f-3641-4216-b74f-4697bcdf763c.png)

## BIO 테스트 

위와는 별개로 BIO에서 어떤 방식으로 요청이 처리되는지 확인하려고 했으나...

BIO를 default로 사용하고 있는 톰캣 버전은 7.0.x이다. 해당 버전을 사용하고 있는 가장 latest spring boot는 1.12로 확인이 되었다!

![image](https://user-images.githubusercontent.com/66164361/192144273-e4361797-473e-4cb5-b574-4e4f9aea0c12.png)
![image](https://user-images.githubusercontent.com/66164361/192144283-e6327feb-bcab-4611-b250-16dd055dd608.png)
![image](https://user-images.githubusercontent.com/66164361/192144292-a7c2c3f0-09c4-4c83-928e-6116123b9025.png)

수많은 취약점은 뒤로하고...
우선 build.gradle 로 import 하는 것부터 쉽지가 않았다 ㅠ