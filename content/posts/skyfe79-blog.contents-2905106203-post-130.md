---
title: "[SE-0021] 인자 라벨을 사용한 함수 이름 지정"
date: 2025-03-09T00:38:47Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 인자 라벨을 사용한 함수 이름 지정

* 제안: [SE-0021](0021-generalized-naming.md)
* 작성자: [Doug Gregor](https://github.com/DougGregor)
* 리뷰 관리자: [Joe Groff](https://github.com/jckarter)
* 상태: **구현 완료 (Swift 2.2)**
* 결정 노트: [Rationale](https://forums.swift.org/t/review-naming-functions-with-argument-labels/1046/11)
* 구현: [apple/swift@ecfde0e](https://github.com/apple/swift/commit/ecfde0e71c61184989fde0f93f8d6b7f5375b99a)


## 소개

Swift는 함수 타입의 값에 모든 함수(또는 메서드)를 넣을 수 있는 퍼스트클래스 함수를 지원한다. 하지만 함수 이름을 지정할 때는 인자 레이블 없이 기본 이름만 제공할 수 있다(예: `insertSubview`). 오버로드된 함수의 경우, 타입 정보를 기반으로 구분해야 하기 때문에 불편하고 장황해진다. 이 제안은 함수를 참조할 때 인자 레이블을 제공할 수 있게 함으로써 대부분의 경우 타입 컨텍스트를 제공할 필요를 없앤다.

Swift-evolution 스레드: 이 제안의 초안은 [여기](https://forums.swift.org/t/proposal-draft-generalized-naming-for-any-function/787)에서 논의되었다. 이 초안은 게터/세터에 대한 지원도 포함했는데, 이는 Michael Henson이 [여기](https://forums.swift.org/t/proposal-expose-getter-setters-in-the-same-way-as-regular-methods/501)와 [여기](https://forums.swift.org/t/proposal-expose-getter-setters-in-the-same-way-as-regular-methods/501/5)에서 별도로 제안한 내용이었다. Joe Groff는 [이 글](https://forums.swift.org/t/proposal-draft-generalized-naming-for-any-function/787/4)에서 게터/세터를 다루는 데는 렌즈가 더 나은 접근 방식이라고 설득했기 때문에, 이 제안의 현재 버전에서는 게터/세터 관련 내용을 제외했다.


## 동기

Swift에서는 여러 함수나 메서드가 동일한 "기본 이름"을 공유하지만, 매개변수 레이블로 구분되는 경우가 흔하다. 예를 들어, `UIView`에는 `insertSubview`라는 동일한 기본 이름을 가진 세 가지 메서드가 있다:

```swift
extension UIView {
  func insertSubview(view: UIView, at index: Int)
  func insertSubview(view: UIView, aboveSubview siblingSubview: UIView)
  func insertSubview(view: UIView, belowSubview siblingSubview: UIView)
}
```

이 메서드를 호출할 때는 인자 레이블이 각 메서드를 구분한다. 예를 들면 다음과 같다:

```swift
someView.insertSubview(view, at: 3)
someView.insertSubview(view, aboveSubview: otherView)
someView.insertSubview(view, belowSubview: otherView)
```

그러나 함수 값을 생성하기 위해 함수를 참조할 때는 레이블을 제공할 수 없다:

```swift
let fn = someView.insertSubview // 모호함: 세 메서드 중 어느 것인지 알 수 없음
```

어떤 경우에는 타입 어노테이션을 사용해 모호함을 해결할 수 있다:

```swift
let fn: (UIView, Int) = someView.insertSubview    // 정상: insertSubview(_:at:) 사용
let fn: (UIView, UIView) = someView.insertSubview // 오류: 여전히 모호함!
```

후자의 경우를 해결하려면 클로저를 생성해야 한다:

```swift
let fn: (UIView, UIView) = { view, otherView in
  button.insertSubview(view, aboveSubview: otherView)
}
```

이는 상당히 번거로운 작업이다.

추가적인 동기로, Swift는 주어진 메서드에 대한 Objective-C 셀렉터를 요청할 수 있는 방법이 필요하다 (문자열 리터럴을 작성하는 대신). 이러한 연산의 인자는 메서드에 대한 참조가 될 가능성이 높으며, 이는 게터와 세터를 포함한 모든 메서드를 명시적으로 참조할 수 있는 기능의 이점을 누릴 수 있다.


## 제안하는 해결책

Swift에서 함수 이름을 지정할 때 복합 이름(예: `insertSubview(_:aboveSubview:)`)을 사용할 수 있도록 기능을 확장할 것을 제안한다. 구체적으로 다음과 같은 문법을 허용한다.

```swift
let fn = someView.insertSubview(_:at:)
let fn1 = someView.insertSubview(_:aboveSubview:)
```

이와 동일한 문법은 초기화 메서드에도 적용할 수 있다. 예를 들어:

```swift
let buttonFactory = UIButton.init(type:)
```

"주어진 메서드에 대한 Objective-C 셀렉터를 생성하는 작업"은 별도의 제안에서 다룰 예정이다. 그러나 여기서는 제안하는 문법을 활용하는 한 가지 가능성을 보여준다.

```swift
let getter = Selector(NSDictionary.insertSubview(_:aboveSubview:)) // insertSubview:aboveSubview:를 생성한다.
```


## 상세 설계

문법적으로 *primary-expression* 문법은 다음과 같이 변경된다:

    primary-expression -> identifier generic-argument-clause[opt]

에서

    primary-expression -> unqualified-name generic-argument-clause[opt]

    unqualified-name -> identifier
                      | identifier '(' ((identifier | '_') ':')+ ')'

로 변경된다.

괄호 안에서 "+"의 사용은 중요하다. 이는 다음과 같은 경우를 명확히 구분하기 때문이다:

```swift
f()
```

위 코드는 `f`를 호출하는 것으로 해석되며, 인자가 없는 `f`를 참조하는 것으로 해석되지 않는다. 인자가 없는 함수 참조는 여전히 컨텍스트 타입 정보를 통해 명확히 구분해야 한다.

함수 이름을 참조할 때는 선언에 있는 모든 인자를 포함해야 한다. 기본값이 지정된 매개변수나 가변 매개변수의 인자를 생략할 수 없다. 예를 들어:

```swift
func foo(x x: Int, y: Int = 7, strings: String...) { ... }

let fn1 = foo(x:y:strings:) // 정상
let fn2 = foo(x:) // 오류: 'foo(x:)'라는 이름의 함수가 없음
```


## 기존 코드에 미치는 영향

이 기능은 순수하게 추가적인 기능으로, 기존 코드에 아무런 영향을 미치지 않는다.


## 고려했던 대안들

* Joe Groff는 속성을 직접 조작하려는 의도라면, 수동으로 getter/setter 함수를 가져오는 것보다 *lens*가 더 나은 해결책이라고 [지적](https://forums.swift.org/t/proposal-expose-getter-setters-in-the-same-way-as-regular-methods/501/3)했다.

* Bartlomiej Cichosz는 `_`를 플레이스홀더로 사용하는 일반적인 부분 적용 문법을 [제안](https://forums.swift.org/t/proposal-draft-generalized-naming-for-any-function/787/6)했다. 예를 들어:

```swift
aGameView.insertSubview(_, aboveSubview: playingSurfaceView)
```

  모든 인자가 `_`일 경우, 이 방식은 어떤 메서드든 이름을 지정할 수 있는 기능을 제공한다:

```swift
aGameView.insertSubview(_, aboveSubview: _)
```

  하지만 나는 Swift에서 이렇게 일반적인 부분 적용 문법이 필요하지 않다고 판단했다. $ 이름을 사용한 클로저가 거의 동일한 간결함을 제공하면서도, `_` 플레이스홀더가 부분 적용 함수의 인자에 어떻게 매핑되는지에 대한 의문을 없애준다:

```swift
{ aGameView.insertSubview($0, aboveSubview: playingSurfaceView) }
```

* 이름에서 언더스코어를 생략할 수도 있었다. 예를 들어:

```swift
let fn1 = someView.insertSubview(:aboveSubview:)
```

  하지만 이 방식은 첫 번째 인자 레이블이 없는 메서드와 첫 번째 인자 레이블이 있는 메서드를 시각적으로 구분하기 어렵게 만든다. 예를 들어 `f(x:)`와 `f(:x:)`를 비교할 때 차이를 명확히 알기 힘들다. 또한, 시스템의 다른 부분(예: Clang의 `swift_name` 속성)에서는 빈 인자 레이블을 언더스코어로 표기하므로 일관성을 유지해야 한다.




