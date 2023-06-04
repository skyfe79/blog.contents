---
title: "객체 불변성"
date: 2023-06-04T05:47:32Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. 불변성이란?

불변성(Immutability)은 객체가 생성된 후 그 상태를 변경할 수 없도록 하는 개념입니다. 자바스크립트에서는 `Object.freeze()`와 `Object.seal()` 메소드를 이용하여 객체의 불변성을 확보할 수 있습니다.

## 2. Object.freeze() 메소드

`Object.freeze()` 메소드는 객체를 '동결'하여 객체 속성을 추가, 제거, 수정할 수 없도록 만듭니다.

```javascript
let obj = { prop: 5 };
Object.freeze(obj);

obj.prop = 10;  // 변경되지 않음
obj.newProp = 15;  // 추가되지 않음
delete obj.prop;  // 삭제되지 않음. 오류가 발생합니다.

console.log(obj.prop);  // 5
```

위 예제에서, `Object.freeze(obj)` 이후에는 `obj`의 속성을 변경할 수 없습니다. 그러나 `Object.freeze()` 메소드는 얕은(shallow) 동결만을 수행합니다. 즉, 객체의 속성이 다른 객체를 참조하는 경우, 그 참조된 객체는 동결되지 않습니다.

```javascript
let obj = { prop: 5, subObj: { prop: 10 } };
Object.freeze(obj);

obj.subObj.prop = 15;

console.log(obj.subObj.prop);  // 15
```

위 예제에서, `obj`의 `subObj` 속성이 참조하는 객체는 동결되지 않아, `subObj` 객체의 속성은 변경할 수 있습니다.

## 3. Object.seal() 메소드

`Object.seal()` 메소드는 객체를 '봉인'하여 그 후에는 속성을 추가, 제거할 수 없지만, 이미 존재하는 속성의 값을 변경할 수는 있습니다.

```javascript
let obj = { prop: 5 };
Object.seal(obj);

obj.prop = 10;  // 변경 가능
obj.newProp = 15;  // 추가되지 않음. 오류 발생
delete obj.prop;  // 삭제되지 않음. 오류 발생

console.log(obj.prop);  // 10
```

위 예제에서, `Object.seal(obj)` 이후에는 `obj`의 속성을 추가, 제거할 수 없지만, `prop` 속성의 값을 변경할 수 있습니다. `Object.seal()` 메소드도 `Object.freeze()`와 마찬가지로 얕은 봉인만을 수행합니다. 즉, 객체의 속성이 다른 객체를 참조하는 경우, 그 참조된 객체는 봉인되지 않습니다.

```javascript
let obj = { prop: 5, subObj: { prop: 10 } };
Object.seal(obj);

obj.subObj.prop = 15;

console.log(obj.subObj.prop);  // 15
```

위 예제에서, `obj`의 `subObj` 속성이 참조하는 객체는 봉인되지 않아, `subObj` 객체의 속성은 변경할 수 있습니다.

## 4. 불변성의 중요성

객체 불변성은 중요합니다. 특히 동시성 프로그래밍 환경에서는 공유 자원을 꼭 필요할 때만 mutable 하게 만들고 그 이외에는 immutable(불변)하게 만드는 것이 오류 방지를 위해 매우 중요합니다.


