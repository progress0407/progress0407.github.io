---
layout: post
title: "싱글톤을 만드는 또 하나의 방법 :)"
subtitle: ""
date: 2023-01-29 02:00 +0900
categories: backend
tags: java
comments: true
---

싱글톤을 만들 때 Xxx Holder, Enum으로 만들어버리기 (이펙티브 자바) 등등... 다양한 전략이 있다.

어느 날... 필자는 하나의 생각이 떠올렸다.

아니 왜 Abstract 클래스로는 안 만든다는 거지...? 이 친구도 new 연산 못하잖아!

그래서 만들어 버렸다...!!

국내 최초 Abstract Class 싱글톤 !

```java
abstract class AbstractSingleton {

    private static ConcreteSingleton singleton;

    public static AbstractSingleton newInstance() {

        if (singleton == null) {
            singleton = new ConcreteSingleton();
        }

        return singleton;
    }

    static class ConcreteSingleton extends AbstractSingleton {

        private static int myCount = 0;

        public ConcreteSingleton() {
            myCount++;
        }

        public String introduceMe() {
            if (myCount >= 2) {
                throw new RuntimeException("어이쿠 ! 제 자신이 두 개 생성돼 버렸네요... 이런 일은 있을 수 없는데!!");
            }
            System.out.printf("안녕 난 싱글톤이얌 ^^ 내가 생성된 갯수는 %d 야 \n", myCount);
            System.out.println("만일 내가 2개 이상이라면 이 문장이 실행되지 않겠지?? ^^");
        }
    }
}
```

조금 민망하지만 위에 코드가 전부이다... :)
