---
title: "[SE-0017] Unmanaged를 UnsafePointer로 변경하기"
date: 2025-03-09T00:30:59Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## `Unmanaged`를 `UnsafePointer`로 변경하기

* 제안: [SE-0017](0017-convert-unmanaged-to-use-unsafepointer.md)
* 작성자: [Jacob Bandes-Storch](https://github.com/jtbandes)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0017-change-unmanaged-to-use-unsafepointer/2461)
* 버그: [SR-1485](https://bugs.swift.org/browse/SR-1485)


## 소개

Swift 표준 라이브러리의 [`Unmanaged<Instance>` 구조체](https://github.com/apple/swift/blob/master/stdlib/public/core/Unmanaged.swift)는 ARC에 참여하지 않는 타입 안전 객체 래퍼를 제공한다. 이를 통해 사용자가 수동으로 retain/release 호출을 할 수 있다.

관련 자료:  
[Swift Evolution 토론](https://forums.swift.org/t/unmanaged-and-copaquepointer-vs-unsafe-mutable-pointer/295),  
[리팩토링 제안 토론](https://forums.swift.org/t/rfc-proposed-rewrite-of-unmanaged-t/612),  
[리뷰](https://forums.swift.org/t/review-se-0017-change-unmanaged-to-use-unsafepointer/2380)


## 동기

Unmanaged 타입으로 변환하거나 Unmanaged 타입에서 변환하기 위해 다음과 같은 메서드를 제공한다:

```swift
static func fromOpaque(value: COpaquePointer) -> Unmanaged<Instance>
func toOpaque() -> COpaquePointer
```

그러나 `void *`나 `const void *`를 받는 C API는 Swift에서 `COpaquePointer` 대신 `UnsafePointer<Void>`나 `UnsafeMutablePointer<Void>`로 노출된다. 실제로 사용자는 `UnsafePointer` → `COpaquePointer` → `Unmanaged`로 변환해야 하며, 이로 인해 다음과 같은 복잡한 코드가 발생한다:

```swift
someFunction(context: UnsafeMutablePointer(Unmanaged.passUnretained(self).toOpaque()))

info.retain = { Unmanaged<AnyObject>.fromOpaque(COpaquePointer($0)).retain() }
info.copyDescription = {
    Unmanaged.passRetained(CFCopyDescription(Unmanaged.fromOpaque(COpaquePointer($0)).takeUnretainedValue()))
}
```


## 제안하는 해결책

`Unmanaged` API에서 `COpaquePointer` 사용을 `UnsafePointer<Void>`와 `UnsafeMutablePointer<Void>`로 대체한다.

영향을 받는 함수는 `fromOpaque()`와 `toOpaque()`다. [현재 구현](https://github.com/apple/swift/blob/0287ac7fd94af0fb860b5444e1bd26faded88e39/stdlib/public/core/Unmanaged.swift#L32-L54)에서 매우 사소한 수정만 필요하다:

```swift
@_transparent
@warn_unused_result
public static func fromOpaque(value: UnsafePointer<Void>) -> Unmanaged {
    // 널 포인터 검사는 디버그 검사로, 특정 잘못된 포인터 값을 방어한다.
    _debugPrecondition(
      value != nil,
      "널 포인터에서 Unmanaged 인스턴스를 생성하려고 시도했습니다.")

    return Unmanaged(_private: unsafeBitCast(value, Instance.self))
}

@_transparent
@warn_unused_result
public func toOpaque() -> UnsafeMutablePointer<Void> {
    return unsafeBitCast(_value, UnsafeMutablePointer<Void>.self)
}
```

`UnsafeMutablePointer` 타입의 값은 `UnsafePointer`나 `UnsafeMutablePointer`를 받는 함수에 모두 전달할 수 있다. 따라서 간단하고 사용하기 쉽게 `fromOpaque()`의 입력 타입으로 `UnsafePointer`를, `toOpaque()`의 반환 타입으로 `UnsafeMutablePointer`를 선택한다.

위 예제 사용법은 더 이상 변환을 필요로 하지 않는다:

```swift
someFunction(context: Unmanaged.passUnretained(self).toOpaque())

info.retain = { Unmanaged<AnyObject>.fromOpaque($0).retain() }
info.copyDescription = {
    Unmanaged.passRetained(CFCopyDescription(Unmanaged.fromOpaque($0).takeUnretainedValue()))
}
```


## 기존 코드에 미치는 영향

이전에 `COpaquePointer`를 사용해 `Unmanaged` API를 호출하던 코드는 `UnsafePointer`를 사용하도록 변경해야 한다. `COpaquePointer` 버전은 전환을 돕기 위해 다음과 같은 가용성 속성을 유지할 수 있다:

    @available(*, unavailable, message="fromOpaque(value: UnsafeMutablePointer<Void>)를 대신 사용하세요")
    @available(*, unavailable, message="toOpaque() -> UnsafePointer<Void>를 대신 사용하세요")

[`COpaquePointer`를 사용하는 코드](https://github.com/search?q=COpaquePointer&type=Code)는 이 타입에 크게 의존하지 않는 것으로 보이며, 이 변경으로 인해 큰 영향을 받지 않을 것이다.


## 고려한 대안들

- 아무런 변경도 하지 않는 방법이 있다. 하지만 [swift-evolution에서 언급된 바](https://forums.swift.org/t/unmanaged-and-copaquepointer-vs-unsafe-mutable-pointer/295/3)에 따르면, `COpaquePointer`는 더 이상 필요 없는 잔재로 여겨지며, C API와의 더 나은 연결이 필요하다. 따라서 이 방향으로 나아가는 것이 바람직하다.




