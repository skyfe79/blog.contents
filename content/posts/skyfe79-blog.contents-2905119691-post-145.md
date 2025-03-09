---
title: "[SE-0036] 열거형 인스턴스 멤버 구현 시 선행 점 접두사 요구"
date: 2025-03-09T01:14:40Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 열거형 인스턴스 멤버 구현 시 선행 점 접두사 요구

* 제안: [SE-0036](0036-enum-dot.md)  
* 작성자: [Erica Sadun](https://github.com/erica), [Chris Lattner](https://github.com/lattner)  
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)  
* 상태: **구현 완료 (Swift 3.0)**  
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0036-requiring-leading-dot-prefixes-for-enum-instance-member-implementations/2196)  
* 버그: [SR-1236](https://bugs.swift.org/browse/SR-1236)


## 소개

열거형 케이스는 기본적으로 정적 멤버이며, 인스턴스 타입 멤버가 아니다. 구조체와 클래스의 정적 멤버와 달리, 열거형 케이스는 초기화자나 인스턴스 메서드에서 완전한 타입 참조 없이도 언급할 수 있다. 이는 일관성이 떨어지는 방식이다. 다른 경우에는 인스턴스 구현이 정적 멤버에 직접 접근할 수 없다. 이 제안은 더 일관된 개발자 경험을 제공하고, 정적 케이스와 인스턴스 멤버를 명확히 구분하기 위해 선행 점(.)이나 완전한 참조(EnumType.caseMember)를 요구하는 규칙을 도입한다.

*이 논의는 Swift Evolution 메일링 리스트의 [\[Discussion\] Enum Leading Dot Prefixes](https://forums.swift.org/t/discussion-enum-leading-dot-prefixes/1404) 스레드에서 진행되었다. 이 제안은 현재 [API 가이드라인 작업 그룹의 지침](https://swift.org/documentation/api-design-guidelines/)에 따라 lowerCamelCase 형식의 열거형 케이스를 사용한다.*

[리뷰](https://forums.swift.org/t/review-se-0036-requiring-leading-dot-prefixes-for-enum-instance-member-implementations/2020)


## 동기

Swift는 개발자를 대신해 단일 열거형 타입을 명확하게 사용할 때 해당 타입을 자동으로 추론한다. 이 추론 기능은 다음과 같은 switch 문을 작성할 수 있게 해준다:

```swift
switch Coin() {
case .heads: print("Heads")
case .tails: print("Tails")
}
```

Swift 언어 전반에서 선행 점(leading dot)은 "열거형 케이스"를 나타내는 관용적인 약어로 자리 잡았다. `enum` 구현 내부에서 사용할 때는 선행 점이 필요하지 않으며, 정적 케이스 멤버에 접근하기 위해 타입 이름도 필요 없다. 다음 코드는 Swift에서 유효하다:

```swift
enum Coin {
    case heads, tails
    func printMe() {
        switch self {
        case heads: print("Heads")  // 선행 점 없음
        case .tails: print("Tails") // 선행 점 있음
        }
        
        if self == heads {          // 선행 점 없음
            print("This is a head")
        }
        
        if self == .tails {         // 선행 점 있음
            print("This is a tail")
        }
    }

    init() {
        let cointoss = arc4random_uniform(2) == 0
        self = cointoss ? .heads : tails // 선행 점 혼합 사용
    }
}
```

이러한 특성은 언어의 일관성을 해치며, 개발자를 혼란스럽게 할 수 있다. 또한 *최소 놀람의 원칙(Principle of Least Astonishment)*에 위배된다. 따라서 선행 점을 의무화하는 것을 제안한다. 이는 열거형 타입 구현 외부에서 케이스를 참조할 때 사용하는 관례와 일치시켜 준다.


## 상세 설계

이 규칙에 따르면, 컴파일러는 모든 case 멤버에 대해 앞에 점(.)을 요구한다. 이 변경 사항은 다른 정적 멤버에는 영향을 미치지 않는다. 인스턴스 메서드에서는 완전한 참조가 필요하고, 정적 메서드에서는 `self`를 추론한다.

```swift
enum Coin {
    case heads, tails
    static func doSomething() { print("Something") }
    static func staticFunc() { doSomething() } // 점이 필요하지 않음
    static func staticFunc2() { let foo = tails } // 점이 필요하지 않음, 정적 규칙을 따름
    func instanceFunc() { self.dynamicType.doSomething() } // 완전한 참조가 필요함
    func otherFunc() { if self == .heads ... } // 점이 필요함, 초기화도 마찬가지

    /// ...
} 
```


## 고려한 대안

현재 상태를 그대로 유지하는 것 외에도, 언어는 다른 정적 멤버와 마찬가지로 인스턴스 멤버가 완전히 정규화된 타입을 사용해 케이스를 참조하도록 강제할 수 있다.




