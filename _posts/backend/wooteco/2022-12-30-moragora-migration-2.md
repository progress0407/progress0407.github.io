---
layout: post
title: "마지막 미션, 모라고라(체크메이트) 마이그레이션 (2)"
subtitle: "..."
date: 2022-12-30 19:00:00 +0900
categories: backend
tags: wooteco
comments: true
published: true
---

저번에 마이그레이션하면서 쓰디 쓴 맛을 많이 맛보았다...

그러면서 깨닫거나 전략을 수정한 것을 기록하였다.

## 변경 사항

### Docker 포기

본인도 그렇고 주변 지인에게 물어본 결과 도커는 생각보다 많은 메모리를 사용했다. 

AWS에서 제공되는 무료 티어 스펙상 CPU는 괜찮으나 메모리가 1GiB 밖에 되지 않아서 아껴 써야 했다...

따라서 도커를 포기했다.

### 계~속해서 실패했던 HTTPS

분명 옆에 있던 팀원이 30분가량 적용하는 것을 보고 쉽게 접근했던 것 같다.

분명 비슷하게 적용한 것 같은데... 나의 경우 잘 되지 않았다.

문제는 크게 2가지를 꼽을 수 있다.

1. **443 port 허용 하지 않음**

기초적으로 체크해야했던 사항인데 ec2 SG 정책에 443을 허용 안 했던 것 같다. ㅠㅠ 난 바보다.

2. **ufw 적용시, 22번 Port를 허용하지 않음**

가장 중요한 내용이다! 잘못하다간 서버에 두 번 다시 접속 못할 수 있다!!

앞에 있는 내용이야 언제든 재시도를 할 수 있지만 이건 그렇지 않다!

ufw 를 활성화(`ufw enable`) 후 `ufw status`로 확인해서 SSH가 허용되었는 지를 체크해야 한다!

그렇지 않을 경우 터미널로 접속할 수 없다!!

## 알게된 사항 메모

### **★★ufw 작업시 22번 포트를 열자**

**정말 정말 중요하다!!!**

방화벽을 enable 하면 이 순간 22번 포트에 대한 접근도 끊기는 것으로 보인다. 바로 열어줘야 한다!

[깨닫게 해준 블로그](https://milkye.tistory.com/343)

### npm build 퍼센트 안 올라갈 때

빌드시 사용되는 RAM이 부족한 걸로 보인다.

swap을 적용하니 바로 되는 것을 확인할 수 있었다.

### Test 제외하고 Build 하기

```sh
./gradlew build -x test
```

### 혼합된 콘텐츠 차단

크롬에서는 HTTPS와 HTTP가 혼용된 경우 컨텐츠를 차단한다.

![image](https://user-images.githubusercontent.com/66164361/210137555-61fb213f-b949-45b4-b53d-cb90e9893942.png)

나의 경우 정적 자원 제공은 HTTPS, API요청은 8080으로 되어있었다. (하나의 ec2에 있기 떄문...)

따러서 HTTPS 요청이되, uri path로 분기되게끔 처리를 바꾸었다. (테스트용 API 하나에 대해서만 처리했다)

![image](https://user-images.githubusercontent.com/66164361/210138235-0905099d-c47d-42c6-a6a3-f8972b2595f9.png)

감격.. 드디어 로그인이 된다 ... (어휴) ㅋㅋㅋ


## 배포

![image](https://user-images.githubusercontent.com/66164361/210138934-b8aec947-cd95-4e39-8fe4-550c6d4cb8b1.png)

드디어 되었다.

개인적인 시도와 기존과 마이그레이션 전략 등을 생각하다 보니 많이 늦은 것 같다... ~~(많이 놀기도 했구...)~~

백엔드는 괜찮지만 필요할 때는 프론트 친구들이 활용을 곧잘 하곘지...?

난 이만...

- [마이그레이션 Git Issue](https://github.com/woowacourse-teams/2022-moragora/issues/597)

## 기타

- [nginx guide](https://idroot.us/install-nginx-with-lets-encrypt-ubuntu-22-04/)

