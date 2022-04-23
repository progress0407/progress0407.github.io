---
layout: post
title: "우테코 로또 미션 리뷰"
subtitle: "..."
date: 2022-03-01 22:00:00 +0900
categories: backend
tags: wooteco
comments: true
---

### 전반 적인 소감

로또 미션을 진행하면서 사실 기분이 마냥 좋지는 않았다

왜냐하면 많은 크루들이 필자가 그동안 조금씩 부지런히 공부해 온 자바 지식을 굉장히 빠른 속도로 따라잡고 있었기 때문이다

그래서 금~토 약 이틀 간 코딩에 거의 집중을 못하고 있었다 (하루에 고작해야 1 ~ 2시간 정도)

그만큼 슬펐다.. 난 이렇게나 혼나면서 힘들게 출/퇴근 길 인강보고 작은 핸드폰으로 구글링하면서 얻은 지식을

남에게 쉽사리 뺏기는 것 같았다

람다, 스트림을 잘 못다루어서 거의 보름 ~ 한달 가까이 시간만 나면 기선님 인강도 보고 자바의 정석 책/유튜브도 봤던 기억이 있다..

자바 8의 세련된 사용법에 관심이 많았었던 것이었다.

반면 크루들은 리펙터링도 열심히 제때 하면서 기본적인 람다, 스트림을 잘 활용하는 모습을 보여주었다

심지어 자바를 공부한 지 얼마 안돼는 비전공 크루가 수집 컬렉션을 필자보다 잘 활용하는 것을 보기도 했다

> 그래서 지금은 ??

이 글을 업데이트한 시점은 3월 17일이다.

약 한 달 전에 느꼈던 심정인데 어떻게 극복하게 됐는지는 잘 모르겠다

그냥 익숙해졌던 걸까...

알고보니 뒤에서 열심히한 크루들도 있었고,
다년간 노력을 해온 크루, 그냥 머리가 좋은 크루(...),
학교에서 전공자로 매번 잘해오던 크루,
한의사 크루, 서울대, 교대 등 똑똑이 크루,
매일 아침 성실히 일어나며 꾸준히 노력하는 크루 등등...

애당초 공부를 좋아하거나 어딘가에서 하나 쯤은 뛰어난 크루들이 많았다.

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

### `행위`를 가지지 않는 일급 컬렉션? 글쎄...

아래와 같이 단순히 생성하고 값을 내보내거나 크기만을 내보내주는 일급 컬렉션이 있다

```java
public class LottoTickets {

    private final List<LottoTicket> values;

    public LottoTickets(List<String> ticketStrings) {
        values = ticketStrings.stream()
                .map(LottoTicket::new)
                .collect(Collectors.toCollection(LinkedList::new));
    }

    public List<LottoTicket> values() {
        return values;
    }

    public int size() {
        return values.size();
    }
}
```

적절해 보이는가??... 필자는 그렇지 않다고 생각한다, 심지어 불변으로 설계되지도 않았다

누구 코드인가... 그것은 필자 코드이다 ㅠㅠ

행위 없이 상태만을 관리하는 일급컬렉션이면 그냥 컬렉션 그대로 사용하는 것이 낫다 !

왜냐하면 애당초 일급 컬렉션이 List 등 컬렉션을 관리하는데

이에 관련된 행위를 컬렉션 자체에 부여할 수 없기 때문에 컬렉션으로 한 번 감싼 이후 행위를 부여하는 것이기 때문이다 (-라고 생각한다)

따라서 위와 같이 생성 로직 정도만 복잡한 래핑 클래스는 제거 하고

복잡한 생성은 `Factory` 클래스에 위임하는게 낫다고 생각한다

### 불변객체

이때부터 불변객체의 개념이 유행하였다

특히 오찌가 불변객체에 대한 개념을 상세하게 적은 것에 대해

학습로그에서 설명해주는데 재미있게 읽었던 경험이 있다

---

그래서 불변객체가 무엇이냐면

`불변객체`란 변하지 않는 객체이다

외부에서 생성자로 넣거나 `Getter`로 내부의 가변 객체를 변화시키더라도

불변 객체는 그 자체로 변하지 않아야 한다

---

불변객체가 모두 불변객체로 구성되어있다면

불변을 위한 어떤 처리를 추가적으로 해줄 필요가 없다 (생성로직, Getter 로직)

만일 불변 객체 중 어느 하나가 가변객체이라면

이 가변객체에 대해 불변으로 만들어주기 위한 로직을 작성해야 한다

### 느낀점

처음부터 구조화를 잘 해놓지 못하면 나중에 고치기가 정말 힘들다..

간단한 로직 하나만 변경하는데도 여파가 산에서 산으로 가서 굉장히 힘들었다

```java
something(Lotto lotto) {...}
```

위와 같은 메서드는 아래와 같은 메서드보다 변경이 힘들다

```java
something(int lottoiSize) {...}
```

왜냐하면 전자의 경우 `Lotto`의 역할이 바뀌면 `something` 메서드의 역할도 바뀔 가능성이 크기 때문이다
