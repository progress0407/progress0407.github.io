---
layout: post
title: "[JPA-트리편 2] 하위 폴더를 검색하거나 삭제하기 "
subtitle: "..."
date: 2022-01-18 04:00 +0900
categories: backend
tags: jpa
comments: true
---

## 또 다시 퇴근하는 중에

---

> https://progress0407.github.io/backend/2022/01/12/jpa-folder-delete-recur.html  
> 스토리가 여기서 이어집니다 !

지인분의 질문으로 다시 한번 하위 폴더 검색 / 삭제하는 로직을 재구성하게 되었다

> https://developer-ping9.tistory.com/  
> 같은 스터디 조원인 핑구님의 블로그 :)

문의는 해당 테이블들의 상속관계 매핑 전략이었다

`single table` 전략으로 하자니 쿼리는 단순하게 나간다는 장점이 있으나 낭비되는 컬럼들이 문제였고

`table per class` 전략으로 하자니 전송되는 쿼리의 발생 숫자가 문제였다

그래서 나는 상속관계가 어떤지에 대해 제안했다

## 구현한 코드

> 메인 실행부

```java
		Folder aFolder = new Folder();
		aFolder.setName("folder a").setParent(null);

		Folder bFolder = new Folder();
		bFolder.setName("folder b");

		BookMark aBookMark = new BookMark();
		aBookMark.setUrl("www.naver.com").setName("bookmark a");


		BookMark bBookMark = new BookMark();
		bBookMark.setUrl("www.daum.net").setName("bookmark b");

		BookMark cBookMark = new BookMark();
		cBookMark.setUrl("www.google.com").setName("bookmark c");

		aFolder.addChildren(bFolder, aBookMark);
		bFolder.addChildren(bBookMark, cBookMark);

		em.persist(aFolder);
		em.persist(bFolder);

		em.persist(aBookMark);
		em.persist(bBookMark);
		em.persist(cBookMark);

		long rootFileId = aFolder.getId();
		em.flush();
		em.clear();

		// 모든 폴더 찾아내기
		File findRootFile = em.find(File.class, rootFileId);

		out.println("findFolder = " + findRootFile);

		Queue<File> queue = new LinkedList<>();
		queue.offer(findRootFile);

		while (!queue.isEmpty()) {
			File pollFile = queue.poll();
			out.println("pollFile.getName() = " + pollFile.getName());
			if (pollFile instanceof Folder) {
				Folder pollFolder = (Folder)pollFile;
				List<File> pollChildren = pollFolder.getChildren();
				pollChildren.forEach(queue::offer);
			}
		}

		em.remove(findRootFile);
```

원래 롬복의 빌더 패턴을 이용해서 구현하려고 했으나 잘 되지 않아서 본래의 getter/setter 를 이용했다..

그러나 빌더의 메서드 체이닝이 유용했기 때문에 유사하게 커스텀해서 만들어 보았다

> 부모 클래스

```java
@Entity
@SequenceGenerator(
	name = "FILE_SEQ_GEN",
	sequenceName = "FILE_SEQ",
	initialValue = 1
)
@Getter
@DiscriminatorColumn
@Inheritance(strategy = InheritanceType.JOINED)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class File extends BaseEntity {

	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "FILE_SEQ_GEN")
	private long id;

	private String name;

	@JoinColumn(name = "PARENT_ID")
	@ManyToOne(fetch = FetchType.LAZY)
	private File parent;

	public File setName(String name) {
		this.name = name;
		return this;
	}

	public File setParent(File parent) {
		this.parent = parent;
		return this;
	}
}
```

> 데이터 저장 결과
> ![image](https://user-images.githubusercontent.com/66164361/149827604-22ecab7a-8dba-45c3-acae-f88fad00e67d.png)

> 폴더 클래스

자식을 저장 가능하다, 부모에는 없는 기능이다

```java
@Entity
@Getter @Setter
public class Folder extends File {

	@BatchSize(size = 900)
	@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
	private List<File> children = new ArrayList<>();

	public Folder addChildren(File... files) {
		this.children.addAll(Arrays.asList(files));
		Arrays.stream(files).forEach(file -> file.setParent(this));
		return this;
	}

	@Override
	public String toString() {
		return "Folder{}";
	}
}
```

> 데이터 저장 결과
> ![image](https://user-images.githubusercontent.com/66164361/149827612-3f5b6152-2a63-452f-990b-01a8d289a2bb.png)

```java
@Entity
@Getter
public class BookMark extends File {

	private String url;

	public BookMark setUrl(String url) {
		this.url = url;
		return this;
	}

	@Override
	public String toString() {
		return "BookMark{}";
	}
}
```

> 데이터 저장 결과
> ![image](https://user-images.githubusercontent.com/66164361/149827628-f7c8003d-f4cc-4c49-a358-8e4925b4db80.png)

위와 같이 구성했을 때 장점이 있다

```java
aFolder.addChildren(bFolder, aBookMark);
```

위와 같이 자식을 넣을 떄 서로 다른 두 타입을 합쳐서 넣을 수 가 있다.

다른 장점으로는 테이블상에서도 상속관계를 유지할 수 있어서 공간의 낭비가 적다는 것이다

그리고 이름을 뽑아오거나 등의 공통필드에 관한 연산을 할 때 빠르게 처리할 수 있다

또한 상위 File 객체 기준으로 읽어들이면 해당 자료형에 속한 sub type을 모두 불러들일 수 있다

이상 장점이었고...

다른 스터디원인 빡샘님의 의견으로는

> 빡샘님 블로그  
> https://velog.io/@jhp1115

애당초 Folder와 Bookmark의 관계가 (is-a) 관계는 아니더라도 한쪽이 다른 한쪽을 (자료구조상) 포함할 수 있는 관계이기 때문에

상속구조로 나타내기엔 다소 애매하다는 것이다

또한

`XXX_ToOne` 관계는 페치조인을 사용하고, `컬렉션` 부분은 lazy로딩 + `@BatchSize` 사용하라는 내용이 강의상에 있었다구 한다

### 기타 참고 사항

---

> https://luminous-pluto-c36.notion.site/V3-1-d5033e1bdebb45d3ad718886d20f57d0
