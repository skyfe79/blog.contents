---
title: "[SE-0010] StaticString.UnicodeScalarView 추가(Rejected)"
date: 2025-03-09T00:18:57Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## StaticString.UnicodeScalarView 추가

* 제안: [SE-0010](0010-add-staticstring-unicodescalarview.md)
* 작성자: [Lily Ballard](https://github.com/lilyball)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **거부됨**
* 결정 노트: [Rationale](https://forums.swift.org/t/rejected-se-0010-add-staticstring-unicodescalarview/1530)


## 소개

`StaticString`의 하위 문자열(substring)을 생성해도 여전히 `StaticString` 타입으로 유지할 방법이 없다. 이 기능이 필요하다.

[Swift Evolution 토론 스레드](https://forums.swift.org/t/proposal-add-staticstring-unicodescalarview/199), [리뷰](https://forums.swift.org/t/review-se-0010-add-staticstring-unicodescalarview/942)


## 동기

`StaticString`의 부분 문자열을 생성하여 `StaticString`을 기대하는 API에 전달할 수 있으면 유용할 때가 있다. 예를 들어, `__FILE__`에서 파일 이름을 추출하는 경우가 그렇다. 하지만 현재는 `StaticString`이 빈 문자열을 생성하는 기본 `init()` 이니셜라이저 외에는 새로운 인스턴스를 생성할 수 있는 방법을 제공하지 않기 때문에 이를 수행할 수 없다.


## 제안하는 솔루션

새로운 타입 `StaticString.UnicodeScalarView`를 추가하고, 이 타입이 `CollectionType`을 준수하도록 한다. 또한 `StaticString`에 새로운 프로퍼티 `unicodeScalars`를 추가한다. 추가로 `StaticString`에 두 개의 이니셜라이저를 도입한다.

```swift
init(_ unicodeScalars: UnicodeScalarView)
init(_ unicodeScalars: Slice<UnicodeScalarView>)
```

이를 통해 사용자는 유니코드 스칼라 뷰를 조작하여 원하는 슬라이스를 생성하고, 그 결과로 `StaticString`을 만들 수 있다. 이 방법은 `StaticString`을 UTF8 버퍼가 아닌 `UnicodeScalar` 시퀀스로 다룰 수 있는 편리한 방식을 제공한다는 추가적인 이점이 있다.


## 상세 설계

API는 다음과 같은 구조로 설계되었다:

```swift
extension StaticString {
  /// `self`의 값을 [유니코드 스칼라 값](http://www.unicode.org/glossary/#unicode_scalar_value)의 컬렉션으로 반환한다.
  public var unicodeScalars: UnicodeScalarView { get }

  /// 주어진 `UnicodeScalarView`에 해당하는 `StaticString`을 생성한다.
  public init(_: UnicodeScalarView)

  /// 주어진 `UnicodeScalarView` 슬라이스에 해당하는 `StaticString`을 생성한다.
  public init(_: Slice<UnicodeScalarView>)

  /// `StaticString`을 인코딩하는 [유니코드 스칼라 값](http://www.unicode.org/glossary/#unicode_scalar_value)의 컬렉션.
  public struct UnicodeScalarView : CollectionType {

    init(_: StaticString)

    /// `StaticString.UnicodeScalarView` 내의 위치를 나타낸다.
    public struct Index : BidirectionalIndexType, Comparable {
      /// `self`의 다음 값을 반환한다.
      ///
      /// - Requires: 다음 값이 표현 가능해야 한다.
      @warn_unused_result
      public func successor() -> Index

      /// `self`의 이전 값을 반환한다.
      ///
      /// - Requires: 이전 값이 표현 가능해야 한다.
      @warn_unused_result
      public func predecessor() -> Index
    }

    /// `StaticString`이 비어 있지 않다면 첫 번째 `UnicodeScalar`의 위치를 반환한다. 비어 있다면 `endIndex`와 동일하다.
    public var startIndex: Index { get }

    /// 끝을 지난 위치를 반환한다.
    ///
    /// `endIndex`는 `subscript`의 유효한 인수가 아니며, `startIndex`에서 `successor()`를 0번 이상 적용해 항상 도달할 수 있다.
    public var endIndex: Index { get }

    /// `self`가 비어 있는지 여부를 반환한다.
    public var isEmpty: Bool { get }

    public subscript(position: Index) -> UnicodeScalar { get }
  }
}
```


## 기존 코드에 미치는 영향

없음.


## 고려한 대안들

`StaticString`에 직접 `subscript(bounds: Range<Index>)`를 추가할 수도 있다. 하지만 `Index`를 정의하는 좋은 방법이 없다. 이는 `String`이 `CollectionType`을 준수하지 않는 것과 같은 이유 때문이다.

포인터에서 안전하지 않은 이니셜라이저를 노출할 수도 있다. 사용자가 `utf8Start`를 조작해 원하는 포인터를 생성할 수 있게 한다. 하지만 이는 매우 안전하지 않으며, 사용자가 `StaticString`을 받는 코드를 속여 동적 문자열을 받아들이도록 할 수 있다.




