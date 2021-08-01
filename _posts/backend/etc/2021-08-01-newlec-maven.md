---
layout: post
title: "뉴렉쳐 메이븐 강의 메모"
subtitle: "..."
date: 2021-07-26 22:15:00 +0900
categories: backend
tags: etc
comments: true
---

## 뉴렉쳐 메이븐 강의 메모

### Maven

---

- IDE와 별개인 빌드 도구
  - 프로젝트생성, 라이브러리 설정, 컴파일, 테스트, 패키징, 배포

> 다운로드 URL : https://maven.apache.org/download.cgi

- cmd에서 `path` 입력시 `{메이븐경로}/bin`이 있어야 명령(mvn) 입력가능
- `mvn -version` 로 설치 유무 확인하자

### 메이븐 프로젝트 생성

---

`mvn archetype:generate -DgroupId=com.newlecture -DartifactId=javaprj -DarchetypeArtifactId=maven-archetype-quickstart`

> 누군가 만들어 놓은 arche로 생성하자는 뜻이다. 외부로부터 다운받아온다

### 컴파일

---

- `mvn compile`
  - BUILD FAILUE 소스/타겟 버전이 낮을시 발생하는 오류일 경우  
    -> pom.xml 의 `<project>` 자식 태그로 아래 코드를 추가한다

```xml
  <properties>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
```

### 패키징

---

- `mvn package`

- 패키징한 것 실행하기
  - `java -cp target\{프로젝트명.jar} {main 함수가 있는 경로명+클래스명}`
  - java -cp target\javaprj-1.0-SNAPSHOT.jar com.newlecture.App

### Build LifeCycle과 Phase 들

---

내게 중요한 것만들 적었다 !!..

