---
title: "객체 속성과 in 연산자"
date: 2023-06-03T12:44:05Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. in 연산자란?

자바스크립트의 `in` 연산자는 특정 객체가 주어진 속성을 가지고 있는지 확인하는 데 사용됩니다. `in` 연산자는 아래와 같이 사용합니다.

```javascript
let car = {make: 'Hyundai', model: 'Elantra', year: 2020};
console.log('make' in car); // true
console.log('color' in car); // false
```

위의 예시에서 `make`는 car 객체의 속성이므로 `'make' in car`는 `true`을 반환하고, `color`는 car 객체의 속성이 아니므로 `'color' in car`는 `false`을 반환합니다.

## 2. undefined 값과 in 연산자

객체의 속성이 `undefined` 값을 가지고 있는 경우에도 `in` 연산자는 해당 속성이 존재한다고 판단합니다.

```javascript
let car = {make: 'Hyundai', model: 'Elantra', year: 2020, color: undefined};
console.log('color' in car); // true
```


## 3 in 연산자와 프로토타입 체인

`in` 연산자는 객체의 프로토타입 체인을 따라 속성을 찾습니다. 즉, 객체가 상속받은 속성에 대해서도 참을 반환합니다.

```javascript
let human = {name: 'John', age: 30, city: 'Seoul'};
let person = Object.create(human);

console.log('name' in person); // true
console.log('toString' in person); // true
```

객체 리터럴로 생성한 객체는 Object 로 부터 상속을 받습니다. 그래서 위 코드에서 `'toString' in person` 가 true를 반환하는 것입니다.

## 4. hasOwnProperty() 메소드와 in 연산자

만약 프로토타입 체인을 따라가지 않고 객체 자체의 속성만을 확인하려면 `hasOwnProperty()` 메소드를 사용할 수 있습니다.

```javascript
let person = {name: 'John', age: 30, city: 'Seoul'};

console.log(person.hasOwnProperty('name')); // true
console.log(person.hasOwnProperty('toString')); // false
```



