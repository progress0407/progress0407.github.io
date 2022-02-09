---
layout: post
title: "[TDD] JUnit 기초 문법들 정리 모음"
subtitle: "..."
date: 2022-02-09 15:30:00 +0900
categories: backend
tags: etc
comments: true
---

> `JUnit5` 기준으로 작성되었습니다 !

## 항상 햇갈리는 `assertThat`의 클래스

자주 사용하는 API로 `assertThat`가 있는데

보통 static import로 사용할 떄가 많다

이때

```
org.assertj.core.api
org.junit.jupitor.api
```

위 두 `api`가 있는데

**assertj** 를 선택하면 된다 !

```java
import static org.assertj.core.api.Assertions.*;
```

### 참고

> 바다의 JUnit5 사용법 https://youtu.be/EwI3E9Natcw
> ParameterizedTest 사용법 https://gmlwjd9405.github.io/2019/11/27/junit5-guide-parameterized-test.html
