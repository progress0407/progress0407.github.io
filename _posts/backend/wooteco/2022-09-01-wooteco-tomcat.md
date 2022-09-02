---
layout: post
title: "우테코 톰캣 구현"
subtitle: "..."
date: 2022-09-01 21:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

## Application Container 간이식 컨테이너 생성

```java
package support;

import java.util.HashMap;
import java.util.Map;

/**
 * static 유틸 클래스로 생성할 수 없는 객체는 이곳에 저장하여 싱글톤으로 관리합니다.
 */
public class ApplicationContainer {

    private final Map<Class, Object> container = new HashMap<>();

    public ApplicationContainer() {
        container.put(FileUtils.class, new FileUtils());
    }

    public <T> T getSingletonObject(Class<T> clazz) {
        return (T) container.get(clazz);
    }
}

```


# 오답 노트

아래의 코드는 정상 작동되지 않는다! ㅠㅠ

```java
final byte[] bytes = inputStream.readAllBytes();
final String httpRequestString = new String(bytes);
final String uri = httpRequestString.split(" ")[1];
```