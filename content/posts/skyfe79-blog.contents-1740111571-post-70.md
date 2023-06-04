---
title: "객체 메서드와 this"
date: 2023-06-04T04:02:01Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트에서 `this`는 매우 중요한 키워드입니다. `this`는 현재 컨텍스트를 참조하는데 사용되며, 그 컨텍스트는 실행 환경에 따라 달라집니다. 이번 장에서는 객체 메서드 내부에서 `this` 키워드를 어떻게 사용하는지에 대해 자세히 알아보겠습니다.

## 1. this 이해하기

`this`는 일반적으로 객체 메서드 내부에서 사용됩니다. 객체 메서드 내부에서 `this`는 해당 메서드를 호출한 객체를 참조합니다.

```javascript
let person = {
  name: "Kim",
  sayHello: function() {
    console.log("Hello, " + this.name);
  }
};

person.sayHello(); // "Hello, Kim"
```

이 경우 `this.name`은 `person.name`을 참조합니다. 

## 2. this와 일반 함수

`this`는 일반 함수 내에서도 사용할 수 있습니다. 하지만 이 경우 `this`의 값은 전역 객체를 참조합니다. 웹 브라우저에서 전역 객체는 `window` 객체이고 NodeJS에서 전역 객체는 `global` 객체입니다. 

```javascript
function sayHello() {
  console.log("Hello, " + this.myName);
}
sayHello(); // Hello, undefined
```

위 코드를 브라우저에서 실행할 경우, `sayHello()` 의 `this`는 `window 객체`입니다. `window 객체`에는 `myName` 속성이 없기 때문에 `Hello, undefined` 결과가 출력됩니다.

```js
function sayHello() {
  console.log("Hello, " + this.myName);
}

let person = {
  myName: "Kim"
};

sayHello.call(person); // "Hello, Kim"
```

sayHello() 함수에서 사용하는 `myName` 값을 제공하기 위해서는 `myName` 값을 가지고 있는 객체를 `this` 로 바인딩해서 호출해야 합니다. 위 코드에서는 person 객체를 this 로 바인딩해서 호출하고 있습니다. 이처럼 자바스크립트에서는 call, apply, bind와 같은 메서드를 사용하여 `this`를 명시적으로 설정할 수 있습니다. 이것을 명시적인 `this` 바인딩이라고 합니다.

```javascript
function sayHello() {
  console.log("Hello, " + this.name);
}

let person1 = {
  name: "Kim"
};

let person2 = {
  name: "Lee"
};

sayHello.call(person1); // "Hello, Kim"
sayHello.call(person2); // "Hello, Lee"
```

위 코드는 명시적 `this` 바인딩을 사용해  `this`가 참조하는 객체를 제어하면서 `sayHello` 함수를 호출했습니다.

## 3. this와 생성자 함수

`this`는 생성자 함수 내에서도 사용됩니다. 생성자 함수 내에서 `this`는 새로 생성된 객체를 참조합니다.

```javascript
function Person(name) {
  this.name = name;
  this.sayHello = function() {
    console.log("Hello, " + this.name);
  };
}

let person = new Person("Kim");
person.sayHello(); // "Hello, Kim"
```


## 4. this와 화살표 함수

`this`는 화살표 함수에서 동작하는 방식이 다릅니다. 화살표 함수 내부에서 `this`는 화살표 함수를 감싸고 있는 외부 함수나 객체의 `this` 를 사용합니다. 만약 외부 함수나 객체가 없다면 `this`는 전역 객체를 참조합니다.

```javascript
let person = {
  name: "Kim",
  sayHello: () => {
    console.log("Hello, " + this.name);
  }
};

person.sayHello(); // "Hello, undefined"
```


## 5. this 이해의 중요성

자바스크립트에서 `this` 바인딩은 많은 혼동을 일으키는 요소 중 하나입니다. 특히 `this`의 값이 실행 컨텍스트에 따라 달라지기 때문에, 예상치 못한 결과를 초래할 수 있습니다. 따라서 `this`를 사용할 때는 항상 그 컨텍스트를 명확히 이해하는 것이 중요합니다.

- 함수 리터럴로 정의한 함수의 `this`는 함수를 호출할 때 결정됩니다. 
- 그러나 화살표 함수의 `this` 값은 함수를 정의할 때 결정됩니다. 이것을 `Lexical this` 라고 합니다.
	- 즉, 화살표 함수의 바깥 `this` 값이 화살표 함수의 `this` 값이 됩니다. 

