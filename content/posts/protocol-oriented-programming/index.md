---
title: "OOP와 POP(Protocol Oriented Programming)"
date: 2017-05-18T19:03:14+09:00
draft: false
tags: ["OOP", "POP"]
---

Swift 2.0 출시 이후, 스위프트 개발자들 사이에서 POP가 화두가 되어 왔습니다. POP는 Protocol Oriented Programming 약자입니다. OOP는 Object Oriented Programming 약자입니다. POP는 뜻 그대로 프로토콜 중심 프로그래밍이고 OOP는 객체 중심 프로그래밍입니다. POP와 OOP 차이가 무엇일까요? 차이가 무엇이길래 개발자들 사이에서 화두가 되고 있을까요?

답을 생각해 보기 전에 OOP의 핵심이 무엇인지 알아야 합니다. OOP 핵심은 상속입니다. 상속을 통해 타입을 확장합니다. 하지만 여러 객체로부터 상속해야 할 경우 많은 문제가 발생합니다. 단일 상속이어도 클래스 상속 계층이 깊어질수록 문제가 커집니다. OOP 핵심인 상속이 코드가 커질수록 문제가 되어 버립니다. 상속은 IS-A 관계를 표현합니다. IS-A 관계를 약간 틀어서 생각해 보면 &#8220;사람은 동물이다&#8221;를 &#8220;사람은 동물 특성을 가지고 있다&#8221;라고 해석할 수 있습니다. IS-A 가 HAS-A 관계가 됩니다.

POP는 바로 HAS-A 관계를 아주 쉽게 표현할 수 있게 해주는 장치입니다. 즉, POP는 수직구조 표현인 OOP 상속을 수평구조  표현인 합성(Composition)으로 객체를 묘사할 수 있게 도와줍니다. POP는 합성을 통해 타입을 확장합니다. 하지만 POP 이전 방식으로 상속을 제거하고 합성을 구현하려면 그 과정이 무척 귀찮았습니다. 왜 그런지 예제를 통해 알아봅시다. 하늘을 날 수 있고 빨리 달릴 수 있고 불을 쏠 수 있는 슈퍼맨을 합성 방식으로 구현해 봅시다.

```swift
struct Fly {
    func fly() {
        print("I can fly")
    }
}

struct FastRun {
    func fastRun() {
        print("I can run fast")
    }
}

struct FireBeam {
    func fireBeam() {
        print("I can shoot fire beam")
    }
}

class Superman {
    let flyable = Fly()
    let fastRunnable = FastRun()
    let fireBeamShootable = FireBeam()
}
```

위처럼 슈퍼맨을 구현했을 때, 어떤 능력을 사용하려고 하면 해당 능력에 직접 접근해야 합니다.

```swift
let superman = Superman() 
superman.flyable.fly()
```

뭔가 어색합니다. 랩퍼를 작성해주면 어색하지 않을 것 같습니다.

```swift
struct Fly {
    func fly() {
        print("I can fly")
    }
}

struct FastRun {
    func fastRun() {
        print("I can run fast")
    }
}

struct FireBeam {
    func fireBeam() {
        print("I can shoot fire beam")
    }
}

class Superman {
    private let flyable = Fly()
    private let fastRunnable = FastRun()
    private let fireBeamShootable = FireBeam()
    
    // 랩퍼를 작성합니다.
    func fly() {
        flyable.fly()
    }
    
    func fastRun() {
        fastRunnable.fastRun()
    }
    
    func fireBeam() {
        fireBeamShootable.fireBeam()
    }
}
```

위처럼 랩퍼를 작성해 주면 아래처럼 능력을 사용할 수 있습니다.

```swift
let superman = Superman()
superman.fastRun()
superman.fly()
superman.fireBeam()
```

코드가 간결해졌습니다. 하지만 매번 이렇게 랩퍼를 작성해 주면 너무 귀찮은 일일 것입니다. 또 다른 문제는 Superman을 직접 수정할 수 없는 경우라면 다른 능력을 추가하는 일도 어렵게 됩니다. 이제 POP가 등장할 때인것 같네요.

