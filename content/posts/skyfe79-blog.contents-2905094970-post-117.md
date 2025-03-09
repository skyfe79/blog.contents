---
title: "[SE-0008] 옵셔널 시퀀스에 대한 Lazy flatMap 추가"
date: 2025-03-09T00:16:47Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 옵셔널 시퀀스에 대한 Lazy flatMap 추가

* 제안: [SE-0008](0008-lazy-flatmap-for-optionals.md)
* 작성자: [Oisin Kidney](https://github.com/oisdk)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **구현됨 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0008-add-a-lazy-flatmap-for-sequences-of-optionals/748)
* 버그: [SR-361](https://bugs.swift.org/browse/SR-361)


## 소개 ##

현재 Swift 표준 라이브러리에는 두 가지 버전의 `flatMap`이 존재한다. 하나는 변환 후 시퀀스의 시퀀스를 평탄화하는 기능을 수행한다:

```swift
[1, 2, 3]
  .flatMap { n in n..<5 } 
// [1, 2, 3, 4, 2, 3, 4, 3, 4]
```

다른 하나는 `Optional` 시퀀스를 평탄화하는 기능을 제공한다:

```swift
(1...10)
  .flatMap { n in n % 2 == 0 ? n/2 : nil }
// [1, 2, 3, 4, 5]
```

그러나 첫 번째 버전에 대해서만 지연 평가(lazy) 구현이 존재한다:

```swift
[1, 2, 3]
  .lazy
  .flatMap { n in n..<5 }
// LazyCollection<FlattenBidirectionalCollection<LazyMapCollection<Array<Int>, Range<Int>>>>

(1...10)
  .lazy
  .flatMap { n in n % 2 == 0 ? n/2 : nil }
// [1, 2, 3, 4, 5]
```

Swift Evolution 토론: [Lazy flatMap for Optionals](https://forums.swift.org/t/lazy-flatmap-for-optionals/127/3), [Review](https://forums.swift.org/t/review-add-a-lazy-flatmap-for-sequences-of-optionals/548)


## 동기 ##

이미 존재하는 `flatMap`이 중첩된 시퀀스에 대해 지연(lazy) 버전을 제공하는 반면, `Optional` 시퀀스에 대한 지연 버전이 없다는 것은 공백으로 보인다. 지연 시퀀스의 유용성은 잘 문서화되어 있으며, 특히 명령형 중첩 for 루프를 메서드 체인으로 리팩토링할 때 조급하게(eagerly) 처리하면 불필요한 중간 배열을 할당할 수 있다는 점에서 중요하다.


## 제안하는 접근 방식 ##

표준 라이브러리에 이미 존재하는 타입을 활용하면 `flatMap`의 기능을 `map`-`filter`-`map` 체인으로 구현할 수 있다.

```swift
extension LazySequenceType {
  
  @warn_unused_result
  public func flatMap<T>(transform: Elements.Generator.Element -> T?)
    -> LazyMapSequence<LazyFilterSequence<LazyMapSequence<Elements, T?>>, T> {
      return self
        .map(transform)
        .filter { opt in opt != nil }
        .map { notNil in notNil! }
  }
}
```


## 상세 설계 ##

`LazyCollectionType`을 위한 버전은 거의 동일하다:

```swift
extension LazyCollectionType {
  
  @warn_unused_result
  public func flatMap<T>(transform: Elements.Generator.Element -> T?)
    -> LazyMapCollection<LazyFilterCollection<LazyMapCollection<Elements, T?>>, T> {
      return self
        .map(transform)
        .filter { opt in opt != nil }
        .map { notNil in notNil! }
  }
}
```

하지만 "양방향" 버전은 이 방식으로 작성할 수 없다. `FilterBidirectionalCollection`이 존재하지 않기 때문이다.

다른 형태의 `flatMap`은 중첩된 시퀀스에 대해 `flatten` 메서드를 사용한다. 이 메서드는 `CollectionType` 형태와 `BidirectionalIndexType`을 가진 `CollectionType` 형태 모두 존재한다.

그러나 Swift의 현재 타입 시스템은 `Optional` 시퀀스에 대해 비슷한 메서드를 정의하는 것을 허용하지 않는다. 이는 `filter`에 의존해야 함을 의미하며, `filter`는 `SequenceType`과 `CollectionType` 구현만 가지고 있다.


## 기존 코드에 미치는 영향 ##

## 고려된 대안 ##

### 커스텀 구조체 ###

새로운 구조체를 추가하고 `LazySequenceType`에 메서드를 추가하는 방법도 있다:

```swift
public struct FlatMapOptionalGenerator<G: GeneratorType, Element>: GeneratorType {
  private let transform: G.Element -> Element?
  private var generator: G
  public mutating func next() -> Element? {
    while let next = generator.next() {
      if let transformed = transform(next) {
        return transformed
      }
    }
    return nil
  }
}

public struct FlatMapOptionalSequence<S: LazySequenceType, Element>: LazySequenceType {
  private let transform: S.Generator.Element -> Element?
  private let sequence: S
  public func generate() -> FlatMapOptionalGenerator<S.Generator, Element> {
    return FlatMapOptionalGenerator(transform: transform, generator: sequence.generate())
  }
}

extension LazySequenceType {
  public func flatMap<T>(transform: Generator.Element -> T?) -> FlatMapOptionalSequence<Self, T> {
    return FlatMapOptionalSequence(transform: transform, sequence: self)
  }
}
```

하지만 이 구현은 `LazyCollectionType` 버전을 포함하지 않는다. 이를 추가하고 양방향 구현을 포함하려면 표준 라이브러리에 6개의 새로운 타입(3개의 `SequenceType`, 3개의 `GeneratorType`)을 추가해야 한다.


### 새로운 Filter 구조체

표준 라이브러리에 `FilterBidirectionalCollection`을 추가하는 방안을 고려할 수 있다. 현재 이 부분은 기능상의 공백으로 볼 수 있다. 이렇게 하면 두 `flatMap` 버전이 서로 대칭적으로 동작할 수 있으며, 새로운 타입을 최소화할 수 있다.


### Optional을 SequenceType에 맞게 수정하기 ###

이 제안은 광범위하고 독립적인 주제지만, 현재 제안이 해결하려는 문제를 해결할 수 있다. 다만, `Optional`은 아마도 `BidirectionalIndexType`을 가지지 않을 것이므로, 어쨌든 `Optional`에는 양방향 버전의 `flatMap`이 존재하지 않을 것이라는 점을 염두에 두어야 한다.




