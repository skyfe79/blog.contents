---
title: "프로토타입 체인"
date: 2023-05-29T04:48:22Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트 객체는 프로토타입을 통해 상속을 구현합니다. 프로토타입 체인은 상속과 관련된 것으로 호출한 속성과 메서드를 찾아가는 과정입니다. 

자바스크립트 객체는 속성이나 메서드를 찾을 때, 자기 자신에게서 먼저 찾고, 그 다음 부모 객체인 프로토타입에서 찾습니다. 이 과정을 프로토타입 체인이라고 합니다.

```js
const person = {
  name: 'John',
  age: 30,
  greet: function() {
    console.log('Hello, my name is ' + this.name);
  }
};

// person을 상속하여 employee객체를 생성합니다.
const employee = Object.create(person);
employee.salary = 5000;


console.log(employee.name); // "John"
console.log(employee.age); // 30
employee.greet(); // "Hello, my name is John"
console.log(employee.salary); // 5000
```

위 코드에서 직접 정의한 `employee.salary` 속성을 제외하고, `employee.name`, `employee.age`, `employee.greet()`은 모두 person 객체에서 상속된 것입니다. employee 객체는 프로토타입 체인 과정을 통해 해당 속성과 메서드를 찾아 참조하거나 호출합니다.  

프로토타입 체인 과정을 `__proto__` 속성을 사용해 직접 테스트해 볼 수 있습니다.

```js
const ageObj = { age: 19 };
const nameObj = Object.create(ageObj);
nameObj.name = 'John';

const john = Object.create(nameObj);

console.log(john.__proto__.name); 
console.log(john.__proto__.__proto__.age); 
```

위 코드를 실행하면 아래와 같이 결과가 출력됩니다.

```
John
19
```

name과 age 속성을 `__proto__` 를 연속해서 호출해서 찾아 참조했습니다. 


