---
title: "[SE-0035] inout 캡처를 @noescape 컨텍스트로 제한하기"
date: 2025-03-09T01:13:27Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## `inout` 캡처를 `@noescape` 컨텍스트로 제한하기

* 제안: [SE-0035](0035-limit-inout-capture.md)  
* 작성자: [Joe Groff](https://github.com/jckarter)  
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)  
* 상태: **구현됨 (Swift 3.0)**  
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0035-limiting-inout-capture-to-noescape-contexts/1544)  
* 버그: [SR-807](https://bugs.swift.org/browse/SR-807)


## 소개

Swift에서 클로저가 `inout` 매개변수를 캡처하고 해당 컨텍스트를 벗어나는 경우의 동작은 종종 혼란을 일으키는 원인이다. `@noescape` 클로저를 제외하고는 `inout` 매개변수의 암시적 캡처를 허용하지 않아야 한다.

Swift-evolution 스레드: [@noescape 클로저에서만 inout 매개변수 캡처 허용](https://forums.swift.org/t/pitch-only-allow-capture-of-inout-parameters-in-noescape-closures/1223)

[리뷰](https://forums.swift.org/t/review-se-0035-limiting-inout-capture-to-noescape-contexts/1461)


## 동기

`@noescape`가 도입되기 전에도, `inout` 매개변수와 `mutating` 메서드를 클로저에서 사용할 수 있도록 하면서도, `inout` 매개변수가 호출자에게 예상치 못한 별칭(aliasing)이나 수명 연장 문제를 일으키지 않고 로컬에서만 변이(mutate)된다는 강력한 보장을 유지하고 싶었다. Swift는 표준 라이브러리의 컬렉션 연산에서부터 `@autoclosure` 기능을 통해 `&&`와 `||` 같은 연산자 및 어설션(assertion)에 이르기까지 클로저를 광범위하게 사용한다. 따라서 `inout` 매개변수를 전혀 캡처할 수 없다면 매우 제한적일 것이다. Dave Abrahams는 현재의 캡처 의미론을 타협점으로 설계했다: `inout` 매개변수는 **섀도우 복사본(shadow copy)**으로 캡처되며, 이 복사본은 호출자가 반환될 때 원래 인자에 다시 기록된다. 이 방식은 클로저가 호출될 때 `inout` 매개변수가 활성 상태일 때 예상된 의미론(semantics)으로 캡처되고 변이될 수 있게 한다:

```swift
func captureAndCall(inout x: Int) {
  let closure = { x += 1 }
  closure()
}
var x = 22
captureAndCall(&x)
print(x) // => 23
```

그러나 클로저가 탈출(escape)할 때는 *섀도우 복사본*이 원래 인자와 독립적으로 유지되기 때문에 직관적이지 않은 결과가 발생한다:

```swift
func captureAndEscape(inout x: Int) -> () -> Void {
  let closure = { x += 1 }
  return closure
}

var x = 22
let closure = captureAndEscape(&x)
print(x) // => 22
closure()
print("still \(x)") // => still 22
```

이 변경 사항은 지속적으로 혼란과 버그 리포트의 원인이 되었으며, 최근 David Ungar의 IBM Swift 블로그 포스트 ["Seven Swift Snares & How to Avoid Them"](https://developer.ibm.com/swift/2016/01/27/seven-swift-snares-how-to-avoid-them/)에서도 언급되었다. 이는 이 주제에 대한 오랜 불만 중 하나이다.


## 제안하는 해결책

`inout` 매개변수를 암묵적으로 탈출 가능한 클로저에 캡처하는 것을 오류로 처리하도록 제안한다. Swift 1.2에서 명시적인 `@noescape` 어노테이션을 추가했고, 이후 표준 라이브러리 전체에 적절히 적용해 왔다. 이로 인해 기존의 타협은 더 이상 유용성을 잃었고 혼란의 원인이 되었다.


## 상세 설계

`mutating` 메서드 내에서 `inout` 매개변수(예: `self`)를 에스케이프 가능한 클로저 리터럴에서 암시적으로 캡처하면 오류가 발생한다. 단, 명시적으로 캡처하여 불변으로 만든 경우는 예외다.

```swift
func escape(f: () -> ()) {}
func noEscape(@noescape f: () -> ()) {}

func example(inout x: Int) {
  escape { _ = x } // 오류: 클로저가 @noescape가 아닌 경우 inout 매개변수를 암시적으로 캡처할 수 없음
  noEscape { _ = x } // 정상, 클로저가 @noescape임
  escape {[x] in _ = x } // 정상, 불변 캡처
}

struct Foo {
  mutating func example() {
    escape { _ = self } // 오류: 클로저가 mutating self 매개변수를 암시적으로 캡처할 수 없음
    noEscape { _ = self } // 정상
  }
}
```

중첩 함수 선언의 경우, 클로저는 중첩 함수에 대한 참조가 값으로 사용될 때까지 생성하지 않는다. 중첩 함수가 자신을 둘러싼 스코프의 `inout` 매개변수를 참조하는 경우, 에스케이프 클로저를 형성하는 중첩 함수에 대한 참조를 허용하지 않는다.

```swift
func exampleWithNested(inout x: Int) {
  func nested() {
    _ = x
  }
  escape(nested) // 오류: inout을 참조하는 중첩 함수는 에스케이프할 수 없음
  noEscape(nested) // 정상
}
```

구현 상세 사항으로, 이 변경은 클로저에서 참조될 수 있는 `inout` 매개변수에 대한 섀도우 복사본을 생성할 필요를 없앤다. 이 변경 후에도 여전히 허용되는 코드의 경우, 섀도우 복사본이 에스케이프하지 않는다는 것이 확실할 때 항상 이를 제거하는 최적화 단계가 존재하므로 관찰 가능한 영향은 없다.


## 기존 코드에 미치는 영향

이 변경은 현재 `inout` 캡처 의미에 의존하는 코드를 깨뜨릴 수 있다. 특히 영향을 받을 수 있는 몇 가지 합법적인 사례는 다음과 같다:

- 클로저가 로컬 뮤테이션 이후에 파라미터를 캡처하고, 더 이상 뮤테이션하지 않거나 다른 곳에서의 뮤테이션을 관찰하지 않는 경우. 이러한 사용 사례에서는 캡처 리스트를 사용해 `inout` 파라미터를 불변으로 명시적으로 캡처할 수 있다. 이 방법은 더 명시적이고 안전하다.
- `inout` 파라미터가 탈출 가능한 클로저에 의해 캡처되지만, 동적으로 원래 스코프 외부에서 실행되지 않는 경우. 예를 들어, 즉시 스코프에서 적용되는 `lazy` 시퀀스 어댑터에서 파라미터를 참조하거나, `dispatch_async` 작업을 포크하여 파라미터의 다른 부분에 접근하지만 원래 스코프가 종료되기 전에 동기화되는 경우. 이러한 사용 사례에서는 섀도우 복사를 명시적으로 만들 수 있다:

    ```swift
    func foo(q: dispatch_queue_t, inout x: Int) {
      var shadowX = x; defer { x = shadowX }
      
      // 원본 x 대신 shadowX를 비동기적으로 조작
      dispatch_async(q) { use(&shadowX) }
      doOtherStuff()
      dispatch_sync(q) {}
    }    
    ```

마이그레이션을 위해 컴파일러는 위의 수정 사항 중 하나를 제공할 수 있다. 캡처된 `inout`의 사용을 검사하여 캡처 이후 뮤테이션이 있는지 확인하고, 불변 캡처가 더 적절한지 아니면 명시적인 섀도우 복사가 더 적절한지 결정할 수 있다. (또는 단순히 섀도우 복사 수정 사항을 제공할 수도 있다.)

이 변경은 또한 라이브러리가 가능한 한 `@noescape`를 더 많이 사용하도록 압력을 증가시킨다. 이는 [SE-0012](0012-add-noescape-to-public-library-api.md)에서 제안하는 바와 같다.


## 고려한 대안들

이 제안을 확장하는 한 가지 방법은 섀도우 복사 캡처를 위한 새로운 캡처 종류를 도입하는 것이다:

```swift
func foo(inout x: Int) {
  {[shadowcopy x] in use(&x) } // 스트로맨 문법
}
```

하지만 논의 과정에서, 이런 경우가 드물기 때문에 추가적인 복잡성을 감수할 가치가 없다고 판단했다. 새로운 `var` 선언을 통해 명시적으로 복사하는 방식이 훨씬 명확하며, 새로운 언어 지원이 필요하지 않다.




