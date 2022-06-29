---
layout: post
title: "TPS 테스트해보기 (Simple)"
subtitle: "..."
date: 2022-06-20 21:00:00 +0900
categories: backend
tags: spring
comments: true
---

JPA를 공부하면서 성능이 무척 중요하다고 생각하게 되었다.

```java
@SpringBootApplication
@RestController
@Slf4j
public class TpsTestApp {

    public static void main(final String... args) {
        SpringApplication.run(TpsTestApp.class, args);
    }

    @GetMapping("/test/resource")
    @SneakyThrows
    public String test() {
        log.info("ResourceController.test");
        TimeUnit.MILLISECONDS.sleep(1_000);
        return "ok";
    }
}
```

위와 같은 간단한 API를 피크 타임 TPS를 버틸 수 있을지를 테스트해보았다

## Thread 버전

```java
public class Tps_ApiCaller_v1 {

    private static int errorCount = 0;
    private static final int wholeTrialCount = 1300;

    private static RestTemplate restTemplate = new RestTemplate();

    public static void main(final String... args) {
        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < wholeTrialCount; i++) {
            threads.add(new RequestThread());
        }
        for (Thread thread : threads) {
            thread.start();
        }
        threads.forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        });

        out.println("\n\n");
        out.println(wholeTrialCount + "개의 요청 중 ");
        out.println("---------------------------");
        out.println(errorCount + "개 실패 ");
    }

    private static String getRequest() {
        ResponseEntity<String> entity = null;
        try {
            restTemplate.getForEntity("http://localhost:8080/test/resource", String.class);
            return "success";
        } catch (Exception ex) {
            errorCount++;
            err.println(ex.getMessage());
            return "error";
        }
    }

    public static class RequestThread extends Thread {

        @Override
        public void run() {
            getRequest();
        }
    }
}
```

## CompletableFuture

```java
public class Tps_ApiCaller_v2 {

    private static int errorCount = 0;
    private static final int wholeTrialCount = 1300;

    private static RestTemplate restTemplate = new RestTemplate();

    public static void main(final String... args) {

        ExecutorService executorService = Executors.newFixedThreadPool(wholeTrialCount);

        List<CompletableFuture<String>> futures = IntStream.range(0, wholeTrialCount)
                .mapToObj(trialCount -> CompletableFuture.supplyAsync(() -> getRequest(), executorService)
                ).collect(Collectors.toList());

        Map<String, Long> resultMap = futures.stream()
                .parallel()
                .map(CompletableFuture::join)
                .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

        out.println("\n\n");
        out.println(wholeTrialCount + "개의 요청 중 ");
        out.println("---------------------------");
        for (Entry<String, Long> entry : resultMap.entrySet()) {
            out.printf("%s - %d 개\n", entry.getKey(), entry.getValue());
        }
    }

    private static String getRequest() {
        ResponseEntity<String> entity = null;
        try {
            restTemplate.getForEntity("http://localhost:8080/test/resource", String.class);
            return "success";
        } catch (Exception ex) {
            errorCount++;
            err.println(ex.getMessage());
            return "error";
        }
    }
}
```

## 어려웠던 점

비동기 구현...

`CompletableFuture` 보다 일반 `Thread` 를 이용하는게 훨씬 쉽고 빠르다 ㅠ

> 스프링부트는 어떻게 다중 유저 요청을 처리할까? (Tomcat9.0 Thread Pool)  
> https://velog.io/@sihyung92/how-does-springboot-handle-multiple-requests  
> https://d2.naver.com/helloworld/5102792

> how can i count  
> https://stackoverflow.com/questions/25441088/how-can-i-count-occurrences-with-groupby
