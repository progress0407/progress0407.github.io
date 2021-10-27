---
layout: post
title: 자바로 코테를 준비하는 팁들
subtitle: "..."
categories: etc
tags: codingtest
comments: true
published: true
---

## 자바로 코테 준비하기

자바는 파이썬에 비해서 준비할 때 난이도가 다소 높다..

자바로 주요 사용하는 코테용 문법들을 정리해보자

## 입력 받기

```java
        Scanner sc = new Scanner(System.in);

        out.print("n 입력: ");
        int n = sc.nextInt();
        int numbers[] = new int[n];


        out.print("배열 입력: ");
        for (int i = 0; i < n; i++) {
            numbers[i] = sc.nextInt();
        }

        out.println("=== 출력 ==");
        for (int i = 0; i < n; i++) {
            out.print(" " + numbers[i]);
        }

```

콘솔 입출력 예:

```
n 입력: 3
배열 입력: 1 2 3
=== 출력 ==
 1 2 3
```

## 배열

### 배열 정렬

배열을 정렬한다. 이때 원본 배열을 정렬한다. (`불변이 아니다`)

```java
Arrays.sort(splitArr); // 순방향 정렬
Arrays.sort(strArr, Collections.reverseOrder()); // 역 정렬
Arrays.sort(strArr, String.CASE_INSENSITIVE_ORDER); //  대소문자 무시 순방향 정렬
```

기본적으로 인자가 하나인 `Arrays.sort(splitArr)` 과 같은 경우는 `Comparable`을 따른다
`Compartor`를 구현하여 커스텀하게 정렬 순위를 매겨 보자.

```java
Arrays.sort(strArr, new Decending());
```

```java
    private static class Decending implements Comparator {

        @Override
        public int compare(Object o1, Object o2) {

            if (o1 instanceof Comparable && o2 instanceof Comparable) {
                Comparable c1 = (Comparable) o1;
                Comparable c2 = (Comparable) o2;
                return c1.compareTo(c2) * -1;
            }
            return -1;
        }
    }
```

위 예시는 자바의 정석 예시인데 저자 말씀에 따르면 -1 이 아닌 c1와 c2의 위치를 바꾸어도 된다.

디버깅으로 o1과 o2를 조사하였는데.. 예측할 수 없게 값이 들어온다. 0번째 1번째 인덱스 순서대로 들어오지 않는다 ㅠ

순방향으로 하고자 하면 `o1.compareTo(o2)` , 역방향은 그 반대로 기억하면 될 것 같다.

문자열이 한글자일 것이라는 가정하에 (혹은 문자) 순방향으로 재구성 해보면

```java
String[] simpleStr = {"a", "b", "c"};
Arrays.sort(simpleStr, new Ascending());
out.println("Arrays.toString(simpleStr) = " + Arrays.toString(simpleStr));
```

```java
    private static class Ascending implements Comparator<String> {

        @Override
        public int compare(String o1, String o2) {
            int i1 = o1.toCharArray()[0];
            int i2 = o2.toCharArray()[0];
            if (i1 > i2) {
                return 1;
            }
            return -1;
        }
    }
```

위와 같다.
이 때 주의할 것은 i1이 a, i2가 b가 아니다! b, a순서대로 들어오는데.. 다른 예시를 들어 디버그를 돌려보면.. 규칙을 추론하기 힘들다.

경험적으로 위와 같은 상황에서 오름차순 정렬이 됨을 확인하였다..

**사실 위 모든 과정은 프로그래머스:가장큰수 를 풀기 위해 공부한 내용이다**

```java
Integer[] intArr = {120, 45, 60, 446, 386, 9}; // 순서는 이렇게 나와야 함 -> 9 60 45 446 386 120
Arrays.sort(intArr, new CustomDictionarySort());
out.println("Arrays.toString(intArr) = " + Arrays.toString(intArr));
```

```java
private static class CustomDictionarySort implements Comparator<Integer> {
        // 양수면 내림차순
        // 음수면 오름차순
        @Override
        public int compare(Integer o1, Integer o2) { // Integer는 Comparable을 상속받았기 때문에 형변환 필요 X

            char[] a = String.valueOf(o1).toCharArray();
            char[] b = String.valueOf(o2).toCharArray();

            int max = Math.min(a.length, b.length);

            for (int i = 0; i < max; i++) {
                if (a[i] > b[i]) return -1; // 큰것을 기준으로 오름차순 한다
                else if (a[i] < b[i]) return 1;
                // 같은 경우는 판별이 안나기 때문에 다음 것을 본다
            }

            return 0;
        }
    }
```

