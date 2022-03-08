---
layout: post
title: "[스터디할래] whitesheep #11 Enum"
subtitle: ""
date: 2022-03-08 11:00 +0900
categories: backend
tags: java
comments: true
---

> 알게된 것. 난 그동안 Enum을 이넘~이라고 발음했는데 알고보니 이늄이라고 발음 한다.. (...)

### Enum은 `Enum` 이라는 클래스를 상속해서 만들어진다

따라서 기본으로 제공하는 메서드들이 있다.

| 메서드    | 항목                                                                                          |
| --------- | --------------------------------------------------------------------------------------------- |
| name()    | Enum으로 생성된 각 인스턴스의 이름을 호출한다                                                 |
| values()  | Enum으로 생성된 모든 인스턴스를 배열로 불러온다                                               |
| ordinal() | Enum에 정의된 각 인스턴스의 순서를 가져온다, 사실 자바 내부적으로 사용하기 위해 있는 API 이다 |

### Enum은 사실 Abstract Class 다

> 자바의 정석

```java

public class ImitationEnumMain {

    public static void main(final String... args) {
        System.out.println("ImitationEnum.FOO = " + ImitationEnum.FOO);
        System.out.println("ImitationEnum.BOO = " + ImitationEnum.BOO);
    }

    static abstract class ImitationEnum {
        public static ImitationEnum FOO = new ImitationEnum("푸우") {
            @Override
            void doSomething() {
                System.out.println();
            }
        };

        public static ImitationEnum BOO = new ImitationEnum("뿌뿌") {
            @Override
            void doSomething() {
                System.out.println();
            }
        };


        private final String name;

        public ImitationEnum(String name) {
            this.name = name;
        }

        abstract void doSomething();
    }
}
```
