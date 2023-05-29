---
title: "객체 생성과 프로토타입"
date: 2023-05-29T04:29:34Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

프로토타입은 객체의 원형입니다. 자바스크립트는 프로토타입을 사용하여 객체를 생성할 수 있습니다. 

## 객체 리터럴

객체 리터럴은 중괄호(`{}`)를 사용하여 객체를 생성하는 방법입니다. 

```js
const obj = {
  name: 'John',
  age: 30,
  greet() {
    console.log(`Hello, my name is ${this.name}.`);
  }
};
```

위 코드에서 `obj`라는 변수에 객체를 할당했습니다. `obj` 객체는 `name`, `age`, `greet`라는 속성을 가지고 있습니다. 객체 리터럴로 생성된 모든 객체는 자신의 부모 역할을 하는 프로토타입(prototype)인 `Object.prototype`을 가지고 있습니다.

따라서, 만약 `obj`객체에서 `toString()` 메소드를 호출하면, 자바스크립트는 `obj`객체 내부에서 해당 메소드를 찾습니다. 만약 찾지 못하면, 자동으로 `Object.prototype`을 검색하여  `Object.prototype.toString()` 메소드를 호출합니다. 이것을 프로토타입 체인(prototype chain)이라고 합니다.

## Object.create()
`Object.create()` 메서드를 사용하면 프로토타입을 설정하여 객체를 생성할 수 있습니다. `Object.create()` 메서드는 새로운 빈 객체를 만들고, 인수로 전달된 객체를 빈 객체의 프로토타입으로 설정합니다.

```js
let proto = { name: "Alice" };
let alice = Object.create(proto);

console.log(alice.name); // "Alice"
```

객체 proto은 alice의 프로토타입이 됩니다.  아래와 같이 확인할 수 있습니다.

```js
console.log(alice.__proto__); // {name: 'Alice'}
console.log(proto.isPrototypeOf(alice)) // true
```

## 생성자 함수와 new 연산자

생성자 함수(를 사용합니다. 생성자 함수는 일반적인 함수와 동일한 방법으로 정의할 수 있지만, 관례적으로 첫 글자를 대문자로 시작하는 이름을 가지고 있습니다. 생성자 함수는 `new` 연산자로 호출할 수 있습니다.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}
const person1 = new Person("John", 30);

console.log(person1.age);
console.log(person1.name);
console.log(person1.__proto__);
```

위 코드를 실행하면 아래와 같이 결과가 출력됩니다.

```js
30
John
{constructor: ƒ}
	constructor: f Person(name, age)
	[[Prototype]]: Object
```

`Person` 함수가 생성자로 정의되어 있음을 확인할 수 있습니다.

### constructor 속성

생성자 함수로 만든 객체는 constructor 속성을 가지고 있습니다. 이 속성은 해당 객체를 만든 생성자 함수를 가리킵니다.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const person1 = new Person("John", 30);

console.log(person1.constructor === Person); // true
```

위 코드를 실행하면 person1 객체의 constructor 속성을 사용해서 person1 객체의 constructor 속성이 Person 생성자 함수를 가리키는지 확인할 수 있습니다. 결과로 true가 반환됩니다. 

### constructor 속성과 프로토타입

생성자 함수로 만든 객체의 프로토타입에는 constructor 속성이 있습니다. constructor 속성은 객체가 속한 생성자 함수를 가리킵니다. 

위에서 만든 person1 객체는 Person 생성자 함수로부터 만들어졌으므로, Person.prototype이 person1 객체의 프로토타입이 됩니다. 그리고 이 프로토타입은 constructor 속성을 가지고 있습니다.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const person1 = new Person("John", 30);

console.log(Person.prototype.constructor === Person); // true
console.log(person1.__proto__.constructor === Person); // true
console.log(person1.constructor === Person); // true
```

### constructor 속성의 활용

객체의 constructor 속성을 사용해 다른 객체를 생성할 수 있습니다.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const person1 = new Person("John", 30);


function Animal(type) {
  // person1 객체의 constructor 속성을 사용해 다른 객체를 생성할 수 있습니다.
  const { name, age } = new person1.constructor("Mike", 2);
  
  this.type = type;
  this.owner = name;
  this.ownerAge = age;
}

const animal1 = new Animal("Dog");

console.log({ owner: animal1.owner });
console.log({ ownerAge: animal1.ownerAge }); 
```

Animal 생성자 함수에서 person1.constructor를 사용해서 새로운 Person 객체를 만들어서 owner와 ownerAge 속성을 초기화했습니다. 그 결과로 아래와 같이 owner와 owerAge 가 출력됩니다.

```js
{ owner: 'Mike' }
{ ownerAge: 2 }
```



