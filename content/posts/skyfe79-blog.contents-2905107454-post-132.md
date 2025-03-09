---
title: "[SE-0023] API 설계 가이드라인"
date: 2025-03-09T00:42:07Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## API 설계 가이드라인

* 제안: [SE-0023](0023-api-guidelines.md)
* 작성자: [Dave Abrahams](https://github.com/dabrahams), [Doug Gregor](https://github.com/DougGregor), [Dmitri Gribenko](https://github.com/gribozavr), [Ted Kremenek](https://github.com/tkremenek), [Chris Lattner](https://github.com/lattner), Alex Migicovsky, [Max Moiseev](https://github.com/moiseev), Ali Ozer, [Tony Parker](https://github.com/parkera)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-with-modifications-se-0023-api-design-guidelines/1666)


## 리뷰어 노트

이 리뷰는 서로 밀접하게 연관된 세 가지 리뷰 중 하나로, 동시에 진행된다:

* [SE-0023 API 디자인 가이드라인](0023-api-guidelines.md)
  ([리뷰](https://forums.swift.org/t/review-se-0023-api-design-guidelines/1162))
* [SE-0006 표준 라이브러리에 API 가이드라인 적용](0006-apply-api-guidelines-to-the-standard-library.md)
  ([리뷰](https://forums.swift.org/t/review-se-0006-apply-api-guidelines-to-the-standard-library/1163))
* [SE-0005 Objective-C API를 Swift로 더 나은 번역](0005-objective-c-name-translation.md)
  ([리뷰](https://forums.swift.org/t/review-se-0005-better-translation-of-objective-c-apis-into-swift/1164))

이 리뷰들은 서로 강하게 연관되어 있기 때문에 동시에 진행된다 (예: 표준 라이브러리의 API 변경은 특정 가이드라인에 해당하거나, 임포터 규칙이 특정 가이드라인을 구현하는 등). 이러한 상호작용과 논의를 관리하기 위해 다음과 같은 사항을 요청한다:

* **리뷰 코멘트를 작성하기 전에 세 문서의 기본 내용을 이해한다.**
* **각 문서에 대한 리뷰는 해당 리뷰 발표에 대한 응답으로 작성한다.** 문서 간의 상호 참조를 통해 의견을 명확히 전달하는 것은 괜찮으며, 권장한다.


## 서론

널리 사용되는 라이브러리의 설계는 프로그래밍 언어의 전반적인 느낌에 큰 영향을 미친다. 훌륭한 라이브러리는 마치 언어 자체의 확장처럼 느껴지며, 라이브러리 간의 일관성은 전체 개발 경험을 한층 더 높여준다. 우수한 Swift 라이브러리를 구축하는 데 도움을 주기 위해, Swift 3의 주요 목표 중 하나는 API 설계 가이드라인을 정의하고 이러한 설계 원칙을 일관되게 적용하는 것이다.


## 제안하는 솔루션

제안하는 API 디자인 가이드라인은 [https://swift.org/documentation/api-design-guidelines/](https://swift.org/documentation/api-design-guidelines/)에서 확인할 수 있다.

이 가이드라인의 소스 코드는 [https://github.com/apple/swift-internals](https://github.com/apple/swift-internals)에서 확인할 수 있다. 간단한 편집 변경에 대한 풀 리퀘스트는 언제든 환영한다. 더 중요한 변경 사항은 리뷰 프로세스의 일부로 처리해야 한다.


## 기존 코드에 미치는 영향

API 설계 가이드라인의 존재는 기존 코드에 특별한 영향을 미치지 않는다. 그러나 이 가이드라인을 [표준 라이브러리](0006-apply-api-guidelines-to-the-standard-library.md)에 적용하는 동반 제안과 [Clang 임포터](0005-objective-c-name-translation.md)를 통해 적용하는 제안은 기존 코드에 큰 영향을 미칠 것이다. 이 두 제안은 상당수의 API를 변경하게 된다.




