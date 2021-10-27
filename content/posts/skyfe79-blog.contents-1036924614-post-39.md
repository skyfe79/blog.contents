---
title: "VSCode, tailwindcss 적용 팁"
date: 2021-10-27T03:09:47Z
author: "skyfe79"
draft: false
tags: ["tips"]
---

`@apply`를 사용하여  tailwindcss 스타일을 정의하여 엘리먼트 간에 스타일을 공유할 수 있다. 
vue 프로젝트에 tailwindcss를 적용할 때, `@apply` 사용하면 오류로 표시되어 불편하다. 

![img1](https://user-images.githubusercontent.com/309935/138993458-0efd4c41-59ef-4020-b14f-efc5cde77f1e.png)



style에 `lang="postcss`을 적용하면 이 문제가 해결된다.

![img2](https://user-images.githubusercontent.com/309935/138993575-a2d91384-cef5-4bd4-9f3a-e8cff054539d.png)


