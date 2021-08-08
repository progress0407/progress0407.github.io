---
layout: post
title: "Java에서 알아낸 Tip들"
subtitle: ""
date: 2021-08-07 13:30:00 +0900
categories: backend
tags: java
comments: true
---

### DTO를 작성할때 실제 테이블의 컬럼이 너무 지저분하다면..

> 실제 SM 등 운영을 하다보면 원장 테이블 컬럼들이 굉장히 지저분하여 DTO로 작성하기 꺼림직한 것들이 있다..

이럴 경우 실제 테이블의 모든 컬럼을 받는 DTO를 하나 만들고 내가 진짜로 쓸 DTO를 만들어서 상속을 받자

예시는 아래의 코드다

```java
public class MyMain {
    public static void main(String[] args) {
        PrintingDomain pd = new PrintingDomain();
        pd.print();
    }
}
```

```java
public class PrintingDomain {
    void print() {
        SubDomain sd = new SubDomain();
        Arrays.stream(sd.getClass().getDeclaredFields()).iterator().forEachRemaining(field -> System.out.println("field = " + field.getName()));
        System.out.println("sd.name = " + sd.name);
        System.out.println("sd.name = " + sd.phomeNumber1);
    }
}
```

```java
public class SuperDomain {
    String name = "super name";
    String phomeNumber1 = "super phomeNumber1";
    String phomeNumber2 = "super phomeNumber2";
    String phomeNumber3 = "super phomeNumber3";
}
```

```java
public class SubDomain extends SuperDomain {
    String name = "my sub name";
    String phomeNumber1 = "my sub phomeNumber1";
}
```

~~저 코드를 작성하다 보니 회사에서 테이블볼때의 안좋은 추억과 함께 화가 난다 ...~~

실제로 상속받은 DTO 객체가 상위 객체를 Override한 것을 볼 수 있다
