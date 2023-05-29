---
title: "프로토타입과 속성 검색"
date: 2023-05-29T13:41:30Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트 객체의 정보인 속성과 메서드 내용을 때로는 검색하거나 존재 유/무를 판단해야 할 때도 있습니다. 자바스트립트는 다양한 연산자와 메서드로 이를 지원합니다.

## in 연산자

in 연산자는 객체 내부에 특정 속성이 존재하는지 확인할 때 사용하는 자바스크립트 연산자입니다. in 연산자는 객체의 속성 이름을 문자열로 받아서 해당 객체 내에 그 속성이 있는지 여부를 불리언(Boolean) 값으로 반환합니다.

아래와 같이 in 연산자를 사용합니다.

```js
속성이름 in 객체
```

여기서, `속성이름`은 문자열로 된 속성 이름이며, `객체`는 해당 속성을 찾으려고 하는 객체입니다. 아래와 같이 사용할 수 있습니다.

```js
const myObj = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  }
};

console.log('name' in myObj); // true
console.log('gender' in myObj); // false
console.log('city' in myObj); // false
```

`myObj`라는 객체에 대해 `in` 연산자를 사용하면, 해당 객체 내부에 `'name'` 속성이 존재하므로 첫 번째 `console.log()` 구문의 결과값은 true가 됩니다. 그러나 `'gender'`와 `'city'` 속성은 존재하지 않으므로 두 번째와 세 번째 `console.log()` 구문의 결과값은 false가 됩니다.

in 연산자는 불리언(Boolean) 결과값을 반환하기 때문에 다양하게 활용할 수 있습니다.

```js
const myObj = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  }
};

if ('name' in myObj) {
	...
}
```

특정 속성의 존재 여부에 따라서 로직을 달리 구현할 수 있습니다.  in 연산자는 for...in 루프에서도 사용할 수 있습니다.  for...in 루프는 객체의 모든 속성과 메서드에 대해 반복문을 실행합니다.

```js
const myObj = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  }
};

for (const prop in myObj) {
  console.log(`${prop}: ${myObj[prop]}`);
}
```

## hasOwnProperty 메서드

hasOwnProperty 메서드는 객체에 특정 속성이 존재하는지 확인할 수 있는 메서드입니다. 단, hasOwnProperty 메서드는 객체의 프로토타입 체인을 검색하지 않고, 해당 객체 자체에서만 특정 속성이 있는지를 확인한다는 사실을 기억해야 합니다.

hasOwnProperty 메서드의 사용 방법은 매우 간단합니다.

```js
object.hasOwnProperty(property)
```

여기서 object는 객체이며, property는 해당 object 객체에서 찾고자 하는 속성 이름의 문자열 값입니다.

```js
const obj = { a: 1 };
console.log(obj.hasOwnProperty('a')); // true
console.log(obj.hasOwnProperty('toString')); // false
```

위 예제에서 obj에는 a라는 속성이 있으므로 `hasOwnProperty('a')`의 결과값은 true 입니다. 하지만 obj에는 toString이라는 속성은 없으므로 `hasOwnProperty('toString')`의 결과값은 false가 됩니다. 하지만 toString 메서드는 부모 객체인 Object가 가지고 있기 때문에 아래와 같이 메서드 호출은 성공합니다.

```js
obj.toString(); // [object Object]
```

hasOwnProperty 메서드를 사용해 두 개의 객체를 병합할 수 있습니다.

```js
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

function mergeObjects(obj1, obj2) {
    const result = {};

    for (let prop in obj1) {
        if (!result.hasOwnProperty(prop)) {
            result[prop] = obj1[prop];
        }
    }

    for (let prop in obj2) {
        if (!result.hasOwnProperty(prop)) {
            result[prop] = obj2[prop];
        }
    }

    return result;
}

console.log(mergeObjects(obj1, obj2)); // { a: 1, b: 2, c: 4 }
```

mergeObjects 함수는 두 개의 객체를 병합하여 새로운 객체를 반환합니다. 이 때 각각의 객체에서 중복된 속성은 전자의 속성을 사용합니다. 이런 로직을 만들기 위해서 hasOwnProperty 메서드를 사용했습니다.

## propertyIsEnumerable 메서드

`propertyIsEnumerable` 메서드는 이름 그대로 객체 속성이 열거 가능한지를 확인하는 메서드입니다. 이 메서드는 인자로 받은 속성이 객체 자체에 직접 정의되어 있는지, 상속받은 속성인지 여부와 상관없이, 해당 속성이 열거 가능한지를 판단합니다.

`propertyIsEnumerable` 메서드는 아래와 같이 객체의 프로토타입에 인자로 받은 속성이 존재하면 true를 반환합니다.

```js
const obj = {
  prop1: 'foo',
};

console.log(obj.propertyIsEnumerable('prop1')); // true
console.log(obj.propertyIsEnumerable('toString')); // false (상속받은 속성)
```

