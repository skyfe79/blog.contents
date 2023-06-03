---
title: "객세 속성 삭제하기"
date: 2023-06-03T12:24:02Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트 객체는 속성 삭제가 가능합니다. `delete` 연산자를 사용하여 속성을 삭제할 수 있습니다. 

## 1. 속성 삭제하기

`delete` 연산자를 사용하여 다음과 같이 속성을 삭제할 수 있습니다.

```javascript
let obj = { name: "홍길동", age: 25 };
delete obj.age;
console.log(obj.age); // undefined
console.log('age' in obj); // false
```

`delete` 연산자는 속성을 성공적으로 삭제하면 `true`를 반환합니다. 주의할 점은 삭제하려는 속성이 존재하지 않아도 true를 반환한다는 점입니다.

```javascript
let obj = { name: "홍길동", age: 25 };
console.log(delete obj.age); // true
console.log(delete obj.height); // true
console.log(delete obj.address); // true -> 주의
```

`delete` 연산자는 속성을 삭제할 수 없는 경우에만 false를 반환합니다. 나중에 살펴볼 defineProperty로 삭제할 수 없는 속성을 만들 수 있습니다.

```js
let obj = {}
Object.defineProperty(obj, 'name', {
    value: 'Burt',
    configurable: false // 삭제할 수 없는 속성입니다.
});

console.log(obj); // { name: 'Burt' }
console.log(delete obj.name); // false
console.log('name' in obj); // true
console.log(obj.name); // Burt
```

실행 결과를 보면 삭제할 수 없는 속성을 `delete` 연산자로 삭제할 때, 실패하여 `false`를 반환하는 것을 확인할 수 있습니다.

## 2. `delete` 연산자와 `undefined`

`delete` 연산자를 사용하여 속성을 삭제하면, 그 속성은 더 이상 객체에 존재하지 않습니다. 이는 속성 값을 `undefined`로 설정하는 것과는 다릅니다.

```javascript
let obj = { name: "홍길동", age: 25 };
obj.age = undefined;
console.log('age' in obj); // true

delete obj.age;
console.log('age' in obj); // false
```

또한 `delete` 연산자는 객체의 속성만 삭제할 수 있습니다. 일반 변수에 `delete` 연산자를 사용하면 아무런 효과가 없습니다.

```javascript
let name = "홍길동";
delete name;
console.log(name); // "홍길동"
```

## 3. `delete` 연산자와 배열

`delete` 연산자는 배열의 요소를 삭제할 때도 사용할 수 있습니다. 

```javascript
let arr = ["apple", "banana", "cherry"];
delete arr[1];
console.log(arr[1]); // undefined
```

이 코드는 배열 `arr`의 두 번째 요소를 삭제합니다. `delete` 연산자를 사용하여 배열 요소를 삭제하면 해당 요소는 `undefined`가 됩니다. 하지만 배열의 `length` 속성에는 영향을 주지 않습니다. 이점을 주의해야 합니다.

```javascript
let arr = ["apple", "banana", "cherry"];
delete arr[1];
console.log(arr.length); // 3
```

`delete` 연산자를 사용하면 해당 인덱스의 요소만 삭제되고, 배열의 길이는 변하지 않습니다.  배열에 `undefined` 값이 남게 됩니다.

```javascript
let arr = ["apple", "banana", "cherry"];
delete arr[1];
console.log(arr); // ["apple", undefined, "cherry"]
```

그렇기 때문에 배열에서 요소를 삭제할 때 `delete` 연산자보다 `Array.prototype.splice()` 메서드를 사용합니다. `splice()` 메서드는 배열에서 요소를 삭제하고, 삭제된 요소를 반환합니다.

```javascript
let arr = ["apple", "banana", "cherry"];
arr.splice(1, 1);
console.log(arr[1]); // "cherry"
console.log(arr.length); // 2
```

`splice()` 메서드는 배열에서 요소를 삭제하면 배열의 `length` 속성도 알맞게 조정됩니다. 

배열의 요소를 삭제할 때는 `delete` 연산자가 아닌 `Array.prototype.splice()` 메서드를 사용해야 함을 꼭 기억하세요!



