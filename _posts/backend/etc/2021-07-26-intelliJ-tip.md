---
layout: post
title: "IntelliJ 팁"
subtitle: "..."
date: 2021-07-26 22:15:00 +0900
categories: backend
tags: etc
comments: true
---

## 터미널에서 한글이 깨질때

---

![image](https://user-images.githubusercontent.com/66164361/126995355-750969de-8013-49f4-a66b-0cfc0d688226.png)
위와 같이 환경 변수를 편집합니다

![image](https://user-images.githubusercontent.com/66164361/126995221-3b8736d3-dafb-44d1-a8ed-c9aef01e1885.png)

위와 같이 설장합니다
Shell Path를 `cmd`에서 `git bash` 로 바꾸면 `Linux Shell`이 열립니다

![image](https://user-images.githubusercontent.com/66164361/126995518-cf651f50-fa76-495b-af04-4d5bab17d617.png)

위와 같이 한글이 정상적으로 동작합니다.

### 윈도우 cmd 한글 깨짐 encoding 수정

---

#### 일시적용

cmd에 `chcp 65001` : Change CodePage

- `65001` : UTF-8
- `949` : EUC-KR (한국어 전용)

#### 영구적용

- regedit
  - HKEY_CURRENT_USER
    - Console
      - %SystemRoot%\_system32_cmd.exe 키생성
        - CodePage 키 생성
          - 10진수로 65001 입력

![image](https://user-images.githubusercontent.com/66164361/127558391-940a277c-df24-47d6-81b6-0fb3d11f5d20.png)

### 인텔리제이가 자바를 실행하는 명령어 ?!

---

SubDomain 의 main 함수를 이렇게 실행했다.. 신기해서 올림

![image](https://user-images.githubusercontent.com/66164361/128587548-198d6929-897a-461a-b87e-09c99df92726.png)

```shell
C:\IDEs\Java\jdk_11\bin\java.exe "-javaagent:C:\IDEs\IntelliJ IDEA Community Edition 2020.2.3\lib\idea_rt.jar=11312:C:\IDEs\IntelliJ IDEA Community Edition 2020.2.3\bin" -Dfile.encoding=UTF-8 -classpath C:\IDEs\IntelliJ_workspace\pure_java_project\out\production\pure_java_project designpattern.adapter.n1.SubDomain
```

위 코드를 다시 chunk단위로, 의미별로 나누어 보자

```
C:\IDEs\Java\jdk_11\bin\java.exe
```

- 11버전의 java를 실행해라

```
"-javaagent:C:\IDEs\IntelliJ IDEA Community Edition 2020.2.3\lib\idea_rt.jar=11312:C:\IDEs\IntelliJ IDEA Community Edition 2020.2.3\bin"
```

- IDE는 인텔리제이를 사용한다

```
-Dfile.encoding=UTF-8
```

- UTF-8 인코딩을 사용한다

```
-classpath C:\IDEs\IntelliJ_workspace\pure_java_project\out\production\pure_java_project designpattern.adapter.n1.SubDomain
```

- -classpath에 해당프로젝트의 경로를 1번째 인자로
- 2번째 인자는 main함수가 위치한 패키지 경로 + 클래스명울 전달한다
