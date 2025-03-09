---
title: "[SE-0005] Objective-C API를 Swift로 더 좋게 번역하기"
date: 2025-03-08T23:52:07Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## Objective-C API를 Swift로 더 좋게 번역하기

* 제안: [SE-0005](0005-objective-c-name-translation.md)
* 작성자: [Doug Gregor](https://github.com/DougGregor), [Dave Abrahams](https://github.com/dabrahams)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-with-modification-se-0005-better-translation-of-objective-c-apis-into-swift/1668)


## 리뷰어 노트

이 리뷰는 서로 밀접하게 연관된 세 개의 리뷰 중 하나로, 동시에 진행된다:

* [SE-0023 API 디자인 가이드라인](0023-api-guidelines.md)
  ([리뷰](https://forums.swift.org/t/review-se-0023-api-design-guidelines/1162))
* [SE-0006 표준 라이브러리에 API 가이드라인 적용](0006-apply-api-guidelines-to-the-standard-library.md)
  ([리뷰](https://forums.swift.org/t/review-se-0006-apply-api-guidelines-to-the-standard-library/1163))
* [SE-0005 Objective-C API를 Swift로 더 나은 번역](0005-objective-c-name-translation.md)
  ([리뷰](https://forums.swift.org/t/review-se-0005-better-translation-of-objective-c-apis-into-swift/1164))

이 리뷰들은 서로 강하게 연관되어 있기 때문에 동시에 진행된다 (예: 표준 라이브러리의 API 변경은 특정 가이드라인에 해당하거나, 임포터 규칙이 특정 가이드라인을 구현하는 등). 이러한 상호작용과 논의를 관리하기 위해 다음 사항을 요청한다:

* **리뷰를 작성하기 전에 세 문서 모두에 대해 이해를 해야한다.**
* **각 문서에 대한 리뷰는 해당 리뷰 발표에 대한 응답으로 작성한다.** 리뷰에서 문서 간의 상호 참조를 하는 것은 괜찮으며, 오히려 권장한다. 이는 여러분의 주장을 뒷받침하는 데 도움이 된다.


## 소개

이 제안서는 C와 Objective-C API를 Swift로 매핑하는 역할을 담당하는 Swift의 "Clang Importer"를 개선하는 방법을 설명한다. 특히 Objective-C 함수, 타입, 메서드, 프로퍼티 등의 이름을 [Swift API Design Guidelines][api-design-guidelines]에 더 가깝게 변환하는 방법에 초점을 맞춘다. 이 가이드라인은 Swift 3 개발의 일환으로 진행 중이다. 우리의 접근 방식은 Objective-C [Coding Guidelines for Cocoa][objc-cocoa-guidelines]와 Swift API Design Guidelines의 차이점을 분석하고, 간단한 언어학적 분석을 통해 Objective-C 이름을 더 "Swifty"한 이름으로 자동 변환하는 데 중점을 둔다.

이 변환의 결과는 [Swift 3 API Guidelines Review](https://github.com/apple/swift-3-api-guidelines-review) 저장소에서 확인할 수 있다. 이 저장소에는 Swift 2([`swift-2` 브랜치](https://github.com/apple/swift-3-api-guidelines-review/tree/swift-2))와 Swift 3([`swift-3` 브랜치](https://github.com/apple/swift-3-api-guidelines-review/tree/swift-3))에서의 Objective-C API의 Swift 투영 결과물이 포함되어 있으며, 부분적으로 마이그레이션된 샘플 코드도 제공된다. 또한 [두 브랜치를 비교](https://github.com/apple/swift-3-api-guidelines-review/compare/swift-2...swift-3)하여 전체적인 변화를 확인할 수 있다.


## 동기

Objective-C의 [Cocoa 코딩 가이드라인][objc-cocoa-guidelines]은 Objective-C에서 명확하고 일관된 API를 설계하는 데 탁월한 프레임워크를 제공한다. 그러나 Swift는 다른 언어다. 특히, 강력한 타입 시스템을 가지고 있으며 타입 추론, 제네릭, 오버로딩과 같은 기능을 제공한다. 결과적으로 Objective-C에서 자연스럽게 느껴지는 API가 Swift에서는 불필요하게 장황하게 느껴질 수 있다. 예를 들면:

```swift
let content = listItemView.text.stringByTrimmingCharactersInSet(
   NSCharacterSet.whitespaceAndNewlineCharacterSet())
```

여기서 사용된 API는 Objective-C 가이드라인을 따르고 있다. 같은 코드를 Swift 스타일에 더 가깝게 작성하면 다음과 같다:

```swift
let content = listItemView.text.trimming(.whitespaceAndNewlines)
```

두 번째 예제는 [Swift API 설계 가이드라인][api-design-guidelines]에 더 부합한다. 특히, 컴파일러가 이미 강제하는 타입(뷰, 문자열, 문자 집합 등)을 반복하는 불필요한 단어를 생략했다. 이 제안의 목표는 임포트된 Objective-C API를 더 "Swift스럽게" 만들어 Swift 프로그래머가 Objective-C API를 사용할 때 더 유연한 경험을 제공하는 것이다.

이 제안의 해결책은 Objective-C 프레임워크(예: Cocoa 및 Cocoa Touch 전체)와 Swift에서 사용 가능한 모든 Objective-C API에 동일하게 적용된다. [Swift 코어 라이브러리][core-libraries]는 Objective-C 프레임워크의 API를 재구현하므로, 해당 프레임워크(Foundation, XCTest 등)의 API 변경 사항은 코어 라이브러리의 Swift 3 구현에 반영된다.


## 제안하는 솔루션

이 제안은 Objective-C [Cocoa 코딩 가이드라인][objc-cocoa-guidelines]과 [Swift API 디자인 가이드라인][api-design-guidelines] 간의 차이점을 식별하여, 가이드라인과 Objective-C에서 관찰된 관례를 기반으로 전자를 후자로 매핑하는 변환 규칙 세트를 구축하는 것이다. 이는 Clang 임포트에서 이름을 변환하는 기존 휴리스틱(예: 전역 enum 상수를 Swift의 케이스로 매핑하거나 Objective-C 팩토리 메서드(`+[NSNumber numberWithBool:]`)를 Swift 초기화 구문(`NSNumber(bool: true)`)로 변환하는 것)을 확장한 것이다.

이 제안에서 설명한 휴리스틱은 Objective-C API의 광범위한 집합을 대상으로 반복적인 테스트와 튜닝이 필요하다. 또한 완벽하지는 않을 것이다. 일부 API는 이 변환을 거친 후 Swift에서 이전보다 덜 명확해질 수 있다. 따라서 목표는 대부분의 Objective-C API가 더 "Swift스럽게" 느껴지도록 하고, 덜 명확해진 API의 작성자가 Objective-C 헤더 내에 주석을 추가해 문제를 해결할 수 있도록 하는 것이다.

이 제안은 Clang 임포터에 다음과 같은 몇 가지 관련 변경 사항을 포함한다:

1. **`swift_name` 속성의 적용 범위 확장**: 현재 Clang의 `swift_name` 속성은 enum 케이스와 팩토리 메서드의 제한된 이름 변경만 허용한다. 이를 C 또는 Objective-C 엔티티가 Swift로 임포트될 때 임의의 이름 변경을 허용하도록 일반화해, API 작성자가 프로세스를 더 세밀하게 제어할 수 있도록 한다.

2. **중복 타입 이름 제거**: Cocoa 코딩 가이드라인은 메서드가 각 인수를 설명하도록 요구한다. 이 설명이 해당 매개변수의 타입을 반복할 경우, Swift API 가이드라인의 [불필요한 단어 생략](https://swift.org/documentation/api-design-guidelines#omit-needless-words) 원칙과 충돌한다. 따라서 임포트 시 이러한 타입 이름을 제거한다.

3. **기본 인수 추가**: Objective-C API가 기본 인수의 필요성을 강력히 암시하는 경우, API를 임포트할 때 기본 인수를 추론한다. 예를 들어, 옵션 세트 매개변수는 기본값으로 `[]`를 설정할 수 있다.

4. **첫 번째 인수 레이블 추가**: 메서드의 첫 번째 매개변수가 기본값을 가질 경우, [첫 번째 인수 레이블이 있어야 한다](https://swift.org/documentation/api-design-guidelines#first-argument-label). 해당 메서드에 적합한 첫 번째 인수 레이블을 결정한다.

5. **Boolean 속성에 "is" 접두사 추가**: [Boolean 속성은 수신자에 대한 주장으로 읽혀야 한다](https://swift.org/documentation/api-design-guidelines#boolean-assertions). 하지만 Cocoa 코딩 가이드라인은 [속성에 "is" 사용을 금지한다](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE). 따라서 이러한 속성을 임포트할 때 "is"를 접두사로 추가한다.

6. **값 소문자화**: Swift API 디자인 가이드라인은 타입이 아닌 선언을 소문자로 작성하도록 요구한다. 임포트 시 접두사가 없는 값을 소문자로 변환한다. 이는 열거자(enum 케이스 또는 옵션 세트로 끝나는 경우)와 기타 속성/함수(예: `URLHandler` 속성은 `urlHandler`로 소문자화)를 포함한다.

7. **`compare(_:) -> NSComparisonResult`를 구현한 클래스에 `Comparable` 적용**: `compare`를 구현한 Objective-C 클래스는 모두 순서 비교가 가능하다고 선언한 것이다. `Comparable`은 이 선언을 임포트 과정에서 구현 가능한 연산자로 공식화한다.

이 변환 규칙의 효과를 확인하기 위해 Swift 2에서 임포트된 `UIBezierPath` API의 일부를 살펴보자:

```swift
class UIBezierPath : NSObject, NSCopying, NSCoding {
  convenience init(ovalInRect: CGRect)
  func moveToPoint(_: CGPoint)
  func addLineToPoint(_: CGPoint)
  func addCurveToPoint(_: CGPoint, controlPoint1: CGPoint, controlPoint2: CGPoint)
  func addQuadCurveToPoint(_: CGPoint, controlPoint: CGPoint)
  func appendPath(_: UIBezierPath)
  func bezierPathByReversingPath() -> UIBezierPath
  func applyTransform(_: CGAffineTransform)
  var empty: Bool { get }
  func containsPoint(_: CGPoint) -> Bool
  func fillWithBlendMode(_: CGBlendMode, alpha: CGFloat)
  func strokeWithBlendMode(_: CGBlendMode, alpha: CGFloat)
  func copyWithZone(_: NSZone) -> AnyObject
  func encodeWithCoder(_: NSCoder)
}
```

이 제안의 현재 실험적 구현으로 임포트된 동일한 API는 다음과 같다:

```swift
class UIBezierPath : NSObject, NSCopying, NSCoding {
  convenience init(ovalIn rect: CGRect)
  func move(to point: CGPoint)
  func addLine(to point: CGPoint)
  func addCurve(to endPoint: CGPoint, controlPoint1 controlPoint1: CGPoint, controlPoint2 controlPoint2: CGPoint)
  func addQuadCurve(to endPoint: CGPoint, controlPoint controlPoint: CGPoint)
  func append(_ bezierPath: UIBezierPath)
  func reversing() -> UIBezierPath
  func apply(_ transform: CGAffineTransform)
  var isEmpty: Bool { get }
  func contains(_ point: CGPoint) -> Bool
  func fill(_ blendMode: CGBlendMode, alpha alpha: CGFloat)
  func stroke(_ blendMode: CGBlendMode, alpha alpha: CGFloat)
  func copy(with zone: NSZone = nil) -> AnyObject
  func encode(with aCoder: NSCoder)
}
```

후자의 경우, 원본 API에서 타입 정보를 반복한 여러 단어가 제거되었다. 결과는 Swift API 디자인 가이드라인을 더 잘 따르게 되었다. 예를 들어, 이제 Swift 개발자는 `NSCopying`을 준수하는 객체를 `foo.copy()`와 같이 간단히 호출해 복사할 수 있다. 이전에는 `foo.copyWithZone(nil)`을 호출해야 했다.


## 구현 경험

이 제안에 대한 실험적 구현은 메인 Swift 저장소에서 확인할 수 있다. Objective-C API를 임포트할 때 이 제안을 적용한 결과를 확인하거나 Swift 코드 자체에 적용하기 위해 사용할 수 있는 컴파일러 플래그가 있다. 예를 들어, `utils/omit-needless-words.py` 스크립트를 통해 확인할 수 있다. 주요 플래그는 다음과 같다:

* `-enable-omit-needless-words`: 이 플래그는 Clang 임포터에 대한 대부분의 변경사항(이전 섹션의 1, 2, 4, 5번 항목)을 활성화한다. 현재는 Swift 마스터 브랜치와 [Swift 2.2 브랜치][swift-2_2-branch]에서 Objective-C 모듈의 Swift 인터페이스를 출력하는 데 적합하다. 이 기능은 [Swift 3 API 가이드라인 브랜치][swift-3-api-guidelines-branch]에서도 활성화되어 있다.

* `-enable-infer-default-arguments`: 이 플래그는 Clang 임포터에서 기본 인자 추론(이전 섹션의 3번 항목)을 활성화한다.

* `-swift3-migration`: 이 플래그는 [Swift 2.2 브랜치][swift-2_2-branch]에서만 사용할 수 있으며, Swift 2 이름을 Swift 3 이름으로 기본 마이그레이션을 수행한다. `-fixit-code`, `-fixit-all` 같은 다른 컴파일러 플래그와 Fix-Its를 수집하고 적용하는 스크립트(`utils/apply-fixit-edits.py`)와 함께 사용하면, 제안된 변경사항에 따라 Swift 코드가 어떻게 보일지 확인할 수 있다. 이 과정에서 선언부와 사용부 모두를 업데이트한다.

이 이름들을 사용해 코드를 컴파일하는 "Swift 3 경험"을 실제로 얻으려면, [Swift 3 API 가이드라인 브랜치][swift-3-api-guidelines-branch]를 사용하면 된다. 이 브랜치는 표준 라이브러리의 변경사항과 함께 이러한 기능을 기본적으로 활성화한다.

[swift-2_2-branch]: https://github.com/apple/swift/tree/swift-2.2-branch
[swift-3-api-guidelines-branch]: https://github.com/apple/swift/tree/swift-3-api-guidelines-branch


## 상세 설계

이 섹션은 실험적으로 구현된 규칙 2-5에 대해 자세히 설명한다. 실제 구현은 Swift 소스 트리에서 확인할 수 있으며, 대부분 [lib/Basic/StringExtras.cpp](https://github.com/apple/swift/blob/master/lib/Basic/StringExtras.cpp) 파일의 `omitNeedlessWords` 함수에 포함되어 있다.

이 섹션의 설명은 Objective-C API를 기준으로 작성되었다. 예를 들어, Objective-C 메서드 이름은 "셀럭터"로 표현된다. `startWithQueue:completionHandler:`는 두 개의 셀럭터 조각(`startWithQueue`와 `completionHandler`)로 구성된 셀럭터이다. 이 이름을 Swift로 직접 매핑하면 `startWithQueue(_:completionHandler:)`가 된다.


### 불필요한 타입 이름 제거하기

Objective-C API 이름은 종종 매개변수나 결과 타입의 이름을 포함한다. Swift API에서는 이런 단어들을 생략하는 것이 일반적이다. 아래 규칙들은 이런 단어들을 식별하고 제거하기 위해 설계되었다. [[불필요한 단어 생략](https://swift.org/documentation/api-design-guidelines#omit-needless-words)]


#### 타입 이름 식별하기

아래 설명된 매칭 프로세스는 셀럭터 조각에서 **타입 이름**이라는 문자열의 접미사를 검색한다. 타입 이름은 다음과 같이 정의된다:

* 대부분의 Objective-C 타입에서 *타입 이름*은 Swift가 임포트하는 타입의 이름이며, nullability는 무시한다. 예를 들어:

  |Objective-C 타입       | *타입 이름*                                |
  |-----------------------|--------------------------------------------|
  |`float`                |`Float`                                     |
  |`nullable NSString`    |`String`                                    |
  |`UIDocument`           |`UIDocument`                                |
  |`nullable UIDocument`  |`UIDocument`                                |
  |`NSInteger`            |`NSInteger`                                 |
  |`NSUInteger`           |`NSUInteger`                                |
  |`CGFloat`              |`CGFloat`                                   |

* Objective-C 타입이 블록인 경우, *타입 이름*은 "`Block`"이다.

* Objective-C 타입이 함수에 대한 포인터나 참조인 경우, *타입 이름*은 "`Function`"이다.

* Objective-C 타입이 `NSInteger`, `NSUInteger`, `CGFloat` 이외의 typedef인 경우, *타입 이름*은 기본 타입의 이름이다. 예를 들어, Objective-C 타입이 `float`에 대한 typedef인 `UILayoutPriority`인 경우, 문자열 "`Float`"을 매칭하려고 시도한다. [[약한 타입 정보 보완](https://swift.org/documentation/api-design-guidelines#weak-type-information)]


#### 매칭 규칙

셀럭터 조각에서 불필요한 타입 이름을 제거하기 위해, 타입을 식별하는 셀럭터의 부분 문자열과 매칭해야 한다. 이러한 매칭은 몇 가지 기본 규칙에 따라 이루어진다.

* **매칭은 단어 경계에서 시작하고 끝난다.** 타입 이름과 셀럭터 조각 모두에서 단어 경계는 문자열의 시작과 끝, 그리고 대문자 앞에 위치한다.

  모든 대문자를 단어의 시작으로 간주하면, 특별한 약어 목록이나 접두사 목록을 유지하지 않아도 대문자로 이루어진 약어를 매칭할 수 있다.

  <pre>
  func documentFor<b>URL</b>(_: NS<b>URL</b>) -> NSDocument?
  </pre>

  반면, 부분 단어가 잘못 매칭되는 것을 방지한다.

  <pre>
  var thumbnailPre<b>view</b> : UI<b>View</b>  // 매칭되지 않음
  </pre>

* **매칭된 텍스트는 타입 이름의 끝까지 확장된다.** 타입 이름의 *어떤 접미사*든 매칭을 허용하기 때문에, 다음과 같은 코드:

  <pre>
  func constraintEqualTo<b>Anchor</b>(anchor: NSLayout<b>Anchor</b>) -&gt; NSLayoutConstraint?
  </pre>

  는 다음과 같이 간소화할 수 있다:

  <pre>
  func constraintEqualTo(anchor: NSLayoutAnchor) -&gt; NSLayoutConstraint?
  </pre>

  편리하게도, 접미사로 매칭한다는 것은 `NS`와 같은 모듈 접두사가 매칭이나 간소화를 방해하지 않는다는 의미이다.

매칭은 다음 중 하나 이상의 시퀀스로 이루어진다:

* **기본 매칭**

  * 셀럭터 조각의 부분 문자열이 타입 이름의 동일한 부분 문자열과 일치하면 매칭된다. 예를 들어, `appendString`의 `String`은 `NSString`의 `String`과 매칭된다:

    <pre>
    func append<b>String</b>(_: NS<b>String</b>)
    </pre>

  * 셀럭터 조각의 `Index`는 타입 이름의 `Int`와 매칭된다:

    <pre>
    func characterAt<b>Index</b>(_: <b>Int</b>) -> unichar
    </pre>

* **컬렉션 매칭**

  * 셀럭터 조각의 `Indexes` 또는 `Indices`는 타입 이름의 `IndexSet`과 매칭된다:

    <pre>
    func removeObjectsAt<b>Indexes</b>(_: NS<b>IndexSet</b>)
    </pre>

  * 셀럭터 조각의 복수 명사는 컬렉션 타입 이름과 매칭된다. 단, 명사의 단수 형태가 컬렉션의 요소 타입 이름과 일치해야 한다:

    <pre>
    func arrange<b>Objects</b>(_: <b>[</b>Any<b>Object]</b>) -> [AnyObject]
    </pre>

* **특수 접미사 매칭**

  * 셀럭터 조각의 빈 문자열은 타입 이름의 `Type` 또는 `_t`와 매칭된다:

    <pre>
    func writableTypesFor<b>SaveOperation</b>(_: NS<b>SaveOperation</b><i>Type</i>) -> [String]
    func objectFor<b>Key</b>(_: <b>Key</b><i>Type</i>) -> AnyObject
    func startWith<b>Queue</b>(_: dispatch_<b>queue</b><i>_t</i>, completionHandler: MKMapSnapshotCompletionhandler)
    </pre>

  * 셀럭터 조각의 빈 문자열은 타입 이름에서 *하나 이상의 숫자 뒤에 "D"가 오는 부분*과 매칭된다:

    <pre>
    func pointFor<b>Coordinate</b>(_: CLLocation<b>Coordinate</b><i>2D</i>) -> NSPoint
    </pre>

위 예제에서 이탤릭체로 표시된 텍스트는 효과적으로 건너뛰어지므로, 셀럭터 조각의 굵은 부분이 매칭되고 간소화될 수 있다.


#### 프루닝 제약 조건

다음 섹션에서 설명할 프루닝 단계는 아래 규칙에 따라 제한된다. 단계를 수행할 때 이 규칙 중 하나라도 위반하면 해당 단계를 건너뛴다.

* **셀럭터 조각을 완전히 비우지 않는다.**

* **첫 번째 셀럭터 조각을 Swift 키워드로 변환하지 않는다.** 이는 사용자가 백틱으로 키워드를 이스케이프해야 하는 상황을 피하기 위함이다. Swift에서 첫 번째 Objective-C 셀럭터 조각은 메서드의 기본 이름이나 프로퍼티의 전체 이름이 된다. 이 둘은 Swift 키워드와 일치할 경우 사용자가 백틱을 써야 한다. 예를 들어,

  <pre>
  extension NSParagraphStyle {
  &nbsp;&nbsp;class func default<b>ParagraphStyle</b>() -> NS<b>ParagraphStyle</b>
  }
  let defaultStyle = NSParagraphStyle.<b>default</b>ParagraphStyle()  // OK
  </pre>

  위 코드가 아래와 같이 변환되면,

  <pre>
  extension NSParagraphStyle {
  &nbsp;&nbsp;class func <b>`default`</b>() -> NSParagraphStyle
  }
  let defaultStyle = NSParagraphStyle.<b>`default`</b>()              // 어색함
  </pre>

  반면, 이후 셀럭터 조각은 인자 레이블이 되며, Swift 키워드와 일치해도 백틱이 필요 없다:

  <pre>
  receiver.handle(someMessage, <b>for</b>: somebody)  // OK
  </pre>

* **이름을 "get", "set", "with", "for", "using"으로 변환하지 않는다.** 이는 의미 없는 이름을 생성하는 것을 방지하기 위함이다.

* **접사, 동사, 동명사 바로 앞에 오는 접미사를 제거하지 않는다.**

  이 휴리스틱은 파라미터를 가리키는 명사 시퀀스를 분리하지 않도록 한다. 명사구의 접미사만 제거하면 뒤따르는 파라미터에 대해 의도치 않은 의미를 암시할 수 있다. 예를 들어,

  <pre>
  func setText<b>Color</b>(_: UI<b>Color</b>)
  ...
  button.<b>setTextColor</b>(.red())  <b>// 명확함</b>
  </pre>

  `Color`를 제거하고 `Text`만 남기면 호출 지점이 혼란스러워진다:

  <pre>
  func setText(_: UIColor)
  ...
  button.<b>setText</b>(.red())      <b>// 텍스트를 설정하는 것처럼 보임!</b>
  </pre>

  참고: 명사 목록을 유지하지는 않지만, 만약 유지한다면 이 규칙은 "파라미터 앞에 오는 명사의 접미사를 제거하지 않는다"로 간단히 표현할 수 있다.

* **클래스의 프로퍼티와 일치하는 메서드의 기본 이름에서 접미사를 제거하지 않는다.**

  이 휴리스틱은 클래스의 프로퍼티를 개념적으로 수정하는 메서드에 대해 지나치게 일반적인 이름을 생성하지 않도록 한다.

  <pre>
  var <b>gestureRecognizers</b>: [UIGestureRecognizer]
  func add<b>GestureRecognizer</b>(_: UI<b>GestureRecognizer</b>)
  </pre>

  `GestureRecognizer`를 제거하고 `add`만 남기면 `gestureRecognizers` 프로퍼티를 수정하는 메서드가 지나치게 일반적인 이름을 갖게 된다:

  <pre>
  var gestureRecognizers: [UIGestureRecognizer]
  func add(_: UIGestureRecognizer) <b>// 프로퍼티에 추가한다는 의미를 나타내야 함</b>
  </pre>


#### 코드 간소화 단계

다음과 같은 순서로 코드 간소화 작업을 수행한다:

1. **타입 보존 변환에서 결과 타입을 헤드에서 제거**. 구체적으로,

   * 리시버 타입이 결과 타입과 동일하고
   * 첫 번째 셀럭터 조각의 헤드에서 타입 이름이 일치하며
   * 일치 부분 뒤에 전치사가 오는 경우

   일치 부분을 제거한다. 이 작업은 리시버의 변환된 버전을 생성하는 속성이나 비뮤테이션 메서드에 영향을 준다. 예를 들어:

   <pre>
   extension NS<b>Color</b> {
   &nbsp;&nbsp;func <b>color</b><i>With</i>AlphaComponent(_: CGFloat) -> NS<b>Color</b>
   }
   let translucentForeground = <b>foregroundColor.color</b><i>With</i>AlphaComponent(0.5)
   </pre>

   다음과 같이 변경된다:

   <pre>
   extension NS<b>Color</b> {
   &nbsp;&nbsp;func <i>with</i>AlphaComponent(_: CGFloat) -> NS<b>Color</b>
   }
   let translucentForeground = <b>foregroundColor</b>.<i>with</i>AlphaComponent(0.5)
   </pre>

2. **추가로 붙어 있는 "By" 제거**. 구체적으로,

   * 1단계에서 어떤 것이 제거되었고
   * 남은 셀럭터 조각이 "`By`"로 시작하며 *뒤에 동명사가 오는 경우*

   초기 "`By`"도 제거한다. 이 휴리스틱은 `a = b.frobnicating(c)` 형태의 사용법을 도출할 수 있게 한다. 예를 들어:

   <pre>
   extension NSString {
   &nbsp;&nbsp;func string<b>By</b><i>Applying</i>Transform(_: NSString, reverse: Bool) -> NSString?
   }
   let sanitizedInput = rawInput.<b>stringByApplyingTransform</b>(NSStringTransformToXMLHex, reverse: false)
   </pre>

   다음과 같이 변경된다:

   <pre>
   extension NSString {
   &nbsp;&nbsp;func applyingTransform(_: NSString, reverse: Bool) -> NString?
   }
   let sanitizedInput = rawInput.<b>applyingTransform</b>(NSStringTransformToXMLHex, reverse: false)
   </pre>

3. **시그니처에서 타입 이름과 일치하는 부분을 이전 셀럭터 조각의 꼬리에서 제거**. 구체적으로,

   |제거 대상:                               |일치하는 부분:              |
   |------------------------------------------------|--------------------------------|
   |매개변수를 도입하는 셀럭터 조각          |매개변수 타입 이름         |
   |속성 이름                                |속성 타입 이름          |
   |매개변수가 없는 메서드 이름              |반환 타입 이름            |
       
   예를 들어,

   <pre>
   extension NSDocumentController {
   &nbsp;&nbsp;func documentFor<b>URL</b>(_ url: NS<b>URL</b>) -> NSDocument? // 매개변수 도입
   }
   extension NSManagedObjectContext {
   &nbsp;&nbsp;var parent<b>Context</b>: NSManagedObject<b>Context</b>?       // 속성
   }
   extension UIColor {
   &nbsp;&nbsp;class func darkGray<b>Color</b>() -> UI<b>Color</b>            // 매개변수 없는 메서드
   }
   ...
   myDocument = self.documentFor<b>URL</b>(locationOfFile)
   if self.managedObjectContext.parent<b>Context</b> != changedContext { return }
   foregroundColor = .darkGray<b>Color</b>()
   </pre>

   다음과 같이 변경된다:

   <pre>
   extension NSDocumentController {
   &nbsp;&nbsp;func documentFor(_ url: NSURL) -> NSDocument?
   }
   extension NSManagedObjectContext {
   &nbsp;&nbsp;var parent : NSManagedObjectContext?
   }
   extension UIColor {
   &nbsp;&nbsp;class func darkGray() -> UIColor
   }
   ...
   myDocument = self.<b>documentFor</b>(locationOfFile)
   if self.managedObjectContext.<b>parent</b> != changedContext { return }
   foregroundColor = .<b>darkGray</b>()
   </pre>

4. **메서드의 기본 이름에서 동사 뒤에 오는 타입 이름과 일치하는 부분 제거**. 예를 들어,

   <pre>
   extension UI<b>ViewController</b> {
   &nbsp;&nbsp;func dismiss<b>ViewController</b>Animated(flag: Bool, completion: (() -> Void)? = nil)
   }
   </pre>

   다음과 같이 변경된다:

   <pre>
   extension UIViewController {
   &nbsp;&nbsp;func dismissAnimated(flag: Bool, completion: (() -> Void)? = nil)
   }
   </pre>


##### 순서가 중요한 이유는 무엇일까?

아래 단계 중 일부는 첫 번째 셀럭터 조각의 앞부분에서 일치하는 부분을 제거하고, 일부는 뒷부분에서 제거한다. [제한 조건](#pruning-restrictions)으로 인해 앞부분과 뒷부분 모두를 제거할 수 없는 경우, 앞부분을 먼저 제거하는 단계를 우선 적용하면 메서드 패밀리를 함께 유지할 수 있다. 예를 들어, NSFontDescriptor의 경우:

```swift
func fontDescriptorWithSymbolicTraits(_: NSFontSymbolicTraits) -> NSFontDescriptor
func fontDescriptorWithSize(_: CGFloat) -> UIFontDescriptor
func fontDescriptorWithMatrix(_: CGAffineTransform) ->  UIFontDescriptor
...
```

이 코드는 다음과 같이 변한다:

<pre>
func <b>with</b>SymbolicTraits(_: UIFontDescriptorSymbolicTraits) ->  UIFontDescriptor
func <b>with</b>Size(_: CGFloat) -> UIFontDescriptor
func <b>with</b>Matrix(_: CGAffineTransform) -> UIFontDescriptor
...
</pre>

만약 `SymbolicTraits`를 뒷부분에서 먼저 제거한다면, 지나치게 빈약한 이름을 만들지 않도록 하는 제한 때문에 "`fontDescriptorWith`"를 "`with`"로 줄이는 것이 불가능해진다. 그 결과는 다음과 같다:

<pre>
func <b>fontDescriptorWith</b>(_: NSFontSymbolicTraits) -> NSFontDescriptor // 일관성 없음
func withSize(_: CGFloat) -> UIFontDescriptor
func withMatrix(_: CGAffineTransform) -> UIFontDescriptor
...
</pre>


#### 기본 인자 추가

단일 파라미터 설정 메서드가 아닌 경우, 다음과 같은 상황에서 파라미터에 기본 인자를 추가한다:

* **Nullable trailing closure 파라미터**는 기본값으로 `nil`을 지정한다.

* **Nullable NSZone 파라미터**는 기본값으로 `nil`을 지정한다. Swift에서는 Zones가 거의 사용되지 않으며 항상 `nil`이어야 한다.

* **옵션 세트 타입** 중 타입 이름에 "Options"가 포함된 경우, 기본값으로 `[]`(빈 옵션 세트)를 지정한다.

* **NSDictionary 파라미터** 중 이름에 "options", "attributes", 또는 "info"가 포함된 경우, 기본값으로 `[:]`를 지정한다.

이러한 휴리스틱을 적용하면 다음과 같은 코드:

<pre>
rootViewController.presentViewController(alert, animated: true<b>, completion: nil</b>)
UIView.animateWithDuration(
  0.2, delay: 0.0, <b>options: [],</b> animations: { self.logo.alpha = 0.0 }) { 
    _ in self.logo.hidden = true 
}
</pre>

가 아래와 같이 간결해진다:

```swift
rootViewController.present(alert, animated: true)
UIView.animateWithDuration(
  0.2, delay: 0.0, animations: { self.logo.alpha = 0.0 }) { _ in self.logo.hidden = true }
```


#### 첫 번째 인자 라벨 추가하기

첫 번째 셀럭터 조각에 전치사가 포함된 경우, **마지막 전치사를 기준으로 첫 번째 셀럭터 조각을 분리**한다. 이때 마지막 전치사부터 시작하는 부분을 첫 번째 인자의 *필수* 라벨로 만든다.

이 방법은 많은 API에 대해 첫 번째 인자 라벨을 생성할 뿐만 아니라, 인자의 기본값이 사용되는 호출 지점에서 첫 번째 인자를 지칭하는 단어를 제거한다. 예를 들어, 다음과 같은 코드가 있다고 가정해 보자:

<pre>
extension UIBezierPath {
  func enumerateObjects<b>With</b>(_: NSEnumerationOptions <b>= []</b>, using: (AnyObject, UnsafeMutablePointer<ObjCBool>) -> Void)
}

array.enumerateObjects<b>With</b>(.Reverse) { // OK
 // ..
}

array.enumerateObjects<b>With</b>() {         // ?? 무엇과 함께?
 // ..
}
</pre>

이 코드는 다음과 같이 개선할 수 있다:

<pre>
extension NSArray {
  func enumerateObjects(<b>options</b> _: NSEnumerationOptions <b>= []</b>, using: (AnyObject, UnsafeMutablePointer<ObjCBool>) -> Void)
}

array.enumerateObjects(<b>options:</b> .Reverse) { // OK
 // ..
}

array.enumerateObjects() {               // OK
 // ..
}
</pre>

이렇게 하면 코드의 가독성이 향상되고, 인자의 기본값을 사용할 때 불필요한 혼란을 피할 수 있다.


#### 불리언 속성에 Getter 이름 사용하기

**불리언 속성의 경우, Swift에서 속성 이름으로 getter의 이름을 사용한다. 예를 들어:

```swift
@interface NSBezierPath : NSObject
@property (readonly,getter=isEmpty) BOOL empty;
```

위 코드는 다음과 같이 변환된다.

<pre>
extension NSBezierPath {
  var <b>isEmpty</b>: Bool
}

if path.<b>isEmpty</b> { ... }
</pre>


### 비교 메서드를 구현하는 클래스의 준수성

현재 프로토콜을 비교할 때 개발자들은 일반적으로 `NSDate`를 확장해 `Comparable`을 준수하도록 만들거나, `NSDate`의 `compare(_:) -> NSComparisonResult` 메서드를 직접 사용한다. 이 경우, `NSDate` 객체에 비교 연산자를 사용하면 코드의 가독성이 높아진다. 예를 들어 `someDate < today`와 같은 표현이 `someDate.compare(today) == .OrderedAscending`보다 더 직관적이다. 임포트 과정에서 클래스가 비교를 위한 Objective-C 메서드를 구현했는지 확인할 수 있으므로, 이 메서드를 구현한 모든 클래스는 `Comparable`을 채택한 것으로 임포트된다.

Foundation 클래스를 조사해 보면 `NSDate`뿐만 아니라 이 변경의 영향을 받는 몇 가지 다른 클래스도 있다.

```swift
func compare(other: NSDate) -> NSComparisonResult
func compare(decimalNumber: NSNumber) -> NSComparisonResult
func compare(otherObject: NSIndexPath) -> NSComparisonResult
func compare(string: String) -> NSComparisonResult
func compare(otherNumber: NSNumber) -> NSComparisonResult
```


## 기존 코드에 미치는 영향

제안된 변경 사항은 Objective-C 프레임워크를 사용하는 Swift 코드에 큰 영향을 미친다. Swift 2 코드를 Swift 3 코드로 변환하려면 마이그레이션 도구가 필요하다. [구현 경험](#implementation-experience) 섹션에서 설명한 `-swift3-migration` 플래그는 이러한 마이그레이션 도구의 기본 기능을 제공할 수 있다. 또한, 컴파일러는 이전(변환 전) Objective-C 이름을 참조하는 Swift 코드에 대해 명확한 오류 메시지(Fix-Its 포함)를 제공해야 한다. 이는 앞서 설명한 Fix-Its와 이전 이름을 유지하는 보조 이름 조회 메커니즘을 결합하여 달성할 수 있다.


## 감사의 말

이 제안서에 설명된 자동 번역 기능은 [Swift API 디자인 가이드라인][api-design-guidelines]을 제작하는 과정에서 개발되었다. 이 작업에는 Dmitri Hrybenko, Ted Kremenek, Chris Lattner, Alex Migicovsky, Max Moiseev, Ali Ozer, 그리고 Tony Parker가 함께 참여했다.

비교 가능성에 대한 부록은 원래 [Chris Amanse](https://github.com/chrisamanse)가 [core-libraries] 메일링 리스트에 제안한 내용을 바탕으로 했다. 이후 Philippe Hausler의 리뷰를 거쳐 이 제안서에 맞게 수정되었다.

[objc-cocoa-guidelines]: https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html  "Cocoa 코딩 가이드라인"
[api-design-guidelines]: https://swift.org/documentation/api-design-guidelines "API 디자인 가이드라인"
[core-libraries]: https://swift.org/core-libraries/  "Swift 코어 라이브러리"
[swift-3-api-guidelines-branch]: https://github.com/apple/swift/tree/swift-3-api-guidelines  "Swift 3 API 가이드라인 브랜치"
[swift-2_2-branch]: https://github.com/apple/swift/tree/swift-2.2-branch  "Swift 2.2 브랜치"




