---
title: "[SE-0034] 디버깅 식별자와 라인 제어 문장 구분하기"
date: 2025-03-09T01:12:52Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 디버깅 식별자와 라인 제어 문장 구분하기

* 제안: [SE-0034](0034-disambiguating-line.md)
* 작성자: [Erica Sadun](https://github.com/erica)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [근거](https://forums.swift.org/t/accepted-se-0034-disambiguating-line-control-statements-from-debugging-identifiers/1614)
* 버그: [SR-840](https://bugs.swift.org/browse/SR-840)


## 소개

Swift Evolution SE-0028(0028-modernizing-debug-identifiers.md)이 채택되면서, `#line`의 사용이 오버로딩되었다. 이는 파일 내 호출 지점의 라인 번호를 매핑하는 식별자 역할과 라인 제어 문장의 일부 역할을 동시에 수행한다. 이 제안은 파일과 라인 문법적 소스 제어를 위해 `#line`을 대체할 `#setline`을 제안한다.

이 논의는 [*\[Discussion\]: Renaming #line, the line control statement*](https://forums.swift.org/t/discussion-renaming-line-the-line-control-statement/1307/24) 스레드에서 온라인으로 진행되었다.

[리뷰](https://forums.swift.org/t/review-se-0034-disambiguating-line-control-statements-from-debugging-identifiers/1477)

[개정](https://forums.swift.org/t/accepted-se-0034-disambiguating-line-control-statements-from-debugging-identifiers/1614/3)


## 동기

Swift는 라인 제어 문을 정의하기 위해 다음과 같은 문법을 사용한다:

```
line-control-statement → #line
line-control-statement → #line line-number file-name
line-number → 0보다 큰 10진수 정수
file-name → static-string-literal
```

SE-0028의 구현에서는 `#line`(제어문)이 첫 번째 열에 위치해야 한다는 요구사항을 통해 두 가지 용도를 명확히 구분한다. 이는 임시적인 해결책으로, `#line`을 재명명하는 것이 더 나은 방법이다.

코어 팀은 `#line`을 오버로딩하기 위해 '줄의 첫 번째 토큰'이라는 공백 규칙을 요구하는 것에 만족하지 않았다. Chris Lattner는 기존 `#line` 지시자를 더 구체적이고 목적에 맞는 이름으로 변경하는 것에 대해 논의할 것을 요청했다: "이름과 문법이 결정되면, 지시자를 재명명하고 공백 규칙을 제거할 수 있다."


## 상세 설계

```
line-control-statement → #setline
line-control-statement → #setline line-number file-name
line-number → 0보다 큰 정수
file-name → 정적 문자열 리터럴
```


## 고려한 대안들

더 유연한 문법이 제안되었지만, Lily Ballard가 지적한 바와 같이:

> 이 기능은 최종 사용자가 사용할 것이 아니다. 또한 `#file`과 `#line` 외에는 합리적으로 적용될 수 있는 것이 없다. 이 기능은 소스 파일을 자동 생성하는 도구에서만 사용하기 위해 고안되었다. 여기서 가장 중요한 고려 사항은 가장 단순한 도구에서도 정확하게 생성하기 쉽고 가독성이 좋아야 한다는 것이다. 그리고 이 기능이 `#file`과 `#line`을 넘어서는 어떤 것에도 적용되지 않기 때문에 이 기능을 일반화하려고 시도할 필요가 전혀 없다.

다양한 다른 키워드가 논의에서 제안되었으며, 온라인 토론에서 확인할 수 있다.


## 수용된 폼과 수정된 설계

라인 제어 문법의 수용된 구문은 다음과 같이 변경된다:

```swift
#sourceLocation(file: "foo", line: 42) 
#sourceLocation()    // 원래 위치로 재설정. 
```

* `#`-네임스페이스에서 식별자의 명명 및 대소문자 표기를 합리화하는 방법을 논의한 후, 코어 Swift 팀은 식별자에 [로어 캐멀 케이스](https://en.wikipedia.org/wiki/CamelCase) 모델을 채택했다. 라인 제어 문법은 로어 캐멀 케이스를 사용하며 `#sourceLocation`으로 이름이 변경된다.

* `#setline`의 구문은 다른 #-지시문과 일관성이 없었다. 괄호를 사용하지 않았기 때문이다. 논의 후, 코어 팀은 `file`과 `line` 인자를 위해 괄호와 쉼표로 구분된 콜론으로 구분된 인자와 값 쌍을 사용하도록 호출을 조정했다.




