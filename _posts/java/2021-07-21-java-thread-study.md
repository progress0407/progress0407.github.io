쓰레드 생성은 아래와 같다. 방식은 크게 두가지이다.

```java
class ThreadSapmle extends Thread {
  @Override
  public void run() {
    StringBuffer sb = new StringBuffer("Running ThreadSample ");
  }
}

public Class Main {
    public static void main(String[] args) {
        ThreadSample th = new ThreadSample();
        ThreadSample th2 = new ThreadSample();
        th.start();
        th2.start();
    }
}
```

### 동기화가 되지 않았을 때 발생하는 대표적인 문제

```java
public class ThreadMain {

  public static final int ThreadGenNum = 2;
  public static final int RetryNum = 5;

  public static void main(String[] args) {
    for (int i = 0; i < RetryNum; i++) {
      runThread();
    }
  }

  public static void runThread() {
    ArithmeticThread[] th = new ArithmeticThread[ThreadGenNum];
    CmmMemory cmmMemory = new CmmMemory();
    for (int i = 0; i < ThreadGenNum; i++) {
      th[i] = new ArithmeticThread();
      th[i].setThreadGenNum(ThreadGenNum);
      th[i].setCmmMemory(cmmMemory);
      th[i].start();
    }
    for (int i = 0; i < ThreadGenNum; i++) {
      try {
        th[i].join();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
    System.out.println("cmmMemory.cnt = " + cmmMemory.getCnt());
  }
}
```

아래는 적절하게 예시를 구현하지 못한 부분이다.
```java
public class CmmMemory {
  private int cnt = 0;

  public int getCnt() {
    return cnt;
  }

  public void setCnt(int cnt) {
    this.cnt = cnt;
  }

```

n 개의 쓰레드가 접근하는 공유 저장소를 생각하여 만들었다.


```java
public class ArithmeticThread extends Thread {

  public CmmMemory cmmMemory;
  public int ThreadGenNum;
  public static final int totalRepetitions = 12000;

  public void setThreadGenNum(int threadGenNum) {
    this.ThreadGenNum = threadGenNum;
  }

  public void setCmmMemory(CmmMemory cmmMemory) {
    this.cmmMemory = cmmMemory;
  }

  @Override
  public void run() {

    int repetitions = totalRepetitions / ThreadGenNum;

    for (int i = 0; i < repetitions; i++) {
      cmmMemory.setCnt(cmmMemory.getCnt() + 1);
    }
  }
}
```
위와 같이 되면 set과 get의 메서드에서 synchronized를 하더라도 Thread 2개가 각각에 접근 가능하여 synchronized가 제공하는 안정성을 보장받지 못한다.


CmmMemory에 아래 메서드를 추가하자.
```java
addCnt() {
	this.cnt++;
}
```

그리고 ArithmeticThread 의
```java
cmmMemory.setCnt(cmmMemory.getCnt() + 1);
```
위 코드를
```java
cmmMemory.addCnt();
```
해당 코드로 치환하자.

그러면 synchronized의 중요성을 알려주는 예시를 만들 수 있다.


동기화를 하지 않았을 시에는

![](https://images.velog.io/images/progress0407/post/5212be00-93ac-4c2b-a4e5-21d96e4543da/image.png)


위와 같은 결과가 나온다.

하지만 동기화를 하면
```java
public synchronized void addCnt() { ... }
```
![](https://images.velog.io/images/progress0407/post/c2406d32-6343-4587-b954-696f64972584/image.png)
데이터의 정합성을 지킬 수 있다.

메서드보다 더 작은 범위에서 동기화를 시킬 수 있다.

```java
private Object lock_001 = new Object();

	...

public void addCnt() {
  synchronized (lock_001) {
    this.cnt++;
  }
}

```


그리고 싱글 쓰레드로 하나씩 더하면서 120000000L 에 도달한 것과
2개의 쓰레드로 동기화를해서 같은 숫자로 도달한 것의 차이를 보겠다.

![](https://images.velog.io/images/progress0407/post/19cc9357-8a7b-4eca-a309-535a69a63776/image.png)

![](https://images.velog.io/images/progress0407/post/d2c4f5fe-c088-40cf-b601-d7ccca9e5f6a/image.png)

되려 더 느려졌다..

병렬 계산을 잘못 설계하면 위와 같이 쓰레드의 이점을 살리지 못하는 것 같다

빨라지지 못한 이유는 동기화 걸린 블럭에 한 쓰레드만 접근 가능하기에 싱글 쓰레드로 돌린 것과 마찬가지라 그런 것 같다

근데 10배 이상이나 차이가 날 줄이야.. 설계를 바꿔서 쓰레드가 각각 계산한 것을 서로 더하는 것으로 가보자.

---


설계를 바꾸어 보았다.

이번엔 하나씩 더할 때마다 쓰레드가 해당 메서드에 접근하는 것이 아니라 계산할 커다란 문제를 2개로 쪼갠 후 서로 각각 더한 후에 각 쓰레드가 구한 최종 합을 구하는 것이다.
** 단 오버헤드가 좀 낮아지면서 연산속도가 늘어서 더할 데이터는 10배로 했다.


![](https://images.velog.io/images/progress0407/post/d8a8f938-e5dc-4164-b2d9-73500eeccd2f/image.png)

![](https://images.velog.io/images/progress0407/post/f8007cd1-5f83-449a-a7e5-a6bde7013929/image.png)

![](https://images.velog.io/images/progress0407/post/40915454-01a8-49d1-b3f3-008d7a01f171/image.png)

![](https://images.velog.io/images/progress0407/post/cd83ca6c-3706-4216-9cb2-56b1d25e21f8/image.png)

쓰레드 갯수 1 ~ 8까지는 수행속도가 점점 늘어나며.. 16부터는 거의 같다.
