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

> 스프링부트는 어떻게 다중 유저 요청을 처리할까? (Tomcat9.0 Thread Pool)  
> https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests  

> BIO, NIO Connector Architecture in Tomcat  
> https://velog.io/@jihoson94/BIO-NIO-Connector-in-Tomcat  
