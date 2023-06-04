---
title: "객체 상속"
date: 2023-06-04T05:57:00Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. 프로토타입 상속 이해하기

자바스크립트는 프로토타입을 통해 객체 상속을 구현합니다. 

```javascript
let animal = {
  eat: function() {
    console.log('먹는 중...');
  }
};

let rabbit = {
  jump: function() {
    console.log('점프하는 중...');
  }
};

rabbit.__proto__ = animal;
rabbit.eat();  // 먹는 중...
rabbit.jump();  // 점프하는 중...
```

위 예제에서, `rabbit` 객체는 `animal` 객체를 상속받아 `animal` 객체의 메소드인 `eat()`을 사용할 수 있게 됩니다. 하지만 `__proto__ `을 직접 설정하는 것은 번거롭고 미지의 버그를 만들 수 있습니다.

## 2. Object.create() 메소드

`Object.create()` 메소드를 통해 쉽게 프로토타입을 상속받는 객체를 생성할 수 있습니다. 이 메소드는 주어진 객체를 프로토타입으로 하는 새로운 객체를 생성합니다.

```javascript
let animal = {
  eat: function() {
    console.log('먹는 중...');
  }
};

let rabbit = Object.create(animal);
rabbit.jump = function() {
  console.log('점프하는 중...');
};

rabbit.eat();  // 먹는 중...
rabbit.jump();  // 점프하는 중...
```

위 예제에서, `Object.create(animal)`는 `animal` 객체를 프로토타입으로 하는 `rabbit` 객체를 생성합니다. 따라서 `rabbit` 객체는 `animal` 객체의 메소드인 `eat()`을 사용할 수 있습니다. `__proto__ `을 직접 설정하는 것보다 코드 이해가 쉽습니다.

## 3. Object.create()의 두 번째 인수

`Object.create()` 메소드는 두 번째 인수로 속성 설명자를 받을 수 있습니다. 이를 통해 새로운 객체에 속성을 추가할 수 있습니다.

```javascript
let animal = {
  eat: function() {
    console.log('먹는 중...');
  }
};

let rabbit = Object.create(animal, {
  jump: {
    value: function() {
      console.log('점프하는 중...');
    },
    writable: true,
    enumerable: true,
    configurable: true
  }
});

rabbit.eat();  // 먹는 중...
rabbit.jump();  // 점프하는 중...
```

위 예제에서, `Object.create(animal, { ... })`는 `animal` 객체를 프로토타입으로 하면서 `jump`라는 속성을 가진 `rabbit` 객체를 생성합니다.

## 4. 프로토타입 체인

프로토타입 체인을 통해 자바스크립트는 객체의 상속 관계를 구축합니다.

```javascript
let animal = {
  eat: function() {
    console.log('먹는 중...');
  }
};

let rabbit = Object.create(animal);
rabbit.jump = function() {
  console.log('점프하는 중...');
};

let longEar = Object.create(rabbit);
longEar.listen = function() {
  console.log('듣는 중...');
};

longEar.eat();  // 먹는 중...
longEar.jump();  // 점프하는 중...
longEar.listen();  // 듣는 중...
```

위 예제에서, `longEar` 객체는 `rabbit` 객체를, `rabbit` 객체는 `animal` 객체를 상속받습니다. 이를 통해 `longEar` 객체는 `rabbit` 객체와 `animal` 객체의 메소드를 모두 사용할 수 있습니다.

## 5. Object.create() 사용시 주의할 점

`Object.create()` 메소드는 프로토타입을 상속받는 새로운 객체를 생성하긴 하지만, 객체를 복사하는 것은 아닙니다. 즉, 원본 객체에 변화가 생기면 프로토타입을 상속받은 객체에도 영향을 미칩니다.

```javascript
let animal = {
  eat: function() {
    console.log('먹는 중...');
  }
};

let rabbit = Object.create(animal);
rabbit.jump = function() {
  console.log('점프하는 중...');
};

animal.eat = function() {
  console.log('새로운 먹는 행동...');
};

rabbit.eat();  // 새로운 먹는 행동...
```

위 예제에서, `animal` 객체의 `eat()` 메소드가 변경되면, 이를 상속받은 `rabbit` 객체에서도 `eat()` 메소드의 행동이 변경됩니다. 이런 결과는 때로는 의도하지 않은 결과와 버그를 만들 수 있습니다. `Object.create()`을 이용한 객체 상속을 구현할 때는 반드시 이 내용을 숙지하고 있어야 합니다.


