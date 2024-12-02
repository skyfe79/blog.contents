---
title: "String.prototype.trimStart와 String.prototype.trimEnd 메서드"
date: 2024-11-23T02:57:16Z
author: "skyfe79"
draft: false
tags: ["javascript","web","아마추어 번역","V8","\bECMAScript"]
---

ES2019에서는 [`String.prototype.trimStart()`와 `String.prototype.trimEnd()` 메서드](https://github.com/tc39/proposal-string-left-right-trim)가 새롭게 도입됐다:

```javascript
const string = '  hello world  ';
string.trimStart();
// → 'hello world  '
string.trimEnd();
// → '  hello world'
string.trim(); // ES5
// → 'hello world'
```

이 기능은 이전에 비표준 메서드인 `trimLeft()`와 `trimRight()`로 사용할 수 있었다. 이 메서드들은 하위 호환성을 위해 새로운 메서드의 별칭으로 남아있다.

```javascript
const string = '  hello world  ';
string.trimStart();
// → 'hello world  '
string.trimLeft();
// → 'hello world  '
string.trimEnd();
// → '  hello world'
string.trimRight();
// → '  hello world'
string.trim(); // ES5
// → 'hello world'
```

## `String.prototype.trim{Start,End}` 지원 현황

- Chrome: 버전 66부터 지원
- Firefox: 버전 61부터 지원
- Safari: 버전 12부터 지원
- Node.js: 버전 8부터 지원
- Babel: 지원

## 알림

이 글은 [v8.dev](https://v8.dev/)에 2018년 3월 26일 발행된 [String.prototype.trimStart and String.prototype.trimEnd](https://v8.dev/features/string-trimming) 글을 한국어로 편역한 내용을 담고 있습니다.

