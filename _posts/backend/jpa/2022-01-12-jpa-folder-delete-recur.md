---
layout: post
title: "[BFS] 하위 폴더 최소의 쿼리로 지우기"
subtitle: "..."
date: 2022-01-12 23:00 +0900
categories: backend
tags: jpa
comments: true
---

## 퇴근하는 도중에...

오늘 퇴근 하는 도중에 핑구님이 문의를 하였다

폴더를 재귀적으로 탐색할 것 같은데 이때 수시로 나가게 되는 쿼리에 대한 고민이었다고 한다

얘기를 듣다보니 굉장히 흥미롭게 들리는 주제여서 지하철 내내 해당 문제를 어떻게 풀어나갈 것인가에 대해 생각에 잠기었다

고민하는 와중에 얼떨결에 미팅이 21:30에 잡혔다 ㅋㅋㅋ

BFS 알고리즘 구현이 머릿속에 바로 안그려져서 유튜브로 복습도 하였다 ~~하하~~

원래는 헬스장을 들렸다가 저녁후 씻은 담에 참여하려고 했는데 계속 내내 생각에 쌓여서 코딩부터 시작했다

대략 19:30 ~ 21:00 까지 코딩후 밥먹고 바로 참여한 것 같다

정말 시간가는 줄 몰랐다

아래는 문의에 대해 고민한 흔적들이다.. ㅎㅎ

우선 나는 BFS로 접근을 해보았다...

