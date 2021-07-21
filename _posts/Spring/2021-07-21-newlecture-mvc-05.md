---
layout: post
title: "newlecture: spring MVC (5) : 프론트로 출력하기"
subtitle: "..."
date: 2021-07-21 21:30:00 +0900
categories: backend
tags: spring
comments: true
---

### 문서 출력 방법 4가지

---

- `서블릿 객체`를 얻어서 문자열 출력
- `@ResponseBody` 설정을 통한 문자열 출력
- `ResourceViewResolver`를 이용한 문서 출력
- `TilesViewResolver`를 이용한 문서 출력

#### 서블릿 객체를 얻어서 문자열 출력

```java
    @RequestMapping("index")
    public void index(HttpServletResponse response) {
    PrintWriter out = null;
    try {
        out = response.getWriter();
    } catch (IOException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }

    out.println("Welcome");
//  return "root.index";
    }
```

#### @ResponseBody 설정을 통한 문자열 출력

```java
    @RequestMapping("index")
    @ResponseBody
    public String index(HttpServletResponse response) {
	return "hello";
    }
```
