---
layout: post
title: "우테코 로또 미션 리뷰"
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

### `List.of("")` 와 `new ArrayList<String>()`

---

코드를 리펙터링하는 와중에 만나게 된 컨셉이다

언뜻보기엔 둘다 겉보기엔 차이가 있나 싶을 지도 모른다

실제로 출력해보면 아래와 같이 나온다

```java
System.out.println(new ArrayList<String>());
System.out.println(List.of(""));
```

> 출력

```
[]
[]
```

그러나 아래와 같이 출력을 하게 되면

```java
System.out.println(List.of("").get(0).equals(""));  // #1
System.out.println(new ArrayList<String>().get(0)); // #2
```

> 출력

```
true
Exception in thread "main" java.lang.IndexOutOfBoundsException: Index 0 out of bounds for length 0
```

`#1` 에서는 정상적으로 `true`가 뜨는데
`#2` 에선 인덱스 예외가 뜬다

아래의 예시를 보면 이해할 수 있다

```java
System.out.println(List.of("").size()); // 1
System.out.println(new ArrayList<String>().size());  // 0
```

위 상황은 아래와 같은 코드의 파라미터로 받는 부분에서 발생하였다

```java
public List<LottoTicket> createManualTickets(List<String> manualTicketNumbers) {
    return manualTicketNumbers.stream()
            .map(LottoTicket::new)
            .collect(Collectors.toCollection(LinkedList::new));
}
```

`List.of("")`의 경우 원소를 하나 가지고 있는 컬렉션이기 때문에 스트림에서 소모되어 도메인 규칙에 어긋나서 예외가 발생했지만 (공백을 허용하지 않는다는 규칙이 있다)
`new ArrayList<>()`는 빈 컬렉션이므로 스트림에서 소모가 되지 않았기 때문에 통과되었다
