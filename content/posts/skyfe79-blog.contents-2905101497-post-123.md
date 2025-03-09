---
title: "[SE-0014] AnySequence.init에 제약 추가하기"
date: 2025-03-09T00:27:39Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## `AnySequence.init`에 제약 추가하기

* 제안: [SE-0014](0014-constrained-AnySequence.md)  
* 작성자: [Max Moiseev](https://github.com/moiseev)  
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)  
* 상태: **구현 완료 (Swift 2.2)**  
* 결정 노트: [근거](https://forums.swift.org/t/accepted-se-0014-constraining-anysequence-init/924)  
* 버그: [SR-474](https://bugs.swift.org/browse/SR-474)


## 소개

`AnySequence`가 기본 시퀀스에 대한 호출을 위임할 수 있도록 하려면, 이니셜라이저에 추가적인 제약 조건이 필요하다.

[Swift Evolution 토론](https://forums.swift.org/t/restricting-anysequence-init/254)


## 동기

현재 `AnySequence`는 `SequenceType` 프로토콜 메서드 호출을 기본 시퀀스로 위임하지 않는다. 이로 인해 해당 동작이 필요한 곳에서 동적 다운캐스트가 발생한다. 예를 들어, `SequenceType.dropFirst`나 `SequenceType.prefix`의 기본 구현에서 이런 문제가 나타난다. 더 중요한 점은, 위임이 없으면 `SequenceType` 메서드의 커스텀 구현이 무시된다는 것이다.


## 제안하는 솔루션

[이 PR](https://github.com/apple/swift/pull/220)에서 구현 내용을 확인할 수 있다.

이러한 위임이 가능해지려면, `_SequenceBox`가 기본 시퀀스뿐만 아니라 관련된 `SubSequence`도 '감싸는' 기능을 지원해야 한다. 따라서 기존에 다음과 같이 선언된 코드:

```Swift
internal class _SequenceBox<S : SequenceType>
    : _AnySequenceBox<S.Generator.Element> { ... }
```

다음과 같이 변경된다:

```Swift
internal class _SequenceBox<
  S : SequenceType
  where
    S.SubSequence : SequenceType,
    S.SubSequence.Generator.Element == S.Generator.Element,
    S.SubSequence.SubSequence == S.SubSequence
> : _AnySequenceBox<S.Generator.Element> { ... }
```

이 변경으로 인해 `AnySequence.init`에도 새로운 제약 조건이 추가된다.

변경 전:

```Swift
public struct AnySequence<Element> : SequenceType {
  public init<
    S: SequenceType
    where
      S.Generator.Element == Element
  >(_ base: S) { ... }
}
```

변경 후:

```Swift
public struct AnySequence<Element> : SequenceType {
  public init<
    S: SequenceType
    where
      S.Generator.Element == Element,
      S.SubSequence : SequenceType,
      S.SubSequence.Generator.Element == Element,
      S.SubSequence.SubSequence == S.SubSequence
  >(_ base: S) { ... }
}
```

이러한 제약 조건은 사실 `SequenceType` 프로토콜 자체에 적용되어야 한다(현재는 불가능하지만). 모든 `SequenceType` 구현이 이미 이를 만족할 것으로 예상하기 때문이다. 기술적으로 `S.SubSequence.SubSequence == S.SubSequence`는 이렇게 엄격할 필요는 없으며, 동일한 요소 타입을 가진 시퀀스라면 충분하지만, 현재는 이를 표현할 수 없다.


## 기존 코드에 미치는 영향

새로운 제약 사항은 `SequenceType` 프로토콜을 준수하는 모든 내장 타입에 영향을 미치지 않는다. 이들은 기본적으로 `SubSequence.SubSequence == SubSequence`와 같은 방식으로 구성되기 때문이다. 서드파티 컬렉션의 경우 기본 `SubSequence`(즉, `Slice`)를 사용한다면 문제가 없을 것이다. 하지만 커스텀 `SubSequence`를 사용하는 경우 프로토콜 준수를 중단할 수도 있다.




