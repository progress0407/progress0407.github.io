---
layout: post
title: 생코 리눅스 관련 메모
subtitle: "..."
categories: backend
tags: linux
comments: true
published: true
---

### foreground, background

`ctrl+Z` 를 누르면 `background`로 전환한다  
`fg`를 입력하면 `foreground`로 전환  
`jobs`를 입력하면 `backgound` 목록이 보인다  
`fg {넘버}` 혹은 `fg %{넘버}`를 입력하면 넘버에 해당하는 백을 포로 전환  
`kill %{넘버}` 백그라운드 종료
`kill -9 %{넘버}` 강제 종료  
` ls -alR / > result.txt 2> error.log &`

- 출력의 결과를 txt에
- 에러로그를 따로 저장
- 백그라운드 서비스로 전환하기
- `>` 는 해당 문자열을 새롭게 update, `>>`는 마지막줄에 insert

![image](https://user-images.githubusercontent.com/66164361/127351870-dc3c01fd-5246-40ef-bad7-1e0ba6d8936c.png)

### Daemon

Daemon : 반신 반의, 항상 실행되고 있다

> `서버`라는 프로그램은 데몬에 해당한다 !  
> 반대로 말하면 사용자는 WebBrowser를 통해 언제든 접근하므로 (WebBrowser라는 프로그램은 언제든 키고 끌수 있다)  
> 하지만 서버는 언제든 접근할지 모르니 데몬/서비스 라는 형태의 프로그램으로 구동되어야만 한다!

기본적으로 daemon에 해당하는 프로그램들은 `/etc/init.d` 라는 경로에 위차한다

### cron

정기적으로 실행하는 프로그램/도구

`crontab -e` 스케쥴링 설정

> m h dom mon dow command  
> minute hour dayOfMonth month dayOfWeek command  
> 분 시간 일 월 요일 명령문  
> 1 10  
> : 1시 10분에 한번  
> \*/10  
> : 10분마다 한번

`crontab expression` 이라고 이미지 검색을 하면 여러 정보를 알 수 있다

예제

- `1/* * * * * date >> date.log 2>&1`
  - 2는 표준 에러를 뜻하면 뒤의 1은 표준 출력을 뜻한다  
    에러가 발생했을 때 출력하여 로그에 기록한다  
    &1이 아닌 1을 넣으면 1이란 숫자에 기록한다

### help, man

- 기본적으로 `ls`, `mkdir` 등은 내장 프로그램이다.  
  이것을 명령어처럼 사용하는 것일 뿐이다

- `{명령어} --help` 간단한 메뉴얼
- `man {명령어}` 상세 메뉴얼
  - `/{찾을 문자열}` 로 검색 가능
  - `n`은 뒤로
  - `q`는 빠져나옴
