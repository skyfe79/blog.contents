---
title: "Promise.prototype.finally"
date: 2024-10-21T13:03:55Z
author: "skyfe79"
draft: false
tags: ["javascript","web","V8","\bECMAScript"]
---

`Promise.prototype.finally`는 Promise가 *처리됐을 때*(즉, resolve되거나 reject됐을 때) 실행할 콜백을 등록할 수 있게 한다.

웹 페이지에 표시할 데이터를 가져오는 상황을 생각해보자. 요청이 시작될 때 로딩 스피너를 표시하고, 요청이 완료되면 숨기려고 한다. 문제가 발생하면 대신 오류 메시지를 표시한다.

```javascript
const fetchAndDisplay = ({ url, element }) => {
  showLoadingSpinner();
  fetch(url)
    .then((response) => response.text())
    .then((text) => {
      element.textContent = text;
      hideLoadingSpinner();
    })
    .catch((error) => {
      element.textContent = error.message;
      hideLoadingSpinner();
    });
};

fetchAndDisplay({
  url: someUrl,
  element: document.querySelector('#output')
});
```

요청이 성공하면 데이터를 표시한다. 문제가 발생하면 대신 오류 메시지를 표시한다.

두 경우 모두 `hideLoadingSpinner()`를 호출해야 한다. 지금까지는 `then()`과 `catch()` 블록 모두에서 이 호출을 중복해야 했다. `Promise.prototype.finally`를 사용하면 이를 개선할 수 있다:

```javascript
const fetchAndDisplay = ({ url, element }) => {
  showLoadingSpinner();
  fetch(url)
    .then((response) => response.text())
    .then((text) => {
      element.textContent = text;
    })
    .catch((error) => {
      element.textContent = error.message;
    })
    .finally(() => {
      hideLoadingSpinner();
    });
};
```

이 방식은 코드 중복을 줄일 뿐만 아니라, 성공/오류 처리 단계와 정리 단계를 더 명확히 분리한다. 깔끔하지 않은가!

현재는 `async`/`await`를 사용해 `Promise.prototype.finally` 없이도 같은 작업을 수행할 수 있다:

```javascript
const fetchAndDisplay = async ({ url, element }) => {
  showLoadingSpinner();
  try {
    const response = await fetch(url);
    const text = await response.text();
    element.textContent = text;
  } catch (error) {
    element.textContent = error.message;
  } finally {
    hideLoadingSpinner();
  }
};
```

[`async`와 `await`가 명확히 더 나은 방식](https://mathiasbynens.be/notes/async-stack-traces)이므로, 이를 사용할 것을 권장한다. 다만, 어떤 이유로 일반 Promise를 선호한다면, `Promise.prototype.finally`를 사용해 코드를 더 단순하고 깔끔하게 만들 수 있다.

## `Promise.prototype.finally` 지원 현황

- [Chrome: 버전 63부터 지원](https://v8.dev/blog/v8-release-63)
- Firefox: 버전 58부터 지원
- Safari: 버전 11.1부터 지원
- Node.js: 버전 10부터 지원
- [Babel: 지원됨](https://github.com/zloirock/core-js#ecmascript-promise)

## 알림

이 글은 [v8.dev](https://v8.dev/)에 2017년 10월 23일 발행된 [Promise.prototype.finally](https://v8.dev/features/promise-finally) 글을 한국어로 편역한 내용을 담고 있습니다.

