---
title: "[SE-0016] UnsafePointer와 UnsafeMutablePointer를 Int 및 UInt로 변환하기 위한 이니셜라이저 추가"
date: 2025-03-09T00:30:01Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## UnsafePointer와 UnsafeMutablePointer를 Int 및 UInt로 변환하기 위한 이니셜라이저 추가

* 제안: [SE-0016](0016-initializers-for-converting-unsafe-pointers-to-ints.md)
* 작성자: [Michael Buckley](https://github.com/MichaelBuckley)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0016-adding-initializers-to-int-and-uint-to-convert-from-unsafepointer-and-unsafemutablepointer/2005)
* 버그: [SR-1115](https://bugs.swift.org/browse/SR-1115)
* 이전 버전: [1](https://github.com/swiftlang/swift-evolution/blob/ae2d7c24fff7cbdff754d9a4339e4fb02df5c690/proposals/0016-initializers-for-converting-unsafe-pointers-to-ints.md)


## 소개

사용자가 `Int`와 `UInt`로부터 `Unsafe[Mutable]Pointer`를 생성할 수 있는 것처럼, `Unsafe[Mutable]Pointer`로부터 `Int`와 `UInt`를 생성할 수 있어야 한다. 이를 통해 사용자는 `intptr_t`와 `uintptr_t` 매개변수를 사용하는 C 함수를 호출할 수 있으며, `UnsafePointer`로는 허용되지 않는 더 복잡한 포인터 연산을 수행할 수 있다.

[Swift Evolution 토론](https://forums.swift.org/t/proposal-add-initializers-for-converting-unsafepointers-to-int-and-unit/331), [리뷰](https://forums.swift.org/t/review-se-0016-adding-initializers-to-int-and-uint-to-convert-from-unsafepointer-and-unsafemutablepointer/1899)


## 배경

현재 Swift는 포인터 정렬 확인, 포인터 태깅, XOR 연산(예: XOR 연결 리스트 작업)과 같은 복잡한 포인터 연산을 수행하는 기능이 부족하다. 시스템 프로그래밍 언어로서 Swift는 이러한 문제를 기본적이고 간결하게 해결할 수 있어야 한다.

또한, 일부 C 함수는 `intptr_t`와 `uintptr_t` 타입의 인자를 받는데, Swift는 현재 이러한 함수를 직접 호출할 수 없다. 사용자는 이러한 함수 호출을 C 코드로 감싸야 한다.


## 제안하는 해결책

`Int`와 `UInt`에 `UnsafePointer`, `UnsafeMutablePointer`, `OpaquePointer`로부터 변환할 수 있는 이니셜라이저를 추가할 예정이다.

현재 이 문제를 해결할 수 있는 유일한 방법은 포인터 연산이 필요한 코드를 C로 작성하는 것이다. 이 코드를 Swift로 작성하더라도 C와 마찬가지로 안전하지 않은 작업이기 때문에 안전성 면에서는 차이가 없다. 그러나 사용자가 C 코드를 강제로 작성하지 않아도 되므로 코드가 더 깔끔해질 것이다.


## 상세 설계

초기화 메서드는 내장된 `ptrtoint_Word` 함수를 사용해 구현한다.

```swift
extension UInt {
  init<T>(bitPattern: UnsafePointer<T>) {
    self = UInt(Builtin.ptrtoint_Word(bitPattern._rawValue))
  }

  init<T>(bitPattern: UnsafeMutablePointer<T>) {
    self = UInt(Builtin.ptrtoint_Word(bitPattern._rawValue))
  }

  init(bitPattern: OpaquePointer) {
    self = UInt(Builtin.ptrtoint_Word(bitPattern._rawValue))
  }
}

extension Int {
  init<T>(bitPattern: UnsafePointer<T>) {
    self = Int(Builtin.ptrtoint_Word(bitPattern._rawValue))
  }

  init<T>(bitPattern: UnsafeMutablePointer<T>) {
    self = Int(Builtin.ptrtoint_Word(bitPattern._rawValue))
  }

  init(bitPattern: OpaquePointer) {
    self = Int(Builtin.ptrtoint_Word(bitPattern._rawValue))
  }
}
```

예를 들어, 이 초기화 메서드를 사용하면 Swift에서 XOR 연결 리스트의 다음 주소를 가져올 수 있다.

```swift
struct XORLinkedList<T> {
  let address: UnsafePointer<T>

  ...

  func successor(_ predecessor: XORLinkedList<T>) -> XORLinkedList<T> {
    let next = UInt(bitPattern: address) ^ UInt(bitPattern: predecessor.address)
    return XorLinkedList(UnsafePointer<T>(bitPattern: next))
  }
}
```


## 기존 코드에 미치는 영향

기존 코드에는 아무런 영향이 없다.


## 고려한 대안들

세 가지 대안을 검토했다.

첫 번째 대안은 `Unsafe[Mutable]Pointer`에 `intValue` 함수를 추가하는 것이다. 이 대안은 타입 변환을 가능한 경우 초기화 메서드로 구현하는 것이 더 바람직하다는 이유로 기각했다.

두 번째 대안은 `Unsafe[Mutable]Pointer`에 필요한 포인터 연산 사례를 처리하는 함수를 추가하는 것이다. 이 대안은 모든 포인터 연산 사용 사례를 예측하고 해당 함수를 작성해야 하는 불가능한 작업이거나, `Unsafe[Mutable]Pointer`에 모든 산술 및 비트 연산자를 추가해야 하는 문제가 있었다. 일부 연산은 부호 있는 정수에서만 정의되고, 다른 연산은 부호 없는 정수에서만 정의되기 때문에 `Unsafe[Mutable]Pointer`를 부호 있는 버전과 부호 없는 버전으로 나눠야 했다. 이는 포인터 연산이 필요 없는 사용자에게 불필요한 복잡성을 초래할 것이다. 또한, 이러한 연산을 구현할 때 포인터를 정수로 변환한 후 단일 연산을 수행하고 다시 포인터로 변환하는 과정이 필요하다. 연산을 연쇄적으로 수행할 경우 불필요한 변환이 많이 발생할 것이다.

마지막 대안은 이러한 초기화 메서드를 포기하고 사용자가 모든 복잡한 포인터 코드를 C로 작성하도록 강제하는 것이다. 이 대안은 Swift를 시스템 프로그래밍 언어로 사용하기에 적합하지 않게 만든다는 이유로 기각했다.


## 리비전 1에서 변경된 사항

- 초안 승인 후 제안서를 수정하여 `OpaquePointer`를 포함했다. 원래는 `UnsafePointer`와 `UnsafeMutablePointer`만 포함되어 있었다.




