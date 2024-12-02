---
title: "Function.prototype.toString 메서드의 개선"
date: 2024-11-23T02:54:15Z
author: "skyfe79"
draft: false
tags: ["javascript","web","아마추어 번역","V8","\bECMAScript"]
---

[`Function.prototype.toString()`](https://tc39.es/Function-prototype-toString-revision/) 메서드가 개선되어 이제 공백과 주석을 포함한 정확한 소스 코드 텍스트를 반환한다. 다음 예제는 이전 동작과 새로운 동작을 비교해 보여준다:

```javascript
// function 키워드와 함수 이름 사이의 주석,
// 그리고 함수 이름 뒤의 공백에 주목하자.
function /* 주석 */ foo () {}

// 이전 V8 버전에서:
foo.toString();
// → 'function foo() {}'
//             ^ 주석 없음
//                ^ 공백 없음

// 현재 버전:
foo.toString();
// → 'function /* 주석 */ foo () {}'
```

## 기능 지원 현황

- Chrome: 버전 66부터 지원
- Firefox: 지원
- Safari: 지원하지 않음
- Node.js: 버전 8부터 지원
- Babel: 지원하지 않음

## 알림

이 글은 [v8.dev](https://v8.dev/)에 2018년 3월 25일 발행된 [Revised Function.prototype.toString](https://v8.dev/features/function-tostring) 글을 한국어로 편역한 내용을 담고 있습니다.

