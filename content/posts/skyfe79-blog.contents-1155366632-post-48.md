---
title: "타입스크립트로 Optional<T> 표현하기"
date: 2022-03-01T13:53:13Z
author: "skyfe79"
draft: false
tags: ["typescript"]
---

타입스크립트는 enum 타입을 지원하지만 아쉽게도 Swift나 Rust 처럼 enum에 associated type(이하 연관타입) 을 지원하지 않는다. 연관 타입을 가진 enum을 지원하는 언어는 Optional<T>를 아래와 같이 아주 쉽게 표현할  수 있다.

```swift
enum Optional<T> {
  case some(value: T)
  case none
}
```


## 타입스크립트 `Optional<T>` 구현 - 1차

타입스크립트는 연관 타입이 있는 enum을 지원하지 않기 때문에 class 활용해 비슷하게 흉내 낼 수 있다.

```typescript
class Optional<T> {
  static some<V>(value: V): Optional<V> {
    return new Optional(value);
  }
  static readonly none = new Optional(null)

  private constructor(public value?: T) {}

  get hasSome(): boolean {
    return this.value !== null
  }
}
```

### 사용하기

아래와 같이 사용할 수 있다.

```typescript
const a = Optional.some(10);
const b = Optional.none;
const c = Optional.none;

if (a.value) {
  console.log(a.value);
}

if (b.hasSome) {
  console.log(b.value);
} else {
  console.log('b is none');
}

console.log(a.hasSome);
console.log(b.hasSome);
console.log(b === Optional.none);
console.log(c === Optional.none);
console.log(b === c);
```

### 실행 결과

```
10
b is none
true
false
true
true
true
```

하지만 함수 인자를 Optional로 정의하려 한다며 어떻게 될까?

```typescript
function someFunction(payload: Optional<number | null>) {
  if (payload.value) {
    console.log(payload.value);
  }
}

someFunction(Optional.some(10));
someFunction(Optional.none);
```

null 값을 받기 위해서 `payload: Optional<number | null>` 처럼 이상한 정의문이 필요하다. 그리고 함수의 인자 값을 전달할 때, `Optional.some(10)`과 `Optional.none` 처럼 표현해야 한다. 

## 타입스크립트 `Optional<T>` 구현 - 2차

타입스크립트의 장점 중 하나인 Union Type을 사용하면 아주 깔끔하게 Optional 타입을 구현할 수 있다. 

```typescript
type Optional<Value>  = Value | null;
```

구현이라고 할 것 없이 단순한 타입 정의다.

### 사용하기

```typescript
type Optional<Value>  = Value | null;

const a: Optional<number> = 10;
const b: Optional<number> = null;

console.log(a);
console.log(b);

function someFunction(payload: Optional<number>) {
  if (payload) {
    console.log(payload);
  }
}

someFunction(10);
someFunction(null);
```

### 실행 결과

```
10
null
10
```