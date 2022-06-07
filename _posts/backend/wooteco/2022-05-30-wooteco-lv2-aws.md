---
layout: post
title: "우테코 레벨2 AWS - 배포"
subtitle: "..."
date: 2022-05-30 14:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

# 셋팅한 이후의 명령어

## ssh 접속

- 로컬에 key파일(`.pem`) 있는 곳으로 이동

```sh
ssh -i {자신의 키파일} ubuntu@{자기 인스턴스의 퍼블릭IP}
cd {프로젝트 폴더 이동}/build/libs
java -jar {프로젝트-SNAPSHOT}.jar &
```

## REST 명령어 실행해보기

```sh
curl -X POST "{자신의_인스턴스_주소:포트번호}/api/products" \
-H 'Content-Type: application/json' \
-d '{"name": "치킨", "price": 10000,"imageUrl": "test"}'
```

# 처음부터 셋팅하기

## aws 셋팅

- 인스턴스 시작
- 이름 및 태그에 `ec2-닉네임`
- OS 선택 ubuntu
- 인스턴스 유형(`성능`)에 t3.micro 선택
- 키페어 생성 눌러서 `key-닉네임.pem` 된 것 로컬에 보관
- 네트워크 설정에서 `TECHCOURSE` 선택 , 서브넷 정보는 `TRAINING`
- 퍼블릭IP 비활성화에서 활성화로 변경
- 방화벽 기존 보안 그룹으로 선택
- 보안그룹은 `SG_DEFAULT` 로

## key 등록하기

```sh
cd {자신의 키파일 있는 디렉터리 위치}
ssh -i {자신의 키파일} ubuntu@{자기 인스턴스의 퍼블릭IP}
```

## key 등록 오류 나는 경우 아래 명령어 실행

```sh
chmod 400 {key 파일}
wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add -
sudo add-apt-repository 'deb https://apt.corretto.aws stable main'
sudo apt-get update
sudo apt-get install -y java-11-amazon-corretto-jdk
```

## 레포지토리 가져오기

```sh
mkdir repository
cd repository
git clone https://github.com/woowacourse/jwp-shopping-cart.git
cd jwp-shopping-cart
./gradlew bootJar #jar 파일 생성
```

## 프로젝트 실행

```sh
cd build/libs
java -jar jwp-shopping-cart-0.0.1-SNAPSHOT.jar &
```

## 실행시 포트 번호 사용중일 경우

```sh
lsof -i :8080  # 8080으로 사용중인 포트 번호 검색
kill -9 {PID 번호}
```

> 예시  
> ![image](https://user-images.githubusercontent.com/66164361/170990794-e129f8dc-7460-4380-87b2-dd23c04b16d6.png)

## 프로젝트 실행 확인하기

```sh
curl -X POST "{자신의_인스턴스_주소:포트번호}/api/products" \
-H 'Content-Type: application/json' \
-d '{"name": "치킨", "price": 10000,"imageUrl": "test"}'
```

## 주의사항

절대 키 노출하면 안됀다
깃헙 푸시할때 키 노출하면 단속된다 !

# 그외 크루들에게 공유받은 자료

### 스컬이 준 자료

터미널로 ec2 접속할때마다 ssh 명령어 입력하기 귀찮으면 이거 한번 해보세요.
vi aws.c 를 입력해 aws.c라는 파일을 생성한다.
i를 누르고 아래 코드를 입력한다.

```c
 #include <stdio.h>
 #include <stdlib.h>

int main(void) {
     system("ssh -i key-skull.pem ubuntu@15.164.166.148");
     return 0;
}

```

3. 인증서와 IP를 자신걸로 바꾼다.
4. esc를 누르고 :wq를 입력해 나온다.
5. gcc -g aws.c -o aws를 입력한다.
6. ./aws를 입력해 자신의 인스턴스에 접속한다.

# AWS - 2

> 제이슨 수업

- 사이더 CIDR
- ping : ip 계층까지 요청이 가는지 확인하는 것 !
- telnet : port 번호까지!
- ping `-4` : Use IPv4 only.

![image](https://user-images.githubusercontent.com/66164361/171776472-72f2cd3c-5b6e-4e90-94f8-e17398171b93.png)

- `application.[yml/properties]`에 id/pw 정보는 비워두고
  - 서버가 run할 때 해당 id/pw 정보가 올라가게끔 해야 한다

![image](https://user-images.githubusercontent.com/66164361/171777396-4cddd2d1-d619-44b5-bbe4-d70d1b7d60bc.png)

![image](https://user-images.githubusercontent.com/66164361/171777406-0e31cc23-429c-4adc-86fa-b6e59ccd98f2.png)

# 제이슨

인스턴스를 껐다 켜도 데이터가 휘발되지 않게 만들기
페어당 DB EC2 인스턴스는 하나
이름: ec2-크루1-크루2-db
VPC: TECHCOURSE
서브넷: TRAINING
퍼블릭 IP 자동 할당: 비활성화
보안 그룹: SG-DEFAULT-DB
태그: Key: Role, Value: student
MySQL 접속 정보가 GitHub에 노출되지 않도록 관리하기
