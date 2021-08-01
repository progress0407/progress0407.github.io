---
layout: post
title: "Linux tomcat 구동 : 배포"
subtitle: "..."
categories: etc
tags: linux
comments: true
published: true
---

공부하고 있는 배치 프록르램을 반영할 수 있어야 한다는 생각에 Linux를 다시 복습하게 되었다.

준비한 사항은 아래와 같다.

- VM 이나 클라우드 등에 리눅스 설치
- putty 등 ssh 클라이언트 셋팅
- ftp 설정
- jdk 전역화
- 톰캣 설정법 숙지 (server.xml)

---

8080 에서 80으로 서비스하려면 아래와 같이 해주면 된다.

- VM에 포트포워딩 항목 추가
- server.xml에서 port를 변경
- authbind를 설치하여 `byport/`에 80을 추가
- startup.sh의 실행하는 문장(`exec ..`)에 `authbind --deep` 문구를 추가

![image](https://user-images.githubusercontent.com/66164361/127560483-14ae748f-793a-487e-83ec-2571b46fef7f.png)

오라클 VM 기준으로 `설정 > 네트워크 > 고급 > 포트포워딩 `
위와 같이 추가하면 된다.

톰캣을 다운받아서 (tar.gz 파일) ftp를 통해서
`{현재유저}/download` 디렉터리에 넣어두자
그리고 `tar -xzvf {타겟 압축파일}` 을 통해 풀자
그리고 `/usr/local` 에 `tomcat` 디렉터릴 만들어서 하위에 옮긴다

`{톰캣 설치폴더}/bin/startup.sh` 을 실행하자
그리고 `w3m http://localhost:8080`을 입력하면

![image](https://user-images.githubusercontent.com/66164361/127560547-332ec3d3-0bb7-4925-8287-fc5050e7c0cb.png)

위와같이 웹에서 실행된 화면이 나온다
그리고 VM 밖의 브라우저에서 동일주소로 들어가면 아래와 같이 접속된다.

![image](https://user-images.githubusercontent.com/66164361/127560575-bfd0f200-ce50-42fb-928c-2945c02a93aa.png)

혹시나 접속에 문제가 있으면

서버랑 클라이언트를 재기동하면 된다.. 필자도 안돼서 재기동 했다 ㅠ

---

### 8080에서 정식으로 80으로 하고자 할 경우

톰캣이 켜져 있나 확인후 (아래 셋 중 하나로, 맨 위에껏 추천)

- `ps -ef | grep tomcat`
- `netstat -nap | grep 8080`
- `w3m http://localhost:8080`

>

#### 포트변경

`{톰캣 설치 dir}/conf/server.xml` 을 백업 후 `nano`로 연다

```
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

그러면 위에 있는 것이 포트 번호인데 80 으로 수정

>

#### authbind 권한부여

1. authbind를 `apt` 를 통해서 설치
   `/etc/authbind/byport/` 에 80을 추가
   `chmod 550 80` 으로 권한 부여
   `{톰캣 dir}bin/startup.sh`
2. `bin/startup.sh` 을 편집기로 연 후

```
exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

위 코드를

```
exec authbind --deep "$PRGDIR"/"$EXECUTABLE" start "$@"
```

으로 변경한다.

---

모든 과정을 했다면 아래와 같이 포트번호를 붙이지 않아도 (80이 기본값이므로) 아래와 같이 잘 접속 될 것이다

![image](https://user-images.githubusercontent.com/66164361/127560559-e64a682a-9113-4a72-af74-7f1957551568.png)

참고로 필자는 80으로 들어가면 톰캣이 아니라 작년에 생코에서 공부했던 mysql의 그 bitnami가 뜬다... 왜이럴까..

---

> 참고 :

- war 배포 및 젠킨스
  - https://blog.taejinfish.co.kr/2018/03/21/deploy-war-to-tomcat-manually.html
- Tomcat 디렉토리 구조
  - https://itwarehouses.tistory.com/8?category=727633

#### tomcat 로그 확인

`tail -f {톰캣 경로}/logs/catalina.out`

#### newlec쌤 mvc 9090포트 배포

![image](https://user-images.githubusercontent.com/66164361/127563354-b62ceaa5-ee15-4847-a8b0-8b73018ce4a3.png)

성공하였다 그러나 JDBC관련된 URI로 접근하면 ..?!

![image](https://user-images.githubusercontent.com/66164361/127563576-62c26759-1d74-4b77-961e-7c371094ee2b.png)

```
Request processing failed; nested exception is org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLRecoverableException: IO Error: The Network Adapter could not establish the connection (CONNECTION_ID=~~~)

서버가, 해당 요청을 충족시키지 못하게 하는 예기치 않은 조건을 맞닥뜨렸습니다.

org.springframework.web.util.NestedServletException: Request processing failed; nested exception is org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLRecoverableException: IO Error: The Network Adapter could not establish the connection (CONNECTION_ID=~~~)
	org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1014)
```

예측했던 에러다 이제 네트워크 공부가 살짝 필요하다 !!

#### IP : localhost, 127.0.0.1, 192.168.0.x

> https://ithub.tistory.com/180  
> http://gotocloud.co.kr/?p=320

- `loopback` 호스트에 할당된 IP

  - `localhost`, `127.0.0.1`
  - 서버를 띄운 자신의 기기에서만 접근 가능

- `사설IP` 같은 대역의 사설 IP를 할당받은 모든 기기 접근 가능

  - `192.168.0.x` C클라쓰
  - 쉽게 말하면 공유기의 범위 내

- `공인IP` 어디에서나 접근 가능
  - ipV4의 제한으로.. 모두가 다 쉽게 누릴 수 없음

> 결론  
> VM에서 내 로컬로 접근하게 하려면.. 사설IP로 만들자

#### VirtualBox의 네트워크 설정들 : NAT, Host-Only-Adapter

---

> https://cjwoov.tistory.com/11

##### 용어정리

호스트는 내 로컬 PC/노트북이고  
게스트는 VM  
인터넷은 말 그대로 외부이다

##### NAT

호스트에서 게스트에 접근이 `포트포워딩` 방식으로 접근이 가능하며
그 역은 안된다.
즉 내 VM에서 내 로컬에 설치된 오라클 서버로 접근 못하는 거지.. ~~(거지같당 ㅠ)~~

호스트에서 게스트로 접근할 떄는 전용 IP가 따로 있으며  
`ifconfig` 으로 확인가능하다

대신 게스트가 인터넷에 접근 가능하다
때문에 `apt-get install` 같은 명령어가 사용가능해 보인다

#### Host-Only-Adapter 호스트 접용 어댑터

사설 IP 기준으로 접근한다
덕분에 게스트와 호스트 서로 접근이 가능하다

대신에 게스트가 인터넷에 접근이 불가능하다

> 결론
> 당장에는 생존형 학습이 필요하니 호스트전용으로 설정하고  
> 이후 DB를 게스트 (VM - Linux)에 설치하여 진행하도록 하자

호스트 어댑터로 설정하면
`ifconfig` 을 통해 `192.168.56.x` 로 설정된 것을 볼수있다  
필자의 경우는 `101`번으로 부여되었다

![image](https://user-images.githubusercontent.com/66164361/127568336-a4f64604-d099-423d-926a-6394765c9d7d.png)

이제 요렇게 접근하면 된다 !

#### 오라클 .ora 위치

`C:\app\{사용자 계정명}\product\18.0.0\dbhomeXE\network\admin`

> 개념 하나 기록
> 기존의 VM 접속후 ifconfig 명령어로 알아본 10.0.2.15 는 사설IP이다..
> 우리가 공유기로 공유받는 IP대역 192.168.0.x 라는 사설IP 대역 안의 사설IP라 생각하면 되지 않을까 싶다.. 그리고 이 사설 IP로 들어가려면 포트포워딩을 통해서 접근이 가능하다  
> 또한 ssh포트로 파일전송도 가능하다

---

위 메모로부터 약 4시간 정도의... 삽질끝에..

와 !!! 드디어 되었다.. 정말 와 소리가 나왔다 ㅠㅠㅠ 땡큐베리감사..

![image](https://user-images.githubusercontent.com/66164361/127597934-b669829c-1e03-452b-876a-5bafac9e5760.png)

해당 리스트를 불러온다는것은 게스트가 호스트의 DB를 접속했다는 뜻 !

게스트, 호스트간 양방향 통신이 된것이다 !

어댑터를 동시에 2개 설정이 가능하단 사실을 알면 덜 해매었을 것 같다..

### 최종적으로 설정해야 헀던 것들

---

#### 안해도 되었던 것

- 방화벽 오라클 포트 열어준 것
- VM호스트 어댑터 설정을 내 노트북의 사설IP 대역이랑 같이 맞추어준 것 (192.168.0.101)

#### 해야하는 것

- 리스너 설정 : loopback -> 사설 IP
  - listener.org 에서 설정
  - service.msc에서 OracleOraDB18Home1TNSListener 재실행 (안해도 될 수 있다 !)
  - lisener와 tns가 있는데 listener는 서버쪽에서 클라이언트가 접속하는 통신 환경 설정이고
    tns는 클라이언트가 접속할 때의 환경 설정
    리스너는 반드시 설정되어야 하고 tns는 꼭 이 설정이 아니더라도 다른 방식으로도 접근 할 수 있다고 한다.
- 어댑터 2개 설정 (NAT-포트포워딩(SSH), 호스트어댑터전용)

> 파일 > 호스트 네트워크 관리자
> ![image](https://user-images.githubusercontent.com/66164361/127599683-705189da-c8b9-418c-b98c-0fa7afd4fc1d.png)

> 설정 > 네트워크
> ![image](https://user-images.githubusercontent.com/66164361/127599767-7df85937-d7e9-4835-8487-b38320785687.png) >![image](https://user-images.githubusercontent.com/66164361/127599814-a434d6c9-1446-4120-a9d2-85516b97c4fb.png)

> 참고한 글들  
> https://hjw1456.tistory.com/16 (이곳에서 양방향 접근이 가능하다는 실마리를 얻었다, 마치 페르마의 마지막 정리 같은 단서임..)  
> https://cjwoov.tistory.com/11 (네트워크 설정별 차이 - 체계적으로 알 수 있었다)
> https://blog.startsomething.dev/2018/10/26/virtualbox-%EC%97%90%EC%84%9C-nat-%EC%99%80-host-only-network-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/  
> https://sarc.io/index.php/oracledatabase/186-2014-06-10-01-33-05  
> https://qjadud22.tistory.com/20 > https://whitekeyboard.tistory.com/730 (여기서 어댑터 2개가 가능하단 것을 알았다)  
> https://blog.daum.net/techtip/12415054

-- OracleOraDB18Home1TNSListener
