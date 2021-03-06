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

1. 추가

```
implementation 'org.springframework.boot:spring-boot-devtools'
```

2. 체크

- Build, Execution, Deployment
  - Compiler
    - `Build project automatically`

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

### 일대일 관계에서 FK 는 누구에게?

---

영한쌤 같은 경우 접근을 많이 하는 테이블을 기준으로 FK 를 둔다고 하셨다

예를 들어 `delivery` 보다는 `order` 에 접근을 많이 하기 때문에 order에 `FK` 를 넣는다

- order를 보면서 delivery에 접근을 많이 하며, 그 반대의 경우는 별로 없다고 가정한다

### 다시 한 번 정리해보는 `EAGER`

---

`EAGER` 는 `조인` 뿐만 아니라 연관된 도메인 가져오려는 모든 행위이다

예를들어 일대다 관계인 member, order에서 order.Member가 `EAGER` 로 fetch type이 잡혀있다고 가정을 하자

`JPQL`로 `select o from Order o` 라는 쿼리문을 실행하였다면

`FK`로 잡혀있는 모든 member를 가져온다 (조인이 아니라 `1+N`) 이것 또한 `EAGER` 이다 !

### 이제서야 알게 된 @Entity(`name`) 과 @Table(`name`) 차이

---

항상 생각을 했었다, 어째서 영한님은 `@Entity(name = ... )` 을 사용하지 않고

굳이 `@Table`의 `name`속성을 따로 사용하는 것일까?

오늘 그 이유를 알았다,

> given

```java
@Entity
@Table(name = "orders")
public class Order { ... }
```

> when

```java
em.createQuery("select o from orders as o", Order.class).getResultList();
```

> then

![image](https://user-images.githubusercontent.com/66164361/151705144-33fee1ca-f89d-40f1-8abe-6f4d34417255.png)

위와 같은 에러 코드가 나게 되는데, 이유는 `@Table(name = "orders")` 에 설정된 것이 Entity 명이 아니라 정말 table 명만을 저렇게 지칭한 것이기 때문이다, `Order`란 엔티티 명을 그대로 사용할 수 있게 된다!

만일 `@Entity(name = "orders")` 로 설정한다면 `JPQL`은 `Order` 란 순수 자바 엔티티명을 사용하지 못하고 `orders` 라는 테이블 종속정인 명칭을 사용해야 한다 ! _(ㅠㅠ)_

### LAZY를 사용 안하고 EAGER로 사용하는 실무자들... (강의 내용 중)

---

아직 겪어보진 못했지만 실무자분들이 LAZY가 아닌 EAGER로 잡는 경우가 있는 것 같다

이유는 lazy로딩시 잡으면 트랜잭션 밖에서 예외가 발생하는 몇몇 경우가 있는 것 같은데..

이에 대한 대안책은 transaction을 빨리 가져오거나 OSIV를 이용하거나 궁극적으로는 `fetch join` 을 이용하면 된다고 한다

### 컬렉션에 set을 함부로 하지 말 것

---

영속화 이후 `PersistenceBag` 이라는 하이버네이트가 관리하는 래퍼 클래스가 들어온다

여기서 무언가 다른 객체로 갈아끼우면 하이버네이트가 의도하는 동작대로 되지 않을 수 있다

### 테이블, 컬럼명 생성 전략 `NamingStrategy`

---

부트 기본 전략

- 카멜케이스 → (스몰케이스)언더스코어
  - 예) orderDate → order_date

> 이 부분은 강의내용 pdf를 보고 좀더공부를 해보면 될 것 같다

### 양벙향 연관관계 편의 메서드

---

관계에서 좀 더 컨트롤하는 쪽에 걸어둔다고 하셨다 (.. 조금 모호하게 와닿아서 나는 일대다의 일에 걸어두고 있다)

### 애플리케이션 아키텍쳐 설명 도중

---

