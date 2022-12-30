---
layout: post
title: "마지막 미션, 모라고라(체크메이트) 마이그레이션"
subtitle: "..."
date: 2022-12-20 10:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

## 개요

마지막으로 개인적으로 주어진 미션이 있다.

우아한테크코스에서 했던 팀 프로젝트를 마이그레이션 하기.

테크코스가 끝나고 정말 귀신같이 매일 잠과 휴식을 탐하면서, 부족한 비타민을 채우듯이 쉬었다.

후반으로 갈 수록 재미보다는 힘들다는 느낌이 컸었던 것 같다.

지금은 완충이 되어, 백업 및 EC2 생성 정도만 해두었던 팀 프로젝트를 옮기고자 한다.


## 팀 프로젝트 옮기기 전 사전 준비

> 이건 오늘 이전에 이미 했었던 과정이다.

### 1. 파일 백업

서버는 대략 20개가 조금 안 되었다. 

이렇게 많은 서버를 가진 이유는 아래와 같다.

- 우선 프로덕션과 개발 서버를 분리가 되었다.

- 그리고 WS 셋업된 EC2가 Front와 Back을 나누어 있다.

- 로드밸런싱된 2대의 WAS가 있다. 

- 별도의 소나큐브 서버 (굉장히 무겁기 떄문에 따로 분리)

- Jenkins, CI 서버 2대 (Front, Back)
  - 이것 역시 무겁기도 하고, 특히 프론트 배포시 서버가 거의 멈추는 일까지 일어난다.

- 프로메테우스 및 그라파나 (성능 분석 및 시각화)

- 로커스트 (부하 생성기)

... 이러다 보니까 서버가 무려 20개 가까이 되었다.

따라서 자잘한 프로그램 셋업 과정이 전체 다를 백업하지는 못하고 내가 손대지 않은 부분만 History로 뺴왔다.

그리고 젠킨스 같은 경우 공을 들인 부분이 많아서, shell script의 경우 거의 다 백업해왔다.

### 2. EC2 생성

- 개인 계정으로 서버를 생성했다.


## 마이그레이션 하기

### 도커?

혹시나 그럴 일이 있을 지는 모르지만 필요한 팀원이 있을 경우, 서버 통쨰로 제공할 수 있으면 좋을 것 같다는 생각에 도커를 기반으로 시작했다.

그러나 정작 필자가 도커에 대해 잊은 사항이 많아서 복습을 하면서 진행했다...

### 도커 이미지 고르기

처음에는 WAS가 필요하면 JDK가 설치된 도커를, Nginx가 필요하면 이에 관한 공식 이미지를 찾았다.

개인적으로는 이런 경우 불편했던 것이... Ubuntu가 아닌 다른 무엇으로 설치되는 경우가 있었다. 이게 상당히 불편하다.

특히 데비안 계열이 아닌 경우 특정 프로그램을 사용하지 못하는 것도 있고, 시스템의 특성상 설치가 안 되는 프로그램도 있었다.

따라서 `Ubuntu:22.04` 에 해당하는 이미지를 베이스로 작업하기로 결정했다.

### 여러 대의 도커 컨테이너?

이것 또한 관리상 불편하다고 생각 되었다. 컨테이너를 분리해서 관리하면, 이미지로 백업해두어서 서랍장 분리하듯 역할이 보여서 편할 거라 생각했다.

그러나 이럴 경우 컨테이너 간 네트워킹을 해야 점도 있고 이미지 자체가 분할 되어, 이미지 간의 연관 관계를 생각하면서 작업해야할 것 같단 생각이 들었다.

**따라서 서비스 하나에 대해 하나의 컨테이너만을 사용하기로 결정헀다**

> 참고
> https://www.daleseo.com/docker-networks/

## 도커 관련 사항

docker를 다루면서 고통받은 일들을 기록하였다.

## 컨테이너에 포트 추가하기 by Version-Up

이게... stop/start (생성된 컨테이너 on off) 명령어로는 포트를 추가할 수 없다.

따라서 컨테이너를 버전 업 하는 방식으로 진행해야 한다. (정확한 표현은 아니고 느낌은 이러했다)

```bash
docker stop {컨테이너 명} # 컨테이너 이름이 없다면, 컨테이너 ID(해시)로 대체 가능
docker commit {컨테이너 명}
# ex
docker commit server progress0407/checkmate:2022-1226-1
```

> 참고
> https://medium.com/sjk5766/%EC%8B%A4%ED%96%89%EC%A4%91%EC%9D%B8-container%EC%97%90-port-or-volume-%EC%B6%94%EA%B0%80-ae8889344c68

여기서 생성된 이미지 기준으로 `run` 명령어를 통해 새로운 컨테이너를 추가하면 된다

## 도커 네이밍

이것 역시 컨벤션이 있다. 아래와 같다.

`{docker hub id}/{repository name}:{version}`

나의 경우 아래로 네이밍 했다.
`progress0407/checkmate:2022-1222-1`

