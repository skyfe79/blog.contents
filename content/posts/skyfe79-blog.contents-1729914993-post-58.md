---
title: "Prototype 이란?"
date: 2023-05-29T02:57:44Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

프로토타입은 객체를 정의하고 상속을 구현하는 방법입니다. 자바스크립트 객체는 다른 객체를 상속할 수 있습니다. 자바스크립트에서는 모든 객체가 `__proto__`라는 숨겨진 속성을 가지고 있는데, 이 속성은 해당 객체의 부모 역할을 합니다. myObj 객체는 자바스크립트 `Object` 객체를 상속하고 있습니다.

```js
const myObj = { name: 'John' };
console.log(myObj.__proto__); // Object {}
```

`__proto__` 속성을 직접 설정하여 상속 관계를 만들 수도 있습니다.

```js
let child = {};
let parent = { name: "Alice" };

child.__proto__ = parent;

console.log(child.name); // "Alice"
```

`child.__proto__ = parent` 를 통해 child와 parent 간에 상속 관계를 만들었습니다. 좀 더 많은 속성과 함수를 가진 객체도 상속할 수 있습니다.

```js
var person = {
  name: "John",
  age: 30,
  greet: function() {
    console.log("Hello, my name is " + this.name + " and I'm " + this.age + " years old.");
  }
};

var student = {
  major: "Computer Science"
};

student.__proto__ = person;

console.log(student.name); 
console.log(student.age); 
student.greet();
```

위 코드를 실행하면 아래와 같이 결과가 출력됩니다.

```
John
30
Hello, my name is John and I'm 30 years old.
```

student 객체가 person 객체의 속성과 메서드를 상속하여 사용할 수 있음을 확인할 수 있습니다. 이처럼 자바스크립트의 모든 객체가 가지고 있는 내부 속성 `__proto__` 를 사용해 객체간의 상속 관계를 만들 수 있습니다.

