---
title: "Object.getOwnPropertyDescriptor()"
date: 2023-06-04T04:49:21Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. Object.getOwnPropertyDescriptor() 소개

`Object.getOwnPropertyDescriptor()`는 특정 객체의 특정 속성에 대한 속성 설명자를 반환하는 메소드입니다. 이 메소드는 첫 번째 인자로 대상 객체를, 두 번째 인자로 속성 이름을 받습니다.

```javascript
let obj = { property1: 'Hello, world!' };
let descriptor = Object.getOwnPropertyDescriptor(obj, 'property1');
console.log(descriptor.value);  // 'Hello, world!'
```

`Object.getOwnPropertyDescriptor()`를 사용하면 객체의 속성에 대한 세부 정보를 얻을 수 있습니다. 반환된 속성 설명자는 `value`, `writable`, `enumerable`, `configurable` 필드를 포함할 수 있으며, 속성이 접근자 속성인 경우 `get`과 `set` 필드를 포함할 수 있습니다.

```javascript
let obj = { property1: 'Hello, world!' };
let descriptor = Object.getOwnPropertyDescriptor(obj, 'property1');
console.log(descriptor.value);  // 'Hello, world!'
console.log(descriptor.writable);  // true
console.log(descriptor.enumerable);  // true
console.log(descriptor.configurable);  // true
```

## 2. 접근자 속성에 대한 설명자 가져오기

`Object.getOwnPropertyDescriptor()`를 사용하면 접근자 속성에 대한 속성 설명자도 얻을 수 있습니다.

```javascript
let obj = {
  get property1() {
    return 'Hello, world!';
  },
  set property1(value) {
    console.log('Setting property1 to ' + value);
  }
};
let descriptor = Object.getOwnPropertyDescriptor(obj, 'property1');
console.log(typeof descriptor.get);  // 'function'
console.log(typeof descriptor.set);  // 'function'
```

`Object.getOwnPropertyDescriptor()`는 속성이 존재하지 않는 경우 `undefined`를 반환합니다.

```javascript
let obj = { property1: 'Hello, world!' };
let descriptor = Object.getOwnPropertyDescriptor(obj, 'property2');
console.log(descriptor);  // undefined
```

## 3. 프로토타입 체인과 Object.getOwnPropertyDescriptor()

프로토타입 체인을 통해 상속받은 속성에 대한 설명자는 `Object.getOwnPropertyDescriptor()`로 얻을 수 없습니다.

```javascript
let obj1 = { property1: 'Hello, world!' };
let obj2 = Object.create(obj1);
let descriptor = Object.getOwnPropertyDescriptor(obj2, 'property1');
console.log(descriptor);  // undefined
```

또한 심볼을 키로 사용하는 속성에 대한 설명자를 가져오려면 `Object.getOwnPropertySymbols()`와 같은 다른 메소드를 사용해야 합니다. `Object.getOwnPropertyDescriptor()`는 문자열 키에 대한 속성 설명자만 가져옵니다.

## 4. ES6와  Object.getOwnPropertyDescriptor()

ES6의 클래스에 정의된 메소드에 대한 속성 설명자를 얻기 위해 `Object.getOwnPropertyDescriptor()`를 사용할 수 있습니다.

```javascript
class MyClass {
  myMethod() {}
}

let descriptor = Object.getOwnPropertyDescriptor(MyClass.prototype, 'myMethod');

console.log(descriptor.writable); // false
console.log(descriptor.enumerable); // false
console.log(descriptor.configurable); // true
```




