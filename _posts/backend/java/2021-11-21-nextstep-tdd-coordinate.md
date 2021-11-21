---
layout: post
title: "플레이그라운드 TDD: 좌표계산기 구현도중 Lazy생성"
subtitle: ""
date: 2021-11-21 16:30:00 +0900
categories: backend
tags: java
comments: true
---

# 좌표 계산기와 다형성, Lazy 생성

---

좌표계산기 피드백을 보았다

피드백 내용은

Factory Method 패턴에 사용된 `if`문을 제거하는 것이다

```java
if(삼각형) {
  return new 삼각형(점들);
}
if(사각형) {
  return new 사각형(점들);
}
if(선때기) {
  return new 선때기(점들);
}
```

리펙터링을 시도해보는 도중.. 장벽을 만났다.

문제의 내 코드는 아래와 같다.

```java

private static Map<Integer, AbstractFigure> oldFigureMap = new HashMap<>();

static Figure getInstance(List<Point> points) {
    initFigures(points);
    if (figure != null) {
        return figure;
    }
    throw new IllegalArgumentException("유효하지 않은 도형입니다.");
}

private static void initFigures(List<Point> points) {
    oldFigureMap.put(Line.LINE_POINT_SIZE, new Line(points));
    oldFigureMap.put(Triangle.TRIANGLE_POINT_SIZE, new Triangle(points));
    oldFigureMap.put(Rectangle.RECTANGLE_POINT_SIZE, new Rectangle(points));
}
```

위와 같이 oldFigureMap에 해당 도형을 미리 map에 넣고 꺼내려던 것인데

이게 넣을 때 부터 `new`연산자에 의해 생성이 되는 것이 문제이다.

생성이 될 때 이게 무슨 각형인지 **검증**하는 부분이 있는데

이 검증 절차를 통과하지 못하여 만들어지지 못하는 부분이 있었던 것..

그래서 이 부분을 어떻게 생성을 **Lazy**하게 막을 것인지 고민하였지만..

그런게 자바 문법에 있기나 해? 라는 생각과 함께 도저히 정답을 모르겠어서

자바지기님의 코드를 보았다..

결과는 놀랄 노자...

Lazy하게 생성할 수가 있던 것이다 !!!

어떻게??

## Lambda Expression 을 이용한 Lazy 생성

---

람다식을 이용하여 Lazy하게 해당 생성자를 호출할 수 있었다.

이건 정말 상상도 못하였다

```java

private static Map<Integer, Function<List<Point>, AbstractFigure>> figureMap;

static {
    figureMap = new HashMap<>();
    figureMap.put(Line.LINE_POINT_SIZE, Line::new);
    figureMap.put(Triangle.TRIANGLE_POINT_SIZE, Triangle::new);
    figureMap.put(Rectangle.RECTANGLE_POINT_SIZE, Rectangle::new);
}

static Figure getInstance(List<Point> points) {
    AbstractFigure figure = figureMap.get(points.size()).apply(points);
    if (figure != null) {
        return figure;
    }
    throw new IllegalArgumentException("유효하지 않은 도형입니다.");
}
```

우선 `Map`에 더 이상 `new 도형`이 아닌 아직 생성하지 않을 생성에 사용될 `함수`를 인자로 넘긴다

정확히는 `메서드 참조`를 넘기고

해당 도형이 맞다면 Map에서 끄낸 해당 함수식을 수행한다 !!

## 후기

---

알고보니 `Optional`의 `orElseGet` 이 Lazy하게 호출하는 것으로 알고 있었는데

이렇게 내가 직접 사용하게 될 줄은 상상도 하지 못하였다