![image](https://user-images.githubusercontent.com/66164361/149159460-cafa7386-0fee-45c8-9d83-bc060b9afc91.png)

BFS로 접근한 이유는 보통 폴더를 여러개 수직으로 넣어서 쓰기 보다는 한 폴더내에 여러 파일을 적재하기 때문이다

실제로 필자의 바탕화면에 있는 폴더를 보았다

![image](https://user-images.githubusercontent.com/66164361/149166576-8b5ac2f0-9167-4dec-a53f-c2d5fc1e6ff9.png)

위와 같이 파일이 대략 2만개로 가정하였다

또한 필자가 가지고 있는 폴더중 깊이 깊은 폴더를 보았다

![image](https://user-images.githubusercontent.com/66164361/149167012-6995f12f-9e2d-4158-b30b-d27fc77439f5.png)

깊이는 5이다 !

폴더 하나당 가지고 있는 평균적인 폴더+파일의 갯수를 x라 했을 때

대략적으로 depth가 깊어짐에 따라 생기는 폴더+파알의 갯수는 x의 거듭제곱만큼 커진다고 가정하였다

즉..

> x^1 + x^2 ... x^5
> 대략 총합이 x^6으로 가정했을 때

x는 5.3 정도가 나온다

폴더 하나당 5.3개의 폴더+파일을 가지고 있는 셈이라고 가정할 수 있다

![image](https://user-images.githubusercontent.com/66164361/149170909-f0c75010-f42f-4c13-8100-ed9e409fbff6.png)

폴더가 k개 있다면 총합은 거듭제곱스럽게 커진다

(밑변의 수식은 오해의 소지가 있을 것 같습니다! 정확하지 않다 !)

따라서 DFS보다는 BFS로 갯수를 파악하는게 좋을 것이라고 생각하였다

아래는 위 상황을 생각하면서 구현해본 코드이다

```java
@Getter
@Setter
@ToString
private abstract static class File {
  private String id;
  private String parentId;
  private String name;
  private boolean isDeleted;
}
```

추상 클래스로 정의했지만 이곳에서 사용은 하지 않는다 ㅜ

```java
@Getter
@Setter
private static class Folder extends File {
  private String id;
  private String parentId;
  private String name;
  private boolean isDeleted;

  public Folder(String id, String parentId, String name) {
    this.id = id;
    this.parentId = parentId;
    this.name = name;
    this.isDeleted = false;
  }

  @Override
  public String toString() {
    return "Folder{" +
      "name='" + name + '\'' +
      ", isDeleted=" + isDeleted +
      '}';
  }
}
```

단방향으로 부모의 ID를 갖는 폴더 클래스를 정의했다

아래의 Bookmark도 마찬가지다

```java
@Getter
@Setter
private static class Bookmark extends File {
  private String id;
  private String parentId;
  private String name;
  private boolean isDeleted;

  public Bookmark(String id, String parentId, String name) {
    this.id = id;
    this.parentId = parentId;
    this.name = name;
    this.isDeleted = false;
  }

  @Override
  public String toString() {
    return "Bookmark{" +
      "name='" + name + '\'' +
      ", isDeleted=" + isDeleted +
      '}';
  }
}
```

```java
private static class BookmarkRepository /*extends DataRepository*/ {

  public static void add(Bookmark bookmark) {
    // DataRepository.add(bookmark);
    bookmarks.add(bookmark);
  }

  public static List<Bookmark> findByParentId(String id) {
    return bookmarks.stream()
      .filter(bookmark -> bookmark.getParentId().equals(id) && !bookmark.isDeleted())
      .collect(Collectors.toList());
  }

  private static void deleteBookmarks(List<Bookmark> bookmarks) {
    for (Bookmark bookmark : bookmarks) {
      out.println("delete Bookmark = " + bookmark);
      bookmark.setDeleted(true);
    }
  }
}
```

```java
private static class FolderRepository /*extends DataRepository*/ {

  public static void add(Folder folder) {
    // DataRepository.add(folder);
    folders.add(folder);
  }

  public static List<Folder> findByParentId(String id) {
    return folders.stream()
      .filter(folder -> folder.getParentId().equals(id) && !folder.isDeleted())
      .collect(Collectors.toList());
  }

  private static void deleteFolders(List<Folder> folders) {
    for (Folder folder : folders) {
      out.println("delete Folder = " + folder);
      folder.setDeleted(true);
    }
  }
}
```

```java
/*
	private static abstract class DataRepository {

		public static void add(File file) {
			files.add(file);
		}

		public static List<File> findByParentId(String id) {
			return files.stream()
				.filter(file -> file.getParentId().equals(id) && !file.isDeleted())
				.collect(Collectors.toList());
		}

	}
*/
```

두 Repository를 포함하는 저장소를 정의해보았지만 오늘은 사용하지 못했다

```java
public static void main(String[] args) {
  initSampleData();

  Queue<String> queue = new LinkedList<>();
  queue.offer("/");


  List<Folder> tempFolders = new ArrayList<>();
  List<Bookmark> tempBookmarks = new ArrayList<>();


  while (!queue.isEmpty()) {
    String parentId = queue.poll();
    List<Folder> folders = FolderRepository.findByParentId(parentId);
    List<Bookmark> bookmarks = BookmarkRepository.findByParentId(parentId);

    // 폴더를 큐에 넣기
    for (Folder folder : folders) {
      queue.offer(folder.getId());
    }

    // 북마크를 큐에 넣기
    for (Bookmark bookmark : bookmarks) {
      queue.offer(bookmark.getId());
    }

    // list 에 먼저 담고...

    // 폴더 삭제
    if(folders.size() > 0) {
      out.println("-------------------- delete query :: folder :: start --------------------");
      FolderRepository.deleteFolders(folders);
      out.println("-------------------- delete query :: folder :: end --------------------\n\n");
    }

    // 북마크 삭제
    if (bookmarks.size() > 0) {
      out.println("-------------------- delete query :: bookmark :: start --------------------");
      BookmarkRepository.deleteBookmarks(bookmarks);
      out.println("-------------------- delete query :: bookmark :: end --------------------\n\n");
    }
  }
}
```

이때 위처럼

개수를 체크하는 로직(`.size() > 0`)이 들어가지 있지 않으면

값이 없음에도 불구하고 쿼리가 나갈 수 있다고 생각하였다 !!

```java
private static void initSampleData() {
  String uuid = getUuid();
  FolderRepository.add(new Folder(uuid, "/", "폴더 A"));

  String uuid2 = getUuid();
  FolderRepository.add(new Folder(uuid2, uuid, "폴더 B"));

  String uuid3 = getUuid();
  BookmarkRepository.add(new Bookmark(uuid3, uuid, "북마크 A"));

  String uuid4 = getUuid();
  FolderRepository.add(new Folder(uuid4, uuid2, "폴더 C"));

  String uuid5 = getUuid();
  FolderRepository.add(new Folder(uuid5, uuid2, "폴더 D"));

  String uuid6 = getUuid();
  BookmarkRepository.add(new Bookmark(uuid6, uuid4, "북마크 B"));

  String uuid7 = getUuid();
  BookmarkRepository.add(new Bookmark(uuid7, uuid4, "북마크 C"));

  String uuid8 = getUuid();
  BookmarkRepository.add(new Bookmark(uuid8, uuid5, "북마크 D"));
}
```

초기 데이터를 생성하였다

![image](https://user-images.githubusercontent.com/66164361/149171613-ca4eaef1-1533-47d5-9c91-42f22c897834.png)

대략 위와 같은 자료구조이다

```java
	private static String getUuid() {
		return UUID.randomUUID().toString();
	}
```

> 실행결과

```
-------------------- delete query :: folder :: start --------------------
delete Folder = Folder{name='폴더 A', isDeleted=false}
-------------------- delete query :: folder :: end --------------------


-------------------- delete query :: folder :: start --------------------
delete Folder = Folder{name='폴더 B', isDeleted=false}
-------------------- delete query :: folder :: end --------------------


-------------------- delete query :: bookmark :: start --------------------
delete Bookmark = Bookmark{name='북마크 A', isDeleted=false}
-------------------- delete query :: bookmark :: end --------------------


-------------------- delete query :: folder :: start --------------------
delete Folder = Folder{name='폴더 C', isDeleted=false}
delete Folder = Folder{name='폴더 D', isDeleted=false}
-------------------- delete query :: folder :: end --------------------


-------------------- delete query :: bookmark :: start --------------------
delete Bookmark = Bookmark{name='북마크 B', isDeleted=false}
delete Bookmark = Bookmark{name='북마크 C', isDeleted=false}
-------------------- delete query :: bookmark :: end --------------------


-------------------- delete query :: bookmark :: start --------------------
delete Bookmark = Bookmark{name='북마크 D', isDeleted=false}
-------------------- delete query :: bookmark :: end --------------------
```

## 발표후 피드백

사람들이 모두 숨도 안쉬는 것처럼 집중해서 경청해주어서 너무 고마웠다 ㅠㅠ

우선 미스터박님의 피드백으로

- Bookmark가 굳이 `queue`에 넣을 필요가 없다

  - `BFS`를 처음 보시는 것 같았는데 바로 알아채리셨다.. 예리함.. ㄷ.ㄷ~~
    실제로도 필요 없었고 저부분은 지웠다 !

- spring data jpa 의 deleteAll 은 쿼리가 갯수만큼 나간다
  - JPQL이나 queryDsl의 문법을 활용하자 !

그리고 오디님의 조언으로는

- 갯수를 체크하는 로직을 Repository를 내부에 놓자
  - 사용자가 API를 사용할 때 편하게
  - SRP 객체에 메세지를 보내자 !
- 삭제할 대상을 list에 임시로 담아두고 삭제 로직은 while문 밖에 보내어 쿼리를 줄인다

아래는 다시 구현해본 삭제 쿼리이다

```java
Queue<String> queue = new LinkedList<>();
queue.offer("/");

List<Folder> tempFolders = new ArrayList<>();
List<Bookmark> tempBookmarks = new ArrayList<>();

while (!queue.isEmpty()) {
  String parentId = queue.poll();
  List<Folder> folders = FolderRepository.findByParentId(parentId);
  List<Bookmark> bookmarks = BookmarkRepository.findByParentId(parentId);

  // 폴더를 큐에 넣기
  for (Folder folder : folders) {
    queue.offer(folder.getId());
  }

  tempFolders.addAll(folders);
  tempBookmarks.addAll(bookmarks);
}

out.println("-------------------- delete query :: folder :: start --------------------");
FolderRepository.deleteFolders(tempFolders);
out.println("-------------------- delete query :: folder :: end --------------------\n\n");

out.println("-------------------- delete query :: bookmark :: start --------------------");
BookmarkRepository.deleteBookmarks(tempBookmarks);
out.println("-------------------- delete query :: bookmark :: end --------------------\n\n");
```

```java
private static void deleteFolders(List<Folder> folders) {
  if(folders.size() == 0) {
    return;
  }
  for (Folder folder : folders) {
    out.println("delete Folder = " + folder);
    folder.setDeleted(true);
  }
}
```

Bookmark는 이하 동일하다 !

> 실행결과

```
-------------------- delete query :: folder :: start --------------------
delete Folder = Folder{name='폴더 A', isDeleted=false}
delete Folder = Folder{name='폴더 B', isDeleted=false}
delete Folder = Folder{name='폴더 C', isDeleted=false}
delete Folder = Folder{name='폴더 D', isDeleted=false}
-------------------- delete query :: folder :: end --------------------


-------------------- delete query :: bookmark :: start --------------------
delete Bookmark = Bookmark{name='북마크 A', isDeleted=false}
delete Bookmark = Bookmark{name='북마크 B', isDeleted=false}
delete Bookmark = Bookmark{name='북마크 C', isDeleted=false}
delete Bookmark = Bookmark{name='북마크 D', isDeleted=false}
-------------------- delete query :: bookmark :: end --------------------

```
