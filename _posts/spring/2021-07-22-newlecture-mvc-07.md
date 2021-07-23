---
layout: post
title: "newlecture: spring MVC (7) : POST 입력"
subtitle: "..."
date: 2021-07-22 23:10:00 +0900
categories: backend
tags: spring
comments: true
---

### 파일 업로드 (browser to server)

---

`enctype`을 `application/x-www-form-urlencoded`에서
`multipart/form-data` 로 바꾼 순간 전송할 때의 데이터 형태가 바뀝니다

![image](https://user-images.githubusercontent.com/66164361/126787276-d5d3f3b1-9812-436f-9415-1a4089b59069.png)

#### 설정하기

---

xml을 아래와 같이 설정합니다

```xml
	<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<!-- setting maximum upload size 300MB(1024-1024*300) -->
		<property name="maxUploadSize" value="314572800"></property>
	</bean>
```

java 코드

```java
    public String reg(String title, String content, MultipartFile file, String category, @RequestParam("back-exercises") String[] backExercises,
	    @RequestParam("rd-back-exercise") String rdBackExercise) {

	String fileName = file.getOriginalFilename();
	long fileSize = file.getSize();

	System.out.printf("fileName : %s , fileSize : %d \n", fileName, fileSize);
```

html 코드

```html
<form
  action="reg"
  method="post"
  enctype="application/x-www-form-urlencoded"
></form>
```

위 코드를 아래와 같이 변경합니다.

```html
<form action="reg" method="post" enctype="multipart/form-data"></form>
```

### 물리 경로 얻기와 파일 저장

---

로컬 이클립스에서 개발할지라도 실제 서버에 배포가 되면

현재 경로와 전혀 다른 경로를 지니게 됩니다.

로컬에서든, 개발, 운영에서든 변치 않는 현재 서비스가 동작하고 있는 배포 주소를 알아야 합니다

```java
@Autowired
private ServletContext ctx;

... SomethingMethod(...) {
  String realPath = ctx.getRealPath("/static/upload");
}
```

```java
@Autowired
private HttpServletRequest request;


... SomethingMethod(...) {
    String realPath = request.getServletContext.getRealPath("/static/upload");
}
```

위 둘중의 하나의 방법으로 알아낼 수 있습니다.
저장은 아래와 같이 합니다

```java
File saveFile = new File(realPath);
multipartFile.transferTo(saveFile);  // 인자로 넘어온 binary file
```

실제 java 코드는 아래와 같습니다

```java

@Autowired
private ServletContext ctx;

public String reg(MultipartFile file, ...) {

	String fileName = file.getOriginalFilename();
	String webPath = "/static/upload";
	String realPath = ctx.getRealPath(webPath);

	File savePath = new File(realPath);

	if(!savePath.exists()) { // 실제 파일경로가 없을 경우
	    savePath.mkdirs(); // 중간 경로를 포함하여 생성

	realPath += File.separator + fileName; // 시스템별 알맞는 디렉터리 구분자 적용
	File saveFile = new File(realPath);

	file.transferTo(saveFile);
```
