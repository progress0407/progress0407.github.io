---
layout: post
title: "[wooteco] 웹의 캐시"
subtitle: "..."
date: 2022-09-02 15:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

# 캐시


- HTTP/1.1 공식 문서
  - RFC 2616

> https://youtu.be/UxNz_08oS4E

Hit Cloud Front 

방금 브라우저가 클라우드 프론트에서 hit 한 데이터를 가져왔다!

Refresh Hit Cloud Front

클라우드 프론트에서 Refresh 을 해서 가져온 것!

# 오답 노트

> 안 되는 설정들 모음

### 캐시 컨트롤 : 0단계 - 휴리스틱 캐싱 제거하기

```yaml
spring:
  web:
    resources:
      cache:
        cachecontrol:
          max-age: 3600
```

```java
@Configuration
public class CacheWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(final InterceptorRegistry registry) {
    }


    @Override
    public void addResourceHandlers(final ResourceHandlerRegistry registry) {
        final CacheControl cacheControl =
                CacheControl.maxAge(1, TimeUnit.DAYS)
                        .cachePrivate()
                        .mustRevalidate();

        registry.addResourceHandler("/")
                .addResourceLocations("classpath:/static/")
                .setCacheControl(cacheControl);
    }

}
```



```java
@GetMapping("/")
public String index(final HttpServletResponse response) {
    response.setHeader("Cache-Control", "no-cache, private");
    return "index";
}
```

> 위는 정상 설정 
그러나 로컬한 설정이라 제거했다


### 1단계 - HTTP Compression 설정하기

```java
@GetMapping("/")
public String index(final HttpServletResponse response) {
    response.setHeader("Transfer-Encoding", "gzip");
    return "index";
}
```

### 2단계 - ETag/If-None-Match 적용하기

### 3단계 - 캐시 무효화(Cache Busting)
