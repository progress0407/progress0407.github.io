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

## HttpResponseBuilder

```java
final String response = HttpResponseBuilderOld.builder()
                    .addStatus(HttpStatus.FOUND)
                    .add(HttpHeader.LOCATION, "/401.html")
                    .build();

public class HttpResponseBuilder {

    private final LinkedList<String> responseHeaders = new LinkedList<>();
    private boolean hasBody = false;

    private HttpResponseBuilder() {
    }

    public static HttpResponseBuilder builder() {
        return new HttpResponseBuilder();
    }

    public HttpResponseBuilder add(HttpHeader header, Object value) {
        final String responseHeader = header.apply(value.toString());
        responseHeaders.add(responseHeader);
        return this;
    }

    public HttpResponseBuilder add(HttpHeader header, String value) {
        final String responseHeader = header.apply(value);
        responseHeaders.add(responseHeader);
        return this;
    }

    public HttpResponseBuilder add(String header, String value) {
        responseHeaders.add(String.format("%s %s ", header, value));
        return this;
    }

    public HttpResponseBuilder addStatus(HttpStatus statusCode) {
        final String responseHeader = HTTP_1_1_STATUS.apply(statusCode);
        responseHeaders.add(responseHeader);
        return this;
    }

    public HttpResponseBuilder addCooke() {
        final String responseHeader = HttpHeader.SET_COOKIE.apply(new HttpCookie());
        responseHeaders.add(responseHeader);
        return this;
    }

    public HttpResponseBuilder addCooke(final Session session) {
        final String responseHeader = HttpHeader.SET_COOKIE.apply(session);
        responseHeaders.add(responseHeader);
        return this;
    }

    /**
     * 아래와 같은 Body를 포함한 문자열이 추가됩니다. <p />
     * "Content-Length: 123<p />
     * <p />
     * This is Body"
     */
    public HttpResponseBuilderOld body(String body) {
        add(HttpHeader.CONTENT_LENGTH, body.getBytes().length);
        responseHeaders.add("");
        responseHeaders.add(body);
        hasBody = true;
        return this;
    }

    public String build() {
        if (!hasBody) {
            responseHeaders.add("");
        }
        return String.join("\r\n", responseHeaders);
    }
}
```


## uri 정보와 메서드 정보만으로 support 메서드 동적으로 만들기 (실패)


```java
public class LoginController extends AbstractController {

    @Override
    public String uri() {
        return "/login";
    }

    public boolean support(String uri) {
        return true;
    }

    protected void doGet(HttpRequest request, HttpResponse response) throws Exception {
    }
}
```

위와 같은 정보 만으로 uri 와 Get, Post 정보 요청 유무에 따라 support(지원 유무)를 만드려고 하였으나... 실패하였다


```java
private static void scanPackageInternal(final String packagePath)
            throws InstantiationException, IllegalAccessException, InvocationTargetException, NoSuchMethodException {
    final Reflections reflections = new Reflections(packagePath);
    classes = reflections.get(Scanners.SubTypes.of(AbstractController.class).asClass());
    for (final Class<?> clazz : classes) {
        final AbstractController controller = (AbstractController) clazz.getDeclaredConstructor().newInstance();
        controllers.add(controller);
    }
    for (final AbstractController controller : controllers) {
        final String uri = (String) controller.getClass().getDeclaredMethod("uri").invoke(controller);
        List<String> httpMethods =
                Arrays.stream(controller.getClass().getDeclaredMethods())
                        .map(it -> it.getName())
                        .filter(it -> it.startsWith("do"))
                        .map(it -> it.replaceAll("do", ""))
                        .map(it -> it.toUpperCase())
                        .collect(Collectors.toList());
        classMap.put((u, ms) -> uri.contains(u) && httpMethods.contains(ms), controller);
    }
    System.out.println("classMap = " + classMap);
}

public static Controller findFromUri(final String uri, final String httpMethod) {
    final AbstractController controller = classMap.get(uri, httpMethod); // 이부분이 문제... key에 FunctionanlInterface가 들어가면 곤란하다!!
    if (controller == null) {
        throw new IllegalArgumentException("해당 URI 가 존재하지 않습니다");
    }
    return controller;
}
```

아쉽다... 하하하

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

## 쿠키

![image](https://user-images.githubusercontent.com/66164361/188798258-50fdf965-e6eb-4e8f-bc3a-cecbf164989a.png)

