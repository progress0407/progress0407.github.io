---
layout: post
title: "[폴더/북마크 시리즈 3] 리펙터링 편"
subtitle: "..."
date: 2022-02-01 21:00 +0900
categories: backend
tags: java
comments: true
---

## 아는 분의 코드에 if/else문이 ...

---

해당 편은 지인분의 jpa 리펙터링 1 ~ 2 시리즈와 연결됩니다

jpa의 join 상속 전략을 바탕으로 만들었는데 OOP를 활용하자 하는 의미에서 리펙터링을 시도하였던 내용을 작성했습니다

### 기존의 코드

---

```java

if (폴더만 있을 경우) {
  로직1

} else if (파일만 있을 경우) {
  로직2

} else {
 로직3

}
```

위와 같이 폴더/파일 중 어느 한쪽이 없는 경우에 따라 케이스별 로직이 달랐기에 if/else 분기를 타는 로직이었으며

if ~ else 의 로직은 거의 동일하였습니다

### 우선 머가 되었든 if else를 제거해 보자

---

> 테스트 코드

```java
@SpringBootTest
class FileRepositoryTest {

	@Autowired
	private FileRepository fileRepository;

	@Autowired
	private FolderRepository folderRepository;

	@Autowired
	private BookmarkRepository bookmarkRepository;

	@AfterEach
	void tearDown() {
		fileRepository.deleteAll();
		folderRepository.deleteAll();
		bookmarkRepository.deleteAll();
	}

	@Test
	@DisplayName("폴더와 북마크 모두 있을 경우")
	void folder_and_bookmark_all_exist() {

		Folder folderA = new Folder();
		folderA.setName("folder a");
		folderA.setShare(false);

		Folder folderB = new Folder();
		folderB.setName("folderB b");
		folderB.setShare(false);

		Bookmark bookmarkA = new Bookmark();
		bookmarkA.setUrl("www.google.com");

		folderA.addChildren(folderB, bookmarkA);

		fileRepository.saveAll(Arrays.asList(folderA, folderB, bookmarkA));
		folderRepository.saveAll(Arrays.asList(folderA, folderB));
		bookmarkRepository.save(bookmarkA);

		Folders folders = new Folders(folderRepository.findAll());
		Bookmarks bookmarks = new Bookmarks(bookmarkRepository.findAll());

		Files files = new Files(folders, bookmarks);

		files.somethingAllDo();
	}

	@Test
	@DisplayName("폴더만 있을 경우")
	void only_folder_exist() {

		Folder folderA = new Folder();
		folderA.setName("folder a");
		folderA.setShare(false);

		Folder folderB = new Folder();
		folderB.setName("folderB b");
		folderB.setShare(false);

		folderA.addChildren(folderB);

		List<Folder> saveList = Arrays.asList(folderA, folderB);

		fileRepository.saveAll(saveList);
		folderRepository.saveAll(saveList);

		Folders folders = new Folders(folderRepository.findAll());
		Bookmarks bookmarks = new Bookmarks();

		Files files = new Files(folders, bookmarks);

		files.somethingAllDo();

	}

	@Test
	@DisplayName("북마크만 있을 경우")
	void only_bookmark_exist() {

		Bookmark bookmarkA = new Bookmark();
		bookmarkA.setUrl("www.google.com");

		fileRepository.save(bookmarkA);
		bookmarkRepository.save(bookmarkA);

		Folders folders = new Folders();
		Bookmarks bookmarks = new Bookmarks(bookmarkRepository.findAll());

		Files files = new Files(folders, bookmarks);
		files.somethingAllDo();
	}
}
```

> 도메인 객체 File, Folder, Bookmark

```java
@Entity
@Getter
@Setter @ToString
@NoArgsConstructor
@Inheritance(strategy = InheritanceType.JOINED)
public class File {

	@Id @GeneratedValue(generator = "item_id_generator")
	@GenericGenerator(name = "item_id_generator", strategy = "uuid")
	private String id;

	private String name;

	@ToString.Exclude
	@JoinColumn(name = "PARENT_ID")
	@ManyToOne(fetch = FetchType.LAZY)
	private File parent;

	public File(String id, String name) {
		this.id = id;
		this.name = name;
	}
}

```

