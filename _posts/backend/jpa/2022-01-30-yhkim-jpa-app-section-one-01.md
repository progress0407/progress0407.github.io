---
layout: post
title: "[김영한 - JPA 실전 활용 1편] 웹 애플리케이션 개발"
subtitle: "..."
date: 2022-01-30 14:00 +0900
categories: backend
tags: jpa
comments: true
---

## 강의 내용을 기록한 것!

> 이미 앞에서 spring core + web mvc + jpa 등을 학습하면서 얻은 지식들이 많기 때문에 많은 내용을 기입하지는 않고 간략한 메모 정도만을 다룹니다 !! ㅎ

- devtools 설정
  - devtools 의존성 + settings에서 `Build project automatically` 설정을 체크해주어야 한다 (금방 찾는다 ! 구글링 할 것 !)
  - html, java 파일을 서버 재기동 없이 반영하여 빠르게 개발 가능

## 설정 포인트들 !

---

- h2 버전 200 이상에서는

```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop;MVCC=TRUE
```

`;MVCC=TRUE` 부분을 입력하면 런타임 에러가 나기 때문에 지워주어야 한다

- junit 4를 사용하기 때문에 아래의 코드를 `dependencies`에 추가한다

```
testImplementation 'org.springframework.boot:spring-boot-starter-test'
testImplementation("org.junit.vintage:junit-vintage-engine") {
  exclude group: "org.hamcrest", module: "hamcrest-core"
}
```

### h2 설정

---

- `jdbc:h2:~/jpashop` (최소 한번)
- `~/jpashop.mv.db` 파일 생성 확인
- 이후 부터는 `jdbc:h2:tcp://localhost/~/jpashop` 이렇게 접속

### CQS

> MemberRepository 타이핑 중에 ...  
> ![image](https://user-images.githubusercontent.com/66164361/151688382-c4f6342b-728f-4928-bf15-9b8080bab120.png)

> 커맨드와 쿼리를 분리해라 ??

이 말 뜻이 이해가 안되어 조사를 조금 해보았다

https://www.inflearn.com/questions/27795

내가 이해한 느낌으로는 Member를 반환시에 해당 객체에 `Setter` 등이 있을 경우에는 변경이 가능하기 때문에 부수효과의 가능성이 있다, 때문에 원천적으로 차단하되, 편의적으로 id는 조회가 가능하게끔 반환값으로 지정해준 것 같다 반환된 Long은 **불변** 이니깐 !

`CQS`는 `Command Query Separation` 의 약자인데
커맨드랑 쿼리를 분리하란 것이다..

대충 파악한 느낌으로는

커맨드 - 상태를 조작

- 예) `Collections.sort(..)` 등

쿼리 - 불변

- 예) `자바8 스트림`

### 간단한 테스트 코드

---

```java
@RunWith(SpringRunner.class) // 스프링과 관련된 테스트를 할 것이다
@SpringBootTest
public class MemberRepositoryTest {

	@Autowired
	MemberRepository memberRepository;

	@Test
	@Transactional
	@Rollback(false) // @Test + @Transactional 으로 인한 롤백이 안되게끔 한다
	public void testMember() {
		// given
		Member member = new Member();
		member.setUsername("memberA");

		// when
		Long saveId = memberRepository.save(member);
		Member findMember = memberRepository.find(saveId);

		// then
		Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
		Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
		Assertions.assertThat(findMember).isEqualTo(member);
		System.out.println("(findMember == member) = " + (findMember == member));

		System.out.println("member hashCode = " + member.hashCode());
		System.out.println("findMember hashCode = " + findMember.hashCode());

		System.out.println("member = " + member);
		System.out.println("findMember = " + findMember);

		System.out.println("member class name = " + member.getClass().getSimpleName());
		System.out.println("findMember class name = " + findMember.getClass().getSimpleName());
	}
}
```

### @Transactional 과 관련된 어노테이션

---

`@Transactional`에 `@Test` 가 있을 경우 테스트 후 db를 롤백한다

```
transaction.TransactionContext   : Rolled back transaction for test: [DefaultTestContext@7582ff54 testClass = MemberRepositoryTest, testInstance = jpa.app.shop.MemberRepositoryTest@3023f157 ...
```

`@Rollback(false)` 를 추가하면 롤백하지 않고 `insert`된 데이터를 볼 수 있다

### terminal 명령어로 배포하기

프로젝트 폴더에서 `./gradlew clean build` 로 빌드 후

`/build/libs` 의 jar 파일에 대해 아래 명령어를 수행한다

```
java -jar {jar파일명}
예) java -jar shop-0.0.1-SNAPSHOT.jar
```

### p6spy

---

로깅을 편하게 해주는 설정이다 아래 의존성을 추가해주면 된다

```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'
```

설정 후

![image](https://user-images.githubusercontent.com/66164361/151696986-938bfdb8-2484-4083-897f-e4b31676ddba.png)

## 그외

---

- `p6spy`란 것이 있다 개발 과정에 생기는 쿼리들을 추적할 때 로그를 이쁘게 출력해줄 수 있게끔 해주는 것으로 보인다
  - 대신 커스텀하는 게 까다로워 보인다
  - 비용이 비싸기에 운영서버에서는 사용을 금한다
