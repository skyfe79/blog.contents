---
title: "객체 속성과 속성 설명자"
date: 2023-06-04T04:18:19Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트에서 객체 속성은 단순히 키와 값 두 부분으로 이루어져 있지 않습니다. 각 속성은 다양한 속성 설명자를 지고 있습니다. 이번 장에서는 객체 속성과 속성 설명자에 대해 알아보겠습니다.

## 1. 속성 설명자 이해하기

속성 설명자(Property Descriptor)는 속성의 메타데이터를 담고 있는 객체입니다. 속성 설명자는 속성의 값(`value`), 쓰기 가능 여부(`writable`), 열거 가능 여부(`enumerable`), 설정 가능 여부(`configurable`) 정보를 포함하고 있습니다.

```javascript
let person = {
  name: "Kim"
};

let descriptor = Object.getOwnPropertyDescriptor(person, 'name');

console.log(descriptor);
```

실행 결과, 아래와 같이 속성 설명자가 출력됩니다.

```js
{
  value: "Kim",
  writable: true, 
  enumerable: true, 
  configurable: true
}
```

## 2. value 속성

`value` 는 속성의 값입니다. `Object.getOwnPropertyDescriptor` 메서드를 사용하면 `value` 속성을 확인할 수 있습니다.

```javascript
let person = {
  name: "Kim"
};

let descriptor = Object.getOwnPropertyDescriptor(person, 'name');
console.log(descriptor.value); // "Kim"
```

## 3. writable 속성

`writable` 은 속성의 값(`value` 속성 값)을 변경할 수 있는지를 나타냅니다. 이 속성이 `false`로 설정되면 해당 속성은 읽기 전용이 됩니다.

```javascript
let person = {
  name: "Kim"
};

Object.defineProperty(person, 'name', { writable: false });

person.name = "Lee"; // 오류 발생
```

위 코드를 실행하면 `Uncaught TypeError: Cannot assign to read only property 'name' of object '#<Object>'` 오류가 발생합니다.

## 4. enumerable 속성

`enumerable` 은 속성을 열거할 수 있는지를 나타냅니다. 이 속성이 `false`로 설정되면 해당 속성은 `for-in` 루프나 `Object.keys` 메서드 등에서 열거 되지 않습니다.

```javascript
let person = {
  name: "Kim",
  age: 30
};

Object.defineProperty(person, 'age', { enumerable: false });

console.log(Object.keys(person)); // ["name"]
```

위 코드에서 `age` 속성을 열거 되지 않는 것을 확인할 수 있습니다.

## 5. configurable 속성

`configurable` 은 속성의 설명자를 변경하거나 속성을 삭제할 수 있는지를 나타냅니다. 이 속성이 `false`로 설정되면 해당 속성 설명자를 더 이상 변경할 수 없고, 속성을 삭제할 수도 없습니다.

```javascript
let person = {
  name: "Kim"
};

Object.defineProperty(person, 'name', { configurable: false });

delete person.name; // 오류 발생
```

위 코드를 실행하면 `Uncaught TypeError: Cannot delete property 'name' of #<Object>` 오류가 발생합니다.

## 6. 속성 설명자를 사용해 속성 정의하기

`Object.defineProperty` 메서드를 사용하면 속성 설명자를 정의하여 새로운 속성을 만들거나 기존 속성을 수정할 수 있습니다. 이 메서드를 사용하면 속성의 값(`value`), 쓰기 가능 여부(`writable`), 열거 가능 여부(`enumerable`), 설정 가능 여부(`configurable`)를 한번에 지정할 수 있습니다.

```javascript
let person = {};

Object.defineProperty(person, 'name', {
  value: "Kim",
  writable: true,
  enumerable: true,
  configurable: true
});

console.log(person.name); // "Kim"
```



