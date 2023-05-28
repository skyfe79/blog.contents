---
title: "실행 컨텍스트(Execution Context)"
date: 2023-05-28T12:14:14Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트 엔진은 코드를 해석하고 실행하는 복잡성을 관리하기 위해 실행 컨텍스트를 사용합니다. 실행 컨텍스트는 현재 코드가 실행되는 환경에 대한 정보를 담고 있는 추상적인 개념입니다.

자바스크립트에는 세 가지 실행 컨텍스트 유형이 있습니다:

- Global 실행 컨텍스트: 자바스크립트 엔진에 의해 기본적으로 생성됩니다.
- 함수 실행 컨텍스트: 함수가 실행될 때마다 생성됩니다.
- Eval 실행 컨텍스트: eval 함수 내에서 생성됩니다.

ui.dev가 제작한 [JavaScript Visualizer 도구](https://ui.dev/javascript-visualizer/)를 사용하여 실행 컨텍스트, 호이스팅, 클로저 및 스코프가 어떻게 작동하는지 쉽게 시각화할 수 있습니다. 이 도구를 사용하여 JavaScript 실행 컨텍스트 및 작동 방식을 이해하는 데 도움을 받을 수 있습니다.

### JavaScript Visualizer 사용 예시

[JavaScript Visualizer 도구](https://ui.dev/javascript-visualizer/) 로 이동하여 아래 코드를 입력합니다. 단, ES5 코드를 입력해야 합니다.

```js
var name = "John";  
function sayHello() {   
	console.log('Hello, ' + name); 
}
sayHello();
```

`Step` 함수를 누르면 `Global` 실행 컨텍스트가 `Creation` 단계부터 시작하여 `Execution` 단계로 이동하며 실행되는 것을 확인할 수 있습니다.


<img width="1070" alt="SCR-20230528-riwz" src="https://github.com/skyfe79/blog.contents/assets/309935/3697f555-d396-4abf-b3f3-2e20ec30f28f">


<img width="1126" alt="SCR-20230528-rjhk 1" src="https://github.com/skyfe79/blog.contents/assets/309935/37ac9cc2-a797-41c3-9bc0-678dd3005b02">


위의 예제에서 전역 실행 컨텍스트 부분은 전체 코드가 실행되는 환경을 나타내며, 함수 실행 컨텍스트 부분은 `sayHello` 함수가 실행될 때 생성되는 환경을 나타냅니다. 실행 컨텍스트는 변수, 함수 선언 그리고 인자 값 같은 정보를 보유하며, 필요에 따라 새로운 변수 및 함수를 만들 수 있습니다.

## 전역 실행 컨텍스트(Global Execution Context)

JavaScript 엔진은 코드를 실행하기 전에 새로운 실행 컨텍스트를 만드는데, 이 새로운 실행 컨텍스트를 전역 실행 컨텍스트라고 합니다. 

전역 실행 컨텍스트는 자바스크립트 엔진에 의해 기본적으로 생성된 실행 컨텍스트입니다. 함수나 객체 안에 속하지 않은 전역 코드는 전역 실행 컨텍스트에서 실행됩니다.

JS Visualizer 에서 `Run` 버튼을 누르기 전에 코드를 작성하지 않은 채로 접속하면, 우리는 전역 실행 컨텍스트가 생성된 것을 확인할 수 있습니다.


<img width="1125" alt="SCR-20230528-rkgf" src="https://github.com/skyfe79/blog.contents/assets/309935/69de5d82-88a7-4b42-b36b-104ae4a90a09">


자바스크립트의 실행 컨텍스트는 두 가지 요소로 구성됩니다.

- 전역 객체: 현재 환경 어디에서나 사용 가능한 변수와 함수를 제공합니다. 브라우저에서 전역 객체는 window이고 Node.js 전역 객체는 global입니다.
- this 객체: this 키워드는 코드가 속한 실행 컨텍스트의 객체를 가리킵니다.

자바스크립트 엔진은 코드가 없는 상황에서도 전역 실행 컨텍스트를 생성합니다. 자바스크립트 실행시 전역 실행 컨텍스트를 하나만 갖도록 제한하는데 자바스크립트가 단일 스레드 프로그래밍 언어이기 때문입니다.

두 수를 더하는 코드 예제를 통해 전역 실행 컨텍스트와 함수 실행 컨텍스트가 어떻게 작동하는지 살펴 봅시다.

```javascript
var number1 = 10;
var number2 = 10;


function sum(n1, n2) { 
  return n1 + n2;
}
```


<img width="1087" alt="SCR-20230528-rmzl" src="https://github.com/skyfe79/blog.contents/assets/309935/68faa36f-cc42-4b9d-af16-33b54eb7329f">


`Step` 버튼을 한 번 클릭하면 전역 실행 컨텍스트의 단계가 `생성(Creation)` 단계로 변한 것을 확인할 수 있습니다. 

전역 실행 컨텍스트는 생성 단계(Creation)와 실행 단계(Execution)로 나뉩니다.

### Creation 단계
- 전역 객체(Global object)와 this 키워드가 생성됩니다.
- 변수와 함수에 할당되는 메모리가 할당됩니다.
- 변수들은 undefined 값을 가집니다.

수행 예시：
```javascript
// Creation 단계
var a;
console.log(a); // undefined
a = 'hello';
```
위 코드에서 변수 a는 undefined 값을 가지고 있는데, 이는 메모리에 할당된 후 값을 가지지 않기 때문입니다.

### Execution 단계
- 코드의 수행이 시작합니다.
- 함수와 변수에 값이 할당됩니다.

수행 예시：
```javascript
// Execution 단계
a = 'hello';
console.log(a); // 'hello'
```

위 코드에서 변수 a에 값을 할당한 후, 값을 출력합니다.

`Step` 버튼을 계속해서 클릭하면 전역 실행 컨텍스트의 phase 가 변경되고 각 변수에 값이 할당되는 것을 확인할 수 있습니다.

![g-e-c](https://github.com/skyfe79/blog.contents/assets/309935/2d41a51c-8a75-44f3-9ff4-8ec4b9e117a1)

## 함수 실행 컨텍스트(Function Execution Context)

함수 실행 컨텍스트는 함수가 실행될 때 생성됩니다. 즉, 함수가 호출될 때마다 해당 함수에만 적용되는 환경을 컨텍스트 형태로 만들어 주는데요. 이 환경은 변수, 함수 등의 정보를 담아두는데 주로 사용합니다. 

함수 실행 컨텍스트는 크게 두 가지 요소로 이루어져 있습니다. 첫째는 Activation Object이고 둘째는 Scope Chain입니다.

### Activation Object(활성화 객체)

Activation Object는 함수가 호출될 때 마다 만들어지는 객체입니다. 이 객체는 함수의 매개변수와 지역 변수 등을 저장하기 위해 생성됩니다. Activation Object는 매개변수나 지역 변수와 같은 식별자 이름을 key로 가지고, 그에 해당하는 값들을 value로 가지게 됩니다. 

다만, 전역 컨텍스트에서는 Activation Object가 생성되지 않습니다. 전역 컨텍스트에서는 window 객체가 Activation Object처럼 동작하기 때문입니다.

### Scope Chain(스코프 체인)

스코프 체인은 변수를 검색할 때 사용하는 목록입니다. 함수 실행 컨텍스트 내부에서는 스코프 체인이 생성됩니다. 이때 스코프 체인은 Activation Object와 상위 컨텍스트의 스코프 체인을 차례로 따라가며 식별자를 검색하게 됩니다.

## 함수 실행 컨텍스트 실습하기

```js
function sum(a, b) {
  return a + b;
}

function invokeSum() {
  return sum(10, 20);
}

var result = invokeSum();
console.log(result);
```

위 예제 코드에서 `invokeSum()` 함수가 실행되면 `sum()` 함수가 호출됩니다. 그리고 `sum()` 함수가 호출되면 함수 실행 컨텍스트가 만들어지게 됩니다. 

![fec02](https://github.com/skyfe79/blog.contents/assets/309935/a9a62a9a-25ae-4ae0-860c-495b2d4a1710)

위 코드를 실행하면 console에 30이 출력됩니다.  함수 실행 컨텍스트에는 이와 같은 변수들의 정보 뿐만 아니라, 현재 실행 컨텍스트의 this 값, arguments 객체 등의 정보도 저장되어 있습니다. 함수 실행 컨텍스트는 다른 프로그래밍 언어에서도 거의 동일한 원리로 작동합니다. 

### 함수를 실행할 때 생성되는 새로운 실행 컨텍스트

함수를 실행할 때마다 새로운 함수 실행 컨텍스트가 생성됩니다. 이 실행 컨텍스트 안에서는 `arguments`라는 특별한 값에 접근할 수 있습니다. arguments 값은 함수를 실행할 때 전달한 인자(argument)들입니다.

예를 들어,

```js
function add(a, b) {
  return a + b;
}

add(1, 2);
```

위와 같은 함수에서 `add(1,2)`를 실행할 때, 새로운 함수 실행 컨텍스트가 생성됩니다. 이 컨텍스트 안에서는 `a=1, b=2`로 할당됩니다. 그리고 arguments 값은 `[1,2]`가 됩니다. 

함수 안에서 또 다른 함수를 실행할 수 있습니다. 이때, 내부 함수는 외부 함수에서 정의된 변수와 arguments 값에 접근할 수 있습니다. 

```js
function outer() {
  const a = 10;
  function inner() {
    console.log(a);
  }
  inner();
}

outer(); // 10 출력
```

위 코드에서는 outer 함수 내부에 inner 함수가 정의되어 있습니다. inner 함수 내에서는 outer 함수 안에서 a라는 변수가 선언되어 있는데, inner 함수는 a에 접근할 수 있습니다. inner 함수를 실행할 때, 새로운 함수 실행 컨텍스트가 생성되며, 이 컨텍스트 안에서는 arguments 값이 없습니다.


<img width="1131" alt="SCR-20230528-sdqo" src="https://github.com/skyfe79/blog.contents/assets/309935/ab3d6d4c-dc82-4340-90f9-856deb2a9f39">


각 함수 호출마다 새로운 함수 실행 컨텍스트가 생성되며, 그 안에서는 arguments 값과 변수 접근이 가능합니다. 이러한 함수 실행 컨텍스트는 함수가 호출되고 실행되는 순간에 생성되며, 함수가 실행을 마칠 때까지 유지됩니다.


## Eval 실행 컨텍스트

`eval` 함수는 문자열을 실행 가능한 JavaScript 코드로 변환하기 위해 만들어졌습니다. 이 기능이 매우 강력해 보이지만, `eval` 함수의 권한을 제어할 수 없기 때문에 사용하지 않는 것이 좋습니다.

`eval` 함수는 악성 공격 취약합니다. `eval` 함수가 받는 문자열이 악성 문자열일 경우, 데이터베이스나 어플리케이션을 완전히 파괴할 수 있습니다. 이러한 이유로 `eval` 함수는 사용하지 않는 것이 좋으며, 추천되지 않습니다.

#### 세부정보

- `eval` 함수는 문자열을 실행 가능한 JavaScript 코드로 변환합니다.
- `eval` 함수는 악성 공격에 취약합니다.
- 악성 문자열이 포함된 함수를 실행하면 데이터베이스나 어플리케이션이 완전히 파괴될 수 있습니다.
- 이러한 이유로 `eval` 함수는 사용하지 않는 것이 좋습니다.

예제:

```js
eval('2 + 2'); // 4
eval('console.log("Hello World");'); // Hello World
eval('alert("hello")'); // hello 얼럿
```

그러나 이 문자열이 안전하게 검증되지 않으면 `eval` 함수를 사용할 때 안전하지 않을 수 있습니다.

```javascript
eval('location.href="https://google.com"'); // google.com으로 이동합니다.
```

위 예제에서는 `google.com` 으로 이동해서 안전하지만 만약 해킹 코드를 다운로드 받는 스크립트가 심어진 페이지로 이동될 수 있습니다. 따라서 `eval` 함수의 사용은 추천하지 않습니다.

## Execution Context vs. Scope

프로그래밍 분야에는 많은 용어들이 있습니다. 이 용어들이 서로 비슷한 의미를 가지는 경우가 많아 개발자들끼리 혼동이 생길 때가 있습니다. JavaScript에서는 Execution Context와 Scope라는 두 가지 용어가 있는데, 이 둘은 완전히 다른 개념입니다.

### Scope와 Execution Context의 차이점

Scope는 함수를 기반으로 합니다. Scope는 함수 내에서 변수에 대한 접근을 관리합니다. JavaScript에서는 전역 스코프와 함수 스코프(또는 지역 스코프) 두 가지 스코프만 있습니다.

Execution Context는 객체 지향적입니다. Execution Context는 현재 코드가 실행되는 환경에 대한 정보를 가지는 추상적 개념입니다. 함수의 context는 해당 함수의 this 키워드 값입니다.

## Scope의 예

```javascript
var a = 1;

function example() {
  var b = 2;
  console.log(a); // 1
}

console.log(b); // ReferenceError: b is not defined
```

위 코드에서는 전역 코드에서 변수 a를 선언하고, 함수 example 내에서 변수 b를 선언합니다. 함수 example 내에서는 변수 a에 접근할 수 있지만, 함수 바깥에서는 변수 b에 접근할 수 없습니다.

## Execution Context의 예

```javascript
var a = 1;

function example() {
  console.log(this); // 전체 global 객체를 출력합니다.
}

example();
```

위 코드에서는 전역 코드에서 변수 a를 선언하고, 함수 example을 정의합니다. 함수 example 내에서는 this 키워드를 사용해서 현재 context를 출력합니다.

## 마무리

JavaScript의 핵심 개념 이해는 애플리케이션 개발에 매우 중요합니다. Execution Context는 JavaScript 코드가 어떻게 작동하는지를 이해하는 데 매우 중요한 개념입니다. 모든 JavaScript 코드에 존재하며, hoisting, closures, scope 등 다른 JavaScript 개념들을 이해하기 위한 꼭 먼저 이해하고 있어야 합니다.

> 이 글은 [Understanding Execution Context in JavaScript](https://www.telerik.com/blogs/understanding-execution-context-javascript) 글을 편역한 것입니다.