```java
@Entity
@Getter @Setter @ToString
@NoArgsConstructor
public class Folder extends File {

	private boolean isShare;

	@OneToMany
	@ToString.Exclude
	private List<File> children = new ArrayList<>();

	public Folder(String id, String name, boolean isShare) {
		super(id, name);
		this.isShare = isShare;
	}

	public void addChildren(File... files) {
		Arrays.stream(files).forEach((File file) -> file.setParent(this));
		children.addAll(Arrays.asList(files));
	}
}
```

```java
@Entity
@Setter
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class Bookmark extends File {

	String url;
}
```

> 일급 컬렉션

```java
@NoArgsConstructor
public class Folders extends Files {

	private List<Folder> folders = new ArrayList<>();

	public Folders(List<Folder> folders) {
		this.folders = folders;
	}

	@Override
	public Boolean isExist() {
		return folders.size() > 0;
	}

	@Override
	public Boolean isNotExist() {
		return !isExist();
	}

	@Override
	public void somethingDo() {
		out.println("---------------------------");
		out.println("#here Folders : 동작 수행");
		out.println("---------------------------");
	}
}
```

```java
@NoArgsConstructor
public class Bookmarks extends Files {

	private List<Bookmark> bookmarks = new ArrayList<>();

	public Bookmarks(List<Bookmark> bookmarks) {
		this.bookmarks = bookmarks;
	}

	@Override
	public Boolean isExist() {
		return bookmarks.size() > 0;
	}

	@Override
	public Boolean isNotExist() {
		return !isExist();
	}

	@Override
	public void somethingDo() {
		out.println("---------------------------");
		out.println("#here Bookmarks : 동작 수행");
		out.println("---------------------------");
	}
}
```

```java
@NoArgsConstructor
public class Files {

	private Folders folders;
	private Bookmarks bookmarks;

	private static final Map<Supplier<Boolean>, Voider> FILES_MAP = new HashMap<>();

	public Files(Folders folders, Bookmarks bookmarks) {
		this.folders = folders;
		this.bookmarks = bookmarks;

		FILES_MAP.put(this::isExist, this::somethingDo);
		FILES_MAP.put(() -> this.folders.isExist() && this.bookmarks.isNotExist(), this.folders::somethingDo);
		FILES_MAP.put(() -> this.bookmarks.isExist() && this.folders.isNotExist(), this.bookmarks::somethingDo);
	}

	public Boolean isExist() {
		return folders.isExist() && bookmarks.isExist();
	}

	public Boolean isNotExist() {
		return !isExist();
	}

	public void somethingDo() {

		out.println("---------------------------");
		out.println("#here Files : 동작 수행 (폴더와 북마크가 모두 있을 경우) ");
		out.println("---------------------------");
	}

	public void somethingAllDo() {

		for (Map.Entry<Supplier<Boolean>, Voider> entry : FILES_MAP.entrySet()) {
			Supplier<Boolean> isFilesExist = entry.getKey();
			if (isFilesExist.get()) {
				Voider somethingDo = entry.getValue();
				somethingDo.execute();
			}
		}
	}

	@FunctionalInterface
	private interface Voider {
		void execute();
	}
}
```

위 코드의 문제점은 `OCP` 원칙이 지키지 않는 다는 것입니다

`OCP` 로직을 추가해도 기존의 클라이언트 코드를 변경하지 않는다

예를들어 `Tag`라는 객체가 추가되었을 때 `Files` 클래스의 생성자의 코드는 아래처럼 수정이 되어야 합니다

```java
public Files(Folders folders, Bookmarks bookmarks, Tags tags) {

    ...

		FILES_MAP.put(() -> this.folders.isExist() && this.bookmarks.isNotExist() && this.tags.isNotExist(), this.folders::somethingDo);
		FILES_MAP.put(() -> this.bookmarks.isExist() && this.folders.isNotExist() && this.tags.isNotExist(), this.bookmarks::somethingDo);
    FILES_MAP.put(() -> this.tags.isExist() && this.bookmarks.isNotExist() && this.folders.isNotExist() , this.bookmarks::somethingDo);
	}
```

