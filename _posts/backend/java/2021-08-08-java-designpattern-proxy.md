---
layout: post
title: "Java 프록시 패턴"
subtitle: ""
date: 2021-08-08 13:30:00 +0900
categories: backend
tags: java
comments: true
---

프록시 패턴 공부/구현한 것 !

> 모든 코드는 깃허브에 있습니다 !!  
> https://github.com/progress0407/intelliJ-pure-java-proejct/tree/main/src/designpattern/proxy/n1

- 먼저 부하가 적은, 메타데이터 정보 정도를 가진 Proxy 클래스를 호출하게 한다
- 부하가 높은 작업을 호출하면 Proxy를 통해 Real 클래스를 호출한다

---

```java
public class MyProgram {
    public static void main(String[] args) {
        List<Thumbnail> proxyThumbnails = new ArrayList<>();

        proxyThumbnails.add(new ProxyThumbnail("/movie?name=darknight", "배트맨 다크나이트"));
        proxyThumbnails.add(new ProxyThumbnail("/movie?name=sen-chihiro", "센과 치히로의 행방불명"));
        proxyThumbnails.add(new ProxyThumbnail("/movie?name=home-alone", "나 홀로 집에"));
        proxyThumbnails.add(new ProxyThumbnail("/movie?name=new-world", "신세계"));
        proxyThumbnails.add(new ProxyThumbnail("/movie?name=gurren-lagann", "천원돌파 그렌라간"));

        proxyThumbnails.stream().iterator().forEachRemaining(thumbnail -> thumbnail.showTitle());

        System.out.println("___________________________________________________");

        proxyThumbnails.stream().iterator().forEachRemaining(thumbnail -> thumbnail.showPreview());
    }
}
```

```java
public interface Thumbnail {
    void showTitle();
    void showPreview();
}
```

```java
public class ProxyThumbnail implements Thumbnail {

    private String movieUrl;
    private String title;

    public ProxyThumbnail(String movieUrl, String title) {
        this.movieUrl = movieUrl;
        this.title = title;
    }

    @Override
    public void showTitle() {
        System.out.println("제목은"  + this.title + "\" 입니다");
    }

    @Override
    public void showPreview() {
        ((Thumbnail) new RealThumbnail(this.movieUrl, this.title)).showPreview();
    }
}
```

```java
public class RealThumbnail implements Thumbnail {

    private String movieUrl;
    private String title;

    public RealThumbnail(String movieUrl, String title) {
        this.movieUrl = movieUrl;
        this.title = title;
    }

    @Override
    public void showTitle() {
        System.out.println("\"" + this.movieUrl + "\" 의 제목은 \"" + this.title + "\" 입니다");
    }

    @Override
    public void showPreview() {
        System.out.println("기다리는 중입니다 ...");
        System.out.println(this.movieUrl + "로 부터 " + this.title + "를 다운 받습니다");
    }
}
```