현재 예제에서는 Controller에서 Repository로 바로 접근하는 것도 허용한다고 하셨다

단순히 Service객체가 Repository를 불러오는 정도의 책임만 가지고 있으면 유연성이 떨어진다고 생각하였다 (실용적인 관점에서 좋지 않음)

실제로 본인도 MVC 구조에 대해 생각할때.. 굳이 이런것까지 service를 타야해? 라는 것도 있었던 것을 생각해보면 수긍이 간다 !

### EntityMangerFactory 를 주입받고 싶으면

---

`@PersistenceUnit` 를 추가하면 된다 !

- 스프링 데이터 JPA는 `@Autowired` 로 주입이 가능하다 !

따라서 생성자 주입 방식으로 사용 가능 !

```java
@PersistenceUnit
private EntityManagerFactory emf;
```

### 애플리케이션 개발 순서

---

핵심 domain 계층 → 웹계층

도메인 → 리포지토리 → 서비스 → 테스트코드 작성 및 검증 → 컨트롤러, 웹

### @Transactional(readOnly = true)

---

기본적으로는 쓰기 가능모드이지만 대게 읽기 전용 메서드가 많으므로 service에는 읽기전용으로 건 후에

세부 변경 가능의 메서드에 기본 어노테이션을 걸어두면 된다 !

### validator 검증 로직 (회원 이름 중복 검증)

---

검증 로직을 넣어도 멀티 쓰레드 환경에서는 중복 검증 로직을 뚫고 가입이 이루어질 수 있다

때문에 최후의 방어로 DB에 유일키 제약 조건을 걸어두어야 한다

### 테스트코드: 회원가입에서 insert 쿼리가 안 나가는 이유

---

`@SpringBootTest` + `@Transactional` 시 기본적으로 `롤백`이 기본 전략이기 떄문이다

롤백시에 캐시에 있는 데이터를 flush하지 않고 말끔하게 지운다

보고 싶다면 !

1. `@Rollback(false)` 로 하거나

   - `@Commit` 으로도 가능하다 !

2. 엔티티매니저를 주입받아서 수동 flush 해주면 된다

### 테스트 코드 작성시 예외에 대한 로직

---

```java
@Test
public void 중복_회원_예외() {

  //... 예외 로직

  try {
    memberService.join(userB);
  } catch (MemberDuplicateException e) {
    return;
  }

  // then
  fail("예외가 발생하여 이 코드에 도달하지 못해야 한다");
}
```

더 간단하게 작성하는 법은 아래와 같다

```java
@Test(expected = MemberDuplicateException.class)
public void 중복_회원_예외_V2() {

  //... 예외 로직
  memberService.join(userB);

  // then
  fail("예외가 발생하여 이 코드에 도달하지 못해야 한다");
}
```

### 테스트 환경시 별도의 설정을 적용하고 싶어!

---

`src` 폴더 밑에 main / test로 나뉘는데 실환경은 main의 설정, 테스트 환경은 test가 우선권을 가진다 !

> h2 인메모리 설정  
> https://www.h2database.com/html/cheatSheet.html

h2 db는 자바로 만들어졌기 때문에 부트 실행시 JVM위에 돌아갈 수 있도록 부트가 지원해준다 !

url 설정 : `jdbc:h2:mem:test`

그리고 부트는 굳이 설정하지 않고 yml 파일만 있으면 h2 인메모리로 동작한다 !!

### DDD에 대한 짧은 언급

---

보통 `Item` 객체에 재고를 증가한다면 서비스에서 getter를 통해 호출후 setter로 +1 된 값을 대입한다

DDD에서는 그러지 않고 `Item`에 책임을 위임한다

### 인텔리J 기능 : for -> lambda

---

충격과 공포...

