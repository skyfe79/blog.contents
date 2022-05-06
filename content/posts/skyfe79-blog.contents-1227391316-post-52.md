---
title: "VSCode에서 HTML API 자동완성을 쉽게 사용하는 방법은?"
date: 2022-05-06T03:58:35Z
author: "skyfe79"
draft: false
tags: ["tips","javascript"]
---

Visual Studio Code에서 HTML API를 쓸 때, 타입스크립트를 적용하지 않고 자동완성 기능을 사용할 수 있다. VSCode는 JSDoc을 지원하기 때문에 [JSDoc](https://jsdoc.app/tags-type.html)형식으로 타입을 지정하는 주석을 작성하면 된다.  짧은 코드를 작성할 때 꽤 유용하다.

예를 들어 HTML5 Canvas API를 사용할 때, 아래와 같이 사용하면 자동완성을 지원 받을 수 있다.

```js
/** @type {HTMLCanvasElement} */
const canvas = document.getElementById('my-canvas');
canvas.setAttribute('width', '1000');
canvas.setAttribute('height', '1000');
```

<img width="552" alt="스크린샷 2022-05-06 오후 12 55 42" src="https://user-images.githubusercontent.com/309935/167064470-6aa18947-adb7-42c5-89ae-9872973f692a.png">

```js
/** @type {HTMLCanvasElement} */
const canvas = document.getElementById('my-canvas');
canvas.setAttribute('width', '1000');
canvas.setAttribute('height', '1000');

/** @type {CanvasRenderingContext2D} */
const context = canvas.getContext('2d');
```

<img width="597" alt="스크린샷 2022-05-06 오후 12 56 14" src="https://user-images.githubusercontent.com/309935/167064514-3f8e58af-d3ad-424d-81cd-10b8ae9def7e.png">