POP는 프로토콜에 구현을 담아 놓은 것입니다. 그렇기 때문에 레고처럼 블럭을 조합해 원하는 타입을 만들 수 있습니다. 위 코드를 POP로 변경해 보겠습니다.

```swift
protocol Flyable {
    func fly()
}

extension Flyable {
    func fly() {
        print("I can fly")
    }
}

protocol FastRunnable {
    func fastRun() 
}

extension FastRunnable {
    func fastRun() {
        print("I can run fast")
    }
}

protocol FireBeamShootable {
    func fireBeam()
}

extension FireBeamShootable {
    func fireBeam() {
        print("I can shoot fire beam")
    }
}
```

우선 프로토콜을 선언하고 프로토콜 확장을 통해 기본 구현을 담습니다. 이 프로토콜을 조합하여 새로운 Superman1 을 만들어 보겠습니다.

```swift
class Superman1 : Flyable, FastRunnable, FireBeamShootable {
}
```

사용해 볼까요?

```swift
let superman1 = Superman1()

superman1.fastRun()
superman1.fly()
superman1.fireBeam()
```

같은 FastRunnable 을 사용하지만 타입에 따라 그 구현을 달리하고플 때는 어떻게 할까요? swift 타입 제약을 사용하면 됩니다. 참고로 타입제약은 클래스와 프로토콜에만 적용할 수 있습니다.

```swift
...
extension FastRunnable where Self: Superman1 {
    func fastRun() {
        print("[Superman1] I can run fast")
    }
}

extension FastRunnable where Self: Superman2 {
    func fastRun() {
        print("[Superman2] I can run fast")
    }
}
...
```

이렇게 하고 Superman1과 Superman2 타입을 만들어 보겠습니다.

```swift
class Superman1 : Flyable, FastRunnable, FireBeamShootable {
    
}

class Superman2 : Flyable, FastRunnable, FireBeamShootable {
    
}
```

사용해 봅시다.

```swift
superman1.fly()
superman1.fireBeam()
superman1.fastRun()

superman2.fly()
superman2.fireBeam()
superman2.fastRun()
```

아래처럼 출력됩니다.

```
I can fly
I can shoot fire beam
[Superman1] I can run fast

I can fly
I can shoot fire beam
[Superman2] I can run fast
```

Superman1과 Superman2는 모두 직접 만든 타입입니다. 만약 직접 수정할 수 없는 타입에 FastRunnable 을 추가하고 싶을 때는 어떻게 할까요? 타입을 확장하면 됩니다. 말은 안 되지만 Swift의 Int 타입에 FastRunnable 능력을 추가해 보겠습니다.

```swift
extension Int: FastRunnable {
    func fastRun() {
        print("[\(self)] I can run fast")
    }
}

10.fastRun()
```

아주 쉽게 추가할 수 있습니다. POP를 사용하면 IS-A관계를 HAS-A관계로 아주 쉽게 변경할 수 있습니다. 귀찮게 랩퍼 코드를 작성하지 않아도 됩니다. 한가지 더 중요한 점은 POP가 레이어를 제공한다는 점입니다. FastRunnable 코드를 플랫폼마다 또는 버전마다 다르게 작성해야 할 때, Superman1 은 그 사실을 몰라도 됩니다. FastRunnable 프로토콜 확장에서 플랫폼 또는 버전에 따라 다른 구현을 담으면 되기 때문입니다.

POP는 전혀 새로운 것이 아닙니다. mixin 이름으로 예전부터 존해해 왔던 개념입니다. 이번 Google I/O 2017 에서 안드로이드 공식 개발 언어가 된 Kotlin도 interface에 구현을 담을 수 있고 타입을 확장하는 기능을 제공합니다. POP 또는 mixin기법을 사용하면 프로토콜 및 인터페이스에 담는 세부 구현 코드는 달라도 한 단계 위 추상레이어 코드는 서로 다른 플랫폼 간에도 비슷하게 작성할 수 있습니다.





