---
layout: post
title: "우테코 레벨2 지하철 미션"
subtitle: "..."
date: 2022-05-04 23:30:00 +0900
categories: backend
tags: wooteco
comments: true
---

베루스와 페어가 되면서 많은 것을 배웠다

## 배운 것들

### RestAssured는 테스트 프레임워크를 Hamcret를 쓴다

> 참고
> https://www.baeldung.com/java-junit-hamcrest-guide

참고로 `Hamcret` 는 `Matchers` 의 애너그램이다 !

### `@DirtiesContext` 는 비용이 비싸다

그것도 매우 비싸다...

이 어노테이션 하나만 없어도 실행속도가 4배는 빨라지는 것을 목격했다

약 25초 -> 6초

> 참고 (4.4를 보면 된다 !)
> https://www.baeldung.com/spring-dirtiescontext

### 지인에게 배운것

> resources 에 있는 JSON 파일을 통해서 Domain 객체 생성
> https://stackoverflow.com/questions/60402338/get-json-file-from-resources-folder