- compile
- test
- **package**

  ![image](https://user-images.githubusercontent.com/66164361/127745379-59cddb71-9569-495d-8748-08ac3104cc2c.png)

  - phase 단계명
  - plug-in 단계를 수행하는 실질적인 프로그램명 / 버전
    - phase와 매칭된 plug-in이 없을 수도 있다
  - goal 플러그인의 하위 항목

> 플러그인목록 조회 : `mvn help:describe -Dcmd=package`

> 플러그인 : https://maven.apache.org/plugins/index.html

### 이클립스에서 메이븐 프로젝트 로드

---

- `import` > `Maven` > `Existing Maven Projects`

### plug-in 설정으로 JDK 버전 변경해보기

---

pom.xml 의 자식태그에 다음 코드 추가

```xml
  <build>
  	<plugins>
  		<plugin>
  			<artifactId>maven-compiler-plugin</artifactId>
  			<version>3.8.1</version>
  			<configuration>
  				<source>1.8</source>
  				<target>1.8</target>
  			</configuration>
  		</plugin>
  	</plugins>
  </build>
```

> pom.xml 변경시 `Maven` > `Update Proejct`를 해주자

이렇게 되면 compiler의 plug-in 버전과 함께 프로젝트의 jdk 버전도 오버라이딩된다.

만일 하고자 하는게 jdk 버전만 바꾸는 것이라면 위에서 작성했던

```xml
  <properties>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
```

이 설정으로 가는 것이 간편하다

### 웹 프로젝트로 변경

---

pom.xml 을 수정하자

```xml
<project>
  ...
  <packaging>jar</packaging>
  ..
```

`jar`를 `war` 로 바꾸고 업데이트를 하자

`Project Explorer` 로 보면 구조가 변경되는 것을 볼 수 있다 (`webapp` 디렉터리 추가 등)

이렇게 되면 pom.xml에 에러가 생기는데 web.xml이 없기 떄문

프로젝트의 webapp 폴더 밑에 `WEB-INF` 폴더를 만들고

톰캣의 webapps > ROOT > `web.xml` 을 복붙을 하자

webapp 밑에 index.html 을 추가하고 run 해서 tomcat 설정을 마치면 실행되는 것을 볼 수 있다

### 흠

---

webapp 밑에 index.jsp를 추가하자

그러면 선언부에 아래와 같은 에러가 나오는 것을 볼 수있다

```
The superclass "javax.servlet.http.HttpServlet" was not found on the Java Build Path
```

Servlet library가 없기 때문이다

#### Local Import

프로젝트 우클릭 > Build Path > Libraries탭 > `Add Library` > `Server Runtime` 우리가 설정했던 것과 맞는 톰캣을 설정하자

> 톰캣 자체가 라이브러리는 아니지만 톰캣 안에 lib가 있다. 이 lib 안에 jsp, servlet-api, el 등 우리가 사용해 온 lib가 있음 !

위 설정의 문제는 절대경로로 임포트되는데 이때 절대경로로 잡힌다

회사와 집이 경로가 다른 경우.. 설정이 꺠짐 ㅠ

#### Remtote Import

https://mvnrepository.com/

위와 같은 곳에서 임포트해온다

저장되는 곳은 default로 밑에와 같다

- `C:\Users\{사용자계정}\.m2\repository`
- `C:\Users\progr\.m2\repository`

이미 가져 온 lib라면 내 로컬 경로에서 캐시한다

여러 다른 프로젝트가 있더라도 remote에서 가져오지 않고 로컬에서 땡겨오는 것이다

> import 가 잘 되었는지 확인하려면 webapp 밑에 index.jsp를 넣고
> 아래 샘플 코드를 추가해서 run하면 된다

```html
<h1>JSP Index Page ${3+4 }</h1>
```

### 라이브러리에 문제가 있을 경우

---

예를들어 lib를 땡겨왔는데 그 lib 자체가 다운로드가 되다 말거나, 무언가의 이유로 jar 파일 자체가 잘못된 경우가 있다

이 경우 수많은 jar파일들 중에 무엇이 잘못되었는지 알기가 힘들다

`.m2\repository` 밑의 파일을 모두 지워주고 이클립스 프로젝트를 다시 열자

그러면 자동으로 가져온다

### 라이브러리 인덱싱 검색

---

> 그런데 나는 이 방법이 편하지가 않다... 때문에 간소화해서 메모합니다요ㅠ

pom.xml > dependencies 탭 > Add 버튼

을 누르면 검색이 되게끔 하는 것이다

`Maven Repositories` view를 열어서 Global Repositories > 우클릭 > Rebuild index을 누른다

빠르면 1시간.. 보통 3~5시간 걸린다 한다 ...

### 커스텀 라이브러리 export, import

---

자신의 Project를 JAR로 export, import 할 수 있다

샘플 프로젝트를 만들어 보자

- Maven Project 생성 > 심플 프로젝트 클릭
  - groupId > com.newlecture
  - artifactId > examlib
    - finish 버튼 생성

/examlib/src/main/java 디렉터리에
/com/newlecture/examlib/entity 패키지 추가후
Exam.java 를 생성해서

```java
public class Exam {
    private int kor;
    private int eng;
    private int math;

    public int total() {
	return kor + eng + math;
    }

    public float avg() {
	return (kor + eng + math) / 3.0f;
    }

  // Getter, Setter, ToString, Construcotr ... 등 추가
}
```

#### Jar로 Export 한 경우

- Build Path 에서 추가한다

#### 1. Add JARs

상대경로로 참조된다

![image](https://user-images.githubusercontent.com/66164361/127758899-f1db85e3-7efa-4aa0-9b81-d71f851148fb.png)

#### 2.Add External JARs

- 내 프로젝트 이외의 장소에 있을 때

- 절대경로로 참조된다

![image](https://user-images.githubusercontent.com/66164361/127758852-8930f04f-8a51-47b1-bb06-0dd7ead6ebac.png)

#### 3. Add External Libary

- JUnit, Tomcat 등 공식적인 JAR를 추가할 때 사용
- 보통 이런 파일들은 pom으로 끌고 온다..

#### Maven으로 Export, Import

#### Import

- 프로젝트 우클릭 > Run As > `Maven Install`

- `.m2/repository/{groupId + artifactId}` 에 jar파일 생성됨
- 예) C:\Users\progr\.m2\repository\com\newlecture\examlib\0.0.1-SNAPSHOT

#### Export

- Maven Repository view 에서 Local Repository > Rebuild Index
- pom.xml > Dependencies 탭 > Add > 커스텀 lib 검색
