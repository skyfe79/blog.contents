---
title: "[SE-0027] 문자열에 코드 유닛 초기화 메서드 추가(Rejected)"
date: 2025-03-09T00:56:45Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 문자열에 코드 유닛 초기화 메서드 추가

* 제안: [SE-0027](0027-string-from-code-units.md)
* 작성자: [Zachary Waldowski](https://github.com/zwaldowski)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **거부됨**
* 결정 근거: [Rationale](https://forums.swift.org/t/rejected-se-0027-expose-code-unit-initializers-on-string/1529)


## 시작하며

문자열과 바이트 표현 간의 변환은 객체 직렬화, 바이너리 및 텍스트 파일 포맷, 네트워크 인터페이스, 암호화 등 다양한 문제를 해결하는 데 중요한 역할을 한다. Swift는 이러한 유틸리티를 제공하지만, 현재는 `String.Type.fromCString(_:)`와 `String.Type.fromCStringRepairingIllFormedUTF8(_:)`를 통해서만 접근할 수 있다.

관련 논의는 swift-evolution [스레드](https://forums.swift.org/t/faster-lower-level-external-string-initialization/974)와 [초안 제안](https://forums.swift.org/t/faster-lower-level-external-string-initialization/974/4)에서 확인할 수 있다.


## 동기

파서를 개발하던 중, 한 동료가 Swift의 유니코드 타입에 대한 벤치마킹 작업을 꼼꼼히 수행했다. 그는 `String.Type.fromCString(_:)` ([사용 예시](https://gist.github.com/zwaldowski/5f1a1011ea368e1c833e#file-fromcstring-swift))가 자신이 찾은 가장 빠른 방법이라고 확신했다. 나는 고집이 세고 초보자답게, Swift의 `UnicodeCodecType`을 통해 더 나은 방법을 찾을 수 있을 거라 생각하며 회의적이었다.

stdlib 소스 코드를 읽고 직접 테스트를 해본 결과, 동료의 말은 사실이었다. `fromCString`은 `String.Type._fromCodeUnitSequence(_:input:)`의 유일한 공개 사용자이며, 이 메서드는 버퍼 복사를 통해 효율적이고 안전한 초기화를 제공한다. 여러 시도 끝에, 현재 제공되는 `String` API들은 성능이 훨씬 떨어지면서도 더 많은 유니코드 안전성을 보장하지 못한다는 결론에 이르렀다.

물론 `fromCString(_:)`도 완벽한 해결책은 아니다. 이 메서드는 UTF-8 인코딩과 널 종결자를 강제한다. 이는 원본 버퍼를 복사하거나, 종결자를 추가해야 할 경우 훨씬 느린 문자별 추가 방식을 사용해야 함을 의미한다. 이는 길이를 미리 지정하는 포맷이나 다른 종결자를 사용하는 비정형 데이터에 적용된다. 또한 이 방식은 문자열 자체가 널 문자를 포함할 수 없게 만든다. 마지막으로, `fromCString(_:)` 생성자는 사용자 코드에서 이미 계산된 경우에도 `strlen`을 호출해야 한다.


## 제안하는 솔루션

`String.Type._fromCodeUnitSequence(_:input:)`와 동등한 기능을 공개 API로 제공하려고 한다:

```swift
static func decode<Encoding: UnicodeCodecType, Input: CollectionType where Input.Generator.Element == Encoding.CodeUnit>(_: Input, as: Encoding.Type, repairingInvalidCodeUnits: Bool = default) -> (result: String, repairsMade: Bool)?
```

편의를 위해, 여기서 `Bool` 플래그를 더 일반적인 경우에 사용할 수 있는 두 개의 `String` 초기화 메서드로 분리한다:

```swift
init<...>(codeUnits: Input, as: Encoding.Type)
init?<...>(validatingCodeUnits: Input, as: Encoding.Type)
```

마지막으로, `String.Type.fromCString(_:)` 및 `String.Type.fromCStringRepairingIllFormedUTF8(_:)`와 더 직접적으로 호환되기 위해, 길이가 알려지지 않은 포인터 기반 문자열에 대해 이 생성자들을 오버로드한다:

```swift
init(cString: UnsafePointer<CChar>)
init?(validatingCString: UnsafePointer<CChar>)
```


## 상세 설계

[전체 구현](https://github.com/apple/swift/compare/master...zwaldowski:string-from-code-units)을 참조한다.

먼저 [Swift 3.0](https://github.com/apple/swift/commit/f4aaece75e97379db6ba0a1fdb1da42c231a1c3b) 버전의 `CString` 생성자를 백포트한 다음, 입력과 코덱에 대해 제네릭으로 만든다.

이 작업은 내부 API의 이름을 변경하는 비교적 간단한 과정이다. 이니셜라이저와 라벨, 순서는 표준 라이브러리의 다른 캐스팅되지 않은 이니셜라이저와 일치하도록 선택했다. "Sequence"는 잘못된 명칭이었기 때문에 제거했다. "input"은 향후 개선을 위해 일반적인 이름으로 유지했다.

새로운 생성자는 기본값에 대한 기대를 바꾼다. `fromCString`은 잘못된 코드 유닛 시퀀스에서 실패할 수 있지만, `init(cString:)`은 무조건 성공한다. Swift 3에 맞춰 개발된 이 방식은 "아마도 올바른 선택"일 것이다.

백포트된 생성자는 Swift 3.0 네이밍 가이드라인을 따르며, 이 제안을 구현한 후에는 더 이상 변경이 필요하지 않을 것이다.

새 API는 기존의 `strlen` 방식으로 계속 작동하는 오버로드를 제공하면서도, `UnsafeBufferPointer`를 통해 임의의 코드 유닛 시퀀스를 지정할 수 있게 한다. 이러한 저수준 성능 이점은 성능에 민감한 코드에서 매우 중요하다. 알려지지 않은 길이의 버퍼에서 읽을 때, 복사본을 최소로 유지하는 것이 핵심이다.

`String.Type._fromWellFormedCodeUnitSequence(_:input:)`의 사용은 새로운 공개 API로 대체되었다.


## 기존 코드에 미치는 영향

`String.Type.fromCString(_:)`와 `String.Type.fromCStringRepairingIllFormedUTF8(_:)`는 각각 `String.init(validatingCString:)`과 `String.init(cString:)`으로 대체되었다. 이는 앞서 논의한 기본 기대와는 반대되는 변경이라는 점에 유의해야 한다.

기존 메서드는 새로운 시그니처를 사용하도록 업데이트되었으며, Swift 3.0에서 제거될 예정이라는 것을 나타내는 deprecation 속성을 포함하고 있다.


## 고려한 대안들

* 아무것도 하지 않음

이 방법은 최적이 아니다. 많은 사용 사례에서 `String`이 이 생성자를 지원하지 않으면 순수 Swift 구현의 성능에 제약이 생긴다.

* `String.UTF8View`와 `String.UTF16View` 솔루션

(참고: "`String.append(_:)` 속도 향상")

`String.UTF8View`와 `String.UTF16View`를 `String.UnicodeScalarView`처럼 가변적으로 만들고, `append(_:)`와 `appendContentsOf(_:)`를 분할 상환 O(1) 시간 복잡도로 구현한다. 특히 `String.UTF16View`의 경우, `String.UnicodeScalarView`에서 `append(_:)`를 가져오는 간단한 변경으로 충분하다. 이 방법은 `String.Type._fromWellFormedCodeUnitSequence(_:input:)`를 대체하는 등 고급 사용 사례에 적합하다.

API 유지 관리 측면에서 이 방법이 장기적으로 더 나은 해결책일 수 있지만, 현재 제안하는 방법은 영향도가 비교적 낮다.

* 프로토콜 지향 API

`SequenceType`에 `func decode<Encoding>(_:)`와 같은 메서드를 추가한다. 이 메서드가 문자열 처리와 직접적으로 관련이 있는지 명확하지 않으며, `where Generator.Element: UnsignedIntegerType`과 같은 타입 제약을 추가해야 할 수도 있다. 하지만 이는 기존에 존재하지 않는 타입 제약을 도입하는 것이다.

* `NSString` 브리지 속도 향상

브리지 코드를 읽어봤지만 왜 느린지 정확히 알 수 없다. 아마도 버그일 가능성이 있다.

* `String.append(_:)` 속도 향상

`_StringCore`의 성장 전략을 완전히 이해하지는 못했지만, `reserveCapacity(_:)`를 사용해도 문서화된 분할 상환 O(1) 시간 복잡도를 보이지 않는 것 같다. 사전 논의에서 한 사용자가 `reserveCapacity`가 아무런 동작을 하지 않는 것처럼 보인다고 언급했다.




