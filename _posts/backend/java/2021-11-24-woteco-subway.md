---
layout: post
title: "우테코 3기 기출 - 지하철 노선도"
subtitle: ""
date: 2021-11-24 00:10:00 +0900
categories: backend
tags: java
comments: true
---

# 람다 표현식과 Void의 사용

---

최근에 우테코 3기에서 나온 지하철 노선도 기출을 풀고 있었다

어렵다 하하 ㅎㅎ..

문제의 코드는 아래당!

```java
private static final Map<String, Supplier<Void>> inputViewMap = new HashMap<>();

static {
    inputViewMap.put("1", InputViewStation::addStation);
    inputViewMap.put("2", InputViewStation::deleteStation);
    inputViewMap.put("3", InputViewStation::stations);
}
```

위 코드에서 보는 것처럼 `inputViewMap`은 **`Supplier<Void>`** 란 값을 가진다

이 값은 함수를 넘기고 lazy 호출을 위해 사용된다.

문제는 이 다음이다!

```java
private static Void deleteStation() {
    System.out.println(INPUT_DELETE_STATION_NAME);
    String stationName = scanner.nextLine();
    boolean deleteStation = StationRepository.deleteStation(stationName);
    if (deleteStation) {
        System.out.println(INFO_DELETE_STATION);
        return null;
    }
    System.out.println(ERROR_DELETE_STATION);
    return null;
}
```

`void`가 아닌 `Void`라는 래퍼 클래스를 인자로 가지다 보니까 저런 상황이 연출되었다 ㅠㅠ

참조타입이기 때문에 의미가 없어도 `return null` 같은 의미 없는 문장을 넣어주어야 한다..

볼수만은 없다..

이걸 커스텀 람다식으로 해치우자

# @FunctionalInterface 의 활용: ()->()

---

우선 아래와 같은 `interface`를 만들자

```java
@FunctionalInterface
public interface Voider {

    // void to void
    void call();
}
```

그리고 컬렉션 선언부를 바꾸자

```java
private static final Map<String, Voider> inputViewMap = new HashMap<>();
```

그러면 아래처럼 코드가 수정될 수 있다 !!

```java
private static void deleteStation() {
    System.out.println(INPUT_DELETE_STATION_NAME);
    String stationName = scanner.nextLine();~
    boolean deleteStation = StationRepository.deleteStation(stationName);
    if (deleteStation) {
        System.out.println(INFO_DELETE_STATION);
        return;
    }
    System.out.println(ERROR_DELETE_STATION);
}
```

짜잔.. 이제 `null`을 리턴하거나 하는 불상사를 막을 수 있다

## 불변 클래스는 불변 보장이라 수정이 안됀다 ! 다메다메~

---

문제는 아래 상황이다

![image](https://user-images.githubusercontent.com/66164361/143199890-0a8a20dd-b827-457b-af71-6e0196843daa.png)

![image](https://user-images.githubusercontent.com/66164361/143199630-61c82317-c4b5-44d4-84ff-9518db191717.png)

도메인 객체에 값을 추가하려는 것인데

```
java.lang.UnsupportedOperationException
```

라는 예외가 발생한 것

흔히 불변성을 지니는 컬렉션을 조작하려고 했을때 나오는 메세지이다

`List.of` 내부 코드를 보도록 하자

![image](https://user-images.githubusercontent.com/66164361/143200070-0af34e24-e61a-4614-86b9-29a0634bb8cc.png)

위와 같은 점도 있고..

그러나 가장 확실한 이유를 찾았다...

![image](https://user-images.githubusercontent.com/66164361/143203621-503d355a-c8c4-44fc-8327-5c93d28cb8ef.png)

get으로 꺼내온 것은 수정할 이유가 없다는 이유로 위의 불변성을 선언한건데..

이게 문제가 된 것이었다.. 하하하 !!
