---
layout: post
title: "[wooteco] 쓰레드 활용하기"
subtitle: "..."
date: 2022-09-06 22:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

쓰레드 활용하기

# 0단계 - 스레드 이해하기

## ThreadTest

쓰레드는 아래의 두 가지 방법으로 구현 가능하며 각각의 방법에는 장단점이 있다

- Thread를 상속

장점: 쓰레드를 구현하기 편하다

단점: Java는 오로지 하나의 클래스만 상속이 가능하므로 다른 것이 상속이 불가하다

- Runnable을 구현

장점: 인터페이스이므로 다른 것을 상속받을 수 있는 여지가 있다

단점: Thread에 비해 다소 불편

## SynchronizationTest

synchronized 블럭에 대해 다룬다

- 메서드에 걸 수 있다

```java
public synchronized void calculate() {
    synchronized (this) {
        setSum(getSum() + 1);
    }
}
```

- 메서드에 부분적으로 걸 수 있다
  - 동기화가 필요한 부분에만 설정 가능

```java
public void calculate() {
    synchronized (this) {
        setSum(getSum() + 1);
    }
}

Object lock = new Object();
public void calculate() {
    synchronized (lock) {
        setSum(getSum() + 1);
    }
}
```
 
## ThreadPoolsTest

- pool size: 현재 활성화된 쓰레드의 개수
- queue size: 작업 큐에 대기하고 있는 쓰레드의 개수

### 상기하면 좋을 점!

> https://leeyh0216.github.io/posts/truth_of_threadpoolexecutor/  

동시에 작업할 건이 60개, pool Size가 10개, max pool size가 50개, queue size가 100개일 때

보통 생각해보면 thread pool 사이즈가 10개에서 50개까지 늘어나고 queue에 10개가 적재가 될 것이라고 생각할 것이다
(필자도 그랬으니까)

그러나 실험 결과는 그렇지 않다고 알려주고 있다

실제로는 queue에 대기열이 다 찰 때까지 Thread Pool을 늘리지 않는다고 한다

따라서 queue size는 100개이고 작업할 건이 그보다 작으니까 Thread Pool의 개수는 늘어나지 않는다
ㄱ
> Thread Pool을 늘리려면?

queue size가 차는 것이 Thread Pool이 늘어나는 조건이므로 queue size 10정도로 줄인다

그러면 max pool size만큼 동작하는 것을 확인할 수 있다

# 1단계 - 동시성 이슈 확인하기


```java
private void join(final User user) {
    if (!users.contains(user)) { // #1  기존 회원이 있는지 확인후
        users.add(user); // #2  가입을 한다
    }
}
```

쓰레드 모드로 디버깅을 하는 순간 #2에 첫번째로 도달하는 쓰레드를 포획한다.  
이 포획된 쓰레드가 현재 자신이 바라보고 있는 브레이크 포인트에 걸린 쓰레드이다.  
(즉 이 쓰레드에 한정해서 step out 등으로 조작할 수 있다)  
이 쓰레드를 Th-A라고 하고 나머지 한 쓰레드를 Th-B라고 했을때,  
Th-A는 디버거로 찍었기 때문에 개발자가 기다리는 만큼 기다리게 된다.  
이때 Th-A에서 찍은 user는 아직 add가 되지 않았기 때문에 Th-B가 바라보는 users에는 user가 없다.  
따라서 이경우 Th-B는 user를 add하게 된다.  
Th-A는 유효성 검증을 이미 통과하였기 때문에 결국에는 user를 등록하게 된다  

# 2단계 - WAS에 스레드 설정하기

- threads.max
  - 톰캣의 맥스 쓰레드 수
- accept-count
  - 작업 큐의 개수
- max-connections
  - 최대 연결의 갯수



# 기타 알게된 내용 

> https://stackoverflow.com/questions/24678661/tomcat-maxthreads-vs-maxconnections  

7.0은 BIO를 Default로 사용하나, NIO가 모든 면에서 더 낫다고 얘기됨
8.5부터 완전히 NIO 기반

> https://ehdvudee.tistory.com/30  

개인 메모: Tomcat 튜닝 가이드
한글로 되어 있어서 좋다 !

