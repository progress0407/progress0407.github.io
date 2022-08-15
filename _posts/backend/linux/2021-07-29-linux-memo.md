---
layout: post
title: 리눅스 관련 메모
subtitle: "..."
categories: backend
tags: linux
comments: true
published: true
---

### 리눅스에서 telnet처럼 특정 서비스 포트 열렸는지 확인

`apt-get install traceroute` 설치

`traceroute -V` 확인

`traceroute {상대방 ip 또는 도메인} -p {포트번호}` 상대방 포트확인

### vm에서 로컬의 db 접속 가능하게 하기 (미해결

`wf.msc` 방화벽 허용 인바운드 규칙

> https://hjw1456.tistory.com/16

### windows에 설치된 Git Bash 명령어 커스텀

bash를 관리자 권한으로 실행한다

`ll /etc/profile.d/` 이곳으로 이동한다

`aliases.sh`을 열어서 명령어를 커스텀한다

git-prompt 파일은 PS1 프롬프트를 커스텀할 수 있는 것으로 보인다

> https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/How-to-customize-Git-Bash-Shell-prompt-settings  
> https://coding-chobo.tistory.com/72
