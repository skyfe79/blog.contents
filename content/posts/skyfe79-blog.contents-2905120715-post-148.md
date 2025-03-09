---
title: "[SE-0039] 플레이그라운드 리터럴 현대화"
date: 2025-03-09T01:17:41Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 플레이그라운드 리터럴 현대화

* 제안: [SE-0039](0039-playgroundliterals.md)
* 작성자: [Erica Sadun](https://github.com/erica)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [이유](https://forums.swift.org/t/accepted-se-0039-modernizing-playground-literals/1746)
* 버그: [SR-917](https://bugs.swift.org/browse/SR-917)


## 소개

Playground 리터럴은 색상, 파일, 이미지를 토큰화한다. 드래그 앤 드롭 기능을 제공하며, 컨텍스트 내 시각화를 통해 플레이그라운드 콘텐츠를 설계할 때 쉽게 참고하고 조작할 수 있도록 돕는다. 현재 형태의 이 리터럴은 간단한 대괄호 구문을 사용하지만, 이는 컬렉션 리터럴과 충돌하는 문제가 있다. 이 제안은 #available과 #selector의 선례를 따라 플레이그라운드 리터럴을 재설계한다.

*이 논의는 Swift Evolution 메일링 리스트의 [\[Discussion\] Modernizing Playground Literals](https://forums.swift.org/t/discussion-modernizing-playground-literals/1443) 스레드에서 진행되었다. 이 개선안을 제안한 [Chris Lattner](https://github.com/lattner)에게 감사드린다.*

[리뷰](https://forums.swift.org/t/review-se-0039-modernizing-playground-literals/1707)


## 동기

현재 색상, 이미지, 파일 리터럴은 다음과 같이 표현된다:

```swift
[#Color(colorLiteralRed: red, green: green, blue: blue, alpha: alpha)#]
[#Image(imageLiteral: localResourceNameAsString)#]
[#FileReference(fileReferenceLiteral: localResourceNameAsString)#]
```

플레이그라운드 리터럴은 다음과 같은 특징을 가진다:

* `[# #]`로 둘러싸인 컨테이너 안에 나타난다.
* 대문자 카멜 케이스 역할 이름으로 표시된다.
* 첫 번째 라벨 인수는 *literal*이라는 단어를 사용해 각 리터럴 아이템의 구성을 강조한다.

이 접근 방식에는 몇 가지 문제가 있다:

* 감싸는 대괄호가 컬렉션 리터럴과 충돌하여 파싱에 추가 작업이 필요하다.
* 구문이 모던 Swift 컨벤션을 따르지 않는다.
* *literal*이라는 단어는 생성된 아이템을 설명하는데, 리터럴을 생성하기 위해 전달되는 인수를 설명하는 데 사용된다. 현재 사용 방식이 잘못되었다.


## 상세 설계

생성자를 [옥토소프](https://en.wikipedia.org/wiki/Octothorpe)로 구분된 식별자로 단순화하면 언어가 깔끔해지고, 문법 충돌 가능성을 줄이며, 모던 Swift에서 사용되는 다른 식별자와 일관성을 유지한다. 제안하는 식별자는 `#colorLiteral`, `#imageLiteral`, `#fileLiteral`이다.

```swift
color-literal → #colorLiteral(red: unit-floating-point-literal, green: unit-floating-point-literal, blue: unit-floating-point-literal, alpha: unit-floating-point-literal)
unit-floating-point-literal → 0 이상 1 이하의 부동 소수점 숫자

image-literal → #imageLiteral(resourceName: image-resource-name)
image-resource-name → 이미지 리소스 이름을 참조하는 정적 문자열 리터럴

file-literal → #fileLiteral(resourceName: file-resource-name)
file-resource-name → 로컬 리소스 이름을 참조하는 정적 문자열 리터럴
```

이 설계에서:

* 각각의 리디자인된 식별자는 기존 Swift 리터럴과 일치하도록 소문자를 사용한다.
* 인자는 관례에 따라 소문자 카멜 케이스 레이블을 사용한다.
* 각 항목의 역할을 나타내기 위해 식별자에 `literal`이라는 단어를 추가한다.
* 인자는 `red`, `green`, `blue`, `alpha`, `resourceName`으로 단순화하고 표준화한다.

그러나 이러한 인자 레이블은 실제 초기화자에는 적합하지 않다. 리터럴 프로토콜의 초기화자는 리터럴에서 사용됨을 명확히 나타내는 인자 레이블을 사용해야 한다. 이는 두 가지 목적을 달성한다. 첫째, 타입이 리터럴에 대해 특수한 동작을 제공하고 싶을 수 있으며, 이는 일반적인 초기화자에는 적합하지 않을 수 있다. 둘째, 리터럴 초기화자는 엄격한 명명 및 타입 규칙을 따라야 하므로, "특이한" 이름을 부여하면 타입의 표준 인터페이스를 오염시키지 않고 모호성을 줄일 수 있다. 이를 고려하여 Swift는 위의 구문을 아래의 해당 초기화자 사용으로 해석한다:

```swift
protocol _ColorLiteralConvertible {
  init(colorLiteralRed red: Float, green: Float, blue: Float, alpha: Float)
}

protocol _ImageLiteralConvertible {
  init(imageLiteralResourceName path: String)
}

protocol _FileReferenceLiteralConvertible {
  init(fileReferenceLiteralResourceName path: String)
}
```


## 고려한 대안

파일 리소스를 설명할 때 `#fileliteral`보다 `#resourceliteral`이 더 적합할 수 있다.




