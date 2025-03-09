---
title: "[SE-0013] final 아닌 Super 메서드의 부분 적용 제거(Rejected)"
date: 2025-03-09T00:25:27Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## final 아닌 Super 메서드의 부분 적용 제거 (Swift 2.2)

* 제안: [SE-0013](0013-remove-partial-application-super.md)
* 작성자: [Ashley Garland](https://github.com/bitjammer)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **거부됨**
* 결정 노트: [Rationale](https://forums.swift.org/t/rejected-se-0013-remove-partial-application-of-non-final-super-methods/1157)


## 소개

Swift 2.2 이전에는 `super.foo()`와 같은 슈퍼클래스 메서드 호출이 정적 디스패치 방식으로 처리됐다. 이는 함수에 대한 참조를 기록하고, 맹글링된 이름을 통해 직접 호출하는 방식이었다. Swift 2.2부터는 `super`를 통해 호출된 클래스 메서드가 동적 디스패치를 사용한다. 즉, 런타임에 슈퍼클래스의 가상 테이블(vtable)에서 메서드를 조회한다. 그러나 메서드가 `final`로 표시된 경우, 이를 오버라이드할 수 없기 때문에 기존의 정적 디스패치 방식을 그대로 사용한다.

커링(Currying)을 지원하는 메커니즘은 다양한 언커링(uncurrying) 수준에서 함수를 호출할 수 있도록 썽크(thunk)를 생성해야 한다. Swift 3.0에서는 커링이 제거될 예정이므로, 이러한 메커니즘에 더 많은 엔지니어링을 투자하기보다는 `self` 매개변수가 암시적으로 캡처되는 경우를 제외하고, `super`를 통한 비-`final` 메서드의 부분 적용을 허용하지 않는 것을 제안한다.

[Swift Evolution 논의 스레드](https://forums.swift.org/t/swift-2-2-removing-partial-application-of-super-method-calls/264)


## 동기

이번 변경의 동기는 부분적으로 구현상의 고려사항에서 비롯된다. 커리(Curry) 썽크(Thunk) 메커니즘을 위한 구조는 최종 함수 호출이 무엇인지에 대해 많은 가정을 하고 있다. 예를 들어, 정적 `function_ref`를 적용하거나 `class_method`를 통해 동적 디스패치를 수행하는 경우가 있다. 이는 `doFoo(self.foo)`와 같은 코드에서 유래한다(`super` 대신 `self`를 사용). 상당한 리팩토링으로 인한 회귀(regression) 위험을 감수하기보다는, Swift 3.0에서 제거된 커링의 일부만을 도입하는 것이 더 나은 선택이다.


## 상세 설계

설계와 구현 측면에서 이 변경 사항은 간단하다. 의미 분석 단계에서 호출 표현식에 대해 다음 검사를 수행한다: 만약 호출 표현식이 `super`를 기반으로 하고, 참조된 함수가 `final`이 아니며, 애플리케이션이 모든 매개변수를 충족하지 않는다면, 에러 진단을 발생시킨다.


### 예제 코드

#### 잘못된 사용: 비종료 메서드의 부분 적용

```swift
func doFoo(f: () -> ()) {
  f()
}

class Base {
  func foo()() {}
}

class Derived : Base {
  override func foo()() {
    doFoo(super.foo()) // 잘못된 사용 - 두 번째 적용이 이루어지지 않음.
  }
}
```


#### OK: 최종 메서드의 부분 적용

이 경우는 안전하다. 새로운 동적 슈퍼 디스패치 메커니즘이 최종 메서드에는 적용되지 않기 때문이다. 최종 메서드는 원래의 정적 함수 참조로 돌아간다. 어떤 클래스도 원래 구현을 오버라이드할 수 없기 때문이다.

```swift
func doFoo(f: () -> ()) {
  f()
}

class Base {
  final func foo()() {}
}

class Derived : Base {
  func bar() {
    doFoo(super.foo()) // OK - 메서드가 최종 메서드임.
  }
}
```

이 변경 사항의 구현은 [apple/swift/remove-partial-super]에서 확인할 수 있다.


#### OK: 암시적 self를 사용한 부분 적용

이 변경 사항에서도 암시적 self 매개변수의 부분 적용은 여전히 허용된다. `super.foo`를 전달할 때, 사실상 메서드를 부분 적용한 것이다. 즉, 모든 Swift 메서드 호출에 존재하는 `self` 인자를 캡처한 것이다. 이는 안전하다. SILGen에서 명시적인 thunk를 생성할 필요가 없기 때문이다. `partial_apply` 명령어는 추가적인 SIL 코드 없이 클로저를 생성한다.

```swift

func doFoo(f: () -> ()) {
  f()
}

class Base {
  func foo() {}
}

class Derived : Base {
  func bar() {
    doFoo(super.foo) // OK - self만 부분 적용
  }
}
```


## 기존 코드에 미치는 영향

커링을 완전히 제거하기로 결정했으므로, 이 변경이 영향을 미치는 코드는 전체 중 일부분에 불과하다. 일반적으로 `super`를 호출하는 경우는 위임을 위한 경우가 대부분이며, 이때는 모든 인자가 이미 존재하는 것이 일반적이다.


## 고려했던 대안들

유일한 대안은 슈퍼 메서드 디스패치를 thunk 생성 과정에 통합하는 것이다. 이 방법은 SILGen, 심볼 맹글링, IRGen에 깊은 변경이 필요하다. 이렇게 포괄적인 변경을 하면 Swift로 작성된 코드에서 소스 변경 없이 동적 슈퍼 디스패치를 사용할 수 있지만, 현재 제안하는 방식이 합리적인 절충안이라고 판단했다.

[apple/swift/remove-partial-super]: https://github.com/apple/swift/tree/remove-partial-super




