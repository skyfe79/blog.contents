---
title: "git 여러 개의 로컬 브랜치 삭제하기"
date: 2021-08-18T00:22:40Z
draft: false
tags: ["tips","git"]
---

feature 단위로 나누어 작업하던 여러 개의 로컬 브랜치를 한 번에 지우고 싶을 때가 있다. 

## 브랜치 목록

모든 브랜치 목록은 `--list` 옵션을 사용해 구할 수 있다.

```
$ git branch --list
```

특정 이름으로 시작하는 브랜치 목록은 아래처럼 구할 수 있다.

```
$ git branch | grep 'feature'
```

또는 

```
$ git branch --list 'feature*'
```

## 브랜치 삭제

위에서 얻은 목록을 모두 삭제한다.

```
git branch -D `git brranch | grep 'feature'`
```

또는

```
git branch -D `git branch --list 'feature*'`
```