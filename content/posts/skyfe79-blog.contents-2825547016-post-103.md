---
title: "Swift 빌드 기술의 새로운 장"
date: 2025-02-02T01:50:08Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

> 알림: [The Next Chapter in Swift Build Technologies](https://www.swift.org/blog/the-next-chapter-in-swift-build-technologies/) 를 한국어로 번역한 글입니다.
> 애플에서 흥미로운 소식을 발표했습니다. 새로운 Swift Build 시스템인데요. 앞으로 Xcode 에 묶이지 않고도 애플 생태계의 다양한 앱을 개발할 수 있을 것 같습니다. 너무나 반가운 소식인 것 같아 한국어로 번역해 보았습니다.

Swift는 크로스 플랫폼 언어로서 다양한 활용 사례를 지원하며 그 인기가 계속 높아지고 있다. [다양한 임베디드 기기](https://www.swift.org/blog/embedded-swift-examples/)를 지원하고, [웨어러블](https://developer.apple.com/documentation/watchos-apps/building_a_watchos_app)부터 [서버](https://www.swift.org/documentation/server/)에 이르는 폭넓은 형태의 기기, 그리고 다양한 [운영체제](https://www.swift.org/documentation/articles/static-linux-getting-started.html)에서 작동한다. Swift의 이러한 확장에 발맞추어, 생태계 전반에 걸쳐 강력하고 일관된, 그리고 유연한 경험을 제공하는 크로스 플랫폼 빌드 도구에 투자할 가치가 있다.

Swift 빌드 기술의 새로운 장을 여는 기초 단계로서, 애플은 오늘 [Swift Build](https://github.com/swiftlang/swift-build)를 오픈소스로 공개했다. Swift Build는 Swift 프로젝트 빌드를 위한 규칙 집합을 제공하는 강력하고 확장 가능한 빌드 엔진이다. 이는 앱스토어의 수백만 개 앱을 지원하는 Xcode에서 사용되는 엔진이며, 애플의 자체 운영체제 내부 빌드 과정에도 활용된다. 오픈소스 저장소는 리눅스와 윈도우 환경도 지원한다.

## Swift Build 소개

빌드 시스템의 핵심 임무는 개발자가 작성한 입력물(프로젝트 명세와 소스 코드 등)을 커맨드라인 도구, 라이브러리, 애플리케이션과 같은 출력물로 변환하는 것이다. 빌드 시스템은 개발자에게 뛰어난 개발 경험을 제공하는 데 중요한 역할을 한다. 개발자가 프로젝트를 설계하고 작업하는 방식을 결정하는 고수준 기능을 제공하기 때문이다. 더불어 빌드 시스템의 성능과 신뢰성은 개발자의 생산성에 직접적인 영향을 미친다.

Swift Build는 Swift 패키지 매니저나 Xcode와 같은 상위 수준 클라이언트가 요청한 빌드를 계획하고 실행하도록 설계된 기반 컴포넌트이다. 기존 [llbuild](https://github.com/swiftlang/swift-llbuild) 프로젝트를 토대로 다음과 같은 기능을 추가했다:

* Swift 컴파일러와 견고하게 통합되어 Swift 프로젝트를 안정적이고 효율적으로 빌드하도록 조정한다
* 라이브러리, 커맨드라인 도구, GUI 애플리케이션까지 다양한 제품 유형을 지원하며 고급 빌드 설정 옵션을 제공한다
* Swift와 C 코드를 빌드할 때 병렬 처리를 극대화하는 빌드 그래프 최적화 기능을 제공한다

## Swift Build 로드맵

Xcode의 빌드 엔진과 비교하면 Swift 패키지 매니저의 빌드 엔진은 상대적으로 단순한 구조를 가진다. 애플 플랫폼에서는 패키지를 빌드하는 두 가지 방식이 존재하며, 이 두 구현 방식의 동작이 일치하지 않아 사용자들이 혼란을 겪었다. 이제 Xcode의 빌드 엔진을 Swift 프로젝트에 기여하고 Swift 컴파일러와 함께 오픈소스로 개발함으로써 이러한 문제를 해결하고 모든 Swift 사용자에게 뛰어난 빌드 경험을 제공할 수 있게 되었다.

이번 릴리스를 통해 SwiftPM은 모든 플랫폼에서 통합된 빌드 실행 엔진을 제공할 기회를 얻게 되었다. 이러한 변화는 사용자에게 투명하게 적용되며, 기존의 모든 패키지와 완벽한 호환성을 유지하면서도 일관된 크로스 플랫폼 경험을 제공한다. 동시에 이는 모든 플랫폼과 도구에서 새로운 기능과 개선사항을 가능하게 하는 토대가 되며, 새로운 성능 최적화와 개발자 중심 기능을 실현한다.

이러한 비전을 향한 첫 걸음으로, 오늘 개발팀은 SwiftPM에서 대체 빌드 엔진으로 Swift Build를 지원하기 위한 통합 과정을 시작하는 풀 리퀘스트를 제출했다. 앞으로 수개월 동안 커뮤니티와 협력하여 빌드 시스템 통합 작업을 완료할 계획이다. 이를 통해 사용자들은 모든 플랫폼과 프로젝트 모델에서 향후 도구 개선의 혜택을 누릴 수 있게 된다.

이는 건강한 패키지 생태계를 지속적으로 발전시키는 중요한 단계라고 확신한다. 개발자들은 어떤 IDE를 사용하고 어떤 플랫폼을 대상으로 하든 일관되고 세련된 개발 경험을 누릴 수 있을 것이다. Swift 포럼을 통해 이 작업에 대한 더 자세한 내용을 공유할 예정이며, 다른 개발자들의 피드백도 기대한다!

## Swift Build 프로젝트 참여 안내

Swift 코드 빌드 방식을 함께 발전시켜 나갈 커뮤니티 구성원을 기다리고 있다. GitHub의 Swift 조직에서 [`swift-build` 저장소](https://github.com/swiftlang/swift-build)를 찾을 수 있다. 이 저장소에는 빌드 방법과 기여 방법을 설명하는 README와 문서가 포함되어 있다. 풀 리퀘스트와 이슈를 통한 기여를 환영하며, [Swift 포럼](https://forums.swift.org/c/development/swift-build/)에서 개선을 위한 피드백과 아이디어도 기다린다.

Swift 빌드 시스템의 새로운 장이 시작되었다. Swift가 열어갈 무한한 가능성을 함께 만나보자!