큰 숫자대로(사전순) 정렬할 수 있도록 구상한 것이다

결과는 원하는 정렬 순서에 맞게 아래와 같다

```
[9, 60, 45, 446, 386, 120]
```

결과적으로... 값이 같아서 ==0 인 자명한 케이스를 제외하고는 양수 혹은 음수를 결정해주어야 한다.

이것은 논리적으로 이유를 찾는 것 보다는 둘 중에 하나일 수 밖에 없기에, 결과를 본 후 음/양을 선택하면 될 것 같다.

### 배열 자르기

`from <= x < to` 와 같은 범위로 배열을 자를 수 있다. 이때 원본 배열은 불변 보장

```java
int[] splitArr = Arrays.copyOfRange(array, start, end);
```

### 배열 출력

디버깅으로 상세 원인을 찾기 보다는 단순히 현재 배열을 출력하고 싶을 때가 있다

`Arrays` 도우미 클래스의 API를 활용하자.

1차원일 때는 `toString`, 2차원 이상일 때는 `deepToString`을 사용하면 된다

```java
  int[] oneDime = {11, 22, 33};
  int[][] twoDim =  {{1, 2, 3}, {4, 5, 6}};
  int[][][] threeDim = {{{1, 2, 3}, {4, 5, 6}}, {{7, 8, 9}, {10, 11, 12}}};

  for (int i = 0; i < 2; i++) {
      for (int j = 0; j < 3; j++) {
          out.printf("twoDim[%d][%d] = %d,  ", i, j, twoDim[i][j]);
      }
      out.println();
  }

  out.println("Arrays.toString(oneDime) = " + Arrays.toString(oneDime));
//        out.println("Arrays.deepToString(oneDime) = " + Arrays.deepToString(oneDime)); // Object[] 변환 에러 Integer[]는 가능하다

  out.println("Arrays.deepToString(twoDim) = " + Arrays.deepToString(twoDim));
//        out.println("Arrays.toString(twoDim) = " + Arrays.toString(twoDim)); // 의미 없는 주소 출력

  out.println("Arrays.deepToString(threeDim) = " + Arrays.deepToString(threeDim));
```

### 배열의 총 합

문자열 더하기
예) "123" + "045" = "123045"

```java
String answer = Arrays.stream(nums).map(String::valueOf).reduce((a, b) -> a + b).orElseGet(()->"");
```

### 배열의 최댓값

```java
int[] arr = {19, 14, 10, 17};
int max = Arrays.stream(arr).max().orElse(-1);
```

JAVA 8 전 버전일 경우

```java
int max = arr[0];
        for (int i = 1; i < n; i++) {
            max = Math.max(max, arr[i]);
        }
```

> 참고: https://stackoverflow.com/questions/31378324/how-to-find-maximum-value-from-a-integer-using-stream-in-java-8/31378866

## 형 (Type)

자바는 Integer와 int가 다르며 당연하게도 '1'과 "1" 등 문자와 문자열의 1도 다르다

(int) 1과 (Integer) 1과 '1' 과 "1" 은 모두 다르다

### '3'을 (int) 3으로

```java
char a = '3';

out.println(a - '0');
//'3' - '0'
```

### int를 char[] 로

```java
int i = 1234;
char[] chars = ("" + i).toCharArray();
// 혹은
String.valueOf(1234).toCharArray();
```

> 참고: https://stackoverflow.com/questions/12192805/convert-an-integer-to-an-array-of-characters-java

### int[] 를 Integer[]로

```java
Integer[] nums = IntStream.of(numbers).boxed().toArray(Integer[]::new);
// 혹은
Integer[] nums = Arrays.stream(numbers).boxed().toArray(Integer[]::new);
```

> 참고: https://stackoverflow.com/questions/880581/how-to-convert-int-to-integer-in-java

### int[] 를 List<Integer> 로

```java
int[] ints = {1, 2, 3};
List<Integer> intList = new ArrayList<Integer>(ints.length);
for (int i : ints) intList.add(i);
```

> 참고: https://stackoverflow.com/questions/1073919/how-to-convert-int-into-listinteger-in-java
