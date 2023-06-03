---
title: "객체 생성"
date: 2023-06-03T10:38:24Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트에서 객체는 다른 언어(C++, Java 등)에 비해 단순합니다. 자바스크립트의 객체는 이름(또는 '키')과 값으로 구성된 속성의 집합으로 볼 수 있습니다. 자바스크립트는 객체를 생성하는 다양한 방법을 제공합니다. 


## 1. 객체 리터럴을 이용한 객체 생성

자바스크립트에서 객체를 생성하는 가장 간단한 방법은 객체 리터럴을 사용하는 것입니다. 이는 중괄호 `{}`를 사용하여 객체를 직접 정의하는 방식입니다.

```javascript
let obj = { name: "홍길동", age: 25 };
console.log(obj.name); // "홍길동"
console.log(obj.age); // 25
```

## 2. new Object()를 이용한 객체 생성

또 다른 방법은 `new Object()` 생성자를 사용하는 것입니다. 이 방식은 객체 리터럴 방식보다 명시적이지만, 코드가 약간 더 길어질 수 있습니다.

```javascript
let obj = new Object();
obj.name = "홍길동";
obj.age = 25;
console.log(obj.name); // "홍길동"
console.log(obj.age); // 25
```

## 3. Object.create()를 이용한 객체 생성

`Object.create()` 메소드를 사용하면 기존 객체를 상속받는 새 객체를 생성할 수 있습니다. 자바스크립트는 객체 상속을 프로토타입을 사용하여 구현합니다.

```javascript
let person = { type: "인간" };
let me = Object.create(person);
console.log(me.type); // "인간"
```


## 4. 생성자 함수를 이용한 객체 생성

생성자 함수를 이용하면 객체를 생성하고 초기화할 수 있습니다. 생성자 함수는 일반적으로 첫 글자를 대문자로 작성합니다.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

let me = new Person("홍길동", 25);
console.log(me.name); // "홍길동"
console.log(me.age); // 25
```


## 5. 클래스를 이용한 객체 생성

ES6부터는 클래스 문법을 이용해 객체를 생성할 수 있습니다. 클래스는 생성자 함수 방식보다 간결하고 명확한 문법을 제공합니다. 타 언어의 문법과 비슷하기 때문에 쉽게 익히고 사용할 수 있습니다.

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

let me = new Person("홍길동", 25);
console.log(me.name); // "홍길동"
console.log(me.age); // 25
```

## 6. 왜 이렇게 많을까?

자바스크립트에서 객체를 생성하고 초기화하는 방법은 다양합니다. 그것은 편리성, 프로토타입 그리고 자바스크립트의 발전 역사와 관련 있습니다. 자바스크립트는 인터넷의 역사와 함께 꽤 오랜 역사를 가진 언어입니다. 따라서 자바스크립트로 작성된 오픈소스 코드에는 다양한 객체 생성 방법을 사용하고 있기 때문에 해당 코드를 이해하고 사용하기 위해서는 자바스크립트의 다양한 객체 생성 방법을 잘 알고 있어야 합니다.

