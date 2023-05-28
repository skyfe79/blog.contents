---
title: "undefined와 null"
date: 2023-05-28T10:27:50Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트를 사용하면서 제일 처음 경험한 어려움은 undefinded 와 null 구분이었습니다. 둘 다 비슷한 것 같은데 언제 어떻게 구분해서 사용하는지 판단하기 어려웠습니다.

## 1. undefined

undefined는 변수를 선언하지 않거나 값을 할당하지 않았을 때 **자동으로 설정**되는 값입니다.

```javascript
let a;
console.log(a); // undefined
```

위 코드에서 변수 a를 선언만 하고 값을 할당하지 않았기 때문에, 결과로 undefined가 출력됩니다. 객체의 속성값을 찾으려고 할 때 해당 속성값이 존재하지 않으면 undefined가 반환됩니다.

```javascript
let obj = {name: 'John'};
console.log(obj.age); // undefined
```

함수가 값을 반환하지 않을 때도 undefined가 반환됩니다.

```js
function odd(x) {
	if (x % 2 === 1) {
		return x;
	}
}
console.log(odd(2)); // undefined
```

## 2. null

null은 **의도적으로** 값이 없음을 나타내는 데이터 타입입니다. null은 어떤 변수나 속성값에 대해서도 할당 가능하며, 이를 통해 해당 변수나 속성값에 아직 값을 할당할 수 없음을 명시적으로 나타낼 수 있습니다.

```javascript
let b = null;
console.log(b); // null
```

위 코드에서 b 변수에 null 값을 할당했기 때문에, 결과로 null이 출력됩니다.

## 3. 언제 undefined를 쓰고 null을 써야 할까?

개인적으로 undefined와 null을 아래 기준으로 나누어 사용하고 있습니다.

undefined는 값이 할당되지 않은 변수나 객체의 속성값 등의 상황에서 자바스크립트 엔진이 자동으로 설정하는 값인 반면에, null은 의도적으로 값이 없음을 나타내기 위해 개발자가 명시적으로 할당하는 값입니다.

따라서 개발자가 명시적으로 undefined를 할당하는 것은 되도록이면 피해야 한합니다. 값이 없음을 명확하게 표현하기 위해서는 null 을 사용해야 합니다. 그런데 이 규칙은 강제성이 없기 때문에 지키기 어렵습니다. 애초에 이렇게 구분해서 써야 하는 것 자체가 자바스크립트 언어의 문제가 아닐까...생각합니다.

undefined는 typeof 연산자를 이용해서 확인하면 `undefined`가 출력되지만, null은 이상하게도 `object`가 출력됩니다.

```javascript
let c;
console.log(typeof c); // undefined

let d = null;
console.log(typeof d); // object
```

`typeof null === 'null'` 이 출력되는 것은 자바스크립트의 버그로 자바스크립트 최초 구현이 담고 있던 문제였지만 이를 수정하게 되면 그로 인한 파장을 감당하기 어려워 아직까지 그대로 존재하고 있다고 합니다.🥶

