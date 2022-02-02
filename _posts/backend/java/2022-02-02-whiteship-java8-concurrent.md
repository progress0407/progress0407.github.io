---
layout: post
title: "자바8 (whitesheep) - Concurrent"
subtitle: ""
date: 2022-02-02 20:30 +0900
categories: backend
tags: java
comments: true
---

## 기본 쓰레드

---

- `main(..)` 도 하나의 쓰레드이다 !
  - ```java
      System.out.println(Thread.currentThread().getName());
    ```

### 쓰레드 선언

---

2가지 정도의 선언 법을 소개하셨다

1. 쓰레드 상속

> 선언부

```java
static class MyThread extends Thread {
		@Override
		public void run() {
			System.out.println(Thread.currentThread().getName());
		}
	}
```

> 사용부

```java
MyThread myThread = new MyThread();
myThread.start();
```

2. `Runnable` 주입

```java
Thread thread = new Thread(new Runnable() {
  @Override
  public void run() {
    System.out.println(Thread.currentThread().getName());
  }
});
thread.start();
```

아래와 같이 람다로 치환 가능하다

```java
Thread thread = new Thread(() -> System.out.println(Thread.currentThread().getName()));
```

### sleep, interrupt

---

```java
Thread thread = new Thread(() -> {
  while (true) {
    System.out.println("Thread: " + Thread.currentThread().getName());
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) { // 쓰레드를 종료시킨다기 보다 예외를 발생시키는 것 뿐이다 !
      System.out.println("catch ! " + e.getMessage());
      return;
    }
  }
});
thread.start();

System.out.println("Hello ! : " + Thread.currentThread().getName());

try {
  Thread.sleep(1000);
  thread.interrupt();
} catch (InterruptedException e) {
  e.printStackTrace();
}
```

`interrupt`를 했다고 해서 해당 쓰레드를 강제로 종료(`return`) 하지는 않는다

별도로 `return` 문장을 넣어서 종료할 수 있게끔 처리를 해주어야 한다

```java
Thread thread = new Thread(() -> {
  String threadName = Thread.currentThread().getName();
  System.out.println("Thread: " + threadName);
  try {
    Thread.sleep(2000);
  } catch (InterruptedException e) {
    throw new IllegalArgumentException();
  }
  System.out.println(threadName + " is now exit !");
});
thread.start();

System.out.println("Hello ! : " + Thread.currentThread().getName());

try {
  thread.join();
} catch (InterruptedException e) {
  e.printStackTrace();
}

System.out.println(thread + " is finished");
```

`join` 문을 넣으면 해당 쓰레드를 기다린다
