---
title: "[SE-0032] Sequence에 first(where:) 메서드 추가"
date: 2025-03-09T01:11:06Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## `Sequence`에 `first(where:)` 메서드 추가

* 제안: [SE-0032](0032-sequencetype-find.md)
* 작성자: [Lily Ballard](https://github.com/lilyball)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [근거](https://forums.swift.org/t/accepted-se-0032-add-find-method-to-sequence/2462)
* 버그: [SR-1519](https://bugs.swift.org/browse/SR-1519)
* 이전 버전: [1](https://github.com/swiftlang/swift-evolution/blob/d709546002e1636a10350d14da84eb9e554c3aac/proposals/0032-sequencetype-find.md)


## 소개

`Sequence`에 새로운 확장 메서드 `first(where:)`를 추가한다. 이 메서드는 조건에 맞는 첫 번째 엘리먼트를 반환한다.

Swift-evolution 포럼에서 이 주제에 대한 논의는 **Add find method to SequenceType**라는 제목의 제안으로 시작되었다.

Swift-evolution 스레드: [Proposal: Add function SequenceType.find()](https://forums.swift.org/t/proposal-add-function-sequencetype-find/825)

[리뷰](https://forums.swift.org/t/review-se-0032-add-find-method-to-sequencetype/2381)


## 동기

특정 조건을 만족하는 시퀀스의 첫 번째 엘리먼트를 찾는 것은 종종 유용하다. `Collection`의 경우 `index(of:)`나 `index(where:)`를 호출한 후 결과 인덱스를 `subscript`에 전달할 수 있지만, 이는 다소 번거롭다. `Sequence`의 경우, 전체 시퀀스를 필터링하고 배열을 생성하지 않는 수동 루프 외에는 이를 쉽게 수행할 방법이 없다.

사람들이 `seq.lazy.filter(predicate).first`와 같은 코드를 작성하는 것을 보았지만, 이는 실제로 지연 평가(lazy evaluation) 방식으로 동작하지 않는다. 왜냐하면 `.first`는 `Collection`에서만 사용 가능한 메서드이기 때문이다. 이는 `filter()` 호출이 `LazySequenceProtocol.filter()`가 아닌 `Sequence.filter()`로 해석되어 배열을 반환하게 된다는 것을 의미한다. 사용자들은 대체로 이를 인지하지 못하므로, 예상보다 훨씬 더 많은 작업을 수행하게 된다.


`Sequence`에 `first(where:)` 메서드를 추가해보자. 이 메서드는 조건을 받아 해당 조건을 만족하는 첫 번째 요소를 반환한다. 조건을 만족하는 요소가 없으면 `nil`을 반환한다.

```swift
extension Sequence {
    func first(where predicate: (Element) -> Bool) -> Element? {
        for element in self {
            if predicate(element) {
                return element
            }
        }
        return nil
    }
}
```

이 구현은 시퀀스를 순회하면서 주어진 조건(`predicate`)을 만족하는 첫 번째 요소를 찾는다. 조건을 만족하는 요소를 발견하면 즉시 반환하고, 전체 시퀀스를 순회해도 조건을 만족하는 요소가 없으면 `nil`을 반환한다. 이 메서드는 시퀀스에서 특정 조건을 만족하는 첫 번째 요소를 찾을 때 유용하게 사용할 수 있다.


## 상세 설계

`Sequence`에 다음 확장을 추가한다:

```swift
extension Sequence {
  /// `predicate`가 `true`를 반환하는 첫 번째 요소를 반환한다. 
  /// 해당 값이 없으면 `nil`을 반환한다.
  public func first(where predicate: @noescape (Self.Iterator.Element) throws -> Bool) rethrows -> Self.Iterator.Element? {
    for elt in self {
      if try predicate(elt) {
        return elt
      }
    }
    return nil
  }
}
```


## 기존 코드에 미치는 영향

이 기능은 순수하게 추가적인 기능이다. 기존 코드에는 아무런 영향을 미치지 않는다.

이론적으로는 `seq.filter(predicate).first`나 `seq.lazy.filter(predicate).first`를 자동으로 `seq.first(where: predicate)`로 변환할 수 있을 것이다. 하지만 기존 코드도 여전히 문제없이 컴파일될 것이다.


## 고려한 대안

없음




