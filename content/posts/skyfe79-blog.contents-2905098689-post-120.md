---
title: "[SE-0011] typealias 키워드를 associatedtype으로 대체하여 연관 타입 선언하기"
date: 2025-03-09T00:21:30Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## `typealias` 키워드를 `associatedtype`으로 대체하여 연관 타입 선언하기

* 제안: [SE-0011](0011-replace-typealias-associated.md)
* 작성자: [Loïc Lecrenier](https://github.com/loiclec)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **구현 완료 (Swift 2.2)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0011-replace-typealias-keyword-with-associatedtype-for-associated-type-declarations/990)
* 버그: [SR-511](https://bugs.swift.org/browse/SR-511)


## 소개

현재 `typealias` 키워드는 두 가지 타입을 선언하는 데 사용된다:

1. 타입 별칭 (기존 타입의 대체 이름)
2. 연관 타입 (프로토콜의 일부로 사용되는 타입에 대한 플레이스홀더 이름)

이 두 가지 선언은 서로 다르며, 별도의 키워드를 사용해야 한다. 이렇게 하면 둘의 차이를 명확히 강조할 수 있고, 연관 타입 사용에 대한 혼란을 줄일 수 있다.

제안하는 새로운 키워드는 `associatedtype`이다.

[리뷰 스레드](https://forums.swift.org/t/review-replace-typealias-keyword-with-associatedtype-for-associated-type-declarations/880)


## 동기

프로토콜에서 `typealias`를 연관 타입 선언에 재사용하는 것은 여러 면에서 혼란을 초래한다.

1. 프로토콜 내부의 `typealias`가 다른 곳에서와는 다른 의미를 가진다는 점이 명확하지 않다.
2. 연관 타입의 존재를 초보자에게 숨기기 때문에, 오해를 일으킬 수 있는 코드를 작성할 가능성이 높아진다.
3. 프로토콜 내부에서 구체적인 타입 별칭이 금지된다는 점이 명확하지 않다.

특히, **2 + 3**의 경우 프로그래머가 다음과 같은 코드를 작성할 수 있다.

```swift
protocol Prot {
    typealias Container : SequenceType
    typealias Element = Container.Generator.Element
}
```

이때 `Element`가 `Container.Generator.Element`에 대한 타입 별칭이 아니라, `Container.Generator.Element`를 기본값으로 하는 새로운 연관 타입이라는 사실을 깨닫지 못할 수 있다.

반면, 아래 코드는

```swift
protocol Prot {
    typealias Container : SequenceType
}
extension Prot {
    typealias Element = Container.Generator.Element
}
```

`Element`를 `Container.Generator.Element`에 대한 타입 별칭으로 선언한다.

이러한 언어적 세부 사항은 현재 주의 깊게 고려해야만 이해할 수 있다.


## 제안하는 해결책

연관 타입을 선언할 때 `typealias` 키워드 대신 `associatedtype`을 사용한다.

이 방법은 앞서 언급한 문제를 해결한다:

1. `typealias`는 이제 타입 별칭 선언에만 사용할 수 있다.
2. 초보자는 프로토콜을 생성할 때 연관 타입에 대해 배워야 한다.
3. 누군가 프로토콜 내부에서 타입 별칭을 생성하려고 하면 오류 메시지를 표시할 수 있다.

이 방법은 이전 코드 예제에서 보여준 혼란을 제거한다.

```swift
protocol Prot {
    associatedtype Container : SequenceType
    typealias Element = Container.Generator.Element // 오류: 프로토콜 내부에서 타입 별칭을 선언할 수 없음, 대신 프로토콜 확장을 사용하세요
}
```

```swift
protocol Prot {
    associatedtype Container : SequenceType
}
extension Prot {
    typealias Element = Container.Generator.Element
}
```

고려된 대체 키워드: `type`, `associated`, `requiredtype`, `placeholdertype`, …


## 제안하는 접근 방식

연관 타입을 선언하기 위해, Swift 2.2에서 `associatedtype`을 추가하고 `typealias`를 더 이상 사용하지 않도록(deprecate) 하며, Swift 3에서는 `typealias`를 완전히 제거하는 것을 제안한다.


## 기존 코드에 미치는 영향

`associatedtype`으로의 전환은 단순히 하나의 키워드를 다른 키워드로 대체하는 것이기 때문에, 기존 코드를 손상시키지 않고도 쉽게 자동화할 수 있다.


## 메일링 리스트

- [원본](https://forums.swift.org/t/introduce-associated-type-keyword/201)
- [대체 키워드](https://forums.swift.org/t/se-0011-re-considering-the-replacement-keyword-for-typealias/669)




