---
title: "[SE-0033] Objective-C 상수를 Swift 타입으로 임포트하기"
date: 2025-03-09T01:12:07Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## Objective-C 상수를 Swift 타입으로 임포트하기

* 제안: [SE-0033](0033-import-objc-constants.md)
* 작성자: [Jeff Kelley](https://github.com/SlaunchaMan)
* 리뷰 관리자: [John McCall](https://github.com/rjmccall)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0033-import-objective-c-constants-as-swift-types/1706)


## 소개

Objective-C 파일에 정의된 상수 목록이 있을 때, Swift에서 이 상수들을 Enum이나 Struct로 임포트할 수 있도록 속성을 추가한다. `RawRepresentable`을 사용해 원래 타입으로 변환하면, API에 문자열을 전달하는 대신 더 타입 안전한 객체를 사용할 수 있다. 이 방법은 Swift의 코드 완성 기능을 활용할 수 있을 뿐만 아니라, Swift 코드와 Objective-C 코드를 더 읽기 쉽고 초보자에게 친근하게 만들어준다.

[Swift-evolution 스레드](https://forums.swift.org/t/pitch-import-objective-c-constants-as-enums/1114)

[리뷰](https://forums.swift.org/t/review-import-objective-c-constants-as-swift-types/1486)


## 동기

현재 많은 Objective-C API는 가능한 값을 나열하기 위해 상수, 특히 `NSString` 상수를 사용한다. `HKTypeIdentifiers.h`에서 신체 측정을 위한 작은 예제를 살펴보자 (가독성을 위해 주석과 가용성 매크로는 생략):

```Objective-C
HK_EXTERN NSString * const HKQuantityTypeIdentifierBodyMassIndex;
HK_EXTERN NSString * const HKQuantityTypeIdentifierBodyFatPercentage;
HK_EXTERN NSString * const HKQuantityTypeIdentifierHeight;
HK_EXTERN NSString * const HKQuantityTypeIdentifierBodyMass;
HK_EXTERN NSString * const HKQuantityTypeIdentifierLeanBodyMass;
```

이 선언들은 Swift로 임포트되면 다음과 같이 변환된다:

```Swift
public let HKQuantityTypeIdentifierBodyMassIndex: String
public let HKQuantityTypeIdentifierBodyFatPercentage: String
public let HKQuantityTypeIdentifierHeight: String
public let HKQuantityTypeIdentifierBodyMass: String
public let HKQuantityTypeIdentifierLeanBodyMass: String
```

이 제안의 목표는 이러한 문자열을 `enum` 또는 `RawRepresentable`을 준수하는 `struct`로 캡슐화하여 `String` 타입으로 표현하는 것이다. 왜 이런 작업을 할까? 이러한 API를 사용할 때, 상수를 사용하는 메서드는 단순히 `NSString` 파라미터를 받는다. 아래 나열된 메서드들이 그 예이다. 초보 프로그래머에게는 이 메서드들이 다른 파일에 정의된 "특별한" 값을 받는다는 사실이 명확히 드러나지 않는다:

```Objective-C
+ (nullable HKQuantityType *)quantityTypeForIdentifier:(NSString *)identifier;
+ (nullable HKCategoryType *)categoryTypeForIdentifier:(NSString *)identifier;
+ (nullable HKCharacteristicType *)characteristicTypeForIdentifier:(NSString *)identifier;
+ (nullable HKCorrelationType *)correlationTypeForIdentifier:(NSString *)identifier;
```

`NSError` 도메인과 같은 경우, 미리 정의된 상수가 있더라도 사용자 정의 값을 사용해도 괜찮다. 하지만 HealthKit 식별자와 같은 경우는 그렇지 않다. 전자의 경우는 `struct`로 잘 처리할 수 있지만, 후자의 경우는 `enum`이 더 적합하다. 이러한 케이스를 추가함으로써 기존 Objective-C 코드를 더 잘 문서화할 수 있고, Swift에서 더 강력한 기능을 제공할 수 있다. 프레임워크를 처음 접하는 프로그래머가 문서를 볼 때, 사용해야 하는 문자열이나 다른 타입의 상수가 어디에서 오는지 즉시 명확히 알 수 있다.


[Doug Gregor가 swift-evolution에서 제안한 것처럼](https://forums.swift.org/t/pitch-import-objective-c-constants-as-enums/1114/2), `typedef`와 새로운 속성을 결합하면 내부적으로 원래 타입을 유지하면서도 코드를 깔끔하게 정리할 수 있다. 다음은 구조체와 열거형을 사용한 예제다:


### 열거형(Enums)

HealthKit 식별자는 열거형을 사용하기에 적합한 사례다. API 사용자가 새로운 항목을 추가할 수 없기 때문이다.

```Objective-C
typedef NSString * HKQuantityTypeIdentifier __attribute__((swift_wrapper(enum));

HK_EXTERN HKQuantityTypeIdentifier const HKQuantityTypeIdentifierBodyMassIndex;
HK_EXTERN HKQuantityTypeIdentifier const HKQuantityTypeIdentifierBodyFatPercentage;
HK_EXTERN HKQuantityTypeIdentifier const HKQuantityTypeIdentifierHeight;
HK_EXTERN HKQuantityTypeIdentifier const HKQuantityTypeIdentifierBodyMass;
HK_EXTERN HKQuantityTypeIdentifier const HKQuantityTypeIdentifierLeanBodyMass;
```

이 추가 타입 정보를 통해 Swift 임포트가 크게 개선된다.

```Swift
enum HKQuantityTypeIdentifier : String {
    case BodyMassIndex
    case BodyFatPercentage
    case Height
    case BodyMass
    case LeanBodyMass
}
```

Objective-C의 메서드 시그니처는 새로운 `typedef`를 사용한다.

```Objective-C
+ (nullable HKQuantityType *)quantityTypeForIdentifier:(HKQuantityTypeIdentifier)identifier;
```

이를 통해 Swift에서 `quantityTypeForIdentifier()`를 호출할 때 `rawValue`를 호출해 내부 `String`을 추출할 필요 없이 구조체 상수를 직접 사용할 수 있다. Swift 코드는 다음과 같다.

```Swift
let quantityType = HKQuantityType.quantityTypeForIdentifier(.BodyMassIndex)
```

타입이 열거형으로 임포트되면, 다른 모듈이나 API 사용자가 추가 항목을 더할 수 없다.


### 구조체

NSError 도메인은 사용자가 새로운 항목을 추가할 수 있기 때문에 구조체로 정의하기 적합하다. 아래는 선언된 도메인의 일부이다:

```Objective-C
typedef NSString * NSErrorDomain __attribute__((swift_wrapper(struct)));

FOUNDATION_EXPORT NSErrorDomain const NSCocoaErrorDomain;
FOUNDATION_EXPORT NSErrorDomain const NSPOSIXErrorDomain;
FOUNDATION_EXPORT NSErrorDomain const NSOSStatusErrorDomain;
FOUNDATION_EXPORT NSErrorDomain const NSMachErrorDomain;
```

이 코드는 Swift에서 다음과 같이 임포트된다:

```Swift
struct NSErrorDomain : RawRepresentable {
    typealias RawValue = String

    init(rawValue: RawValue)
    var rawValue: RawValue { get }

    static var Cocoa: NSErrorDomain { get }
    static var POSIX: NSErrorDomain { get }
    static var OSStatus: NSErrorDomain { get }
    static var Mach: NSErrorDomain { get }
}
```

구조체로 임포트되는 타입의 경우, 다른 모듈에서 상수 인스턴스를 추가해야 한다면 확장으로 임포트된다. 예를 들어:

```Objective-C
extern NSString * const NSURLErrorDomain;
```

이 코드는 Swift에서 다음과 같이 임포트된다:

```Swift
extension NSErrorDomain {
    static var URL: NSErrorDomain  { get }
}
```


### 네이밍 규칙

새로운 상수(constants)의 이름은 타입 이름과 주어진 케이스에서 공통 접두사와 접미사를 제거하여 결정한다. 이름은 대문자로 작성하며, Swift 코드에서 사용하는 다음과 같은 예시를 참고할 수 있다:

```Swift
struct NSSortOptions : OptionSetType {
    init(rawValue rawValue: UInt)
    static var Concurrent: NSSortOptions { get }
    static var Stable: NSSortOptions { get }
}
```

```Swift
enum NSSearchPathDirectory : UInt {
    case ApplicationDirectory
    case DemoApplicationDirectory
    case DeveloperApplicationDirectory
    …
    case AllApplicationsDirectory
    case AllLibrariesDirectory
    case TrashDirectory
}
```


## 기존 코드에 미치는 영향

기존의 수정되지 않은 Objective-C 코드는 변경 없이 그대로 임포트된다. 이러한 어노테이션을 사용해 Objective-C 코드를 활용하는 기존 Swift 코드는 새로운 타입을 사용하도록 업데이트해야 한다. 하지만 모든 타입 정보가 존재하기 때문에 마이그레이션이 가능하며, Swift 코드는 상수에 대해 Objective-C 이름을 그대로 사용한다.


## 고려한 대안들

이 제안의 두 번째 초안은 `struct`만 사용했다. 일부 타입은 열거형(enum)으로 임포트하는 것이 적절하지 않기 때문이다(예: 오류 도메인). 이번 버전은 열거형을 다시 추가해, 적절한 타입에 따라 두 가지 버전을 모두 사용할 수 있도록 했다.

첫 번째 초안은 `struct` 대신 `enum`을 사용했고 `RawRepresentable`을 포함하지 않았다. 리스트 토론을 거쳐 `RawRepresentable`을 사용하도록 수정했다. `RawRepresentable`을 사용하면 다른 모듈에서 정의한 새로운 값을 더 쉽게 추가할 수 있다. 단, 이 제안의 목적과 `enum`이 완벽히 일치하지 않는다는 우려가 있었다. 이러한 식별자 목록은 `switch` 문에서 자주 사용되지 않기 때문이다.

원래 아이디어는 `NS_CASE_LIST_BEGIN`과 `NS_CASE_LIST_END` 같은 마커를 사용해 코드 영역을 표시한 다음, 마커 내부에 선언된 상수를 열거형으로 임포트하는 것이었다. 그러나 Doug의 `typedef` 솔루션은 메서드에도 주석을 달 수 있어, Swift에서 사용할 수 있는 타입 정보를 더 많이 제공한다.




