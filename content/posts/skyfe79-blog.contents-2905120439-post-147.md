---
title: "[SE-0038] 패키지 매니저 C 언어 타겟 지원"
date: 2025-03-09T01:16:52Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 패키지 매니저 C 언어 타겟 지원

* 제안: [SE-0038](0038-swiftpm-c-language-targets.md)
* 작성자: [Daniel Dunbar](https://github.com/ddunbar)
* 리뷰 관리자: [Rick Ballard](https://github.com/rballard)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0038-package-manager-c-language-target-support/1569)
* 버그: [SR-821](https://bugs.swift.org/browse/SR-821)


## 소개

이 제안은 C, C++, Objective-C, Objective-C++ 언어(이하 단순히 "C" 언어로 통칭)에 대한 초기 패키지 매니저 지원을 추가하는 내용이다. 이 제안의 범위는 C 언어로만 구성된 타겟을 지원하는 데 한정된다. C와 Swift 소스가 혼합된 타겟을 지원하는 내용은 포함하지 않는다.

[Swift Evolution 리뷰 스레드](https://forums.swift.org/t/review-se-0038-package-manager-c-language-target-support/1462)


## 동기

Swift는 Clang 모듈 시스템을 활용해 C 기반 언어와 쉽게 상호 운용할 수 있다. Swift 패키지가 C 타겟을 포함할 수 있게 하여, 이를 단일 패키지의 일부로 Swift에 직접 노출하고자 한다.

이 기능은 개발자에게 Swift와의 연결이 부족하거나 미흡한 API에 접근해야 할 때, 또는 Swift보다 낮은 수준의 C에서 구현하는 것이 더 적합한 동작을 구현할 때 C로 "되돌아갈" 수 있는 간단한 메커니즘을 제공한다.


## 제안하는 솔루션

이 제안은 C 소스 코드로만 구성된 타겟을 허용하도록 기존 컨벤션 기반 시스템을 확장한다. 컨벤션은 다음과 같이 수정된다:

1. 기존 디렉토리에서 타겟을 정의할 때, 파일 확장자로 식별되는 C 소스 코드 집합을 포함할 수 있다. 타겟에 C 소스 코드가 포함된 경우, 모든 소스 코드 파일은 C 소스 코드여야 한다.

   Objective-C와 Objective-C++ 소스 파일 확장자도 지원하지만, 이들은 본질적으로 이식성이 떨어진다.

2. C 타겟은 `Includes` 또는 `include`라는 특수 서브디렉토리를 가질 수 있다. 이 두 이름 중 하나만 사용할 수 있다.

   이 디렉토리의 헤더 파일은 C 타겟의 "내보낸" 인터페이스로 간주되며, 다른 타겟에서 사용할 수 있게 된다.

   `Includes`와 `include`라는 이름은 이 폴더가 공개 API를 정의한다는 것을 명확히 전달하지 못한다는 점에서 다소 아쉽다. 하지만 이는 C 언어 헤더 파일을 구성하는 데 있어 기존에 널리 사용되는 컨벤션이며, 더 나은 대안이 없는 상황이다.

3. include 디렉토리에 타겟 이름과 일치하는 헤더 파일이 있는 경우, 이 헤더 파일은 모듈 맵 구성 시 "우산 헤더"로 처리된다.

4. include 디렉토리에 "module.modulemap"이라는 파일이 있는 경우, 이 파일은 타겟의 모듈을 정의하는 것으로 간주되며, 모듈 맵이 자동으로 생성되지 않는다.

5. Swift 타겟과 마찬가지로, `main.c`(또는 `main.cpp` 등)라는 소스 파일의 존재 여부로 실행 파일과 라이브러리를 구분한다.

다음은 C 라이브러리와 Swift 타겟을 포함하는 패키지를 정의하는 예시 레이아웃이다:

    example/src/foo/include/foo/foo.h
    example/src/foo/foo.c
    example/src/foo/util.h
    example/src/bar/bar.swift

이 예시에서 `util.h`는 `foo` 타겟의 구현에 내부적으로 사용되는 반면, `include/foo/foo.h`는 `foo` 라이브러리의 내보낸 API로 사용된다.

패키지 매니저는 이러한 타겟을 빌드하는 기능을 자연스럽게 지원한다:

1. 패키지 매니저는 (a) Swift에서 사용할 수 있도록 내보낸 API 헤더를 모두 포함한 합성 모듈 맵을 구성하고, (b) 이 디렉토리에 대한 헤더 검색 경로를 이 타겟에 종속된 다른 타겟에 제공한다.

   모듈 맵은 완전히 평평한 헤더 레이아웃(예: ``src/foo/include/*.h``) 또는 단일 서브디렉토리(예: ``src/foo/include/foo/**.h``)를 가진 타겟에 대해서만 합성된다. 다른 구조는 라이브러리 작성자가 명시적으로 모듈 맵을 제공해야 한다. 이 부분은 실제 경험을 통해 재검토할 수 있다.

2. 대부분의 패키지는 include 디렉토리에 패키지 또는 타겟 이름을 서브폴더로 포함하도록 권장한다(예: ``src/foo/include/foo/``). 하지만 이는 필수 사항이 아니며, 전통적으로 헤더를 `/usr/include`에 직접 설치하는 레거시 프로젝트의 경우 이 컨벤션을 사용하지 않는 것이 유용할 수 있다. 이는 해당 프로젝트의 클라이언트 코드가 설치된 라이브러리를 사용하는 버전과 소스 호환성을 유지할 수 있게 한다.

3. C 언어 타겟은 기존 패키지 매니저 기능과 통합될 것으로 기대한다. 예를 들어, C 언어 타겟은 테스트 기능을 사용해 테스트할 수 있어야 한다(비록 이러한 테스트는 C 언어 테스트 프레임워크가 패키지 매니저에서 사용 가능해질 때까지 Swift로 작성해야 할 것이다).

이 제안에서는 다루지 않는 C 언어 지원과 관련된 몇 가지 명백한 기능이 있다:

1. 특정 타겟만 하위 **패키지**에 API를 내보내야 한다는 선언이 필요할 것으로 예상한다(예를 들어, 위 패키지는 `bar` 타겟을 클라이언트에 내보내고 C 타겟을 구현 세부 사항으로 유지할 수 있다).

2. 이 제안에서는 컴파일러 인수를 제어하는 방법을 제공하지 않는다. 기존 디버그 및 릴리스 구성에 대해 고정된 컴파일러 플래그 집합을 사용해 지원할 것이다. 이러한 플래그를 수정해야 할 필요성을 고려한 향후 제안을 기대한다.

3. 이 기능은 표준을 준수하는 모든 C 컴파일러를 지원할 수 있도록 구축할 예정이지만, 주로 Clang을 컴파일러로 사용하는 것을 지원할 것이다(물론 모듈 지원은 Clang을 필요로 한다).

4. 타겟이 자체 모듈 맵을 정의해야 할 필요가 있을 수 있다. 필요한 경우, 이는 ``include`` 디렉토리에 ``module.modulemap``으로 추가될 것으로 예상한다. 하지만 초기 기능이 구현되고 사용 사례가 명확해질 때까지 이 지원의 구현을 미룰 예정이다.


## 상세 설계

패키지 매니저는 다음과 같은 추가 동작을 수행한다:

1. 프로젝트 모델을 확장하여 C 언어 타겟을 탐지하고 문제를 진단한다. 예를 들어, C와 Swift 소스 파일이 혼합된 경우를 확인한다.

2. 빌드 시스템을 확장하여 각 C 언어 타겟을 컴파일하고 링크한다. `llbuild`의 기존 C 소스 코드 컴파일 지원을 활용하고, GCC 호환 컴파일러 스타일의 헤더 의존성 정보를 수집하여 증분 빌드를 수행한다.

3. 타겟을 빌드할 때, 패키지 매니저는 빌드 중인 타겟의 의존성 전이 폐쇄에 있는 각 C 언어 타겟의 include 디렉토리에 대한 추가 헤더 검색 경로 인자를 자동으로 추가한다.

   현재 패키지 내에 있는 타겟에는 `-iquote`를, 패키지 의존성에서 오는 타겟에는 `-I`를 헤더 검색 인자로 사용한다. 이를 통해 프로젝트는 패키지 내부와 외부 헤더를 구분하기 위해 `#include "foo/foo.h"`와 `#include <foo/foo.h>` 구문을 적절히 사용할 수 있다.

4. include가 있는 각 C 언어 타겟에 대해 모듈 맵 파일을 합성한다. 모듈 맵은 include 디렉토리에 있는 모든 헤더를 명시적으로 열거하여 구성한다. 결정론적 동작을 보장하기 위해 이를 사전순으로 정렬하지만, 문서에서는 각 API를 설명하는 헤더가 "독립적으로" 포함될 수 있어야 함을 사용자에게 전달한다. 즉, 어떤 순서로도 포함될 수 있어야 한다.

5. Swift 타겟을 빌드할 때, 의존성 폐쇄에 있는 각 C 언어 타겟에 대해 합성된 모듈 맵을 Clang에 명시적으로 전달한다(`-fmodule-map-file=<PATH>` 인자 사용). 이를 통해 Swift Clang 임포터가 헤더 검색 메커니즘을 통해 찾지 않고도 해당 모듈을 찾을 수 있다.

이 지원은 Clang뿐만 아니라 GCC 호환 컴파일러를 사용하여 C 타겟을 빌드할 수 있도록 설계되었다.


## 기존 패키지에 미치는 영향

이전에는 지원되지 않았기 때문에 기존 패키지에 심각한 영향은 없다. 기존 패키지에서 C 계열 소스 코드를 빌드하려고 시도하지만, 이는 바람직한 변화일 것이다.

위에서 언급한 규칙을 따르지 않는 기존 C 언어 프로젝트에 미치는 영향을 고려할 필요가 있다.

* 대부분의 프로젝트는 이러한 규칙을 따르지 않는다. 그러나 이는 모든 "간단한" 규칙에서 예상되는 현상이다. 기존 C 언어 프로젝트의 상당 부분이 "그냥 작동"할 수 있도록 하는 다른 간단한 규칙이 존재하지 않는다고 판단한다.

  C 타겟을 설명하는 매니페스트 파일에 특정 오버라이드를 허용하여, 올바른 `Package.swift` 파일만 추가하면 일부 프로젝트가 패키지 매니저와 함께 작동하도록 할 계획이다. 기존 C 프로젝트에 대해 옵션을 테스트할 수 있게 되면 허용할 정확한 오버라이드를 결정할 것이다.

  패키지 매니저는 이미 기존 프로젝트를 지원하기 위해 명시적으로 설계된 "시스템 모듈" 패키지에 대한 지원을 제공한다. 이 제안에서 설명한 C 언어 타겟 지원은 Swift 프로젝트를 지원하기 위해 작성된 새로운 C 코드를 대상으로 한다. 깔끔하고 간단한 규칙을 채택하는 것이 이러한 목표를 지원하는 최선의 접근 방식이라고 생각한다.

* 기존 프로젝트를 *사용하는* 기존 소스 코드(예: `libpng`를 사용하는 소스 파일)는 수정 없이 잘 구성된 패키지를 사용할 수 있다. 이는 상당한 장점으로 간주된다. 왜냐하면 업스트림 프로젝트가 주요 트리에 적절한 패키지 매니저 지원을 통합하는 데 도움이 될 수 있기 때문이다.


## 비모듈 헤더의 위험성

이 제안의 일환으로, C 언어 타겟에 대한 모듈 맵을 합성하여 Clang AST 임포터를 통해 Swift에서 사용할 수 있도록 한다.

이 제안의 주요 위험 요소는 여러 C 언어 타겟이 Swift 모듈로 임포트될 때, 정의된 모듈 맵이 없는 공통 C 헤더를 참조하는 경우다. 이러한 헤더를 "비모듈 헤더"라고 한다. 이 상황은 Linux에서 자주 발생할 수 있다. Linux 시스템은 일반적으로 공통 헤더에 대한 모듈 맵을 제공하지 않기 때문이다. 또한 OS X에서도 타겟이 패키지 생태계에 속하지 않은 타사 헤더를 참조할 때 발생할 수 있다.

이러한 상황에서 현재 컴파일러의 동작은 사용자가 이해하기 어렵다. 컴파일러가 기대하는 모델은 모든 콘텐츠가 모듈화되어 있다는 것이다. 하지만 이 모델이 깨지고 두 모듈이 중복된 콘텐츠를 포함할 경우, (a) 모듈 구현의 미묘한 차이로 인해 때로는 정상적으로 동작할 수 있고, (b) 컴파일러가 이를 진단하지 않는다. 이로 인해 혼란스러운 오류가 발생하거나, 컴파일러가 더 엄격해질 경우 의도치 않은 패키지 변경으로 이어질 수 있다.

궁극적으로, 이 문제에 대한 예상되는 해결책은 Swift에서 사용하는 모든 콘텐츠에 대해 더 많은 모듈 맵을 제공하는 작업을 계속 진행하는 것이다. 이 제안을 통합하면 해당 작업이 더 높은 우선순위를 갖게 되고, 컴파일러 구현에서 해결해야 할 추가 문제가 드러날 수 있다.


## 고려한 대안

### C 언어 지원 생략

C 언어 타겟을 직접 지원하지 않고, 외부 빌드 시스템과 기능을 활용해 Swift 패키지에 통합하는 방안을 고려했다. 이러한 기능을 이 제안과 별도로 추가할 수도 있지만, C 타겟에 대한 기본 지원을 제공하는 것이 가치 있다고 판단했다. 이는 Swift 프로젝트 내에서 소량의 C 코드를 쉽게 통합할 수 있게 해주며, Swift 패키지 매니저가 Swift 외의 프로젝트에서도 유용하게 사용될 수 있도록 하는 장기적인 방향과도 일치한다.


### C 언어 타겟 헤더 레이아웃 제한 사항

각 `include` 디렉토리가 타겟 이름과 일치하는 하위 디렉토리를 가져야 한다는 명명 규칙을 요구하는 방안을 고려했다. 이 방식은 해당 타겟의 모든 클라이언트가 `#include <foo/header.h>`와 같은 구문으로 헤더를 포함하도록 보장한다는 장점이 있다.

하지만 이 방식은 전통적인 C 라이브러리에서 헤더를 이 형식으로 설치하지 않는 경우 패키지의 사용성을 떨어뜨린다. 또한 프로젝트가 자체 헤더에 추가적인 조직화를 적용하는 능력도 제한한다. 예를 들어, LLVM은 최상위 헤더를 `llvm`과 `llvm-c`(C++ API와 C API를 구분하기 위해)로 나누는 규칙을 따르고 있다.

헤더 레이아웃을 제한해야 할 다른 기능이나 개발 영역이 없었기 때문에, 불필요한 제한을 가하는 것보다 이를 단순히 권장 사항으로 처리하는 것이 더 나은 선택이라고 판단했다.




