---
layout: post
title: "우테코 로또 리뷰"
subtitle: "..."
date: 2022-03-01 22:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

### BigDecimal

![image](https://user-images.githubusercontent.com/66164361/156176608-08851d04-8a6d-427a-b303-9d70a9739cbe.png)

`BigDecimal`을 도입했는데 위와 같은 명쾌한 답변을 받았다.. 그렇다 기껏해야 비싼 객체를 사용해서 연산까지 다헀는데

정작 기능을 사용하는 곳은 저렴한 `double`을 보내니 ㅠㅠ... 이렇게 저렴이 저렴이할 수가 없도다..

재사용성을 생각하면 보다 기능이 많은 `BigDecimal`을 보내자

### `.collect(Collectors.toUnmodifiableSet())` 는 순서를 보장하지 못한다 !

한참 디버깅했었는데 수동으로 입력한 로또가 계속 순서를 보장하지 못하는 것이다..

위에서 반환하는 `Set`의 구현체가 다른 종류의 순서를 보장하는 `Set`이 아니다

> 문제의 코드

```java
    private Set<LottoNumber> createLottoNumbersFromString(String lottoNumbersString) {
        return Arrays.stream(lottoNumbersString.split(","))
                .map(LottoNumber::new)
                .collect(Collectors.toUnmodifiableSet());
```

위와 같이할 때 `ImmutableCollections$SetN` 이 반환되는데 이친구가 순서를 보장 못한다

참고로 `Collections.unmodifiableSet(...)`으로 반환되는 타입은 `Collections$unmodifiableSet` 으로 위와는 또 다르다

순서 보장이 필요한 경우 아래 둘 중 하나로 취사선택하면 된다

> 스트림

```java
    private Set<LottoNumber> createLottoNumbersFromString(String lottoNumbersString) {
        return Arrays.stream(lottoNumbersString.split(","))
                .map(LottoNumber::new)
                .collect(Collectors.toCollection(LinkedHashSet::new));
```

> For

```java
    private Set<LottoNumber> createLottoNumbersFromString(String lottoNumbersString) {
        LinkedHashSet<LottoNumber> lottoNumbers = new LinkedHashSet<>();
        for (String lottoNumberString : lottoNumbersString.split(",")) {
            lottoNumbers.add(new LottoNumber(lottoNumberString))
        }
        return Collections.unmodifiableSet(lottoNumbers);
```

> 참고
> https://stackoverflow.com/questions/54826658/collectors-toset-changes-order
