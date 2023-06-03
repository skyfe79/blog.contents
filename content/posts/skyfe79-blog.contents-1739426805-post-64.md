---
title: "객체 속성 추가와 수정"
date: 2023-06-03T11:11:58Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트의 객체는 특별한 설정(freeze 또는 seal)을 하지 않는 한 항상 수정 가능합니다. 그래서 실시간으로 객체에 속성을 추가하거나 수정할 수 있습니다.

## 1. 속성 추가하기

객체에 속성을 추가하는 것은 간단합니다. 이미 선언된 객체에 새로운 속성을 추가할 때는 점 표기법이나 대괄호 표기법을 사용하여 추가할 수 있습니다.

```javascript
let obj = { name: "홍길동", age: 25 };
obj.job = "개발자";
console.log(obj.job); // "개발자"
obj["language"] = "자바스크립트";
console.log(obj.language); // "자바스크립트"
```

## 2. 속성값 수정하기

객체의 속성값을 수정하는 것도 매우 간단합니다. 속성에 새로운 값을 할당하면 됩니다.

```javascript
let obj = { name: "홍길동", age: 25 };
obj.name = "이순신";
console.log(obj.name); // "이순신"
```

## 3. 동적 이름을 가진 속성 추가하기

대괄호 표기법을 사용하면, 변수를 사용하여 동적으로 속성 이름을 정의할 수 있습니다.

```javascript
let obj = { name: "홍길동", age: 25 };
let key = "job";
obj[key] = "개발자";
console.log(obj.job); // "개발자"
```

이 특징을 활용하여 객체에 동적 속성을 쉽게 추가할 수 있는 함수를 만들 수 있습니다.

```js
function addDynamicProperty(obj, propertyName, propertyValue) {
  // 동적 이름을 가진 속성을 추가합니다.
  obj[propertyName] = propertyValue;
}

const myObj = {};
const dynamicPropertyName = "dynamicProperty";
const dynamicPropertyValue = "Hello, World!";

// 함수를 호출해 myObj에 동적 속성을 추가합니다.
addDynamicProperty(myObj, dynamicPropertyName, dynamicPropertyValue);

// 아래와 같이 동적 속성에 접근할 수 있습니다.
console.log(myObj.dynamicProperty); // Hello, World!
console.log(myObj["dynamicProperty"]); // Hello, World!
```

## 4. 메서드 추가하기

객체의 속성 값으로 함수를 정의하면, 그 함수는 객체의 메서드가 됩니다. 메서드는 객체의 동작을 나타냅니다.

```javascript
let obj = {
  name: "홍길동",
  age: 25,
  sayHello: function() {
    console.log("안녕하세요, " + this.name + "입니다.");
  }
};
obj.sayHello(); // "안녕하세요, 홍길동입니다."
```

객체 메서드는 속성의 값이 함수인 것만 달라서 이전 예제처럼 동적으로 메서드를 객체에 추가할 수 있습니다.

```js
obj.sayGoodBye = function() {
	console.log("안녕히 계세요, " + this.name + "은 이만 가 보겠습니다.");
}
obj.sayGoodBye(); // "안녕히 계세요, 홍길동은 이만 가 보겠습니다."
```

## 5. 속성 삭제하기

`delete` 연산자를 사용하면 객체의 속성을 삭제할 수 있습니다.

```javascript
let obj = {
  name: "홍길동",
  age: 25,
  sayHello: function() {
    console.log("안녕하세요, " + this.name + "입니다.");
  }
};

console.log('age' in obj); // true
delete obj.age;
console.log('age' in obj); // false

console.log('sayHello' in obj); // true
delete obj.sayHello;
console.log('sayHello' in obj); // false
```



