---
title: "객체의 rest와 spread 속성"
date: 2024-10-20T12:28:09Z
author: "skyfe79"
draft: false
tags: ["web","V8","\bECMAScript"]
---

객체의 rest와 spread 속성에 대해 설명하기 전에, 비슷한 기능을 먼저 살펴보자.

## ES2015의 배열 rest와 spread 요소

ECMAScript 2015에서는 배열 구조 분해 할당을 위한 rest 요소와 배열 리터럴을 위한 spread 요소를 도입했다.

```javascript
// 배열 구조 분해 할당을 위한 rest 요소:
const primes = [2, 3, 5, 7, 11];
const [first, second, ...rest] = primes;
console.log(first);  // 2
console.log(second); // 3
console.log(rest);   // [5, 7, 11]

// 배열 리터럴을 위한 spread 요소:
const primesCopy = [first, second, ...rest];
console.log(primesCopy); // [2, 3, 5, 7, 11]
```

- Chrome: 버전 47부터 지원
- Firefox: 버전 16부터 지원
- Safari: 버전 8부터 지원
- Node.js: 버전 6부터 지원
- Babel: 지원

## ES2018: 객체의 rest와 spread 속성 🆕

그렇다면 새로운 점은 무엇일까? [tc39 Object Rest/Spread Properties for ECMAScript 제안](https://github.com/tc39/proposal-object-rest-spread)에서 객체 리터럴에도 rest와 spread 속성을 사용할 수 있게 했다.

```javascript
// 객체 구조 분해 할당을 위한 rest 속성:
const person = {
    firstName: 'Sebastian',
    lastName: 'Markbåge',
    country: 'USA',
    state: 'CA',
};
const { firstName, lastName, ...rest } = person;
console.log(firstName); // Sebastian
console.log(lastName);  // Markbåge
console.log(rest);      // { country: 'USA', state: 'CA' }

// 객체 리터럴을 위한 spread 속성:
const personCopy = { firstName, lastName, ...rest };
console.log(personCopy);
// { firstName: 'Sebastian', lastName: 'Markbåge', country: 'USA', state: 'CA' }
```

Spread 속성은 여러 상황에서 [`Object.assign()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)보다 더 우아하고 좋은 방법을 제공한다:

```javascript
// 객체의 얕은 복사:
const data = { x: 42, y: 27, label: 'Treasure' };
// 기존 방식:
const clone1 = Object.assign({}, data);
// 새로운 방식:
const clone2 = { ...data };
// 두 방식 모두 다음과 같은 결과를 얻는다:
// { x: 42, y: 27, label: 'Treasure' }

// 두 객체 병합:
const defaultSettings = { logWarnings: false, logErrors: false };
const userSettings = { logErrors: true };
// 기존 방식:
const settings1 = Object.assign({}, defaultSettings, userSettings);
// 새로운 방식:
const settings2 = { ...defaultSettings, ...userSettings };
// 두 방식 모두 다음과 같은 결과를 얻는다:
// { logWarnings: false, logErrors: true }
```

하지만 spread가 setter를 다루는 방식에는 미묘한 차이가 있다:

1. `Object.assign()`은 setter를 트리거하지만, spread는 그렇지 않다.
2. 상속된 읽기 전용 속성을 통해 `Object.assign()`이 자체 속성을 생성하는 것을 막을 수 있지만, spread 연산자는 그렇지 않다.

- Chrome: 버전 60부터 지원
- Firefox: 버전 55부터 지원
- Safari: 버전 11.1부터 지원
- Node.js: 버전 8.6부터 지원
- Babel: 지원

[Axel Rauschmayer의 글](http://2ality.com/2016/10/rest-spread-properties.html#spread-defines-properties-objectassign-sets-them)에서 이러한 주의점에 대해 더 자세히 설명하고 있다.

## 알림

이 글은 [v8.dev](https://v8.dev/)에 2017년 6월 6일 발행된 [Object rest and spread properties](https://v8.dev/features/object-rest-spread) 글을 한국어로 편역한 내용을 담고 있습니다.