![image](https://user-images.githubusercontent.com/66164361/153636441-c890d6ae-f494-4657-9c2b-a7dde381c995.png)

위와 같이 `alt + enter`를 누른 후 하이라이트된 부분을 선택하면

![image](https://user-images.githubusercontent.com/66164361/153636494-2a11e9aa-f5f9-49ea-b594-64e293520d04.png)

이처럼 람다 & 스트림으로 치환된다 !!

### 다시 살펴보는 cascade

---

order와 {delivery, orderItem} 정도의 관계에서만 사용해야 한다!

예를들어 delivery는 다른 객체를 부모로 두지 않는다.. 딱 이정도의 관계에서 사용해야 한다!

나) 즉 order가 aggregate의 최상단이며 나머지가 그 자식들일때 !

    -	private owner ..? 강의상에서 언급한 개념인데 ㅠ 잘 못들었다

또 생성, 삭제등과 같은 행위에서 라이프 사이클을 동일한 타이밍에 관리할 때

정리하면 아래와 같은 조건을 만족할 떄 `cascade` 옵션을 사용한다

1. 주인이 오직 하나 일 떄

2. 생애주기를 동일하게 관리할 때

위 사항이 아닐 때는 쓰지 아니한다

예를 들어 delivery가 중요해서 다른 곳에서 참조해서 사용할 때는 사용하면 아니된다

### 정정 팩토리 메서드 : OrderItem.createOrderItem

---

위와 같이 정적 생성하는 메서드가 별도로 있을 때는 이외의 모든 스타일의 생성자는 막는게 좋다

- 예측하지 못한 방식의 생성이 있을 수 있기 떄문 !

따라서 jpa는 `protected` 까지 허용하므로 기본생성자의 접근 제어자를 `protected`로 변경하자 ! ㅎ

jpa사용중 `protected`는 쓰지 말란 뜻이다 !

### 도메인 모델 패턴과 트랜잭션 스크립트 패턴

---

- 도메인 모델 패턴

  - 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것
    - 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할을 한다
  - http://martinfowler.com/eaaCatalog/domainModel.html

- 트랜잭션 스크립트 패턴
  - 엔티티에는 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것
  - http://martinfowler.com/eaaCatalog/transactionScript.html

#### SpringBootTest 보다는 단위테스트를...

빠르게 통과시킬 수 있는 단위테스트가 더 중요하다고 말씀하였다

### 다시 한번 살펴보는 BindingResult

---

![image](https://user-images.githubusercontent.com/66164361/153770854-f6cc7506-c1f3-438f-9042-f850868fabb3.png)

위와 같은 검증 어노테이션을 통해서 아래와 같이 검증 메세지를 화면단에서 받을 수 있다

![image](https://user-images.githubusercontent.com/66164361/153770832-09890bd0-2d80-4b47-98ad-798f09b28b6d.png)

스프링이 BindingResult 란 객체를 통해서 처리를 해주며 아래와 같이 에러필드의 메세지에 있다는 것을 확인할 수 있음!

- 프론트 친구들에게 에러메세지를 ajax로 객체를 전송해줄 수 있다 !

![image](https://user-images.githubusercontent.com/66164361/153770792-735cd318-c153-49dc-93a2-62e3a825016f.png)

### 폼 객체를 그냥 쓸지, 아니면 순수한 엔티티 통으로 쓸지

---

요구사항이 정말 단순하다면 엔티티를 써도 되겠지만 보통은 그리 단순하지 않다

폼객체를 통해서 사용하는 것이 좋다

왜냐하면 `@NotEmpty` 같은 검증이 붙으면 엔티티가 점점 지저분해진다

그래서 별도로 화면에 맞는 객체를 사용하기 위해 폼 객체나 DTO를 사용해야 한다 !

### API를 만들 때는 절대 엔티티를 반환하면 안됀다 !

---

API는 하나의 스펙이다

그런데 예를 들어 Member 엔티티에 필드가 하나 추가되었다면 기존의 API 스펙과 맞지 않게 된다

- 불안 전한 API 스펙

그리고 추가된 필드가 주민번호나 비밀번호 등이라면 외부로 노출되는 셈이다

### Setter를 사용한 코드는 지양하자

---

아래는 안 좋은 코드이다

![image](https://user-images.githubusercontent.com/66164361/153771915-d952a656-71c5-4e5c-9f3a-e05a36dc6713.png)

setter를 지우고 createBook과 같은 static 메서드를 활용해서 만드는 것이 좋다

![image](https://user-images.githubusercontent.com/66164361/153772100-91f16362-d350-454f-9bdb-79b43e2c2f3f.png)

위에 처럼 !

### 수정할 때 item의 id에 대한 검증 필요

---

이 유저가 이 item에 대한 권한이 있는지를 체크해주는 검증 로직이 있어야 한다 !

그렇지 않으면 크래커에 의해 다른 사용자의 item이 수정될 수 있다

### `merge` 가 무엇이지?

---

아래 코드 처럼 등록하는게 merge이다

```java
@Transactional
public Item updateItem(Long itemId, Book param) {
  Item findItem = itemRepository.findOne(itemId);

  findItem.setPrice(param.getPrice());
  findItem.setName(param.getName());
  findItem.setStockQuantity(param.getStockQuantity());

  return findItem;
}
```

- 먼저 가져올 엔티티를 조회한 후
- 변경되는 값들을 찾아서 모두 변경한다
  - 이때 모든 엔티티 필드에 대해 변경함
- 그 다음에 영속성 컨텍스트에 저장한 엔티티를 새로 반환한다

```java
Item mergedItem = em.merge(item)
```

이때 item은 준영속, mergedItem 은 영속상태이다

### `merge` 는 함부로 쓰면 안돼

---

변경감지와 다르게 엔티티를 통째로 갈아끼우기 때문에 조심해야 한다!

만일 화면에서 price를 넘겨주지 않으면 엔티티에 price가 `null`이 들어올 위험이 있는데

이때 `merge`를 호출하면 기존의 price 값이 **null로 덮어 씌워**진다

> 결론! `merge` 보다는 `더티체킹` + `persist` 를 사용해라 !

### 더티 체킹할 때 set 보다는 ...

---

`setPrice` 등의 메서드 보다

`changeItem(Item)` 등의 메서드를 제공하자

만일 필자라면 item 객체를 통째로 `changeItem`을 넘겨서 관리 포인트를 하나로 만들 것이다 !

아래 처럼

```java
public void changeItem(Item param) {
  price = param.price;
  name = param.name;
  stockQuantity = param.stockQuantity;
}
```

이렇게 하면 `set` 메서드를 정의해두지 않아도 된다 ㅎ

### Controller에서 member, item 등을 찾는다면 ?

---

위 말은 controller에서 memberRepository 등을 다이렉트로 접근한다는 의미와 동일하다

위 상황에서 member 객체를 가져오면 `@Transactional` 에 의한 영속성 관리가 안 되기 때문에

service로 넘어온 이 객체는 더티 체킹 등이 되지 않는다 !

### 엔티티에는 setter 대신에 비즈니스 메서드를

모든 필드에 setter가 열려있으면 추적하기 어려워진다.

그렇다고 엔티티는 불변객체가 아닌 변경 가능한 객체로 설계되어있고 변경 감지를 이용해서 객체를 수정할 수 있다.

> 참고  
> https://www.inflearn.com/questions/86314  
> https://www.inflearn.com/questions/15944  
> https://yoonbing9.tistory.com/28

## 그외

---

- `p6spy`란 것이 있다 개발 과정에 생기는 쿼리들을 추적할 때 로그를 이쁘게 출력해줄 수 있게끔 해주는 것으로 보인다

  - 대신 커스텀하는 게 까다로워 보인다
  - 비용이 비싸기에 운영서버에서는 사용을 금한다

- modelmapper 기능을 이용해서 반복되는 dto to entity 매핑 작업을 줄일 수 있다.
