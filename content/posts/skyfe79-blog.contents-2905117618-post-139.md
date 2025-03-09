---
title: "[SE-0030] 프로퍼티 동작(Withdrawn)"
date: 2025-03-09T01:09:10Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 프로퍼티 동작

* 제안: [SE-0030](0030-property-behavior-decls.md)
* 작성자: [Joe Groff](https://github.com/jckarter)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **철회됨**
* 대체 제안: [SE-0258](0258-property-wrappers.md)
* 결정 노트: [근거](https://forums.swift.org/t/rejected-se-0030-property-behaviors/1546)


## 소개

프로퍼티 구현 패턴은 반복적으로 등장한다. 컴파일러에 고정된 패턴 집합을 하드코딩하기보다, 이러한 패턴을 라이브러리로 정의할 수 있는 일반적인 "프로퍼티 동작" 메커니즘을 제공하는 것이 바람직하다.

[Swift Evolution 토론](https://forums.swift.org/t/proposal-property-behaviors/594)<br/>
[리뷰](https://forums.swift.org/t/review-se-0030-property-behaviors/1385)


## 동기

특정 언어 기능을 통해 프로퍼티에 대한 몇 가지 중요한 패턴을 지원하려고 시도했지만, 이러한 지원은 범위와 유용성이 제한적이었다. 예를 들어 Swift 1과 2는 `lazy` 프로퍼티를 기본 언어 기능으로 제공한다. 지연 초기화는 흔히 사용되며, 프로퍼티를 `Optional`로 노출하지 않기 위해 종종 필요하기 때문이다. 이 언어 지원 없이는 동일한 효과를 얻기 위해 많은 보일러플레이트 코드가 필요하다:

```swift
class Foo {
  // lazy var foo = 1738
  private var _foo: Int?
  var foo: Int {
    get {
      if let value = _foo { return value }
      let initialValue = 1738
      _foo = initialValue
      return initialValue
    }
    set {
      _foo = newValue
    }
  }
}
```

`lazy`를 언어에 내장하는 것은 몇 가지 단점이 있다. 언어와 컴파일러를 더 복잡하게 만들고 직교성을 떨어뜨린다. 또한 유연성이 부족하다. 지연 초기화에는 다양한 변형이 있지만, 모든 경우에 대해 언어 지원을 하드코딩하는 것은 바람직하지 않다. 예를 들어, 어떤 애플리케이션은 지연 초기화가 동기화되기를 원할 수 있지만, `lazy`는 단일 스레드 초기화만 제공한다. `lazy`의 표준 구현은 값 타입에도 문제가 있다. `lazy` 게터는 `mutating`이어야 하므로, 불변 값에서 접근할 수 없다. 또한 인라인 저장소는 메모이제이션 작업에 최적이 아니다. 값의 복사본 간에 캐시를 재사용할 수 없기 때문이다. 값 중심의 메모이제이션 프로퍼티 구현은 값을 직접 변경하지 않기 위해 클래스 인스턴스를 사용해 캐시된 값을 외부에 저장하는 등 매우 다를 수 있다.

지연 초기화 외에도 중요한 프로퍼티 패턴이 있다. 다단계 초기화를 지원하기 위해 "지연된", 한 번 할당 후 불변인 프로퍼티를 사용하는 것이 종종 합리적이다:

```swift
class Foo {
  let immediatelyInitialized = "foo"
  var _initializedLater: String?

  // initializedLater는 사용자 코드에서 옵셔널이 아닌 'let'처럼 동작해야 한다.
  // 한 번만 할당할 수 있고, 할당 전에는 접근할 수 없다.
  var initializedLater: String {
    get { return _initializedLater! }
    set {
      assert(_initializedLater == nil)
      _initializedLater = newValue
    }
  }
}
```

암시적 언래핑 옵셔널(IUO)을 사용하면 이를 간단히 구현할 수 있지만, 옵셔널이 아닌 'let'에 비해 안전성을 많이 포기한다. IUO를 다단계 초기화에 사용하면 불변성과 nil 안전성을 모두 포기하게 된다.

또한 `didSet`/`willSet`과 같은 애플리케이션별 프로퍼티 기능도 있다. 이는 제한된 기능을 위해 언어 복잡성을 증가시킨다. 이미 언어에 내장된 기능 외에도, 동기화된 접근, 복사, 다양한 종류의 프록시 등 끝없이 많은 일반적인 프로퍼티 동작이 존재하며, 이들 모두 보일러플레이트를 제거하기 위한 언어적 관심이 필요하다.


## 제안하는 해결책

언어 내에서 **프로퍼티 동작(property behaviors)**을 구현할 수 있도록 하는 방안을 제안한다. `var` 선언 시 키워드 뒤에 대괄호를 사용해 **동작**을 지정할 수 있다:

```swift
var [lazy] foo = 1738
```

이 코드는 `lazy`에 대한 **프로퍼티 동작 선언**에 따라 `foo` 프로퍼티를 구현한다:

```swift
var behavior lazy<Value>: Value {
  var value: Value? = nil
  initialValue

  mutating get {
    if let value = value {
      return value
    }
    let initial = initialValue
    value = initial
    return initial
  }
  set {
    value = newValue
  }
}
```

프로퍼티 동작은 영향을 받는 프로퍼티의 저장, 초기화, 접근을 제어할 수 있다. 이를 통해 `lazy`, 옵저버, 그리고 기타 특수한 경우의 프로퍼티 기능을 위한 별도의 언어 지원이 필요 없어진다.


## 예제 살펴보기

세부 설계를 설명하기 전에, 다양한 동작이 적용될 수 있는 잠재적인 활용 사례를 간단히 살펴본다.


### Lazy

현재 `lazy` 속성 기능을 속성 동작으로 재구현할 수 있다.

```swift
// 속성 동작은 `var behavior` 키워드 조합으로 선언한다.
public var behavior lazy<Value>: Value {
  // 동작은 속성을 뒷받침하는 저장소를 선언할 수 있다.
  private var value: Value?

  // 동작은 속성의 초기화 표현식을 `initialValue` 속성 선언과 바인딩할 수 있다.
  initialValue

  // 동작은 저장소를 위한 초기화 로직을 선언할 수 있다.
  // (저장 속성은 인라인에서도 초기화할 수 있다.)
  init() {
    value = nil
  }

  // 인라인 초기화도 지원되므로 `var value: Value? = nil`도 동일하게 작동한다.

  // 동작은 속성을 구현하는 접근자를 선언할 수 있다.
  mutating get {
    if let value = value {
      return value
    }
    let initial = initialValue
    value = initial
    return initial
  }
  set {
    value = newValue
  }
}
```

`lazy` 동작으로 선언된 속성은 동작에서 제공하는 `Optional` 타입의 저장소와 접근자로 뒷받침된다:

```swift
var [lazy] x = 1738 // 내부적으로 Int?를 할당하고 nil로 초기화
print(x) // `lazy` getter를 호출하여 속성을 초기화
x = 679 // `lazy` setter를 호출
```


### 지연 초기화

프로퍼티 동작은 "지연된" 초기화 동작을 모델링할 수 있다. 여기서 프로퍼티에 대한 DI 규칙은 컴파일 타임이 아니라 동적으로 적용된다. 이 방식은 다단계 초기화에서 암시적으로 언래핑된 옵셔널을 사용하지 않아도 된다는 장점이 있다. 재할당이 가능한 `var`와 같은 가변형 변종과 `let`처럼 단 한 번만 초기화가 가능한 불변형 변종을 모두 구현할 수 있다.

가변형 구현은 다음과 같다:

```swift
public var behavior delayedMutable<Value>: Value {
  private var value: Value? = nil

  get {
    guard let value = value else {
      fatalError("property accessed before being initialized")
    }
    return value
  }
  set {
    value = newValue
  }
}
```

불변형 구현은 다음과 같다:

```swift
public var behavior delayedImmutable<Value>: Value {
  private var value: Value? = nil

  get {
    guard let value = value else {
      fatalError("property accessed before being initialized")
    }
    return value
  }

  // 초기화를 수행하며, 이미 초기화된 경우에는 오류를 발생시킨다.
  set {
    if let _ = value {
      fatalError("property initialized twice")
    }
    value = initialValue
  }
}
```

이를 통해 다음과 같은 다단계 초기화가 가능하다:

```swift
class Foo {
  var [delayedImmutable] x: Int

  init() {
    // "x"를 아직 알지 못하며, 설정할 필요도 없다.
  }

  func initializeX(x: Int) {
    self.x = x // 'self.x'가 이미 초기화된 경우 크래시가 발생한다.
  }

  func getX() -> Int {
    return x // 'self.x'가 초기화되지 않은 경우 크래시가 발생한다.
  }
}
```


### 프로퍼티 옵저버

프로퍼티 동작은 커스텀 접근자를 선언함으로써 `didSet`/`willSet` 옵저버의 내장 동작을 유사하게 구현할 수 있다:

```swift
public var behavior observed<Value>: Value {
  initialValue

  var value = initialValue

  // 프로퍼티 동작은 접근자 요구사항을 선언할 수 있다.
  // 이 접근자들은 프로퍼티 선언 시 구현되어야 한다.
  // 동작은 접근자의 기본 구현을 제공할 수 있으며, 이를 선택적으로 만들 수 있다.

  // willSet 접근자: 프로퍼티가 업데이트되기 전에 호출된다. 기본적으로 아무 동작도 하지 않는다.
  mutating accessor willSet(newValue: Value) { }

  // didSet 접근자: 프로퍼티가 업데이트된 후에 호출된다. 기본적으로 아무 동작도 하지 않는다.
  mutating accessor didSet(oldValue: Value) { }

  get {
    return value
  }

  set {
    willSet(newValue)
    let oldValue = value
    value = newValue
    didSet(oldValue)
  }
}
```

`didSet`/`willSet`에 대한 일반적인 불만은 옵저버가 *모든* 쓰기 작업에서 실행된다는 점이다. 실제로 값이 변경된 경우에만 호출되는 `didChange` 접근자를 지원하는 동작은 다음과 같이 새로운 동작으로 구현할 수 있다:

```swift
public var behavior changeObserved<Value: Equatable>: Value {
  initialValue

  var value = initialValue

  mutating accessor didChange(oldValue: Value) { }

  get {
    return value
  }
  set {
    let oldValue = value
    value = newValue
    if oldValue != newValue {
      didChange(oldValue)
    }
  }
}
```

예를 들어:

```swift
var [changeObserved] x: Int = 1 {
  didChange { print("\(oldValue) => \(x)") }
}

x = 1 // 아무것도 출력되지 않음
x = 2 // 1 => 2 출력
```

(참고로, 현재의 `didSet`/`willSet`과 마찬가지로, 이 동작 구현도 참조된 클래스 인스턴스를 변경하되 참조 자체는 변경하지 않는 경우에는 변경을 관찰하지 않는다. 또한 현재 제안하는 대로라면 동작은 프로퍼티가 인라인으로 초기화되도록 강제하며, 이는 인스턴스 프로퍼티에는 적합하지 않다. 이 제한은 향후 확장을 통해 해결할 수 있다.)


### 동기화된 프로퍼티 접근

Objective-C는 `atomic` 프로퍼티를 지원한다. 이 프로퍼티는 `get`과 `set` 시에 락을 걸어 프로퍼티 접근을 동기화한다. 이 기능은 때때로 유용하며, Swift에서도 동일한 동작을 구현할 수 있다. ObjC의 `atomic` 프로퍼티 실제 구현은 전역 락을 사용하지만, 설명을 위해 (그리고 `self`를 참조하는 방법을 보여주기 위해) 각 객체별 락을 사용한다:

```swift
// 프로퍼티 접근을 동기화하기 위해 뮤텍스를 소유하는 클래스
public protocol Synchronizable: class {
  func withLock<R>(@noescape body: () -> R) -> R
}

// Behavior는 프로퍼티가 속한 타입을 암시적인 `Self` 제네릭 파라미터로 참조할 수 있다.
// 확장에서와 마찬가지로 'where' 절을 사용해 제약을 적용할 수 있다.
public var behavior synchronized<Value where Self: Synchronizable>: Value {
  initialValue

  var value: Value = initialValue

  get {
    return self.withLock {
      return value
    }
  }
  set {
    self.withLock {
      value = newValue
    }
  }
}
```


### `NSCopying`

많은 Cocoa 클래스는 명시적인 복사가 필요한 값 타입 객체를 구현한다. Swift는 현재 Objective-C의 `@property(copy)`와 유사한 동작을 제공하기 위해 `@NSCopying` 속성을 제공한다. 이 속성은 프로퍼티가 설정될 때 새로운 객체에서 `copy` 메서드를 호출한다. 이 동작을 다음과 같이 정의할 수 있다:

```swift
public var behavior copying<Value: NSCopying>: Value {
  initialValue

  // 초기화 시 값을 복사한다.
  var value: Value = initialValue.copy()

  get {
    return value
  }
  set {
    // 재할당 시 값을 복사한다.
    value = newValue.copy()
  }
}
```

이는 동작의 가능성을 보여주는 작은 예제이다. 제안하는 설계를 자세히 살펴보자:


## 상세 설계

### 프로퍼티 동작 선언

**프로퍼티 동작 선언**은 `var behavior` 컨텍스트 키워드 클러스터로 시작한다. 이 선언은 동작을 사용하는 프로퍼티의 문법과 유사하게 설계되었다.

```text
property-behavior-decl ::=
  attribute* decl-modifier*
  'var' 'behavior' identifier         // 동작 이름
  generic-signature?
  ':' type
  '{'
    property-behavior-member-decl*
  '}'
```

동작 선언 내부에서는 표준 초기화, 프로퍼티, 메서드, 중첩 타입 선언이 허용된다. 또한 **코어 접근자** 선언(`get`과 `set`)도 사용할 수 있다. **접근자 요구 사항 선언**과 **초기 값 요구 사항 선언**도 컨텍스트 내에서 인식된다.

```text
property-behavior-member-decl ::= decl
property-behavior-member-decl ::= accessor-decl // get, set
property-behavior-member-decl ::= accessor-requirement-decl
property-behavior-member-decl ::= initial-value-requirement-decl
```


### 동작 선언 내 바인딩

동작 선언 내에서 `self`는 암묵적으로 이 동작을 사용해 인스턴스화된 프로퍼티를 포함하는 값에 바인딩된다. 전역 또는 로컬 범위에서 독립적으로 존재하는 프로퍼티의 경우, 이 값은 빈 튜플 `()`이 된다. 정적 또는 클래스 프로퍼티의 경우, 이 값은 메타타입이 된다. 동작 선언 내에서 `self`의 타입은 추상적이며, 암묵적인 제네릭 타입 매개변수 `Self`로 표현된다. 동작의 제네릭 시그니처에 `Self`에 대한 제약을 추가해 프로토콜 멤버를 `self`에서 사용할 수 있게 할 수 있다:

```swift
protocol Fungible {
  typealias Fungus
  func funge() -> Fungus
}

var behavior runcible<Value where Self: Fungible, Self.Fungus == Value>: Value {
  get {
    return self.funge()
  }
}
```

동작 내에서 `self`에 대한 조회는 암묵적이지 않으며 항상 명시적으로 해야 한다. 왜냐하면 한정되지 않은 조회는 동작 자체의 멤버를 참조하기 때문이다. `self`는 `mutating` 메서드 내에서만 변경 가능하며, 이때 `Self` 타입에 클래스 제약이 없다면 `inout` 매개변수로 간주된다. 동작의 저장소를 초기화하는 인라인 초기화자나 `init` 선언 내에서는 `self`에 접근할 수 없다. 이는 컨테이너의 초기화 단계에서 실행될 수 있기 때문이다.

동작 내 정의는 동작의 다른 멤버를 한정되지 않은 조회로 참조할 수 있다. 혼동이 있을 경우, 동작의 이름을 통해 한정된 조회를 사용할 수 있다(`self`는 이미 포함된 값을 의미하기 때문):

```swift
var behavior foo<Value>: Value {
  var x: Int

  init() {
    x = 1738
  }

  mutating func update(x: Int) {
    foo.x = x // 동작 저장소에 대한 참조를 명확히 함
  }
}
```

동작에 *접근자 요구 선언*이 포함된 경우, 선언된 접근자 이름은 라벨이 지정된 인자를 가진 함수로 바인딩된다:

```swift
var behavior fakeComputed<Value>: Value {
  accessor get() -> Value
  mutating accessor set(newValue: Value)

  get {
    return get()
  }
  set {
    set(newValue: newValue)
  }
}
```

동작 자체의 *핵심 접근자* 구현인 `get { ... }`와 `set { ... }`는 이 방식으로 참조할 수 없다는 점에 유의한다.

동작에 *초기값 요구 선언*이 포함된 경우, `initialValue` 식별자는 프로퍼티의 초기값 표현식을 평가하는 get-only 계산 프로퍼티로 바인딩된다.


### Behavior의 프로퍼티와 메서드

Behavior는 프로퍼티와 메서드 선언을 포함할 수 있다. Behavior 프로퍼티에 의해 생성된 저장 공간은 해당 프로퍼티를 사용하는 범위로 확장된다.

```swift
var behavior runcible<Value>: Value {
  var x: Int = 0
  let y: String = ""
  ...
}
var [runcible] a: Int

// expands to:

var `a.[runcible].x`: Int
let `a.[runcible].y`: String
var a: Int { ... }
```

공개(public) behavior의 경우, 이는 본질적으로 *취약*하기 때문에 저장 공간을 추가하거나 제거하는 것은 호환성을 깨뜨리는 변경이다. 이러한 문제는 탄력적인(resilient) 타입을 저장 공간으로 사용함으로써 해결할 수 있다. 인스턴스화된 프로퍼티는 또한 behavior의 잠재적 사용자에게 보여질 수 있는 타입이어야 한다. 이는 공개 behavior가 공개(public) 또는 내부(internal)이며 사용 가능한(available) 타입을 저장 공간으로 사용해야 함을 의미한다. 이는 인라인 가능한(inlineable) 함수에 대한 제한과 유사하다.

메서드와 계산된 프로퍼티(computed property) 구현은 기본적으로 `self`와 그 저장 공간에 대해 불변(immutable) 접근만을 갖는다. 단, `mutating`으로 표시된 경우는 예외이다. (계산된 프로퍼티와 마찬가지로, setter는 명시적으로 `nonmutating`으로 표시되지 않는 한 기본적으로 `mutating`이다).


### Behavior에서의 `init` 사용

Behavior의 저장 공간은 초기화되어야 한다. 이를 위해 인라인 초기화를 사용하거나, 초기화자 내부에 `init` 선언을 사용할 수 있다.

```swift
var behavior inlineInitialized<Value>: Value {
  var x: Int = 0 // 인라인 초기화
  ...
}

var behavior initInitialized<Value>: Value {
  var x: Int

  init() {
    x = 0
  }
}
```

Behavior는 최대 하나의 `init` 선언을 포함할 수 있으며, 이 `init`은 매개변수를 받지 않아야 한다. 또한, `init` 선언은 가시성 수정자를 사용할 수 없다. `init`의 가시성은 항상 Behavior 자체의 가시성과 동일하다. 인라인 초기화나 `init` 선언의 본문에서는 `self`를 참조할 수 없다. 이들은 속성이 포함된 값의 초기화 과정에서 실행되기 때문이다.


### 초기값 요구 사항 선언

*초기값 요구 사항 선언*은 특정 동작(behavior)을 사용해 선언된 프로퍼티가 반드시 초기값 표현식을 포함해야 한다는 것을 명시한다.

```swift
initial-value-requirement-decl ::= 'initialValue'
```

프로퍼티 선언에서 사용된 초기값 표현식은 프로퍼티의 타입으로 강제 변환된 후, 동작의 스코프 내에서 `initialValue` 식별자에 바인딩된다. `initialValue`를 로드하는 것은 get-only 계산 프로퍼티처럼 동작하며, 매번 로드할 때마다 표현식을 평가한다:

```swift
var behavior evalTwice<Value>: Value {
  initialValue

  get {
    // 어떤 이유로든 초기값을 두 번 평가한다.
    _ = initialValue
    return initialValue
  }
}

var [evalTwice] test: () = print("test")

// "test"를 두 번 출력한다
_ = evalTwice
```

동작이 초기값 요구 사항을 포함하는 경우, 해당 동작을 사용해 선언된 프로퍼티는 반드시 초기값 표현식을 가져야 한다. 반대로, 초기값 요구 사항이 없는 동작을 사용할 때는 초기값 표현식이 없어도 된다.


### 접근자 요구사항 선언

*접근자 요구사항 선언*은 특정 동작이 프로퍼티에 접근자 구현을 제공해야 한다는 것을 명시한다. 이 선언은 `accessor` 키워드를 사용하여 정의한다:

```swift
accessor-requirement-decl ::=
  attribute* decl-modifier*
  'accessor' identifier function-signature function-body?
```

접근자 요구사항 선언은 프로토콜의 함수 요구사항 선언과 유사한 역할을 한다. 해당 동작을 사용하는 프로퍼티는 기본 구현이 없는 모든 접근자 요구사항에 대한 구현을 제공해야 한다. 접근자 이름(라벨이 있는 인자 포함)은 동작 선언 내에서 함수로 바인딩된다:

```swift
// 계산 프로퍼티를 재구현
var behavior foobar<Value>: Value {
  accessor foo() -> Value
  mutating accessor bar(bas: Value)

  get { return foo() }
  set { bar(bas: newValue) }
}

var [foobar] foo: Int {
  foo {
    return 0
  }
  bar {
    // 'bas'라는 파라미터 이름은 접근자 요구사항에서 기본적으로 제공되며,
    // 현재 내장 접근자와 동일한 방식으로 작동한다.
    print(bas)
  }
}

var [foobar] bar: Int {
  foo {
    return 0
  }
  bar(myNewValue) {
    // 파라미터 이름을 재정의할 수도 있다.
    print(myNewValue)
  }
}
```

접근자 요구사항은 기본 구현을 지정하여 선택적으로 만들 수 있다:

```swift
// 프로퍼티 옵저버를 재구현
var behavior observed<Value>: Value {
  // 요구사항

  initialValue
  mutating accessor willSet(newValue: Value) {
    // 기본적으로 아무 작업도 수행하지 않음
  }
  mutating accessor didSet(oldValue: Value) {
    // 기본적으로 아무 작업도 수행하지 않음
  }

  // 구현

  init() {
    value = initialValue
  }
  get {
    return value
  }
  set {
    willSet(newValue: newValue)
    let oldValue = value
    value = newValue
    didSet(oldValue: oldValue)
  }
}
```

접근자 요구사항은 가시성 수정자를 가질 수 없다. 접근자는 항상 동작 자체와 동일한 가시성을 가진다.

메서드와 마찬가지로, 접근자는 `mutating`으로 선언되지 않은 경우 동작의 저장소나 `self`를 변경할 수 없다. `mutating` 접근자는 다른 `mutating` 컨텍스트에서만 호출할 수 있다.


### 핵심 접근자 선언

이 동작은 *핵심 접근자*인 `get`과 선택적으로 `set`을 정의해 속성을 구현한다. 동작이 getter만 제공하면 읽기 전용 속성을 생성한다. getter와 setter를 모두 제공하면 변경 가능한 속성을 생성한다. 단, 이 동작을 인스턴스화하는 속성은 여전히 setter의 가시성을 제어할 수 있다. 동작 선언에서 최소한 getter를 제공하지 않으면 오류가 발생한다.


### 프로퍼티 선언에서의 동작 사용

프로퍼티 선언은 임의의 접근자와 함께 동작을 인스턴스화할 수 있는 기능을 제공한다.

```text
property-decl ::= attribute* decl-modifier* core-property-decl
core-property-decl ::=
  ('var' | 'let') behavior? pattern-binding
  ((',' pattern-binding)+ | accessors)?
behavior ::= '[' visibility? decl-ref ']'
pattern-binding ::= var-pattern (':' type)? inline-initializer?
inline-initializer ::= '=' expr
accessors ::= '{' accessor+ '}' | brace-stmt // 모호성 해결에 대한 참고 사항
accessor ::= decl-modifier* decl-ref accessor-args? brace-stmt
accessor-args ::= '(' identifier (',' identifier)* ')'
```

예를 들어:

```swift
public var [behavior] prop: Int {
  accessor1 { body() }
  behavior.accessor2(arg) { body() }
}
```

동일한 선언에서 여러 프로퍼티를 선언할 경우, 동작은 모든 선언된 프로퍼티에 적용된다. `let` 프로퍼티는 아직 동작을 사용할 수 없다.

동작이 접근자를 필요로 하는 경우, 해당 접근자의 구현은 프로퍼티의 접근자 선언에서 이름으로 매칭되어 가져온다. 향후 동작 합성을 지원하기 위해 접근자 정의는 `behavior.accessor`와 같은 정규화된 이름을 사용할 수 있다. 접근자 요구사항이 매개변수를 가지고 있지만, 프로퍼티의 정의에서 매개변수를 명시적으로 명명하지 않은 경우, 동작의 접근자 요구사항 선언에서의 매개변수 레이블이 기본적으로 암묵적으로 바인딩된다.

```swift
var behavior foo<Value>: Value {
  accessor bar(arg: Int)
  ...
}

var [foo] x: Int {
  bar { print(arg) } // `arg`가 암묵적으로 바인딩됨
}

var [foo] x: Int {
  bar(myArg) { print(myArg) } // `arg`가 `myArg`로 명시적으로 바인딩됨
}
```

프로퍼티의 접근자 정의가 동작 요구사항과 일치하지 않으면 오류가 발생한다.

get-only 계산 프로퍼티의 단축 표기법은 동작을 사용하지 않는 계산 프로퍼티에만 허용된다. 접근자를 사용하는 동작을 가진 프로퍼티는 모든 접근자를 명시적으로 명명해야 한다.

동작을 가진 프로퍼티가 인라인 초기화자를 선언하는 경우, 초기화자 표현식은 동작의 초기화자 요구사항에 바인딩된 get-only 계산 프로퍼티의 구현으로 캡처된다. 동작이 초기화자 요구사항을 가지고 있지 않다면, 인라인 초기화자 표현식을 사용하는 것은 오류이다. 반대로, 초기화자를 요구하는 동작에 초기화자 표현식을 제공하지 않는 것도 오류이다.

프로토콜 내부에서는 동작을 사용하여 프로퍼티를 선언할 수 없다.

이 제안에 따르면, 동작을 가진 프로퍼티가 초기값 표현식을 가지고 있더라도 타입은 항상 명시적으로 선언되어야 한다. 또한 동작은 프로퍼티의 외부 초기화를 허용하지 않는다. 이러한 제한은 향후 확장을 통해 해제될 수 있다. 자세한 내용은 아래 **향후 방향** 섹션을 참고한다.


## 기존 코드에 미치는 영향

이 기능 자체는 기존 코드에 영향을 주지 않는 추가적인 기능이다. 하지만 제안하는 미래 방향 중 일부를 고려하면, `lazy`, `willSet`/`didSet`, `@NSCopying` 같은 하드코딩된 언어 기능이 더 이상 필요 없어질 가능성이 있다. 이 기능들을 그대로 유지할 수도 있지만, 라이브러리 기반의 프로퍼티 동작 구현으로 점진적으로 대체하는 것을 선호한다. (물론 이 기능들을 완전히 제거하는 것은 별도의 제안으로 다뤄야 한다.)


## 고려한 대안들

### 새로운 선언 대신 프로토콜(형식적이든 비형식적이든) 사용

이 제안의 이전 버전에서는 프로퍼티 동작을 위해 비형식적인 인스턴스화 프로토콜을 사용했다. 이 방식은 동작을 함수 호출로 변환하는 방식이었다. 예를 들어:

```swift
var [lazy] foo = 1738
```

위 코드는 다음과 같은 코드로 변환될 수 있다:

```swift
var `foo.[lazy]` = lazy(var: Int.self, initializer: { 1738 })
var foo: Int {
  get {
    return `foo.[lazy]`[varIn: self,
                        initializer: { 1738 }]
  }
  set {
    `foo.[lazy]`[varIn: self,
                 initializer: { 1738 }] = newValue
  }
}
```

이 방식에는 몇 가지 단점이 있다:

- 동작들이 네임스페이스를 오염시킬 가능성이 있다. 여러 전역 함수나 타입이 추가될 수 있다.
- 실제로는 모든 동작이 새로운 (보통 제네릭) 타입을 사용해 구현되어야 한다. 이는 타입의 메타데이터 구조를 위한 런타임 오버헤드를 초래한다.
- 프로퍼티 동작 로직이 덜 명확해진다. 비전문화된 언어 구조로 인코딩되기 때문이다.
- 동작의 기능을 결정하는 데 함수 오버로드 해결에 의존해야 한다. 이는 까다로울 수 있으며, 프로퍼티 지향적인 오류 메시지를 얻기 위해 많은 특수 케이스 진단 작업이 필요하다.
- 비형식적 프로토콜을 심각하게 복잡하게 만들지 않고는, 즉시 초기화와 지연 초기화를 지원하거나 `inout` 별칭 규칙을 위반하지 않으면서 프로퍼티의 저장소에 대한 동시 변경 접근을 허용하기 어렵다. 독립적인 동작 선언을 위한 코드 생성은 이 복잡성을 숨길 수 있다.

프로퍼티 동작을 별도의 선언으로 만드는 것은 언어의 크기를 증가시키지만, 동작과 같은 기능에 대한 수요는 분명히 존재한다. 새로운 선언을 통해 더 나은 네임스페이싱, 더 효율적인 코드 생성, 구현을 위한 더 명확하고 설명적인 코드, 더 나은 진단 기능과 표현력을 얻을 수 있다. 이 복잡성은 오늘날 여러 특수 케이스 언어 기능을 제거함으로써, 그리고 미래에는 다른 종류의 동작으로 일반화되거나 모든 것을 포괄하는 매크로 시스템에 흡수됨으로써 그 자체를 상쇄할 수 있다고 주장한다. 예를 들어, 미래의 `func behavior`는 함수 본문을 변환하는 Python 데코레이터와 같은 동작을 제공할 수 있다.


### "템플릿" 스타일 동작 선언 구문

John McCall은 이 제안의 이전 버전에서 속성 동작을 위한 "템플릿" 스타일 구문을 제안했다:

```swift
behavior var [lazy] name: Value = initialValue {
  ...
}
```

이 구문은 선언이 사용을 따르는 관점에서 매력적이며, 이름, 타입, 초기값 바인딩을 편리하게 추가할 수 있는 장점이 있다. 그러나 이와 같은 구문은 Swift에서 전례가 없으며, 초기 검토에서 큰 인기를 얻지 못했다.


### 행동(behavior)을 사용한 프로퍼티 선언 구문

제안하는 `var [behavior] propertyName` 구문 대신 고려할 수 있는 대안은 다음과 같다:

- 다른 종류의 괄호 사용: `var (behavior) propertyName` 또는 `var {behavior} propertyName`.  
  소괄호는 튜플 `var` 선언과 구분이 모호해, 추가적인 파싱이 필요하다.  
  대괄호는 향후 서브스크립트나 함수와 같은 다른 선언에 행동을 확장할 때 더 적합하다.

- 속성(attribute) 사용: `@behavior(lazy)` 또는 `behavior(lazy) var`.  
  가장 보수적인 접근이지만, 다소 번거롭다.

- 행동 함수 이름을 직접 속성으로 사용: 예를 들어 `@lazy`와 같이 사용한다.

- 새로운 키워드 도입: `var x: T by behavior`와 같은 형식을 사용한다.

- 콜론 오른쪽에 표현: `var x: lazy(T)`와 같은 방식.  
  이 표현은 `lazy(T)`가 타입인 것처럼 보이지만, 실제로는 타입이 아니다.

각 대안은 장단점이 있으며, 문법의 명확성과 확장성을 고려해 적절한 방식을 선택해야 한다.


## 향후 발전 방향

여기서 제안한 기능은 상당히 광범위하다. 따라서 초기 제안에 대한 검토 부담을 최소화하기 위해 여러 측면을 별도로 고려할 수 있도록 분리했다:


### 불변 `let` 프로퍼티의 동작

현재는 효과 시스템(effects system)이 없기 때문에, `let` 동작 구현이 `let` 프로퍼티에 기대되는 불변성 가정을 무효화할 가능성이 있다. 따라서 프로그래머가 이를 직접 유지해야 한다. 같은 이유로 계산된 `let`(computed `let`)도 지원하지 않는다. 그러므로 지금은 프로퍼티 동작에서 `let`을 제외하는 것을 권장한다. 불변 계산 프로퍼티나 함수에 대한 포괄적인 설계가 마련되면, 나중에 `let behavior`를 추가할 수 있다.


### 행동(behavior)이 적용된 프로퍼티의 타입 추론

행동이 프로퍼티 타입에 제약을 추가할 때, 행동을 사용해 프로퍼티 타입을 추론하는 과정에서 미묘한 문제가 발생할 수 있다. 다음과 같은 코드를 예로 들어보자:

```swift
var behavior uint16only: UInt16 { ... }

var [uint16only] x = 1738
```

이 경우, 어떤 일이 일어날지 정의하는 방법은 두 가지 이상이 될 수 있다:

1. **행동을 적용하기 전에 초기화 표현식을 독립적으로 타입 체크한다.** 이 경우, `1738`은 기본적으로 `Int` 타입으로 타입 체크된다. 이후 `uint16only` 행동을 적용할 때, 이 행동이 `UInt16` 타입을 요구하므로 오류가 발생한다.

2. **초기화 표현식을 타입 체크하기 전에 행동을 먼저 적용한다.** 이 경우, `uint16only` 행동이 초기화 표현식의 컨텍스트 타입을 `UInt16`으로 제한한다. 따라서 리터럴 `1738`은 `UInt16`으로 성공적으로 타입 체크된다.

두 접근 방식 모두 장단점이 있다. 이러한 문제를 충분히 고려할 수 있도록, 나는 행동이 적용된 프로퍼티는 항상 명시적인 타입을 선언하도록 제안한다. 이렇게 하면 타입 추론 과정에서 발생할 수 있는 모호성을 줄일 수 있다.


### 동작 합성

동작을 합성하는 기능은 유용하다. 예를 들어, 옵저버를 가진 지연 속성(lazy property)을 동기화하는 경우가 있다. 관련하여, 하위 클래스가 상속받은 속성을 `didSet`과 `willSet`을 통해 기본 클래스 구현 위에 동작을 적용해 `override`할 수 있는 것도 유용하다. 선형 합성은 동작이 쌓일 수 있게 함으로써 지원될 수 있으며, 각 동작은 `super`나 다른 매직 바인딩을 통해 아래에 있는 속성을 참조한다. 그러나 이러한 형태의 합성은 "잘못된" 동작 합성을 허용할 수 있기 때문에 위험할 수 있다. `lazy • synchronized`와 `synchronized • lazy` 중 하나는 잘못된 동작을 할 가능성이 있다. 이 문제는 특정 합성을 직접 코드로 작성할 수 있게 함으로써 어느 정도 해결할 수 있다. John McCall은 모든 합성이 완전히 별개의 동작으로 직접 구현되어야 한다고 제안했다. 물론 이는 명백한 지수적 증가 문제를 야기한다. 모든 유용한 동작 조합을 예측하고 수동으로 코딩하는 것은 불가능하다. 이러한 문제들은 신중하게 별도로 고려할 필요가 있으므로, 이 초기 제안에서는 동작 합성을 제외하기로 한다.


### 초기화 표현식의 지연 평가

이 제안은 초기화 표현식 내에서 허용되는 연산을 변경하지 않는다. 특히, 인스턴스 프로퍼티 초기화는 값이 완전히 초기화되기 전에 표현식이 실행될 가능성이 있기 때문에 `self`나 다른 인스턴스 프로퍼티 또는 메서드를 참조할 수 없다.

```swift
struct Foo {
  var a = 1
  var b = a // 허용되지 않음
  var c = foo() // 허용되지 않음

  func foo() { }
}
```

이는 초기화 단계가 완료된 후에만 초기 값 표현식을 평가하는 `lazy` 같은 동작에 불편함을 준다. 특히 `self`를 참조하여 지연 초기화를 하고 싶을 때 더욱 그러하다. 동작을 확장하여 초기화를 "지연된" 것으로 주석을 달면, 초기화 시점에 초기화 표현식이 평가되지 않으면서도 `self`를 참조할 수 있게 할 수 있다. (동작이 기본적으로 항상 취약하다고 간주한다면, 이는 동작 구현에서 추론할 수 있다.)


### 행위(behaviors)를 통한 외부 초기화

이 제안은 외부 초기화(out-of-line initialization)를 지원하는 행위(behaviors)를 허용하지 않는다. 예를 들어 다음과 같은 경우다:

```swift
func foo() {
  // 외부에서 로컬 변수 초기화
  var [behavior] x: Int
  x = 1
}

struct Foo {
  var [behavior] y: Int

  init() {
    // 외부에서 인스턴스 프로퍼티 초기화
    y = 1
  }
}
```

이는 인스턴스 프로퍼티에 있어서 상당히 심각한 제한 사항이다. 이 문제를 해결하기 위해 몇 가지 접근 방식을 고려할 수 있다. 첫 번째로, 행위의 `init` 로직이 외부 초기화를 매개변수로 받을 수 있도록 허용하는 방법이 있다. 이는 초기화 요구 사항에 대한 다른 제약을 두어 `init`에서만 참조할 수 있도록 하는 방식이다. 이는 "지연된(deferred)" 초기화와 반대되는 개념이다. 

또 다른 방법은 선형 행위 합성(linear behavior composition)을 통해 간접적으로 지원하는 것이다. 프로퍼티 스택의 기본 루트 `super` 행위가 일반적인 저장 프로퍼티로 기본 설정되면, 이후에는 일반적인 초기화 규칙을 따를 수 있다. 이는 현재 `didSet`/`willSet`이 동작하는 방식과 유사하다. 하지만 이 방법은 행위가 초기화 동작을 변경할 수 없게 만든다.


### 프로퍼티 이름을 동작에 바인딩하기

프로퍼티 이름을 문자열로 참조할 수 있다면, 이를 활용해 여러 가지 유용한 작업을 수행할 수 있다. 예를 들어, 맵에서 값을 조회하거나 로깅을 하거나, 직렬화하는 등의 작업에 활용할 수 있다. 이와 같은 상황에서 `name` 요구사항을 선언하는 방식을 지원할 수 있다:

```swift
var behavior echo<Value: StringLiteralConvertible>: Value {
  name: String

  get { return name }
}

var [echo] echo: String
print(echo) // => echo
```

이 코드는 `echo`라는 프로퍼티의 이름을 문자열로 반환하는 동작을 정의한다. 이를 통해 프로퍼티의 이름을 직접 활용할 수 있다.


### 오버로딩 동작

동작을 오버로딩할 수 있으면 유용한 경우가 있다. 예를 들어, 개념의 계산된 버전과 저장된 버전에 대해 서로 다른 구현을 제공할 수 있다:

```swift
// 저장 프로퍼티를 위한 동작...
var behavior foo<Value>: Value {
  initialValue

  var value: Value = initialValue
  get { ... }
  set { ... }
}

// 계산 프로퍼티를 위한 동작...
var behavior foo<Value>: Value {
  initialValue

  accessor get() -> Value
  accessor set(newValue: Value)

  get { ... }
  set { ... }
}
```

이러한 오버로딩은 접근자, `Value`에 대한 타입 제약, 그리고/또는 초기화 요구 사항에 따라 해결할 수 있다. 그러나 이 오버로딩 시그니처가 어떻게 정의되어야 하는지, 그리고 초기화 표현식으로부터 타입 추론과의 흥미로운 상호작용은 별도의 논의 주제로 남겨둔다.


### "아웃 오브 밴드" 동작 멤버 접근

프로퍼티에 정규 타입의 일반 멤버가 아닌 아웃 오브 밴드(Out-of-band) 연산을 추가하는 것은 유용하다. 예를 들어, 나중에 다시 계산할 수 있도록 지연 프로퍼티를 `clear`하거나, 구현에 정의된 기본값으로 프로퍼티를 재설정하는 경우가 있다. 이런 기능은 유용하지만, 설계를 복잡하게 만든다. 이러한 멤버에 접근하기 위한 문법과 같은 표면적인 문제 외에도, 이는 동작을 순수한 구현 세부 사항이 아니라 인터페이스로 노출시킨다. 즉, 이러한 동작은 **탄력성(resilience)**, **프로토콜**, **클래스 상속**, 그리고 다른 추상화와의 상호작용을 설계해야 한다. 또한 아웃 오브 밴드 멤버를 동작과 반드시 연결해야 하는지도 고려할 만한 질문이다. 아웃 오브 밴드 멤버를 동작과 독립적인 별도의 기능으로 설계하는 것도 유용할 수 있다.




