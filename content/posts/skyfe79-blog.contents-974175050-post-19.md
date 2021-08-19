---
title: "iOS 시뮬레이터 타임존 변경하기"
date: 2021-08-19T01:15:04Z
draft: false
tags: ["tips","iOS"]
---

iOS 시뮬레이터에서 타임존을 변경하는 옵션이 현재는 없다. 좀 만들어 주지~~ 😭 방법을 찾아 보니 프로젝트의 Scheme 에 환경 변수 값을 등록하여 타임존을 변경할 수 있었다.

<img width="935" alt="settimezone" src="https://user-images.githubusercontent.com/309935/129991832-deedf896-0e08-4dd4-b41b-fcbab4869281.png">

`Environment Variables` 에 `TZ` 변수를 설정하고 원하는 타임존 값을 입력한다. 아래와 같은 값을 입력하면 된다.

- UTC
- PST
- EST
- 또는
- Asia/Seoul
- America/Los_Angeles

그러나 환경 변수를 설정하면 프로젝트 파일이 변경되기 때문에 협업 등을 고려하면 별로 좋지 않은 방법 같다. 아이폰 시뮬레이터를 타임존 옵션을 주어 실행하는 방법이 있을까? 해서 찾아 보았지만 현재까지는 없는 것 같다.🤪