`docker hub id`: progress0407이 나의 도커 허브 ID이다
`repository name`: 이 부분이 참 애매하다... 리포지토리, 프로젝트 최상단 명칭이 moragora인데... 그러나 마지막 프로젝트명인 checkmate로 했다
`version`: latest 혹은 22.04 등으로 명시하지만... 나는 일자 + 일련번호로 했다. 일련번호는 해당일에 대해 추가 커밋이 있는 경우 +1을 하기로 했다.

> [참고, 태그로 rename](https://miiingo.tistory.com/332)

## MySQL 관련 사항

- [한글, 이모티콘 입력 가능한 DB 만들기](https://zibsin.net/1754)

```sql
create database `checkmate-db` default character set utf8mb4 collate utf8mb4_general_ci;
```

사용자 생성

```sql
create user `checkmate-user`@'%' identified by {비번};
```

권한 부여 (편의상 모든 권한을 주었다. DDL 마저도...)

```sql
grant all privileges on `checkmate-db`.* to `checkmate-user`@'%';
```

## HTTPS 적용 문제 (in Docker)

Docker를 사용하기 위해서 Certbot란 프로그램을 설치해야 한다.

그러나 이 과정이 순탄하지 않다.

여러가지 시도를 해보았고 마지막 시도로 지금까지의 이미지를 docker-compose 기반으로 certbot와 함께 새로운 컨테이너를 만드려고 하였다.

그러나 결국은 인내심의 종지부를 찍었다 ㅠㅠ...!!

![image](https://user-images.githubusercontent.com/66164361/209717747-90d8f629-3ec0-4251-96b1-38892df56854.png)

![image](https://user-images.githubusercontent.com/66164361/209717810-11b35077-75e3-42ec-9c0d-757a73838886.png)

생각한 그림은 한 컨테이너 내부에 WS, WAS, Redis 등 모든 것을 넣는 것이었다.

컨테이너가 n개 생성된 시점에서부터 포기하기로 하였다.

## Redis

레디스 기본 포트: `default 6379 and additionally 16379`

- [Redis Session 스토리지 이용](https://escapefromcoding.tistory.com/702)
- https://stackoverflow.com/questions/7537905/how-to-set-password-for-redis

## Git 전략 (?)

결론부터 말하면 포크 프로젝트에 neo라는 새로운 브랜치를 만들어서 진행했다.

기존에는 Git 전략과 Github 전략을 혼합하여 사용했다.

현재는 포크 프로젝트의 `neo`란 브랜치를 만들어서 사용하기로 했다.

(`neo`는 새로운이란 뜻으로 생각해서 사용)

이렇게 한 이유는 매번 소통이 발생하면 굉-장히 오랜 시간을 들여서 작업해야할 수 있기 때문이다...

사실상 팀프로젝트가 끝나서 이렇게 하기로 했다.

## Submodule 변경 사항

기존에 있던 prod, dev, local을 최대한 덜 수정하는 방향으로 작업하기로 했다.

대신에 ec2에서 운영할 `neo-prod`와 로컬에서 사용할 `neo-local` 설정을 추가하였다.

## Nginx 사항

### unknown directive "proxy_path" in {설정 경로}

**결론, 오타가 있었다. `path` -> `pass`**

예제를 따라하면서 작업했었는데, proxy_path라는 예약어를 인식하지 못하는 것 같았다.

- https://stackoverflow.com/questions/36011581/nginx-unknown-directive-proxy-pass

아래글을 따라서 설정을 건드려보았지만 잘 되지는 않았다.

`proxy_path`가 아닌 `proxy_pass`였다

> host not found in upstream "localhost80" in {설정 경로}

이제 메시지가 변경되었다.

자세히 보니 또 오타가 있었다. `localhost80`

수정후 정상 재기동되었다.

### Nginx 403 Faild

필자의 경우 권한문제였다

`chmod`, `chown` 을 체크해보자!

nginx가 찾는 경로 중 어느 하나라도 권한 문제가 있을 경우에 정상적으로 File을 Serving못할 수 있다!


## 프론트 앱 배포

- 프론트 루트 디렉터리에 .env파일에 아래 정보를 기입한다.

```
API_SERVER_HOST={API 서버 URI}
CLIENT_ID={구글 OAuth 관련 ID}
```

```shell
npm run build
# npm run start  # 이러면 nginx를 거치지 않고 바로 배포
```

## 문제 사항

### ec2 먹통 현상

이해가 되지 않는 현상이다. 상당히 높은 빈도로 CPU 과부화 되면서 멈춘다.

혹시 모르지만 docker의 영향이 있던 걸까? WAS, DB, Redis등 종합적으로 비대해지다 보니 이런 일이 발생한 걸까? 아직 알아가는 중이다.

### HTTPS 적용 후 접근 불가

처음에는 80 포트에 접근이 불가했다. 뒤이어서 22번 포트도 접근이 불가능해서 인프라 작업을 할 수 없었다.

아직 원인을 파악하지 못했기 때문에 알아보는 중이다!
