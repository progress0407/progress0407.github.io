---
layout: post
title: "[wooteco] 톰캣 구현"
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

## File Input

```java
final Path path = Paths
            .get("tomcat", "src", "main", "resources", "static", fileName)
            .toAbsolutePath();
final File file = path.toFile();
```

아래의 코드로 대신할 수 있다.
위는 .java가 있는 곳을 읽어가며 아래와 같은 경우 실제로 classLoader에 있는 곳을 읽어간다

```java
URL resource = getClass()
                    .getClassLoader()
                    .getResource("static" + fileName);
String filePath = Objects.requireNonNull(resource)
        .getFile();

final BufferedReader bufferedReader = new BufferedReader(new FileReader(filePath));
```

## POST 전송 QueryString 파싱


```java

postContent = headers[headers.length - 1].replaceAll("\\s+", "");

/**
* TODO 아래의 내용을 파싱해야함
* @   %40
* !   %21
*/

private String findPostContent(final String[] httpRequestStringLines) {
    return Arrays.stream(httpRequestStringLines)
            .filter(it -> it.contains("JSESSIONID"))
            .findAny()
            .orElseThrow(() -> new IllegalArgumentException("Not found JSESSIONID "))
            .split("\\s+")
            [1];
}

public String findPostContent() {
    return nonKeyValues.stream()
            .filter(nonKeyValue -> !StringUtils.isEmpty(nonKeyValue))
            .findAny()
            .orElse("");
}
```

## 공백 문자열이 오는 현상

알고보니 Header와 Body를 안나누었기 때문에 나타난 현상으로 보인다!

readLine() 에서 계속 멈추는 현상이다... 
```java
while (reader.ready()) {
    stringBuffer.append(reader.readLine());
}
```

이때는 Content-Length만큼 미리 다 읽어버리면 된다 !

```java

// IoUtils.readCertainLength(reader, Integer.parseInt(contentLength));

public static String readCertainLength(final BufferedReader reader, final int length) {
        char[] buffer = new char[length];
        try {
            reader.read(buffer, 0, length);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        return new String(buffer);
    }
```
