---
layout: post
title: "Java 객체지향 디자인 패턴 *장 팩토리 메서드"
subtitle: ""
date: 2021-12-09 21:00:00 +0900
categories: backend
tags: java
comments: true
---

> 필자의 해석이 포함되어 있습니다!  
> 실제로 책에서 의도한 의미와는 다를 수 가 있습니다  
> 주의 부탁드립니다 !!

# \*장 : Factory Method.

---

얄코 디자인 패턴을 기준으로 먼저 선 구현

> 출처: 얄코 디자인 패턴. https://youtu.be/q3_WXP9pPUQ

1. `Button` 등의 구현체들의 생성자가 변경되어도 `Factory` 클래스에서만 변경이 된다 **SRP**

2. API를 사용하는 클라이언트는 `Button`등의 클래스를 몰라도 생성이 가능하다. **캡슐화**

```java
public class YalcoFactoryMethod {
	private abstract class Component {
		abstract protected String getName();
		public Component() {
			System.out.println(this.getName() + " 생성");
		}
	}

	private class Button extends Component {
		@Override
		protected String getName() {
			return "버튼";
		}
	}

	private class Switch extends Component {
		@Override
		protected String getName() {
			return "스위치";
		}
	}

	private class Dropdown extends Component {
		@Override
		protected String getName() {
			return "드롭다운";
		}
	}

	private enum ComponentType {
		Button, Switch, Dropdown;
	}

	private class ComponentFactory {
		public Component create(ComponentType componentType) {
			switch (componentType) {
				case Button:
					return new Button();
				case Switch:
					return new Switch();
				case Dropdown:
					return new Dropdown();
			}
			return null;
		}
	}

	public static void main(String[] args) {
		ComponentFactory componentFactory = new YalcoFactoryMethod().new ComponentFactory();
		componentFactory.create(ComponentType.Button);
		componentFactory.create(ComponentType.Switch);
		componentFactory.create(ComponentType.Dropdown);
	}
}
```
