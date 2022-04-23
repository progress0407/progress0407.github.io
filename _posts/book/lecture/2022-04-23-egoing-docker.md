---
layout: post
title: 생활코딩 Docker
subtitle: "..."
date: 2022-04-28 21:00 +0900
categories: book
tags: lecture
comments: true
published: true
---

host: 기존의 환경
container: `host` 위에 설치된 각각의 격리된 공간의 실행환경

또 각각의 `container`에는 앱을 실행하기 위한 라이브러리와 실행파일 정도만이 있다

리눅스에는 이런 앱 실행 방법이 내장되어 있다, 이런 기술 이름을 Container 라고 한다 !

Docker: 부두에서 컨테이너를 다루는 노동자들

---

![image](https://user-images.githubusercontent.com/66164361/164895989-eab90776-eee1-4bb3-9b88-9725cc08178e.png)

> `Official` 이라고 표기되어 있는 것은 도커에서 공식적으로 관리하는 믿을 수 있는 이미지

만일 자신의 컴퓨터가 `Linux`라면 `Docker` 설치시 성능 저하는 따로 없으나

윈도우나 Mac OS 라면 그 위에 `Linux`를 가상으로 올리기 때문에 성능 저하가 있다

> docker images

위 명령어를 쳤을 때 에러가 나지 않는다면 도커가 설치된 것이다

---

`app store`에서 `program` 을 다운받아서 실행시키면 `process` 들이 된다

`docker hub`에서 `image` 를 pull 받아서 run시키면 `container`들이 된다

> 도커 다운
> docker pull httpd

---

도커 명령어 확인 하는 곳

> https://docs.docker.com/engine/reference/commandline/run/

> docker run [`OPTIONS`] IMAGE [`COMMAND`] [`ARG`...]

[`COMMAND`] 에는 `IMAGE` 안에서 실행할 명령어를 입력한다 !

> 도커 실행
> docker run https

> 도커 Process 확인
> docker ps

> 이름을 부여해서 실행
> docker run --name {컨테이너 이름} httpd
> docker run --name ws2 httpd

> 컨테이너 중지
> docker stop ws2

> 도커 프로세스 모든 목록
> docker ps -a

> 중지했던 컨테이너 다시 시작 (한번은 실행했던 것)
> docker start ws2

> 도커 로그 확인
> docker logs ws2

> 로그를 실시간으로 확인
> docker logs -f ws2

> 컨테이너 삭제 (당연히 현재 실행중인 것은 삭제 불가)
> docker rm ws2

> 현재 실행되고 있어도 한방에 삭제하고 싶다
> docker rm --force ws2

> 이미지 자체를 삭제
> docker rmi httpd

// i 는 이미지의 약자다

---

외부 Client 가 있고 서버는 Docker를 사용한다고 하자

Docker 의 외부는 Host고 내부는 Container다

각각은 File System과 Port를 가지고 있다

> docker run -p 80:80 httpd

앞의 80은 host, 뒤의 80은 container다

> ![image](https://user-images.githubusercontent.com/66164361/164896833-36759acf-f951-455b-8496-3581b3333619.png)

웹사이트로 들어가면 아래와 같은 응답이 온다

![image](https://user-images.githubusercontent.com/66164361/164898142-99829f4c-3533-4e20-9986-59f27a59ff1f.png)

---

### 웹서버에 html을 바꾸어보자

`localhost:port`로 접속하면 index페이지를 볼수 있다

> https://hub.docker.com/_/httpd

위 주소에 따르면

웹서버의 경로는 아래와 같다

> `/usr/local/apache2/htdocs/`

> 컨테이너 내부에 어떤 명령을 수행
> docker exec [`OPTIONS`] CONTAINER COMMAND [`ARG`...]
> ex) docker exec ws2 pwd

> 컨테이너 내부의 셸을 실행
> docker exec {컨테이너명} /bin/sh

위와 같은 경우 바로 셸 접속이 끊긴다

> 셸을 끊기지 않게끔 계속 실행
> docker exec -it {컨테이너명} /bin/sh

> 셸 나가기
> exit

본셸(`sh`) 은 기능이 다소 약하다

요새는 `bash shell` 을 쓰는 추세이다

> 배시 셸 실행
> docker exec -it {컨테이너명} /bin/bash

컨테이너의 덕목은 용량이 작은 것!

#### nano 설치!

> apt update
> apt install nano

---

## 호스트와 컨테이너 사이간 디렉터리 연결하기

> docker run -p 7080:80 --name ws2 -v /c/dev/test-projects/egoing/docker/htocs:/usr/local/apache2/htdocs/ httpd
> docker run -p 7080:80 --name ws2 -v C:\dev\test-projects\egoing\docker\htocs:/usr/local/apache2/htdocs/ httpd
> docker run -p 7080:80 --name ws2 -v /c/dev/test-projects/egoing/docker/htocs:/usr/local/apache2/htdocs/:z httpd

- `v` 볼륨을 `현재경로`:`원격경로`

- 그러나 난 어떤 짓을 해도 디렉터리 연결이 되지 않았다...

이점:

- 컨테이너를 제거해도 여전히 파일들은 호스트에 존재
- 버전 관리(`VCS`)
- 에디터로 편리하게 코드를 편집
- 호스트에서 백업 정책 수행

---

지식 지도 서말 !

> https://seomal.com/map/1/129
