---
title: "객체 메서드"
date: 2023-06-04T02:37:56Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. 객체 메서드란?

자바스크립트에서 객체는 속성과 메서드를 가질 수 있습니다. 속성은 객체의 상태를 나타내는 반면, 메서드는 객체의 행동을 정의한다고 생각할 수 있습니다. 객체 메서드는 해당 객체에 연결된 함수객체의 속성을 조작하거나 계산하는 역할을 합니다.

예를 들어, `person` 객체가 있다고 가정해봅시다.

```javascript
let person = {
  firstName: "Kim",
  lastName: "Minjun",
  fullName: function() {
    return this.firstName + " " + this.lastName;
  }
};
```

여기에서 `fullName`은 `person` 객체의 메서드입니다. 이 메서드는 `person`의 `firstName`과 `lastName` 속성을 결합하여 전체 이름을 반환합니다.


## 2.  객체 메서드 정의하기

객체 메서드를 정의하는 방법은 간단합니다. 함수를 속성의 값으로 할당하면 해당 함수가 객체의 메서드로 정의 됩니다.

```javascript
let person = {
  firstName: "Kim",
  lastName: "Minjun",
  fullName: function() {
    return this.firstName + " " + this.lastName;
  }
};
```

위 예시에서 `fullName` 함수가 `person` 객체의 속성 값으로 할당되어 있습니다. 이렇게 하면 `person` 객체 내부에서 `fullName` 메서드를 호출할 수 있습니다.

## 3. ES6에 도입된 좀 더 간결한 메서드 정의

ES6에서는 메서드를 보다 간결하게 정의할 수 있는 문법이 도입되었습니다.

```javascript
let person = {
  firstName: "Kim",
  lastName: "Minjun",
  fullName() {
    return this.firstName + " " + this.lastName;
  }
};
```

위 코드처럼 `fullName` 메서드를 `function` 키워드 없이 정의할 수 있습니다.

## 4. this 키워드 이해하기

객체 메서드 내에서 `this` 키워드를 사용하면, 해당 메서드가 속한 객체를 참조할 수 있습니다. 

```javascript
const person = {
  name: "John",
  age: 20,
  sayHello() {
    console.log(`Hello, my name is ${this.name}.`);
  },
  introduce() {
    console.log(`My name is ${this.name} and I am ${this.age} years old.`);
  }
};

person.introduce(); // My name is John and I am 20 years old.
```

`this.firstName`과 `this.lastName`는 각각 `person` 객체의 `firstName`과 `lastName` 프로퍼티를 참조합니다. 이처럼  메서드에서 `this` 키워드를 통해 객체의 프로퍼티에 접근하거나 수정할 수 있습니다.

## 5. this 를 활용한 메서드 체이닝

메서드 체이닝은 한 객체의 여러 메서드를 연속적으로 호출하는 프로그래밍 패턴입니다. 이 패턴을 사용하면 코드를 더욱 간결하고 읽기 쉽게 만들 수 있습니다. 빌더 패턴 등에서 유용하게 사용됩니다.

```javascript
let person = {
  firstName: "",
  lastName: "",
  setFirstName(firstName) {
    this.firstName = firstName;
    return this;
  },
  setLastName(lastName) {
    this.lastName = lastName;
    return this;
  },
  fullName() {
    return this.firstName + " " + this.lastName;
  }
};

person.setFirstName("Kim").setLastName("Sungcheol").fullName(); // "Kim Sungcheol"
```


## 6. 화살표 함수(arrow function)와 this

화살표 함수는 자신만의 this를 가지지 않습니다. 따라서 화살표 함수로 메서드를 정의하면 해당 메서드 내부에서 `this` 키워드를 사용할 수 없으므로 주의해야 합니다. 다음은 예시입니다.

```javascript
const person = {
  name: "John",
  age: 20,
  sayHello: () => {
    console.log(`Hello, my name is ${this.name}.`);
  }
};

person.sayHello(); // Hello, my name is undefined.
```

위 코드를 실행하면 `Hello, my name is undefined.` 결과가 출력되거나 `Uncaught TypeError: Cannot read properties of undefined (reading 'name')` 오류가 발생할 수 있습니다. 

이렇게 실행되는 이유는 화살표 함수의 this 처리가 기존 함수와 다르기 때문입니다. 다른 장에서 좀 더 자세히 다루겠습니다. 만약 `sayHello()` 메서드가 실행되도록 하려면 생성자 함수를 사용하거나 생성자 함수의 문법 설탕인 `class` 를 사용하면 됩니다.

```js
function Person() {
  this.name = "John";
  this.age = 20;
  this.sayHello = () => {
    console.log(`Hello, my name is ${this.name}.`);
  }
}
const person = new Person();
person.sayHello(); // Hello, my name is John.
```

또는 ES6에 도입된 `class` 로 객체를 정의합니다.

```js
class Person {
  constructor() {
    this.name = "John";
  	this.age = 20;
  }
  sayHello = () => {
    console.log(`Hello, my name is ${this.name}.`);
  }
};
 
const person = new Person();
person.sayHello(); // Hello, my name is John.
```

