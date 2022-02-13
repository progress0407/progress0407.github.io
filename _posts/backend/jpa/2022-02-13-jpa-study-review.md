---
layout: post
title: "금일 JPA 스터디에서 알게된 내용들 !"
subtitle: "..."
date: 2022-02-13 01:00 +0900
categories: backend
tags: jpa
comments: true
---

### 스터디에서 알게된 내용들

> 을 기입하였습니다 ㅎㅎ

---

- 오늘 발표자 빡쌤님의 말씀에 의하면

테스트코드 클래스레벨에 `@Transactional`을 걸어두었을 떄 조심해야 한다

Service의 `@Transactional`이 무시될 수 있다!

---

- `@AfterEach` 보다는 `@BeforeEach` !

왜냐하면 각 테스트 메서드에서 어떤 한 test가 fail이 나면 그 뒤에 수행되는 `@AfterEach` 메서드는 실행이 안될 수 있기 때문이다 !

- 그러나 실험 결과 잘만 실행되는데.. !! 으음?!

---

#### 상황에 따라 달라지는 객체의 동등성

아래의 경우는 당연히 같다

```java
@Transactional
@Test
public void 저장한_객체와_꺼낸_객체는_당연히_같다() {

	// when
	Long savedId = memberService.join(member);
	Member findMember = memberService.findOne(savedId);

	// then
	assertThat(findMember).isEqualTo(member);
}
```

그러나 다음의 경우는 다르다

```java
@Test
public void 영속성_컨텍스트가_다를경우_객체는_서로_같지않다() {

	// when
	// 영속성 컨텍스트_시작 --->
	Long savedId = memberService.join(member);
	// <--- 영속성 컨텍스트 종료

	// 영속성 컨텍스트_시작 --->
	Member findMember = 회원_찾기(savedId);
	// <--- 영속성 컨텍스트 종료

	// then
	assertThat(findMember == member).isFalse();
	assertThat(findMember).isNotEqualTo(member);
}

@Transactional
public Member 회원_찾기(Long id) {
	return memberService.findOne(id);
}
```

우선 회원\_찾기 메서드는 별도로 트랜잭션이 걸려있어서 새로운 트랜잭션이 열린다.
또한 memberService또한 메서드에 트랜잭션이 걸려있어서 별도의 트랜잭션을 가진다

따라서 두 객체는 서로 다른 객체이다

```java
@Transactional
@Test
public void 영속성_컨텍스트가_다를경우_객체는_서로_같지_않지만_동일한_트랜잭션_범위라면_서로같다() {

	// when
	Long savedId = memberService.join(member);
	Member findMember = 회원_찾기(savedId);

	// then
	assertThat(findMember == member).isTrue();
	assertThat(findMember).isEqualTo(member);
}
```

방금 위에서 시행했었던 테스트 메서드 위에 `@Transactional`을 붙이면 pass가 된다

이유는 스프링의 기본 `@Tranasactional` 하위 트랜잭션이 상위에 참여를 하기 때문에 각각의 트랜잭션은 의미가 없어진다

반대로

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

자식 트랜잭션들에 위와 같이 설정을 하게 되면 자식들마다 새로운 트랜잭션을 열게 되므로 테스트는 fail이 일어나게 된다

```java
@Transactional
@Test
public void _1차_캐시를_지우면_두_객체의_동일성은_서로_다르다() {
	em.persist(member);
	Long savedId = member.getId();
	em.flush();
	em.clear();

	Member findMember = em.find(Member.class, savedId);
	assertThat(findMember != member).isTrue();
}
```

메서드 명에서처럼 1차 캐시를 지워진 이후에 찾아오면 두 객체는 서로 다르다 !

---

- 빡샘님 : 테스트 코드도 중복을 제거하는 방향으로 최대한 리펙터링한다
  - 예를들어 회원 객체 생성을 매 메서드마다 작성하고 있으면 공통 필드로 추출한다

---

#### ordi님 `@Transactional(readonly = true)`가 왜 빠를까??

코드 레벨로 내려가다 보니까 `FlushMode.MANUAL` 로 설정되어있는데 `더티체킹`을 하지 않는다고 한다 !

---

### jpa에서의 동등성 문제

---

한때 우리 스터디원 빡샘님이 jpa 동등성 이슈 떄문에 6시간동안 디버깅을 한 적이 있다

`equals` 메서드에 아래와 같이 Hibernate.getClass()로 감싸준다!

이 경우 `프록시`이거나 `진또배기 객체`의경우 모두를 케어해주는 것 같다 자세한건 찾아보아야한다!

```java
Hibernate.getClass()
```