### hasOwnProperty 와 propertyIsEnumerable 차이점

`Object.prototype.hasOwnProperty`  메서드는 객체가 특정 속성을 자체적으로 가지고 있는지 (즉, 상속된 속성이 아닌지) 확인합니다. 이 메서드는 인자로 속성의 이름을 문자열로 받습니다. 해당 속성이 객체에 직접 존재하면 true를 반환하고, 그렇지 않으면 false를 반환합니다.

```js
let obj = { a: 1 }; 
console.log(obj.hasOwnProperty('a')); // true
console.log(obj.hasOwnProperty('b')); // false
```

`Object.prototype.propertyIsEnumerable` 이 메서드는 객체의 특정 속성이 열거 가능한지 확인합니다. "열거 가능"이란, 해당 속성이 `for...in` 루프나 `Object.keys()` 등의 메서드에서 반환될 수 있는지를 의미합니다. 일반적으로 사용자가 정의한 속성은 열거 가능하고, 내장 속성은 그렇지 않습니다. 이 메서드는 인자로 속성의 이름을 문자열로 받습니다. 해당 속성이 객체에 직접 존재하면서 열거 가능하다면 true를 반환하고, 그렇지 않으면 false를 반환합니다.

```js
let obj = { a: 1 }; 
console.log(obj.propertyIsEnumerable('a')); // true
console.log(obj.propertyIsEnumerable('b')); // false
```

두 메서드의 주요 차이점은 `propertyIsEnumerable`이 추가적으로 해당 속성이 열거 가능한지를 확인한다는 점입니다.

### propertyIsEnumerable와 Object.getOwnPropertyDescriptor

`Object.getOwnPropertyDescriptor` 메서드는 객체의 속성을 검색하고, 해당 속성에 대한 정보를 반환합니다. 만약 문자열로 주어진 속성이 없다면 undefined를 반환합니다.

```js
console.log(Object.getOwnPropertyDescriptor(obj, 'prop1'));
```

출력 결과는 아래와 같습니다.

```json
{ 
	value: 'foo', 
	writable: true, 
	enumerable: true, 
	configurable: true
}
```

이 메서드에서 반환하는 정보 중 하나가 `enumerable` 속성입니다. 이 속성값은 `propertyIsEnumerable` 과 동일합니다.

```js
const obj = {
  prop1: 'foo',
};

console.log(Object.getOwnPropertyDescriptor(obj, 'prop1').enumerable); // true
```

## Object.keys 메서드

`Object.keys` 메서드는 인자로 주어진 객체의 열거 가능한 속성 이름을 배열로 반환합니다. 

```js
const obj = { a: 1, b: 2, c: 3 };
console.log(Object.keys(obj)); // ["a", "b", "c"]
```

배열의 경우 인덱스 배열이 반환됩니다.

```js
const arr = ["a", "b", "c"];
console.log(Object.keys(arr)); // ["0", "1", "2"]
```

객체의 속성을 순회하면서 특정 로직을 실행할 수 있습니다.

```js
const obj = { a: 1, b: 2, c: 3 };
const keys = Object.keys(obj);

keys.forEach((key) => {
  console.log(`${key}: ${obj[key]}`);
});
```

`Object.keys` 메서드는 열거 가능한 속성 이름만 반환한다는 점을 주의해야 합니다. 따라서 열거 불가능한 속성이나 상속받은 속성은 포함되지 않음을 기억해야 합니다.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayHello = function () {
  console.log(`Hello, ${this.name}!`);
};

const person = new Person("John");
console.log(Object.keys(person)); // ["name"]
```

`Person` 함수로 생성된 `person` 객체는 `name` 속성과 `sayHello` 메서드를 가지고 있습니다. 하지만 `Object.keys` 메서드는 `name` 속성만 반환하고 있습니다. `sayHello` 메서드는 Person.prototype에서 상속 받은 것이기 때문에 열거 가능한 속성이 아닙니다. 

만약 아래와 같이 자체 속성으로 정의하면 열거 가능한 속성이 됩니다.

```js
function Person(name) {
	this.name = name;
	this.sayHello = function () {
		console.log(`Hello, ${this.name}!`);
	}
}
const person = new Person("John");
console.log(Object.keys(person)); 
```

아래와 같이 출력됩니다.

```js
['name', 'sayHello']
```

## for...in 문

JavaScript에서 for...in문은 객체의 모든 속성에 대해 반복하는 루프입니다. 

```js
for (variable in object) {
   statements
}
```

예를 들어, 다음과 같은 객체가 있다고 가정해 봅시다.

```js
const person = { 
   name: 'John', 
   age: 30, 
   job: 'developer' 
};
```

for...in문을 사용하여 모든 속성을 반복할 수 있습니다.

```js
for(const prop in person) {
    console.log(prop + ': ' + person[prop]);
}
```

출력 결과는 아래와 같습니다.

```
name: John
age: 30
job: developer
```

for...in 문은 상속된 속성도 열거합니다.

```js
const person = { 
   name: 'John', 
   age: 30, 
   job: 'developer' 
};

