---
layout: post
title: "IntelliJ 기본 자바 프로젝트를 Gradle 로 변환하기"
subtitle: "..."
date: 2021-11-02 22:15:00 +0900
categories: backend
tags: etc
comments: true
---

# IntelliJ 에서 자바 프로젝트를 Gradle로 변환

---

기존에 자바 문법을 연습하거나 코테를 공부하기 위한 기본 자바 프로젝트가 있었다

근데 중간중간에 롬복이나 테스트 라이브러리를 사용하고 싶은데

Gradle 프로젝트로 만들지 않아서 불편했었다

그렇다고 Gradle 프로젝트를 새로 하나 파자니 기존의 Git과 연동되어 있던 Java프로젝트를 파기하고 싶지는 않았다 ㅠㅠ

그래서 오늘은 Java -> Gradle 로 Convert 해보는 시간을 갖도록 하자

## 하지만 실패하였다...

---

방금 기존 순수 자바 프로젝트를 Gradle 프로젝트로 전환하는데 실패하였다..

지금 우테코 코테를 준비해야하기 때문에 이작업은 이행하지 못할 것 같다..

어떻게 삽질을 올바로이 하면은 될 것 같지만... ㅠㅠㅠ

지금은 시간이 없다..

## 그래도 기록해보는 Converting Gradle

---

이 파트는 이 방식을 하면서 어떤 점을 추가적으로 보완하면 올바르게 변환이 되지 않았을까 하는 점에서 기록해본다

우선 참고한 사이트는 아래와 같다

> https://www.jetbrains.com/help/idea/gradle.html#convert_project_to_gradle

대강 번역해 보면

1. IDEA를 연다

2 ~ 3. 프로젝트 최상단에 새 파일 `build.gradle` 을 만든다

![image](https://user-images.githubusercontent.com/66164361/142761090-5ee9fbf0-fd6e-4ea6-98c1-3db2c14abdfa.png)

4. 위 파일안에 아래와 같은 스크립트를 넣고 프로젝트를 다시 연다.

```
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}
sourceSets {
    main {
        java {
            srcDirs = ['src']
        }
    }
}
dependencies {
    compile 'junit:junit:4.12'
}

```

추가적으로 `settings.gradle`란 파일을 만들고

```
rootProject.name = '프로젝트 이름'
```

위 내용을 기입한다

---

위 사항까지가 공식문서에 나온 내용이다

그리고 아래와 같이 프로젝트를 설정이 되어야 한다

![image](https://user-images.githubusercontent.com/66164361/142761026-6f2c8ce4-8c2d-488d-a719-76a2a35c1a34.png)

`src` 폴더 밑에 `main`과 `test`가 있어야 하며

각각의 폴더 밑에 또 `java`가 있어야 한다

## 프로젝트가 꼬였더라도..

방금해보니까 Git으로 형상관리된 프로젝트를 Git Clone 하니까 다시 원상복구되었다..

Spring은 추가적으로 무언가 만져야 했는데.. 다행이다.. 하하
