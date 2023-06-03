---
title: "객체 생성과 Object.create()"
date: 2023-06-03T13:39:13Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

[객체 생성](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-1739394264-post-62/)에서 `Object.create()` 를 사용해 보았습니다. 이번 장에서는 `Object.create()`의 두 번째 파라미터에 대해서 자세하게 알아 보려고 합니다.

JavaScript는 프로토타입 기반의 언어입니다. 그래서 객체를 만들 때, 프로토타입으로부터 속성과 메서드를 상속받을 수 있습니다. 프로토타입 상속을 가능하게 하는 메서드 중 하나가 `Object.create()`입니다.

`Object.create()`는 첫 번째 파라미터로 프로토타입을 받고, 두 번째 파라미터로 속성 객체를 받아 새로운 객체를 생성합니다. 참고로 두 번째 파라미터는 선택적인 파라미터입니다.

```js
const obj1 = {a: 1};
const obj2 = Object.create(obj1, {
	b: {
		value: 2, 
		writable: true, 
		enumerable: true, 
		configurable: true
	}
});
console.log(obj2.a); // 1
console.log(obj2.b); // 2
```

## 1. 속성 객체

`Object.create()` 의 두 번째 파라미터는 속성 객체입니다. 속성 객체는 속성 이름을 키로 가지고, 그 값으로 속성 설명자를 갖는 객체입니다. 속성 설명자는 `value`, `writable`, `enumerable`, `configurable` 네 가지 속성을 가질 수 있습니다.

```javascript
{
  prop: {
    value: ... ,
    writable: ... ,
    enumerable: ... ,
    configurable: ...
  },
  ...
}
```

## 2. value 속성

`value` 속성은 속성의 값입니다. 어떤 자바스크립트 값이든 `value` 속성의 값이 될 수 있습니다.

```javascript
var obj = Object.create({}, {
  myProp: {
    value: 'Hello, World!'
  }
});
console.log(obj.myProp); // 'Hello, World!'
```

## 3. writable 속성

`writable` 속성은 속성의 값이 변경 가능한지를 결정합니다. 이 값이 `false`이면 속성의 값을 변경할 수 없습니다.

```javascript
var obj = Object.create({}, {
  myProp: {
    value: 'Hello, World!',
    writable: false
  }
});
obj.myProp = 'Goodbye, World!'; // 아무런 변화가 없다.
console.log(obj.myProp); // 'Hello, World!'
```

`obj.myProp = 'Goodbye, World!';` 코드를 실행하면 아무런 변화가 없거나 
`Uncaught TypeError: Cannot assign to read only property 'myProp' of object '#<Object>'` 오류가 발생합니다.

## 4. enumerable 속성

`enumerable` 속성은 해당 속성이 열거 가능한지를 결정합니다. 이 값이 `true`일 때만 해당 속성을 `for...in` 루프나 `Object.keys()` 등으로 열거할 수있습니다.

```javascript
var obj = Object.create({}, {
  myProp: {
    value: 'Hello, World!',
    enumerable: false
 }
});
console.log(Object.keys(obj)); // []
```

## 5. configurable 속성

`configurable` 속성은 해당 속성의 설명자가 변경 가능한지, 그리고 속성 자체가 삭제 가능한지를 결정합니다. 이 값이 `false`이면 해당 속성의 설명자를 변경하거나 속성 자체를 삭제할 수 없습니다.

```javascript
var obj = Object.create({}, {
  myProp: {
    value: 'Hello, World!',
    configurable: false
  }
});
delete obj.myProp; // 아무런 변화가 없다.
console.log(obj.myProp); // 'Hello, World!'
```

`delete obj.myProp;` 코드를 실행하면 아무런 변화가 없거나 `Uncaught TypeError: Cannot delete property 'myProp' of #<Object>` 오류가 발생합니다.

## 6. Object.create()와 객체 속성 설명자

Object.create()의 두 번째 파라미터인 객체 속성 설명자를 사용하면 새로운 객체를 생성하는 동시에 그 객체의 속성들을 상세하게 설정할 수 있습니다. 따라서 객체 생성과 속성 설정을 한번에 할 수 있어 코드를 간결하게 작성할 수 있습니다. 

```javascript
var obj = Object.create({}, {
  myProp: {
    value: 'Hello, World!',
    writable: true,
    enumerable: true,
    configurable: true
  }
});
console.log(obj.myProp); // 'Hello, World!'
```

`Object.create()`의 두 번째 파라미터인 객체 속성 설명자는 객체의 속성을 제어하는 데 매우 유용한 도구입니다. 따라서 자바스크립트에서 객체를 생성하고 다루는 방법을 이해하기 위해 객체 속성 설명자를 꼭 알아야 합니다.


