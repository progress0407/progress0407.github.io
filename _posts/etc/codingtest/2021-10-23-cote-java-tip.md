---
layout: post
title: 자바로 코테를 준비하는 팁들
subtitle: "..."
categories: etc
tags: codingtest
comments: true
published: true
---

## 자바로 코테 준비하기

자바는 파이썬에 비해서 준비할 때 난이도가 다소 높다..

자바로 주요 사용하는 코테용 문법들을 정리해보자

## 입력 받기

```java
        Scanner sc = new Scanner(System.in);

        out.print("n 입력: ");
        int n = sc.nextInt();
        int numbers[] = new int[n];


        out.print("배열 입력: ");
        for (int i = 0; i < n; i++) {
            numbers[i] = sc.nextInt();
        }

        out.println("=== 출력 ==");
        for (int i = 0; i < n; i++) {
            out.print(" " + numbers[i]);
        }

```

콘솔 입출력 예:

```
n 입력: 3
배열 입력: 1 2 3
=== 출력 ==
 1 2 3
```

## 배열

### 배열 정렬

배열을 정렬한다. 이때 원본 배열을 정렬한다. (`불변이 아니다`)

```java
Arrays.sort(splitArr);
```

### 배열 자르기

`from <= x < to` 와 같은 범위로 배열을 자를 수 있다. 이때 원본 배열은 불변 보장

```java
int[] splitArr = Arrays.copyOfRange(array, start, end);
```
