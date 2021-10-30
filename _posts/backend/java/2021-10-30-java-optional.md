---
layout: post
title: "Optional 사용기"
subtitle: ""
date: 2021-10-09 11:30:00 +0900
categories: backend
tags: java
comments: true
---

# Opional은 주의하면서 사용해야 해

---

Optional은 단순히 `null`을 체크하기 위한 용도라고 생각하고 사용해왔었는데

자칫 잘못사용헀다가는 예측하기 힘든 일을 초래할 수 있다는 걸 근래에 알았다

아직까지 실무에는 java8을 적극적으로 사용하지는 않지만 곧 사용하게 될 수 있으니 기록한다!

## orElse보다는 orElseGet 을..

---

`orElse` 와 `orElseGet`

```java
List<Member> memberRepo = new ArrayList<>();
memberRepo.add(new Member("foo", 30));
memberRepo.add(new Member("bar", 25));
```

회원들 리스트에 담겨 있고

```java
String findName = "foo";
Member member = findOrCreate(memberRepo, findName);
```

회원을 찾는 부분이 있다

회원이 있으면 찾은 회원을 반환해 주고 회원이 없으면 새로 생성한다

```java
private static Member findOrCreate(List<Member> memberRepo, String findName) {
    return memberRepo.stream()
            .filter(e -> e.getName().equals(findName))
            .findFirst()
            .orElse(signUp(findName));
}
```

```java
private static Member signUp(String findName) {
    return new Member(findName, 0);
}
```

문제는 위의 `orElse` 메서드에서 발생한다

Member객체는 아래와 같으며 생성시에 로그를 찍는다

```java
static class Member {

    private String name;
    private Integer age;

    public Member(String name, Integer age) {
        this.name = name;
        this.age = age;
        out.println("ID created !  " + this);
    }

    /* 아래는 생략
    getter
    eequals hashcode
    toString
    */
}
```

생략한 `equals` 메서드는 name이 같다면 같은 객체로 간주되게끔 작성하였다

문제는 `findOrCreate` 메서드의 아래 부분에서 발생하는데

```java
.orElse(signUp(findName));
```

회원을 찾아도 해당 메서드를 수행한다

분명히 있는 회원인데도 새로 생성하는 것!! 이러면.. 논노데스이다...

```java
.orElseGet(()-> signUp(findName));
```

그래서 위처럼 `orElseGet`으로 수정하면 그런 불상사를 막을 수 있다

위 메서드는 정말로 찾지 못했을 때만 호출한다

`orElseGet`은 `Lazy` 하게 호출된다

## 그외

---

- `VO`에서 `Optional`은 `return`할때 사용한다 (`get`)
- 필드에는 `Optional`을 사용하지 말자

> 참고  
> https://homoefficio.github.io/2019/10/03/Java-Optional-%EB%B0%94%EB%A5%B4%EA%B2%8C-%EC%93%B0%EA%B8%B0/ ★★  
> https://stackoverflow.com/questions/38725445/optional-get-without-ispresent-check  
> https://cfdf.tistory.com/34 ★ 회원가입 예제
