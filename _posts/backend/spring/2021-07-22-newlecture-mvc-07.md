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

public String reg(MultipartFile file ...) {

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

#### 다중으로 업로드할 경우

간단합니다, 아래와 같이 수정합니다.

```html
<input type="file" name="files" />
```

```java
public String reg(MultipartFile[] files ...) {

  for (MultipartFile file : files) {
    ...
}
```

### 공지사항 페이지 준비

```xml
<definition name="admin.board.*.*" template="/WEB-INF/view/admin/inc/layout.jsp" >
  <put-attribute name="title" value="관리자 > 공지사항"/>
  <put-attribute name="header" value="/WEB-INF/view/inc/header.jsp" />
  <put-attribute name="visual" value="/WEB-INF/view/admin/inc/visual.jsp" />
  <put-attribute name="aside" value="/WEB-INF/view/admin/inc/aside.jsp" />
  <put-attribute name="body" value="/WEB-INF/view/admin/board/{1}/{2}.jsp" />
  <put-attribute name="footer" value="/WEB-INF/view/inc/footer.jsp" />
</definition>
```

```java
@RequestMapping("/admin/board/notice/")
...
@RequestMapping("list")
public String list() {
return "admin.board.notice.list";
}
```

reg의 경우 GET과 POST요청시 처리할 부분을 나누어야 합니다

#### GET / POST 요청 분리

```java
@Autowired
private HttpServletRequest request; // 혹은 reg 메서드의 인자로 둡니다
...
public String reg(...) {
    if(request.getMethod().equals("POST")) {
      ...
    }
}
```

위와 같이 요청 구분을 나누는 방법도 있습니다만 이렇게 하지 않고 아래와 같이 합니다

```java
@RequestMapping(value="reg", method=RequestMethod.GET)
public String reg() {
  return "admin.board.notice.reg";
}

@RequestMapping(value="reg", method=RequestMethod.POST)
 public String reg(MultipartFile ...) {
   ...
  return "redirect:list";
}
```

스프링 4.x 부터는 더 간단한 애노테이션을 제공합니다

```java
@GetMapping("reg")
@PostMapping("reg")
```

#### clean

> tomcat 우클릭 > clean을 하면 배포 디렉터리가 비워진다
> {워크스페이스}\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\{웹루트}
> project > clean 은 target 디렉터리가 비워진다

> 배포 디렉터리에는 .class뿐만 아니라 img 등이 들어간다
> target디렉터리에는 .class파일만 있는것으로 보인다.

### 데이터 레이어

![image](https://user-images.githubusercontent.com/66164361/127495535-58bff2e8-4de3-48f8-b27c-2ff053c388c1.png)

컨트롤러, 서비스, Dao의 각 역할을 분리한다.
Service는 비지니스 스럽게 Dao는 데이터 처리스러운 작업을 한다

![image](https://user-images.githubusercontent.com/66164361/127495741-6c63a3ca-f219-4327-af45-d7bb84fa1a6c.png)

Dao와 테이블은 1:1 관계이며.. 대게 단조롭고 반복적이다

DB와 연결하는 방식(라이브러리)이 몇가지 있다

- 스프링의 Jdbc 라이브러리 : Jdbc 템플릿
- mybatis
- JPA의 하이버네이트

![image](https://user-images.githubusercontent.com/66164361/127496263-aaabec9d-07fa-4f7d-b98a-73ce60b13a5f.png)

참고) 간소화된 코드량이다

template 메서드

- `query` 리스트 반환
- `queryForObject` 단일 객체 반환
- `queryForList` 하위호환성을 위함

> 보통의 경우 query와 queryForObject 사용하면 된다

```java
	JdbcTemplate template = new JdbcTemplate();
	template.setDataSource(dataSource);
	List<Notice> list = template.query(sql, new BeanPropertyRowMapper<Notice>(Notice.class));
```
