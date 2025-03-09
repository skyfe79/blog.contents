---
title: "[SE-0028] Swift의 디버깅 식별자 현대화"
date: 2025-03-09T01:06:36Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## Swift의 디버깅 식별자 현대화

* 제안: [SE-0028](0028-modernizing-debug-identifiers.md)
* 작성자: [Erica Sadun](https://github.com/erica)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 2.2)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0028-modernizing-swifts-debugging-identifiers-line-etc/1303)
* 버그: [SR-669](https://bugs.swift.org/browse/SR-669)


## 소개

이 제안은 Swift에서 `__FILE__` 및 `__FUNCTION__`과 같은 "스크리밍 스네이크 케이스" 사용을 없애고, 이러한 식별자 인스턴스를 일반적인 옥토소프 접두사가 붙은 소문자 `#identifier` 표현으로 대체하는 것을 목표로 한다.

*이 주제에 대한 Swift-Evolution 논의는 "[Review] SE-0022: Referencing the Objective-C selector of a method" 스레드에서 시작되었고, 이후 "[\[Proposal\] Eliminating Swift's Screaming Snake Case Identifiers](https://forums.swift.org/t/proposal-eliminating-swifts-screaming-snake-case-identifiers/1165)" 스레드에서 계속되었다.*


## 동기

Swift는 기본적으로 `__FILE__`, `__LINE__`, `__COLUMN__`, `__FUNCTION__`, 그리고 `__DSO_HANDLE__` 식별자를 제공한다. 처음 네 개는 현재 소스 코드 위치에 해당하는 문자열과 정수 리터럴로 확장된다. 마지막 식별자는 현재 동적 공유 객체(.dylib 또는 .so 파일)에 대한 `UnsafePointer`를 제공한다. 이러한 기능은 로깅에 매우 유용하며, 실행 흐름을 추적하거나 개발자가 [에러 컨텍스트를 캡처](http://ericasadun.com/2015/08/27/capturing-context-swiftlang/)하는 데 도움을 준다.

현재의 식별자들은 C 언어의 `__FILE__`과 `__LINE__` 매크로에서 유래한 구문을 사용한다. 이 매크로들은 C의 전처리기에 내장되어 있으며, C 언어 파서가 실행되기 전에 확장된다. Swift의 구현은 C와 다르지만 유사한 기능을 제공하며, 안타깝게도 비슷한 심볼을 사용한다. 이 제안은 역사적으로 이어져 온 보기 흉한 스네이크 케이스(예: `__FILE__`)에서 벗어나고자 한다. 이러한 심볼들은 마치 [키워드를 삼키려는 보아뱀](https://s-media-cache-ak0.pinimg.com/originals/59/ea/ee/59eaee788c31463b70e6e3d4fca5508f.jpg)처럼 보이기 때문이다.


## 제안하는 솔루션

옥토소프(octothorpe) 접두사를 사용한 키워드는 여러 가지 장점을 제공한다:

* 기존 `#available` 키워드와 일관성을 유지한다 (D. Gregor)
* 이미 채택된 SE-0022의 `#selector(...)` 접근 방식과 일치한다. 이 방식은 메서드의 Objective-C 셀렉터를 참조한다 (D. Gregor)
* 타겟팅된 코드 완성을 지원한다 (D. Gregor)
* 키워드를 사용하지 않고 컴파일러가 지원하는 표현 타입을 추가한다. `#`은 "여기서 컴파일러 대체 로직을 실행하라"는 관례를 도입한다 (J. Rose)
* 아직 설계되지 않은 매크로 시스템에 대한 단기적인 해결책을 제공한다 (D. Gregor)


## 상세 설계

이 제안서는 다음과 같은 식별자의 이름을 변경한다:

* `__FILE__` -> `#file`
* `__LINE__` -> `#line`
* `__COLUMN__` -> `#column`
* `__FUNCTION__` -> `#function` (*리뷰 중 추가됨*)
* `__DSO_HANDLE__` -> `#dsohandle`

이 식별자들은 기존 `__LINE__` 기능의 마법 같은 동작을 그대로 유지한다. 일반 표현식 컨텍스트에서는 해당 지점의 위치로 확장된다. 기본 인자 컨텍스트에서는 호출자의 위치로 확장된다.

Swift 팀이 고려해야 할 추가 사항은 다음과 같다:

* `#file` 참조에서 `lastPathComponent`를 사용하지 않기 위해 `#filename`을 추가한다.
* `__FUNCTION__`을 `#function`으로 이름을 바꾸어 유지한다. (*리뷰 중 승인됨*)
* `#dsohandle`과 향후 추가될 수 있는 `#sourcelocation`을 포함해 소문자 네이밍 표준을 채택한다.
* `#symbol`을 도입한다. 예를 들어 `Swift.Dictionary.Init(x:Int,y:String)`와 같이 모듈, 타입, 함수를 포함한 컨텍스트를 요약한다. 완전히 정규화된 심볼은 사용자가 원하는 정확한 정보에 접근할 수 있게 한다. 멤버 오버로드를 올바르게 식별하기 위해 파라미터 타입 정보를 포함해야 한다.


## 가능한 미래 확장

[SR-198](https://bugs.swift.org/browse/SR-198)는 기존 식별자를 통합하라는 요청을 제안했다. Swift 팀이 표준화된 소스 위치 타입을 다루기로 결정한다면, 구조화된 `#sourcelocation` 식별자를 추가로 도입할 수 있다. 이는 개별 필드나 키워드에 접근할 수 있는 기능을 제공할 것이다.

요약 기능에 대한 지원으로, Remy Demerest는 다음과 같이 언급했다: "소스 위치가 하나의 객체로 제공되어 전체 내용을 출력할 수 있으면서도 필요에 따라 개별 구성 요소를 사용할 수 있는 아이디어를 매우 좋아한다. 보통 나는 일부 속성만 원하는 경우는 거의 없고, 적절한 형식으로 작성하는 데 시간이 더 걸리기 때문에 포함하지 않는 경우가 많다. 한 번에 모든 내용을 얻을 수 있다면 확실히 이점이 있다."

만약 그러한 타입이 채택된다면, 디버그와 릴리스 로깅을 위해 적절히 구분된 일반적인 출력 요약 표현을 지원할 것을 권장한다. 또는 `#sourcelocation`의 채택과는 별개로 `#context`, `#releasecontext`, `#debugcontext` 요약을 추가할 수도 있다.


## 구현 시 고려사항

Swift에서 이미 존재하는 `#line` 식별자는 줄 번호를 재설정하는 데 사용된다. 현재 `#line` 지시자를 줄 바꿈 후 첫 번째 토큰으로 제한하면 사용 시 명확성을 높일 수 있다. 또는 `#line` 식별자를 `#linenumber`로 변경하는 방법도 있다.


## 리뷰 수락 및 수정 사항

SE-0028 "Swift 디버깅 식별자 현대화"에 대한 리뷰는 2016년 1월 29일부터 2월 2일까지 진행되었다. 이 제안은 수정 사항과 함께 *수락*되었다:

* 코어 팀은 기존의 `__FILE__`, `__LINE__`, `__COLUMN__`, `__FUNCTION__`, `__DSO_HANDLE__` 심볼을 모두 소문자로 변경하고 `#` 네임스페이스에 포함하는 것에 동의했다. 이는 `#file`, `#line`, `#column`, `#function`, `#dsohandle`을 포함한다. 여기서 `__FUNCTION__`을 유지하고, `#line`은 라인의 첫 번째 토큰일 때는 지시문으로 동작하지만 그 외의 경우에는 표현식으로 동작하는 이중 동작을 갖도록 한다. 이러한 심볼 이름 변경은 Swift 언어 내에서의 일관성을 높이며, 모든 심볼을 유지함으로써 기존 구문에서 새로운 구문으로의 원활한 전환 경로를 제공한다.

* 코어 팀은 `#line`이 가지는 "라인의 첫 번째 토큰"이라는 마법 같은 공백 동작에 대해 크게 만족하지 않는다. 이에 대해 더 구체적이고 목적에 맞는 이름으로 기존 `#line` 지시문의 이름을 변경하는 논의를 시작하고자 한다. 이름과 구문이 확정되면 지시문의 이름을 변경하고 공백 규칙을 제거할 수 있다.

* 코어 팀은 `#symbol`을 별도의 제안으로 분리할 것을 요청했다. 이는 더 상세한 설계 작업이 필요하며, 추가 기능이기 때문이다. 예를 들어, 현재 심볼을 맹글링된 이름으로 제공하는 `#mangledname` 표현식을 제공하는 것이 매력적일 수 있다. 이를 디맹글러에 입력하면 현재 심볼의 더 구조화된 형태를 얻을 수 있다.