> In the first implementation of JavaScript, JavaScript values were represented as a type tag and a value. The type tag for objects was `0`. `null` was represented as the NULL pointer (`0x00` in most platforms). Consequently, `null` had `0` as type tag, hence the `typeof` return value `"object"`. ([reference](https://2ality.com/2013/10/typeof-null.html))
> 
> A fix was proposed for ECMAScript (via an opt-in), but [was rejected](https://web.archive.org/web/20160331031419/http://wiki.ecmascript.org:80/doku.php?id=harmony:typeof_null). It would have resulted in `typeof null === "null"`.
> 
> [출처] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#typeof_null

## 4. undefined 사용하기

### 4.1 변수 자동 초기화

변수를 선언하고 값을 할당하지 않으면 해당 변수는 undefined 값을 가집니다. 

```javascript
let x;
console.log(x); // undefined
```

하지만 이렇게 사용하는 것은 매우 좋지 않은 습관입니다. 반드시 값을 초기화한 후 사용하세요.

```js
let x = 0;
console.log(x)
```

만약 값이 없음을 나타내려면 `null` 을 사용해 명시적으로 초기화합니다.

```js
let x = null;
console.log(x);
```

### 4.2. 객체 속성 존재 여부 확인

객체의 속성을 참조할 때 해당 속성이 존재하지 않으면 undefined 값을 반환합니다. 이때, 객체에 해당 속성이 존재하는지 확인하기 위해 undefined를 사용할 수 있습니다.

```javascript
const obj = { a: 1, b: 2 };
console.log(obj.c); // undefined

if (obj.c === undefined) {
  console.log('obj has no property c');
}
```

### 4.3. 함수 매개변수

함수의 매개변수는 값을 전달하지 않아도 자동으로 undefined로 초기화됩니다. 이때, 함수 내부에서 매개변수가 정상적으로 전달되었는지 확인하기 위해 undefined를 사용할 수 있습니다.

```javascript
function sum(a, b) {
  if (a === undefined || b === undefined) {
    return 'Please provide both parameters';
  }
  return a + b;
}

console.log(sum(1)); // Please provide both parameters
```

### 4.4. 배열 요소 존재 여부 확인

배열의 요소를 참조할 때 해당 요소가 존재하지 않으면 undefined 값을 반환합니다. 이때, 배열에 해당 요소가 존재하는지 확인하기 위해 undefined를 사용할 수 있습니다.

```javascript
const arr = [1, 2, 3];
console.log(arr[3]); // undefined

if (arr[3] === undefined) {
  console.log('arr has no element at index 3');
}
```

### 4.5. 반환 값이 없는 함수

함수에서 return 문이 없으면 해당 함수는 undefined 값을 반환합니다. 이때, 반환 값을 확인하고 처리하기 위해 undefined를 사용할 수 있습니다.

```javascript
function greet(name) {
  if (name) {
    return `Hello, ${name}!`;
  }
}

console.log(greet()); // undefined
```

## 5. null 사용하기

null은 JavaScript에서 "존재하지 않음"을 나타내는 값입니다. 따라서 변수가 아직 값이 할당되지 않은 경우나, 존재하지 않는 객체를 참조할 때 null 값을 명시적으로 사용합니다.

### 5.1. 변수 초기화

변수를 선언할 때, 값을 할당하지 않으면 자동으로 undefined가 할당됩니다. 하지만 값이 없음을 나타내는 null 을 명시적으로 할당하는 것이 더 좋습니다.

```javascript
let userInput = null;
```


###  5.2. 객체 참조

객체를 참조할 때, 해당 객체가 존재하지 않을 수 있습니다. 이때 null 값을 사용합니다.

```javascript
let person = {
  name: 'John',
  age: 30
};

if (!person.address) {
  person.address = null;
}
```

위의 코드에서는 person 객체에 address 속성이 없으면, null 값을 할당합니다.

### 5.3. 함수 반환값

일부 함수에서는 명시적으로 반환값이 없을 수 있습니다. 이때도 null 값을 반환합니다.

```javascript
function findUser(id) {
  let user = getUserById(id);
  if (!user) {
    return null;
  }
  return user;
}
```

위의 코드에서는 findUser 함수에서 유저를 찾지 못한 경우, null 값을 반환합니다.

## 6. isNullish

undefined와 null을 비교할 때는 주의해야 합니다. 다음 코드를 봅시다.

```js
console.log(undefined == null); // true
console.log(undefined === null); // false
```

둘 다 false가 맞는 것 같지만 결과는 그렇지 않습니다. 따라서 `값이 없음`을 명확하게 검사하는 타입 검사 도구를 정의해 사용하는 것이 안전합니다.

```js
function isNullish(value) {
	if (value === undefined || value === null) {
		return true;
	}
	return false;
}
const someValue = 'a';

...

if (!isNullish(someValue)) {
	// do something
}
```