수정되어야 할 요소가 많습니다.. ㅠㅠ 생성자 시그니쳐도 바뀌어야할 뿐만 아니라 맵에 담기는 로직도 바뀌어야 합니다

그리고 if ~ else 문만 없지 기존의 Folder와 Bookmark에 있던 로직이 일급 컬렉션 형태로 옮겨간 것 뿐입니다

> 만일 다형성을 이용한 로직으로 변경하고 싶으면 해당 로직을 부모 클래스에 위임하는 방법이 있을 수 있습니다

## 컴포지트 패턴

이때 어떤 특정한 조건을 만족하면 컴포지트 패턴으로의 구조 변경을 시도해볼 수 있다고 생각하였습니다

```java
if(폴더만 존재) { // (1)
  폴더로직1();
  ...
  폴더로직n();

} else if (북마크만 존재){ // (2)
  북마크로직1();
  ...
  북마크로직m();

} else { // 둘다 존재
  폴더로직1();
  ..
  폴더로직n();

  북마크로직1();
  ...
  북마크로직m();
}
```

이떄 중요한 것은 `else` 문의 로직이 정확히 (1) 과 (2) 의 합이어야 한다는 것입니다

로직이 중간에 다르거나 부족하거나 많아도 안됩니다 !

위와 같이 표현이 될 떄 컴포지트 패턴을 이용할 수 있다고 생각하였습니다

### 적용한 코드

---

```java
@Test
	@DisplayName("V2 폴더만 있을 경우")
	public void v2_only_folder() {
		Folder folderA = new Folder();
		folderA.setName("folder a");
		folderA.setShare(false);

		Folder folderB = new Folder();
		folderB.setName("folderB b");
		folderB.setShare(false);

		folderA.addChildren(folderB);

		FileGroup fileGroup = new FileGroup();
		fileGroup.addGroup(folderA, folderB);
		fileGroup.somethingDoAll();
	}

	@Test
	@DisplayName("V2 북마크만 있을 경우")
	public void v2_only_bookmark() {

		Bookmark bookmarkA = new Bookmark();
		bookmarkA.setUrl("www.google.com");

		Bookmark bookmarkB = new Bookmark();
		bookmarkA.setUrl("www.naver.com");

		FileGroup fileGroup = new FileGroup();
		fileGroup.addGroup(bookmarkA, bookmarkB);
		fileGroup.somethingDoAll();
	}

	@Test
	@DisplayName("V2폴더와 북마크 모두 있을 경우")
	void v2_folder_and_bookmark_all_exist() {

		Folder folderA = new Folder();
		folderA.setName("folder a");
		folderA.setShare(false);

		Folder folderB = new Folder();
		folderB.setName("folderB b");
		folderB.setShare(false);

		Bookmark bookmarkA = new Bookmark();
		bookmarkA.setUrl("www.google.com");

		Bookmark bookmarkB = new Bookmark();
		bookmarkA.setUrl("www.naver.com");

		folderA.addChildren(folderB, bookmarkA);

		FileGroup fileGroup = new FileGroup();
		fileGroup.addGroup(bookmarkA, bookmarkB, folderA, folderB);
		fileGroup.somethingDoAll();
	}
```

> 컴퍼지트 패턴의 가장 핵심적인 주인공 _클라쓰_

```java
public class FileGroup extends File {

	private List<File> files = new ArrayList<>();

	public FileGroup() {
	}

	public void addGroup(File... files) {
		this.files.addAll(Arrays.asList(files));
	}

	public void somethingDoAll() {
		for (File file : files) {
			file.somethingDo();
		}
	}

	@Override
	public void somethingDo() {
		if (this.files.size() == 0) {
			return;
		}
		out.println("---------------------------");
		out.println("#here FileGroup : 동작 수행");
		out.println("---------------------------");
	}
}
```

- 그룹은 그룹을 포함할 수 있으며 각 그룹의 리스트를 순회하며 구현체의 로직을 돌린다 !

```java
@Entity
public class Folder extends File {

	@Override
	public void somethingDo() {
		out.println("---------------------------");
		out.println("#here Folder : 동작 수행");
		out.println("---------------------------");
	}
}
```

