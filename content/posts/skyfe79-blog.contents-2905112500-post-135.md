---
title: "[SE-0026] 추상 클래스와 메서드(Rejected)"
date: 2025-03-09T00:56:04Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 추상 클래스와 메서드

* 제안: [SE-0026](0026-abstract-classes-and-methods.md)
* 작성자: David Scrève
* 리뷰 관리자: [Joe Groff](https://github.com/jckarter/)
* 상태: **거부됨**
* 리뷰: ([초안](https://forums.swift.org/t/proposal-draff-abstract-classes-and-methods/965)) ([리뷰](https://forums.swift.org/t/review-se-0026-abstract-classes-and-methods/1580)) ([연기](https://forums.swift.org/t/deferred-se-0026-abstract-classes-and-methods/1705)) ([거부](https://forums.swift.org/t/returning-or-rejecting-all-the-deferred-evolution-proposals/60724))


## 소개

프레임워크와 재사용 가능한 코드를 개발할 때, 부분적으로 추상화되고 부분적으로 구현된 클래스를 만들어야 한다. 프로토콜과 프로토콜 확장은 이를 제공하지만, 클래스처럼 속성을 가질 수 없다. 

부분 클래스는 클래스의 동작과 프로토콜처럼 상속된 클래스에서 메서드를 구현해야 하는 요구 사항을 결합한다.


## 동기

C++의 순수 가상 메서드와 Java, C#의 추상 클래스처럼, 프레임워크 개발에서는 때때로 추상 클래스 기능이 필요하다. 추상 클래스는 일반 클래스와 비슷하지만, 일부 메서드나 프로퍼티가 구현되지 않고 상속받은 클래스에서 구현되어야 한다. 추상 클래스는 다른 클래스를 상속받을 수 있고, 프로토콜을 구현할 수 있으며, 멤버 속성을 가질 수 있다. 이는 프로토콜과는 대조적이다. 추상 클래스에서는 일부 메서드와 프로퍼티만 추상적일 수 있다.

추상 클래스의 목적은 일반적인 동작을 캡슐화하는 데 있다. 이 동작은 추상 클래스 내부에서 알 수 없는 구체적인 구현 메서드가 필요할 수 있으며, 이러한 동작은 추상 클래스 메서드에서 사용되는 속성을 필요로 한다.

예를 들어, 프레임워크에 포함된 일반적인 RESTClient를 생각해 보자:

```swift
class RESTClient {
    
    var timeout = 3000
    
    var url : String {
        assert(false,"Must be overridden")
        return ""
    }
    
    func performNetworkCall() {
        let restURL = self.url
        print("Performing URL call to \(restURL) with timeout \(self.timeout)")
    }
}
```

그리고 이를 구현한 클래스:

```swift
class MyRestServiceClient : RESTClient {
    override var url : String {
        return "http://www.foo.com/client"
    }
    
}
```

여기서 볼 수 있듯이, `url` 프로퍼티는 상속받은 클래스에서 구현되어야 하며, 상위 클래스에서 구현되어서는 안 된다. 이를 우회하기 위해 `assert`를 추가했지만, 이 오류는 컴파일 타임이 아닌 런타임에만 감지되며, 최종 사용자에게 크래시를 일으킬 수 있다.

다른 방법으로는 델리게이트/데이터소스 패턴을 사용할 수 있지만, 델리게이트는 클래스가 제공하는 상속 기능을 사용할 수 없다.


## 제안하는 해결책

메서드나 프로퍼티가 추상적이며 현재 클래스에서 구현되지 않음을 나타내는 새로운 키워드를 추가할 것을 제안한다. 이는 해당 메서드나 프로퍼티가 상속받은 클래스에서 반드시 구현되어야 함을 의미한다. 클래스와 프로퍼티/메서드에 추가해야 하는 `abstract` 키워드를 제안한다:

```swift
abstract class RESTClient {    
     var timeout = 3000

    abstract var url : String { get }
    
    func performNetworkCall() {
        let restURL = self.url
        print("Performing URL call to \(restURL) with timeout \(self.timeout)")
    }
}
```

그리고 구현 예제:

```swift
class MyRestServiceClient : RESTClient {
    override var url : String {
        return "http://www.foo.com/client"
    }
    
}
```


## 상세 설계

추상 클래스는 인스턴스화할 수 없다.

추상 메서드나 프로퍼티는 구현을 포함할 수 없다.

하나 이상의 추상 메서드나 프로퍼티를 포함하는 클래스는 반드시 추상 클래스로 선언해야 한다.

추상 클래스를 상속받은 클래스는 모든 상속된 메서드나 프로퍼티를 구현하지 않으면 추상 클래스로 선언해야 한다.

추상 클래스나 부분적으로 추상 메서드나 프로퍼티를 구현한 상속 클래스를 구현하려고 하면 컴파일 오류가 발생한다.

override 키워드와 관련해, 추상 프로퍼티는 setter, getter, 그리고 옵저버에 적용된다.

추상 프로퍼티를 선언할 때는 반드시 구현해야 할 메서드를 지정해야 한다: get, set, didSet, willSet.

아무것도 지정하지 않으면 setter와 getter만 추상으로 선언된다. 아래와 같이 말이다:

```swift
    abstract var url : String
```

옵저버는 기본적으로 빈 구현을 제공한다.

추상 프로퍼티의 타입은 반드시 명시해야 한다. 타입 추론이 불가능하기 때문이다.


## 기존 코드에 미치는 영향

이 변경 사항은 기존 코드에 영향을 미치지 않는다. 하지만 Swift 3.0에서 안정화 중인 ABI를 변경할 가능성이 있다.


## 고려했던 대안들

첫 번째로 프로토콜과 프로토콜 확장이 필요를 충족할 수 있을 것처럼 보인다. 하지만 실제로는 그렇지 않다. 추상 클래스는 프로토콜이 지원하지 않는 속성(attributes)과 프로퍼티를 가질 수 있다.

다른 대안으로는 프로토콜과 프로토콜 확장에 속성을 추가하는 방법이 있다. 하지만 이 방법은 Objective-C 런타임과의 호환성을 깨뜨릴 수 있다.




