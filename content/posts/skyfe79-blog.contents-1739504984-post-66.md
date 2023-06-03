---
title: "객체 속성 열거하기"
date: 2023-06-03T12:35:50Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---


자바스크립트 객체의 속성 중 열거 가능한 속성은 다양한 방법으로 순회가 가능합니다.

## 1. 'for...in' 문

`for...in` 문을 사용해 객체의 모든 열거 가능한 속성을 순회할 수 있습니다.

```javascript
let person = {name: 'John', age: 30, city: 'Seoul'};
for (let property in person) {
  console.log(`${property}: ${person[property]}`);
}

// 실행 결과
name: John
age: 30
city: Seoul
```

다음 코드는 프로토타입을 상속하여 객체를 만들어 속성을 순회합니다.  `for...in` 문는 객체가 상속받은 프로토타입 체인의 속성까지 포함하여 순회합니다.

```js
let human = {name: 'John', age: 30, city: 'Seoul'};
let person = Object.create(human);
for (let property in person) {
  console.log(`${property}: ${person[property]}`);
}

// 실행 결과
name: John
age: 30
city: Seoul
```

## 2 'Object.keys()' 메소드

'Object.keys()' 메소드는 주어진 객체의 속성 이름들을 배열로 반환합니다. 

```javascript
let person = {name: 'John', age: 30, city: 'Seoul'};
let keys = Object.keys(person);
console.log(keys); // ['name', 'age', 'city']
```

다음 코드를 보면 `for...in` 문과 다르게  'Object.keys()' 메소드는 입 체인에서 상속받은 속성은 무시하고 객체 자체의 속성만을 반환하는 것을 알 수 있습니다.

```js
let human = {name: 'John', age: 30, city: 'Seoul'};
let person = Object.create(human);
let keys = Object.keys(person);
console.log(keys); // []
```

## 3 'Object.values()' 메소드

'Object.values()' 메소드는 주어진 객체의 속성 값들을 배열로 반환합니다. 'Object.keys()'와 마찬가지로, 객체 자체의 속성 값만을 반환합니다.

```javascript
let person = {name: 'John', age: 30, city: 'Seoul'};
let values = Object.values(person);
console.log(values); // ['John', 30, 'Seoul']
```

## 4 'Object.entries()' 메소드

'Object.entries()' 메소드는 주어진 객체의 `[키, 값]` 쌍을 배열로 반환합니다. 이는 객체의 속성을 순회하며 키와 값 정보를 동시에 얻을 때 유용합니다.

```javascript
let person = {name: 'John', age: 30, city: 'Seoul'};
let entries = Object.entries(person);
console.log(entries); // [['name', 'John'], ['age', 30], ['city', 'Seoul']]
```

