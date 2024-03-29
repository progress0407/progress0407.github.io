---
layout: post
title: "[JWT] Secret Key 생성하기"
subtitle: "..."
date: 2022-08-09 15:30:00 +0900
categories: backend
tags: etc
comments: true
---

# Secret Key 를 생성하는 방법

기존에 쓰고 있던 Secret-key 는 아래와 같다

```yaml
security.jwt.token.secret-key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE1MTYyMzkwMjJ9.ih1aovtQShabQ7l0cINw4k1fagApg3qLWiB8Kt59Lno
```

(교육과정 중에 공공연하게 오픈된 key로 사용된 것이니 저 키값은 노출되도 전혀 상관없다!)

위와 같은 key를

```yaml
security.jwt.token.secret-key: apple.banana.pineapple
```

처럼 하면 아래와 같은 오류 메세지를 발견할 수 있다

```java
Caused by: io.jsonwebtoken.security.WeakKeyException: The specified key byte array is 144 bits which is not secure enough for any JWT HMAC-SHA algorithm.  The JWT JWA Specification (RFC 7518, Section 3.2) states that keys used with HMAC-SHA algorithms MUST have a size >= 256 bits (the key size must be greater than or equal to the hash output size).  Consider using the io.jsonwebtoken.security.Keys#secretKeyFor(SignatureAlgorithm) method to create a key guaranteed to be secure enough for your preferred HMAC-SHA algorithm.  See https://tools.ietf.org/html/rfc7518#section-3.2 for more information.
	at app//io.jsonwebtoken.security.Keys.hmacShaKeyFor(Keys.java:96)
	at app//com.woowacourse.auth.support.JwtTokenProvider.<init>(JwtTokenProvider.java:23)
```

아래와 같이 위 클래스를 이용해서 랜덤한 Key값을 만들 수 있다

## 방법 1

```java
SecretKey secretKey = Keys.secretKeyFor(SignatureAlgorithm.HS256);
String secretString = Encoders.BASE64.encode(secretKey.getEncoded());
```

결과 : 0GZgxiG4lJbqJ+svzesLUNfVAXpW7ti6Tmst3rTFqJc=

위 문자열을 3개 합치면 기존의 장바구니때 키 문자열이 되는 것으로 보인다

사실은 하나만 있어도 된다!

## 방법 2

리눅스에서 아래의 명령어를 검색한다

```
 openssl rand -hex 64
```

결과: 403e364df1cc395b7326a7683fa91879d921d17360136331253b4c723e5222858751008f2b60f69bc37637c64c019282911756e8ce3095068739629b8555c3c9