```java
@Entity
public class Bookmark extends File {

	@Override
	public void somethingDo() {
		out.println("---------------------------");
		out.println("#here Bookmark : 동작 수행");
		out.println("---------------------------");
	}
}
```

#### 위 코드의 문제

> 우선 개별 항목에 대한 적용이 문제이다 !  
> 즉 **단건 처리**가 문제야 !

이것이 순수한 자바 코드만의 문제였다면 상관이 없었다.. 그러나 이건 JPA 영역에서 적용할 문제이다

수십여개의 개별 데이터가 트랜잭션을 남발하면서 IO를 왔다갔다 하면서 DML 을 뿜뿜 뽐내면 우리의 WAS, DB가 힘들어 할 것이다 ㅠ

때문에 단건 처리가 아닌 다건처리를 할 수 있는 방법이 필요하다...

### 일급 컬렉션을 이용한 방법

---

일급 컬렉션을 이용하면 컬렉션을 담은 객체이기 때문에 다건 처리가 가능할 것이라 생각하였다

> 테스트 코드

```java
@Test
@Transactional
@Rollback(false)
@DisplayName("V3 폴더와 북마크 모두 있을 경우")
public void v3_folder_and_bookmark_all_exist() {

  Folder folderA = new Folder();
  folderA.setName("folder a");
  folderA.setShare(false);

  Folder folderB = new Folder();
  folderB.setName("folderB b");
  folderB.setShare(false);

  Bookmark bookmarkA = new Bookmark();
  bookmarkA.setUrl("www.google.com");

  Bookmark bookmarkB = new Bookmark();
  bookmarkA.setUrl("www.naver.com");

  folderA.addChildren(folderB, bookmarkA);

  Folders folders = new Folders(Arrays.asList(folderA, folderB));
  Bookmarks bookmarks = new Bookmarks(Arrays.asList(bookmarkA, bookmarkB));

  FileGroup fileGroup = new FileGroup();
  fileGroup.addGroup(folders);
  fileGroup.addGroup(bookmarks);

  fileGroup.somethingDoAll();
}
```

```java
public class FileGroup extends Files {

	private List<Files> fileList = new ArrayList<>();

	public FileGroup() {
	}

	public void addGroup(Files files) {
		fileList.add(files);
	}

	public void somethingDoAll() {
		for (Files files : fileList) {
			files.somethingDo();
		}
	}

	@Override
	public void somethingDo() {
		out.println("---------------------------");
		out.println("#here FileGroup : 동작 수행");
		out.println("---------------------------");
	}
}
```

그러나 아래와 같은 몇가지 문제가 있었다...

#### 과연 이대로 해도 되는가 ... ?

---

![image](https://user-images.githubusercontent.com/66164361/151975050-5b0ee91f-2941-4174-b34b-94007b1aff50.png)

각 일급컬렉션에서 로직 처리를 수행하는데 이때 일급 컬렉션이 `Repository`를 자연스럽게 의존하게 된다

그리고 더불어 일급컬렉션에서 `FileRepository` DI 주입이 안된다.. (이부분은 알아내지 못하였다.. 부트 실행에서도 적용 안된다 !)

#### 본디 되어야할 의존 관계도 ??

---

![image](https://user-images.githubusercontent.com/66164361/151995216-b1c3b933-407f-4dfe-adbd-be8fb0b31f01.png)

위와 같은 그림이 되어야지 않나 생각하고 있다

그러나 정작 비즈니스 로직 처리의 핵심은 도메인에 있는데 이곳에서 트랜잭션을 수행을 하는 것이 맞는가 싶다..

이 부분에 대해서는 고민이 더욱 필요하다...

#### 분기문을 타야 했었던 것인가

---

애당초 사실은 폴더만 존재, 파일만 존재, 모두 존재할 경우를 나누어야만 했었던 것인가에 대한 의구심이 남아 있다..

이전 코드를 살펴보니 의문점이 남는다 !!... 사실 프로젝트 참여 인원이 아닌 필자가 여기까지 고민하는 것이 이상한 것 같다 ..하하

이 고민은 이만 여기서 마무리하고자 한다
