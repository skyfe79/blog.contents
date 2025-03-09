---
title: "[SE-0024] 옵셔널 값 설정자 ??= (Rejected)"
date: 2025-03-09T00:44:42Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 옵셔널 값 설정자 `??=`

* 제안: [SE-0024](0024-optional-value-setter.md)
* 작성자: [James Campbell](https://github.com/jcampbell05)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **거부됨**
* 결정 노트: [이유](https://forums.swift.org/t/rejected-se-0024-optional-value-setter/1528)


## 소개

새로운 연산자인 "옵셔널 값 설정자(Optional Value Setter)"를 소개한다. 이 연산자를 통해 옵셔널 값을 설정하면, 기존에 값이 없는 경우에만 새로운 값이 설정된다.

Swift-evolution 스레드: [해당 제안에 대한 토론 스레드 링크](https://forums.swift.org/t/optional-setting/553)


## 동기

`??` 연산자는 긴 변수명을 다룰 때 도움이 되지 않는 경우가 있다. 예를 들어 `really.long.lvalue[expression] = really.long.lvalue[expression] ?? ""`와 같은 코드는 길고 복잡해진다. 또한 Ruby 같은 언어에서는 파이프 연산자 `really.long.lvalue[expression] ||= ""`를 사용해 동일한 기능을 간결하게 표현할 수 있으며, 이는 매우 인기 있는 문법이다. 이를 통해 해당 언어 출신의 개발자들이 더 쉽게 접근할 수 있다.

간결성과 명확성을 고려할 때, 이 기능을 Swift에 추가하는 것이 좋다고 생각한다. 이를 통해 이전의 긴 코드를

```swift
really.long.lvalue[expression] = really.long.lvalue[expression] ?? ""
```

에서

```swift
really.long.lvalue[expression] ??= ""
```

로 간단하게 줄일 수 있다.


## 제안하는 해결책

이 해결책에서는 이미 값이 있는 경우(즉, .Some) 옵셔널 값이 설정되지 않는다. 이 작업이 발생할 때만 willSet과 didSet이 호출되는 것이 이상적이다.

```swift
var itemsA:[Item]? = nil
var itemsB:[Item]? = [Item()]

itemsA ??= [] // itemsA는 값이 .None이므로 설정됨
itemsB ??= [] // itemsB는 값이 .Some이므로 변경되지 않음
```


## 기존 코드에 미치는 영향

이 기능은 엄격하게 추가적이고 선택적인 기능이므로 기존 코드에 영향을 미치지 않는다. 오히려 앞으로는 코드를 더 간결하게 작성할 수 있게 해준다.


## 고려한 대안들

다른 문법으로 `?=`가 있었지만, `a = a ?? ""`에서 사용된 `??`의 관례와 일치하지 않는다고 판단했다.




