---
title: "[SE-0040] 속성 인자에서 등호를 콜론으로 대체하기"
date: 2025-03-09T01:18:13Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 속성 인자에서 등호를 콜론으로 대체하기

* 제안: [SE-0040](0040-attributecolons.md)
* 작성자: [Erica Sadun](https://github.com/erica)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [이유](https://forums.swift.org/t/accepted-se-0040-replacing-equal-signs-with-colons-for-attribute-arguments/1719)
* 구현: [apple/swift#1537](https://github.com/apple/swift/pull/1537)


## 소개

속성(attribute) 인자는 Swift 언어의 다른 인자와 다르게 동작한다. 호출 시점에서 인자 이름과 전달된 값을 구분하기 위해 콜론 대신 `=`를 사용한다. 이 제안서는 이러한 특수한 경우에 "=" 대신 ":"를 사용하여 속성을 Swift 표준 관행에 맞추는 것을 목표로 한다.

*이 논의는 Swift Evolution 메일링 리스트의 [\[Discussion\] Replacing Equal Signs with Colons For Attribute Arguments](https://forums.swift.org/t/discussion-replacing-equal-signs-with-colons-for-attribute-arguments/1459) 스레드에서 진행되었다. 이 개선 사항을 제안한 [Doug Gregor](https://github.com/DougGregor)에게 감사드린다.*


## 동기

속성(attribute)은 개발자가 선언과 타입에 동작을 제한하는 키워드를 추가할 수 있게 한다. [at-sign](http://foldoc.org/strudel) "@" 접두사로 구분되는 속성은 Swift 컴파일러에게 타입과 선언의 기능, 특성, 제약, 기대 사항을 전달한다. 대표적인 속성으로는 호출 수명을 벗어날 수 없는 매개변수를 나타내는 `@noescape`, 타입의 호출 규약이 Swift, C, 또는 Objective-C 블록 모델을 따르는지 표시하는 `@convention`, 그리고 선언이 특정 플랫폼이나 OS 버전과 호환되는지 열거하는 `@available` 등이 있다. Swift는 현재 약 12개의 고유 속성을 제공하며, 앞으로 언어 업데이트를 통해 이 목록을 확장할 가능성이 높다.

일부 속성은 인자를 받는다. 현재 `@available`, `@warn_unused_result`, `@swift3_migration` 등이 이에 해당한다. 현재 문법에서는 속성 인자 키워드와 값을 구분하기 위해 등호(=)를 사용한다.

```swift
introduced=version-number
deprecated=version-number
obsoleted=version-number
message=message
renamed=new-name
mutable_variant=method-name
```

그러나 등호를 사용하는 방식은 Swift의 다른 매개변수화 호출 패턴과 일치하지 않는다. 이 제안의 범위는 매우 작지만, 문법을 조정해 Swift 전체에 일관성을 더하는 작은 변화를 도입한다.

```swift
parameter name: parameter value
```


## 상세 설계

이 제안은 속성 인자 절을 구성할 때 사용되는 밸런스 토큰에서 `=`를 `:`로 대체하는 방안을 제시한다. 구체적인 내용은 다음과 같다:

```swift
attribute → @ attribute-name attribute-argument-clause<sub>opt</sub>
attribute-name → identifier
attribute-argument-clause → ( balanced-tokens<sub>opt<opt> )
balanced-tokens → balanced-token
balanced-tokens → balanced-token, balanced-tokens
balanced-token → attribute-argument-label : attribute argument-value
```

이 설계는 "현재 Swift 속성에서 `=`를 사용하는 모든 곳을 `:`로 대체한다"는 원칙을 따른다. 예를 들면 다음과 같다:

```swift
@available(*, unavailable, renamed: "MyRenamedProtocol")
typealias MyProtocol = MyRenamedProtocol

@warn_unused_result(mutable_variant: "sortInPlace")
public func sort() -> [Self.Generator.Element]

@available(*, deprecated, message: "it will be removed in Swift 3.  Use the 'generate()' method on the collection.")
public init(_ bounds: Range<Element>)
```


## 고려한 대안

이 제안을 받아들이지 않는 것 외에는 다른 대안이 없다.