Object.prototype.country = 'Korea';

for(const prop in person) {
    console.log(prop + ': ' + person[prop]);
}

// 출력 결과:
// name: John
// age: 30
// job: developer
// country: USA
```

Object.prototype에 country 속성을 추가한 후 for...in문을 사용하여 person 객체의 속성을 열거했습니다. 그 결과, 상속된 country 속성도 출력되었습니다.

이 때, hasOwnProperty 메소드를 사용하면 상속된 속성을 제외하고 객체의 속성만 열거할 수 있습니다.

```js
const person = { 
   name: 'John', 
   age: 30, 
   job: 'developer' 
};

Object.prototype.country = 'Korea';

for(const prop in person) {
    if (person.hasOwnProperty(prop)) {
        console.log(prop + ': ' + person[prop]);
    }
}
// 출력 결과:
// name: John
// age: 30
// job: developer
```

for...in문은 배열에 대해 사용할 수 있지만 권장되지 않습니다. 배열의 요소를 열거하는 것이 아니라, 배열의 인덱스를 열거하기 때문입니다.

```js
const arr = ['apple', 'banana', 'orange'];

for(const prop in arr) {
    console.log(prop);
}

// 출력 결과:
// 0
// 1
// 2
```

배열의 인덱스가 필요한 경우가 아니라 요소가 필요한 경우라면, for...in 문 대신 for...of 문 사용을 권장합니다.

```js
const arr = ['apple', 'banana', 'orange'];

for(const prop of arr) {
    console.log(prop);
}

// 출력 결과:
// apple
// banana
// orange
```

## Reflect.has 메서드

Reflect.has() 메서드는 특정 객체가 특정 속성을 가지고 있는지 여부를 확인합니다. 이 메서드는 기본적으로 Object.prototype.hasOwnProperty()와 같은 역할을 합니다. 그러나 Reflect.has() 메서드는 리플렉션(Reflection) API의 일부로 제공되므로 보다 강력합니다.

Reflect.has() 메서드의 구문은 다음과 같습니다.

```js
Reflect.has(target, propertyKey)
```

- target: 검사할 객체입니다.
- propertyKey: 검사할 객체에서 검사할 속성 이름입니다.

```js
let obj = {
    name: 'John',
    age: 30,
    address: {
        city: 'Seoul',
        country: 'South Korea'
    }
};

console.log(Reflect.has(obj, 'name')); // true
console.log(Reflect.has(obj, 'age')); // true
console.log(Reflect.has(obj, 'address')); // true
console.log(Reflect.has(obj, 'city')); // false
```

위 예제에서는 obj 객체에 대해 Reflect.has() 메서드를 사용하여 속성이 존재하는지 여부를 확인합니다.  `name`, `age`, `address` 속성의 경우, Reflect.has() 메서드는 true를 반환합니다. 마지막  `city` 속성의 경우, Reflect.has() 메서드는 false를 반환합니다.


## Object.getOwnPropertyNames 메서드

`Object.getOwnPropertyNames` 메서드는 인자로 전달된 객체의 모든 속성 이름을 배열로 반환합니다. `for...in` 구문과 차이점은  `for...in` 객체의 프로토타입 체인까지 탐색하여 열거할 수 있는 속성들을 반환합니다. 하지만 `Object.getOwnPropertyNames()`는 객체 자체에 정의된 속성들만 반환합니다.

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayHello = function() {
  console.log(`Hello ${this.name}`);
};

const john = new Person('john', 30);
const props = Object.getOwnPropertyNames(john);

console.log(props); // ['name', 'age']

for (const prop in john) {
    console.log(prop);
}
// name
// age
// sayHello
```

위 예제에서 `john` 객체는 `Person` 생성자 함수로 생성한 객체입니다. `sayHello` 메서드는 `Person` 생성자 함수의 프로토타입에 정의된 속성입니다. 따라서 `for...in` 구문으로 열거할 수 있지만, `Object.getOwnPropertyNames()`로는 반환되지 않습니다.

`Object.getOwnPropertyNames` 를 사용해 객체 복제를 구현할 수 있습니다.

```js
const obj = { name: 'john', age: 30 };
const clone = {};

Object.getOwnPropertyNames(obj).forEach(prop => {
  clone[prop] = obj[prop];
});

console.log(clone); // { name: 'john', age: 30 }
```

객체의 속성 이름들을 정렬하고 싶은 경우, `Object.getOwnPropertyNames()` 메서드를 사용하여 배열 형태로 얻은 후, 배열의 `sort()`를 사용해 정렬 할 수 있습니다.

```js
const obj = { b: 1, a: 2, c: 3 };
const props = Object.getOwnPropertyNames(obj).sort();

console.log(props); // ['a', 'b', 'c']
```



