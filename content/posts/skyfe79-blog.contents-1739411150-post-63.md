---
title: "객체 속성 접근하기"
date: 2023-06-03T10:56:17Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

## 1. 객체 속성

자바스크립트 객체의 속성은 키와 값으로 구성되며, 이는 객체의 상태와 행동을 나타냅니다. 자바스크립트에서는 두 가지 방식으로 객체의 속성에 접근할 수 있습니다

 - 점 표기법(dot notation)
 - 대괄호 표기법(bracket notation)

## 2. 점 표기법으로 속성에 접근하기

점 표기법은 가장 일반적인 방법으로, 객체 이름 뒤에 점(`.`)과 속성 이름을 붙여 사용합니다.

```javascript
let obj = { name: "홍길동", age: 25 };
console.log(obj.name); // "홍길동"
console.log(obj.age); // 25
```


## 3.대괄호 표기법으로 속성에 접근하기

대괄호 표기법은 속성 이름을 문자열로 사용하므로, 동적인 속성 이름이나 유효하지 않은 식별자 이름을 가진 속성에 접근할 때 유용합니다.

```javascript
let obj = { name: "홍길동", age: 25, "favorite food": "김치", "79번지": "주소입니다." };
console.log(obj["name"]); // "홍길동"
console.log(obj["age"]); // 25
console.log(obj["favorite food"]); // "김치"
console.log(obj["79번지"]); // "주소입니다."
```

위 코드에서 공백을 포함하고 있는 속성이나 숫자로 시작하는 속성은 자바스크립트의 유효한 식별자 이름이 아니기 때문에 문자열 속성 이름으로 접근해야 합니다.

