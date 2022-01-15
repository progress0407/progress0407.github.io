---
layout: post
title: "[JPA] 굉장히 Simple한 JPA save vs saveAll 테스트"
subtitle: "..."
date: 2022-01-15 16:00 +0900
categories: backend
tags: jpa
comments: true
---

> 스터디원 중 한명인 빡쌤님이 spring data jpa 설정을 도와주셔서 ~~하하하~~  
> 해당 프로젝트 기반으로 더욱 공부하다 간단한 형태의 테스트를 해보게 되었다  
> 아래는 빡쌤님 블로그다 !  
> https://velog.io/@jhp1115/%EB%B3%B5%ED%95%A9%ED%82%A4-%EB%8D%94-%EA%B9%8A%EC%9D%B4-%EB%93%A4%EC%96%B4%EA%B0%80%EB%B3%B4%EB%8B%A4

---

## save와 saveAll의 성능 테스트

---

### save

save를 1000번씩 10번의 평균을 수행해보았다

```java
@Test
void save() {
  List<Long> elapsedTimes = new ArrayList<>();
  for (int i = 0; i < 10; i++) {
    long start = System.currentTimeMillis();
    for (int j = 0; j < 1000; j++) {
      Post post = Post.builder().name("post " + j).build();
      postRepository.save(post);
    }
    long elapsedTime = (System.currentTimeMillis() - start);
    elapsedTimes.add(elapsedTime);
    System.out.println("elapsed time : " + elapsedTime);
  }
  double avgElapsedTime = elapsedTimes.stream().mapToLong(e -> e).average().getAsDouble();
  System.out.println("#avgElapsedTimeList = " + elapsedTimes);
  System.out.println("#avgElapsedTime = " + avgElapsedTime);
}

```

> 실행결과

```
#avgElapsedTime = 958.5
```

### saveAll

list에 1000개씩 담아서 10번씩 수행한 것의 평균을 측정하였다

```java
@Test
void saveAllTest() {
  List<Long> elapsedTimes = new ArrayList<>();

  for (int i = 0; i < 10; i++) {
    long start = System.currentTimeMillis();
    List<Post> posts = new ArrayList<>();

    for (int j = 0; j < 1000; j++) {
      Post post = Post.builder().name("post " + j).build();
      posts.add(post);
    }

    postRepository.saveAll(posts);

    long elapsedTime = (System.currentTimeMillis() - start);
    elapsedTimes.add(elapsedTime);
    System.out.println("elapsed time : " + elapsedTime);
  }
  double avgElapsedTime = elapsedTimes.stream().mapToLong(e -> e).average().getAsDouble();
  System.out.println("#avgElapsedTime = " + avgElapsedTime);
}
```

> 실행결과

```
#avgElapsedTime = 467.3
```

### 성능 차이가 나는 이유? (추측)

처음에는 saveAll같은 경우 트랜잭션을 한번만 열어서 수행하고
save같은 경우 저장할때마다 트랜잭션을 열어서 수행하는 것의 차이가 있는 것 같다 -라고 생각을 하였으나.. 이건 굉장히 불확실한 결론이라 생각한다

![image](https://user-images.githubusercontent.com/66164361/149622036-ebe47e67-6ee3-42a5-bc92-bde9c798a1ef.png)

아직 프록시와 `@Transactional` 에 대한 이해가 필요한 것 같다

> 참고글  
> https://velog.io/@rainmaker007/spring-data-jpa-save-saveAll-%EB%B9%84%EA%B5%90
