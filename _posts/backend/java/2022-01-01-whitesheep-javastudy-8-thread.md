---
layout: post
title: "[스터디할래] whitesheep #10 쓰레드"
subtitle: ""
date: 2022-01-01 17:00:00 +0900
categories: backend
tags: java
comments: true
---

# 금주의 공부할 내용 !

- Thread 클래스와 Runnable 인터페이스
- 쓰레드의 상태
- 쓰레드의 우선순위
- Main 쓰레드
- 동기화
- 데드락

# `Thread` 클래스와 `Runnable` 인터페이스

- 기본적으로 구현은 `run()` 메서드를 오버라이드한다

- 실행부에서 `start()`메서드를 사용해서 호출해야 쓰레드가 별도의 실행 공간에서 실행된다

### `Thread` 클래스를 상속

```java
class MyThread extends Thread {
		@Override
		public void run() {
			System.out.println("hello my thread");
		}
	}
```

> 실행부

```java
MyThread myThread = new MyThread();
myThread.start();
```

### `Runnable` 인터페이스를 구현

```java
static private class MyRunnableImpl implements Runnable {
		@Override
		public void run() {
			System.out.println("hello runnable");
		}
	}
```

> 실행부

```java
MyRunnableImpl runnable = new MyRunnableImpl();
Thread th = new Thread(runnable);
th.start();
```

### 만일 start가 아닌 run 메서드를 이용한다면??

쓰레드로 실행이 되지 않는다

```java
class MyThread extends Thread {
	@Override
	public void run() {
		sleep(300);
		System.out.println(getName() + " hello my thread");
	}

	private void sleep(int millis) {
		try {
			Thread.sleep(millis);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

위와 같이 쓰레드에 딜레이를 걸어 놓는다

그리고 `run`메서드를 통해서 실행한다

```java
MyThread[] myThread = new MyThread[10];
for (int i = 0; i <10; i++) {
	myThread[i] = new MyThread();
	myThread[i].run(); // 이곳 !!
}
```

그렇게 되면 main 메서드에 순차적으로 0.3초씩 메세지가 쌓이는 것이 보인다

그리고 `start`메서드를 통해서 실행하면 0.3초 뒤 한번에 메세지가 일괄 출력된다

즉 `run`메서드는 쓰레드를 실행시키는 것이 아니라 단순히 클래스에 선언된 `run`메서드를 호출하는 것이다

반면 `start()`는 쓰레드가 작업을 실행하는데 필요한 call stack을 생성한 다음에

`run()`을 호출해서 방금 생성한 stack에 `run()`을 실행한다

# 쓰레드의 상태

| 상태          | 의미                                              | 관련 메서드             |
| ------------- | ------------------------------------------------- | ----------------------- |
| NEW           | 쓰레드 인스턴스를 생성하였지만 아직 실행되지 않음 | new                     |
| RUNNABLE      | 실행 중                                           | start()                 |
| BLOCKED       | 실행 중지, 즉 `monitor lock`이 풀리기를 기다림    |                         |
| WAITING       | 대기 중                                           | sleep(초) 혹은 wait(초) |
| TIMED_WAITING | 특정 시간만큼 대기 중                             | sleep() 혹은 wait()     |
| TERMINATED    | 종료된 상태                                       | join() 이후             |

\*\* 비고 WAITING, TIMED_WAITING 는
`Thread.sleep()` 메서드나 `monitor`객체의 `wait` 메서드로 대기할 때 나오는 상태이다

# 쓰레드의 우선 순위

- 자바의 신 내용을 기준으로 작성해보았다

- 우선순위의 기본값은 5이다 (`getPriority()`)

- 되도록 우선순위는 지정하지 않는것이 좋다고 한다

- 그래도 굳이 지정해야겠다면 `Thread` 클래스의 상수를 이용하자 (아래 그림)

![image](https://user-images.githubusercontent.com/66164361/147872787-a1bbc011-a4e2-4c1c-9915-d31137f0fbe8.png)

# 데몬 쓰레드

- 해당 쓰레드를 실행하는 컨텍스트가 종료되면 보통 쓰레드는 종료되지 않고 실행된다
  - 예를들어 `main()` 는 종료되었지만 쓰레드는 살아 있는 것
- 이때 `setDaemon(true)` 을 지정하여 같이 종료를 시킬 수 있다

# Main 쓰레드

    - main함수도 쓰레드이다 !!

# 동기화

어떤 Target 객체가 있고 이 객체가 상태를 가지고 있을 때

이 객체를 여러 쓰레드가 사용할 떄 동기화 문제가 발생할 수 있다

이것을 해결하기 위한 방법 중 하나로 `synchronized` 예약어를 이용할 수 있다

## synchronized

- 사용 방법은 크게 두가지가 있다
  - 메서드 자체를 synchronized로 선언하는 방법 (`synchronized methods`)
  - 메서드 내 특정 문장만 감싸는 방법 (`synchronized statement`)

## 싱글턴에서의 동기화 문제

필자가 작성한 글에서도 확인해볼 수 있다 .. ㅎㅎ

> https://progress0407.github.io/backend/2021/12/02/%EC%9E%90%EB%B0%94_%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-6-Singleton.html

# 데드락

교착상태이다

- 특정 조건이 갖추어지면 락이 걸린다

참고

- resumeQ, stopO, suspend(） 는 쓰레드를 교착상태 (dead-lock）로 만들기 쉽기 때문에 deprecated되었다．
