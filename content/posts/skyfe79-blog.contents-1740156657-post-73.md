---
title: "객체 복사"
date: 2023-06-04T05:46:58Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. 얕은 복사와 깊은 복사 이해하기

자바스크립트에서 객체를 복사할 때 고려해야 할 중요한 개념으로 깊은 복사(deep copy)와 얕은 복사(shallow copy)가 있습니다. 

얕은 복사는 객체의 최상위 속성을 복사합니다. 그러나 그 속성이 참조 타입(객체나 배열 등)일 경우 참조 값만 복사하게 됩니다. 따라서 원본 객체나 복사본에서 중첩된 객체를 변경하면, 그 변경사항이 서로에게 영향을 줍니다.

```javascript
let obj1 = { a: 1, b: { c: 2 }};
let obj2 = Object.assign({}, obj1);
obj2.b.c = 3;
console.log(obj1.b.c); // 3
```

깊은 복사는 객체의 모든 레벨에서 속성을 복사합니다. 중첩된 객체도 새로운 객체로 복사되므로, 원본 객체와 복사본은 완전히 독립적입니다.

```javascript
let obj1 = { a: 1, b: { c: 2 }};
let obj2 = JSON.parse(JSON.stringify(obj1));
obj2.b.c = 3;
console.log(obj1.b.c); // 2
```

깊은 복사는 객체의 모든 레벨에서 복사본을 생성하므로, 원본 객체의 변경이 복사본에 영향을 미치지 않습니다. 그러나, `JSON.parse(JSON.stringify())`을 이용한 깊은 복사는 함수, 심볼, 순환 참조 등 JSON으로 변환할 수 없는 값을 가진 속성을 복사할 수 없다는 한계가 있습니다.

이러한 한계를 극복하기 위해 여러 외부 라이브러리들이 개발되어 있습니다. 그 중에서도 `lodash` 라이브러리의 `_.cloneDeep()` 메소드는 깊은 복사를 수행하면서 위에서 언급된 한계를 극복할 수 있습니다.

`lodash` 라이브러리를 사용하여 깊은 복사를 수행하는 예제는 다음과 같습니다:

```javascript
let _ = require('lodash');
let obj1 = {
  a: 1,
  b: {
    c: 2,
    d: function() { console.log('hello'); },
    e: Symbol('symbol'),
    f: [1, 2, 3]
  }
};
let obj2 = _.cloneDeep(obj1);
```

위의 예제에서, `obj2`는 `obj1`의 깊은 복사본이며, `obj1`에 변화가 있어도 `obj2`에는 영향을 미치지 않습니다. 또한, 함수, 심볼, 배열 등 `JSON.stringify()`에서는 제대로 복사되지 않는 값을 포함한 모든 속성이 복사됩니다.

하지만, `lodash` 역시 완벽하지 않습니다. 더 복잡한 객체 구조, 예를 들어 순환 참조나 특정 내장 객체들에 대해서는 여전히 문제가 발생할 수 있습니다. 따라서, 깊은 복사를 수행할 때에는 어떤 메소드나 라이브러리를 사용하더라도 주의가 필요합니다.

## 2. Object.assign() 이란?

자바스크립트에서 `Object.assign()` 메소드는 하나 이상의 소스(`source`) 객체로부터 모든 열거 가능한 속성을 타겟(`target`) 객체로 복사하는데 사용됩니다. 이 메소드를 이용하면 새로운 객체를 생성하고 그것을 복제본으로 사용할 수 있습니다.

```javascript
let obj = { a: 1 };
let copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
```

`Object.assign()` 메소드는 첫 번째 인수로 주어진 객체에 나머지 인수로 주어진 객체의 속성을 복사합니다. 복사는 소스(`source`) 객체의 키-값 쌍에 대해 수행되며, 같은 키를 가진 속성이 타겟(`target`) 객체에 이미 존재할 경우 그 값은 소스(`source`) 객체의 값으로 덮어씌워집니다.

```javascript
let obj1 = { a: 1, b: 2 };
let obj2 = { b: 3, c: 4 };
let obj3 = Object.assign(obj1, obj2);
console.log(obj3); // { a: 1, b: 3, c: 4 }
```

`Object.assign()`은 얕은 복사를 수행합니다. 이는 객체 내부에 중첩된 객체가 있는 경우 그 내부 객체의 참조를 복사한다는 것을 의미합니다. 따라서, 내부 객체를 수정하면 복제본과 원본 모두에 영향을 미칩니다.

```javascript
let obj = { a: 1, b: { c: 2 }};
let copy = Object.assign({}, obj);
copy.b.c = 3;
console.log(obj.b.c); // 3
```

깊은 복사를 수행하려면 `JSON.parse()`와 `JSON.stringify()`를 사용할 수 있습니다. 이 방법은 객체의 모든 레벨에서 새로운 복제본을 만듭니다. 그러나 이 방법은 함수나 심볼, 순환 참조 등 JSON으로 변환할 수 없는 값을 가진 속성을 복사할 수 없다는 단점이 있습니다.

```javascript
let obj = { a: 1, b: { c: 2 }};
let copy = JSON.parse(JSON.stringify(obj));
copy.b.c = 3;
console.log(obj.b.c); // 2
```

