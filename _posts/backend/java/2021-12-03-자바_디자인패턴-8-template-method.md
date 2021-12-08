---
layout: post
title: "Java 객체지향 디자인 패턴 *장 템플릿 메서드"
subtitle: ""
date: 2021-12-07 22:30:00 +0900
categories: backend
tags: java
comments: true
---

> 필자의 해석이 포함되어 있습니다!  
> 실제로 책에서 의도한 의미와는 다를 수 가 있습니다  
> 주의 부탁드립니다 !!

# \*장 : Template Method.

---

이것 역시 얄코 디자인 패턴을 기준으로 먼저 선 구현후 책을 보기로 하였다

> 출처: 얄코 디자인 패턴. https://www.youtube.com/watch?v=q3_WXP9pPUQ&t=483s

```java
public class YalcoTemplateMethodMain {

	public static void main(String[] args) {
		NaverMapView naverMapView = new NaverMapView();
		naverMapView.initMap();
	}

	private static abstract class MapView {

		protected abstract void connectMapServer();
		protected abstract void showMapOnScreen();
		protected abstract void moveToCurrentLocation();

		public void initMap() {
			connectMapServer();
			showMapOnScreen();
			moveToCurrentLocation();
		}
	}

	private static class NaverMapView extends MapView {
		@Override
		protected void connectMapServer() {
			System.out.println("네이버 지도 서버에 연결");
		}

		@Override
		protected void showMapOnScreen() {
			System.out.println("네이버 지도를 보여줌");
		}

		@Override
		protected void moveToCurrentLocation() {
			System.out.println("네이버 지도에서 현재 좌표로 이동");
		}
	}

	private static class KakaoMapview extends MapView {
		@Override
		protected void connectMapServer() {
			System.out.println("카카오 지도 서버에 연결");
		}

		@Override
		protected void showMapOnScreen() {
			System.out.println("카카오 지도를 보여줌");
		}

		@Override
		protected void moveToCurrentLocation() {
			System.out.println("카카오 지도에서 현재 좌표로 이동");
		}
	}

}
```

위와 같은 `abstract`를 이용한 구현이 있다

핵심은 상속 뿐만 아니라 일정한 절차를 거치는 메서드가 있을때 이 메서드를 호출해서 나머지 구현체를 호출하는 것이다

말그대로 템플릿(일정한 형식)을 지닌 메서드를 실행하는 것이다