# NIO에 대해 알게 된 내용들


> BIO vs NIO에 대한 직관적인 설명이 들어 있다
> https://stackoverflow.com/questions/4752130/java-thread-per-connection-model-vs-nio

> 의역  

```
In a HTTP server, most connections are keep-alive connections, they are idle most of times. It would be a waste of resource to pre-allocate a thread for each.

HTTP 서버의 경우 대부분의 연결은 connection을 유지한다. 그리고 커넥션과 유지된 쓰레드는 유휴 상태이다. 개별 쓰레드에게 커넥션을 미리 할당하는 것은 자원 낭비이다.

For MMORPG things are very different. I guess connections are constantly busy receiving instructions from users and sending latest system state to users. A thread is needed most of time for a connection.

그러나 `MMORPG` 의 경우는 매우 다른데, 유저로부터 명령을 수행 받거나, 서버의 최신 상태를 유저에게 끊임없이 보내야 하기 때문에 커넥션이 매우 바쁘다고 추측할 수 있다.

If you use NIO, you'll have to constantly re-allocate a thread for a connection. It may be a inferior solution, to the simple fixed-thread-per-connection solution.

NIO의 경우 끊임 없이 쓰레드에게 커넥션을 재할당해야 하기 때문에 간단한 고정 쓰레드 방식에 비해 열등한 방법일 수 있다.

The default thread stack size is pretty large, (1/4 MB?) it's the major reason why there can only be limited threads. Try reduce it and see if your system can support more.

기본 쓰레드 사이즈는 1/4MB 정도나 된다. 이것이 쓰레드를 한정시키는 주요 이유이다. 이걸 줄이려고 노력하고 시스템이 지원할 수 있는지를 보아야 한다.


However if your game is indeed very "busy", it's your CPU that you need to worry the most. NIO or not, it's really hard to handle thousands of hyper active gamers on a machine.

그러나 게임 서버가 실제로 매우 바쁘다면, CPU를 가장 먼저 걱정해야 한다. NIO에 관계 없이, 컴퓨터에서 수천 명의 활동적인 게이머의 요청을 처리하는 것은 정말 어렵다.
```

> NIO, BIO에 대한 basic, core한 설명
> https://stackoverflow.com/questions/55871787/what-is-fundamental-difference-between-nio-and-bio-in-tomcat

폴러가 이벤트를 받아서 쓰레드에게 전달해주고, 읽고 처리하고 쓰기 작업이 다 되면 폴러에게 다시 돌려준다


# 참고

> 스프링부트는 어떻게 다중 유저 요청을 처리할까? (Tomcat9.0 Thread Pool)  
> https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests  

> BIO, NIO Connector Architecture in Tomcat  
> Connector의 변경사항 (9.0 dropped the BIO Connector)
> https://velog.io/@jihoson94/BIO-NIO-Connector-in-Tomcat  

> 오라클 동기화 번역본  
> https://www.daleseo.com/synchronization/  

> ThreadPoolExecutor를 오해하고 사용한 것에 대해 작성  
> https://leeyh0216.github.io/posts/truth_of_threadpoolexecutor/

> 이곳을 통해 maxThread를 설정하는게 worker thread를 설정하는 것으로 알 수 있다  
> https://github.com/apache/tomcat/blob/8301307ce76540e92c6068d8277038d133da862c/java/org/apache/tomcat/util/net/AbstractEndpoint.java#L1120

> Worker에 대한 설명이 있다, 설명이 많고 어렵  
> https://blog.actorsfit.com/a?ID=00500-fff6744c-2f08-436f-a448-90b8a9105166  

> NIO, BIO에 대한 심도 깊은 설명  
> https://velog.io/@hyunjong96/Spring-NIO-Connector-BIO-Connector  

> Tomcat Async 처리과정 디버깅  
> https://meteorkor.tistory.com/18  

> NIO를 코드 레벨에서 설명한다  
> https://anmolsehgal.medium.com/non-blocking-io-java-nio-b18e53a92bad  

> Netty 의 Worker 쓰레드 질문  
> https://groups.google.com/g/netty-ko/c/Pgtlp0R2zxk  