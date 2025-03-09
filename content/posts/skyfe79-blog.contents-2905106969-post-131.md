---
title: "[SE-0022] 메서드의 Objective-C 셀렉터 참조하기"
date: 2025-03-09T00:40:52Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 메서드의 Objective-C 셀렉터 참조하기

* 제안: [SE-0022](0022-objc-selectors.md)
* 작성자: [Doug Gregor](https://github.com/DougGregor)
* 리뷰 관리자: [Joe Groff](https://github.com/jckarter)
* 상태: **구현 완료 (Swift 2.2)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0022-referencing-the-objective-c-selector-of-a-method/1194)
* 구현: [apple/swift#1170](https://github.com/apple/swift/pull/1170)


## 소개

Swift 2에서는 Objective-C 셀럭터를 `Selector` 타입 컨텍스트 내에서 문자열 리터럴로 작성했다. 예를 들어, `"insertSubview:aboveSubview:"`와 같은 방식이다. 이 방식은 오류가 발생하기 쉬운 문제가 있다. 이 제안은 이러한 문제를 해결하기 위해 Swift 메서드 이름을 참조하는 `Selector` 초기화 구문을 도입하려고 한다.

Swift-evolution 관련 스레드: [여기](https://forums.swift.org/t/proposal-draft-referencing-the-objective-c-selector-of-a-method/1022), [리뷰](https://forums.swift.org/t/review-se-0022-referencing-the-objective-c-selector-of-a-method/1118), [승인 후 수정](https://forums.swift.org/t/amendment-se-0022-referencing-the-objective-c-selector-of-a-method/2737)


## 동기

셀럭터 이름으로 문자열 리터럴을 사용하는 방식은 오류가 발생하기 쉽다. 문자열이 올바른 셀럭터 형식인지 확인할 방법이 없을 뿐만 아니라, 알려진 메서드나 의도한 클래스의 메서드를 참조하는지도 알 수 없다. 또한, [Objective-C API의 자동 이름 변경](0005-objective-c-name-translation.md) 작업을 고려할 때, Swift 이름과 Objective-C 셀럭터 간의 연결 관계가 명확하지 않다. 메서드의 Swift 이름을 기반으로 명시적인 "셀럭터 생성" 구문을 제공하면, 개발자가 실제로 사용되는 Objective-C 셀럭터에 대해 고민할 필요가 없어진다.


## 제안하는 해결책

새로운 표현식 `#selector`를 도입해 메서드 참조를 통해 셀렉터를 생성할 수 있도록 한다. 예를 들어:

```swift
control.sendAction(#selector(MyApplication.doSomething), to: target, forEvent: event)
```

여기서 "doSomething"은 MyApplication의 메서드이며, Objective-C에서는 완전히 다른 이름을 가질 수도 있다:

```swift
extension MyApplication {
  @objc(jumpUpAndDown:)
  func doSomething(sender: AnyObject?) { … }
}
```

Swift 메서드 이름을 지정하고 `#selector` 표현식이 Objective-C 셀렉터를 자동으로 생성하도록 함으로써, 개발자가 수동으로 이름을 변환할 필요가 없어지고, 메서드가 존재하며 Objective-C에 노출되었는지 정적으로 확인할 수 있다.

이 제안은 [인자 레이블을 사용한 함수 이름 지정 제안](https://forums.swift.org/t/proposal-draft-2-naming-functions-with-argument-labels/1018)과 함께 사용할 수 있다. 이를 통해 메서드와 인자 레이블을 함께 지정할 수 있다:

	let sel = #selector(UIView.insertSubview(_:atIndex:)) // "insertSubview:atIndex:" 셀렉터 생성

`#selector` 문법 도입과 함께, 문자열 리터럴을 사용해 셀렉터를 생성하는 방식은 더 이상 사용하지 않도록 권장한다. Swift 2.2에서 이 방식을 더 이상 사용하지 않도록 하고, Swift 3에서는 완전히 제거하는 것이 이상적이다.

또한, 문자열 리터럴로 셀렉터를 생성하는 코드를 메서드 참조로 변환하는 특별한 마이그레이션 지원을 도입해야 한다. 이를 잘 수행하기 위해서는 컴파일러/마이그레이터가 특정 Objective-C 셀렉터를 가진 모든 선언을 찾고, 어떤 것을 참조할지 결정해야 한다. 이는 간단하지 않지만, 가능한 작업이며, 다른 참조를 문자열 기반 초기화 문법(예: `Selector("insertSubview:atIndex:")`)으로 마이그레이션할 수 있다.


## 상세 설계

`#selector` 표현식의 하위 표현식은 반드시 `objc` 메서드에 대한 참조여야 한다. 구체적으로, 입력 표현식은 Objective-C 메서드에 대한 직접적인 참조여야 하며, 괄호로 묶이거나 "as" 캐스트를 사용할 수 있다. 이는 같은 이름의 Swift 메서드를 구별하는 데 유용하다. 예를 들어, 다음은 "매우 일반적인" 예제다:

```swift
let sel = #selector(((UIView.insertSubview(_:at:)) as (UIView) -> (UIView, Int) -> Void))
```

`#selector` 내부의 표현식은 `.`으로 구분된 일련의 인스턴스 또는 클래스 멤버로 제한되며, 마지막 구성 요소는 `as`를 사용해 명확히 할 수 있다. 특히, 이는 `#selector` 내부에서 메서드 호출을 수행하는 것을 금지하며, `#selector`의 하위 표현식이 평가되지 않고 이로 인한 부작용이 발생하지 않음을 명확히 한다.

`#selector`의 전체 문법은 다음과 같다:

<pre>
selector → #selector(<i>selector-path</i>)

selector-path → <i>type-identifier</i> . <i>selector-member-path</i> <i>as-disambiguation<sub>opt</sub></i>
selector-path → <i>selector-member-path</i> <i>as-disambiguation<sub>opt</sub></i>

selector-member-path → <i>identifier</i>
selector-member-path → <i>unqualified-name</i>
selector-member-path → <i>identifier</i> . <i>selector-member-path</i>

as-disambiguation → as <i>type-identifier</i>
</pre>


## 기존 코드에 미치는 영향

`#selector` 표현식의 도입은 기존 코드에 아무런 영향을 미치지 않는다. 하지만 문자열 리터럴을 셀렉터로 사용하는 문법을 더 이상 지원하지 않으면 기존 코드가 동작하지 않는 문제가 발생한다. 이 경우 새로운 `#selector` 표현식을 사용하거나, 문자열을 이용해 `Selector`를 명시적으로 초기화하는 방식으로 코드를 마이그레이션할 수 있다.


## 고려한 대안들

주요 대안은 [타입 안전 셀럭터](https://forums.swift.org/t/type-safe-selectors/108)다. 이 방식은 `@objc` 메서드의 타입과 셀럭터를 포함하는 새로운 "셀럭터" 호출 규약을 도입한다. 타입 안전 셀럭터의 주요 장점은 타입 정보를 포함할 수 있어 타입 안전성을 높인다는 점이다. 해당 논의에 따르면, `MyClass.observeNotification`을 참조하면 다음과 같은 타입의 값을 생성한다:

```swift
@convention(selector) (MyClass) -> (NSNotification) -> Void
```

Objective-C API 중 셀럭터를 받는 API는 타입 정보를 제공할 수 있다. 예를 들어 Objective-C 속성이나 타입이 지정된 `SEL`을 위한 새로운 문법을 통해 셀럭터 기반 API의 타입 안전성을 개선할 수 있다. 개인적으로 타입 안전 셀럭터는 잘 설계된 기능이지만, 구현할 가치가 없다고 생각한다. 기존 Objective-C API와의 상호 운용성을 제외하면 클로저가 일반적으로 더 선호되기 때문이다. 이 기능을 Swift와 Clang에 추가하는 비용은 상당히 크며, 상당수의 Objective-C API에서 이를 채택해야만 가치가 있다. iOS에서는 상대적으로 적은 수의 API(100개 정도)가 해당되며, 이 중 많은 API가 이미 블록/클로저 기반의 변형을 제공하고 있어 선호된다. 따라서 이 제안에서는 더 간단한 기능을 구현하는 것이 더 복잡하지만 타입 안전성이 높은 대안보다 적합하다.

문법적으로 `@selector(method reference)`는 Objective-C와 더 가깝게 일치하지만, Swift에서는 `@`가 항상 속성을 의미하기 때문에 적합하지 않다.

이 제안의 초기 버전은 마법 같은 `Selector` 초기화자를 사용하는 것을 제안했다. 예를 들어:

```swift
let sel = Selector(((UIView.insertSubview(_:at:)) as (UIView) -> (UIView, Int) -
```

그러나 이 구문이 마법처럼 보이고 인스턴스 생성처럼 보이지만 실제로 Swift에서 초기화자로 표현할 수 없다는 점, 그리고 `#`이 이미 특수 표현을 나타내는 데 사용된다는 점(예: `#available`) 때문에 공개 검토가 끝난 후 `#selector` 구문으로 변경되었다.


## 변경 이력

- 2016-05-20: 제안이 승인된 후 `#selector` 내부의 하위 표현식 구문을 제한하기 위해 수정됨. 특히 메서드 호출을 허용하지 않도록 변경. 원래는 유효한 Swift 표현식 모두를 지원했음.




