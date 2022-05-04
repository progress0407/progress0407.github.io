---
layout: post
title: "[엘리 - 비동기] callback hell, promise, async-await"
subtitle: "..."
date: 2022-05-03 00:30:00 +0900
categories: frontend
tags: javascript
comments: true
---

## callback hell

## Promise

promise 의 `then`는 값을 바로 전달할 수도 있고 또 다른 promise 객체를 전달할 수 있다 (`promise chaining`)
![image](https://user-images.githubusercontent.com/66164361/166152972-a67b535a-bd63-48e0-98e8-e0a9df63bfb0.png)

### CallBack To Promise

> 내가 개선한 버전

```js
class UserStorage {
  loginUser(id, password, onSuccess, onError) {
    setTimeout(() => {
      if (id === "philz" && password == "swcho") {
        onSuccess(id);
      } else {
        onError(new Error("not found"));
      }
    }, 1_000);
  }

  getRoles(user, onSuccess, onError) {
    setTimeout(() => {
      if (user === "philz") {
        onSuccess({ name: "philz", role: "admin" });
      } else {
        onError(new Error("no access"));
      }
    }, 1_000);
  }
}

const userStorage = new UserStorage();
const id = prompt("enter your id");
const password = prompt("enter your password");

const login = () =>
  new Promise((resolve, reject) => {
    userStorage.loginUser(id, password, resolve, reject);
  });

const getRole = (userWithRole) =>
  new Promise((resolve, reject) => {
    userStorage.getRoles(userWithRole, resolve, reject);
  });

login() //
  .then(getRole)
  .catch(console.error)
  .then((user) => alert(`Hello ${user.name}, you have a ${user.role} role`))
  .catch(console.error);
```

## Async-Await
