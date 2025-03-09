---
title: "[SE-0009] 인스턴스 멤버 접근 시 self 사용 강제화(Rejected)"
date: 2025-03-09T00:17:33Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 인스턴스 멤버 접근 시 self 사용 강제화

* 제안: [SE-0009](0009-require-self-for-accessing-instance-members.md)
* 작성자: [David Hart](https://github.com/hartbit)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **거부됨**
* 결정 노트: [Rationale](https://forums.swift.org/t/rejected-se-0009-require-self-for-accessing-instance-members/930)


## 소개

현재 Swift 버전(2.1)에서는 클로저 내에서 인스턴스 멤버에 접근할 때 `self`를 사용해야 한다. 이 제안은 이를 모든 멤버 접근으로 확장하는 것을 제안한다(이것은 Objective-C에서 본질적으로 그렇듯이). 이는 인스턴스 프로퍼티와 로컬 변수를 구분하고, 인스턴스 함수와 로컬 함수 또는 클로저를 구분하는 데 유용하다.

[Swift Evolution 토론 스레드](https://forums.swift.org/t/proposal-re-instate-mandatory-self-for-accessing-instance-properties-and-functions/125)


## 동기

이 제안은 인스턴스 프로퍼티와 로컬 변수, 그리고 인스턴스 함수와 로컬 함수/클로저를 명확히 구분한다. 이는 여러 가지 장점을 가진다:

* 사용 지점에서 가독성이 높아진다.
* 클로저 문맥에서만 `self`를 요구하는 것보다 일관성이 있다.
* 학습 관점에서 혼란을 줄인다.
* 개발자가 로컬 변수를 사용하려고 했지만 실수로 인스턴스 프로퍼티를 사용하거나(또는 그 반대), 컴파일러가 경고를 통해 버그를 방지할 수 있다.

이 제안으로 방지할 수 있는 버그의 예시([Rudolf Adamkovic 제공](https://forums.swift.org/t/proposal-re-instate-mandatory-self-for-accessing-instance-properties-and-functions/125/4)):

```swift
class MyViewController : UIViewController {
	@IBOutlet var button: UIButton!
        var name: String = "David"

	func updateButton() {
		// var title = "Hello \(name)"
		button.setTitle(title, forState: .Normal) // 이 줄을 주석 처리하지 않았지만 컴파일러는 경고하지 않고, 실수로 UIViewController의 title을 참조하게 됨
		button.setTitleColor(UIColor.blackColor(), forState: .Normal)
	}
}
```

API 디자인 가이드라인은 API 작성용이지만, 여전히 Swift의 기본 원칙을 대표한다고 생각한다. 첫 두 가지 원칙은 다음과 같다:

* 사용 지점에서의 명확성이 가장 중요한 목표다. 코드는 작성하는 것보다 읽는 경우가 훨씬 많다.
* 명확성이 간결함보다 중요하다. Swift 코드는 간결할 수 있지만, 가능한 한 적은 문자로 코드를 작성하는 것은 목표가 아니다. Swift 코드의 간결성은 강력한 타입 시스템과 보일러플레이트를 자연스럽게 줄이는 기능의 부산물이다.

이 제안은 이러한 목표와 직접적으로 일치한다고 믿는다.


## 반론

커뮤니티의 두 멤버가 제기한 반론은 현재 동작이 "클로저 내에서 self의 캡처 의미를 더 두드러지게 만든다"는 것이다. 이는 사실이지만, 저자는 그 유용성이 부족하다고 생각한다.

다음 코드에서 우리는 `foobar`가 예외를 던지는 함수이고 `barfoo`는 예외를 던지지 않는다는 것을 명확히 알 수 있다.

```swift
try foobar()
barfoo()
```

그러나 클로저 내에서 `self`를 사용하는 예제를 보자:

```swift
foobar({
	print(self.description)
})
```

위 코드에서 `self` 키워드는 힌트를 제공하지만 확실한 정보를 제공하지는 않는다:

* `self`는 컴파일러에 의해 메모리 문제 가능성을 암시하기 위해 강제되었을 수 있다.
* 클로저가 non-escaping이라면 `self`는 프로그래머의 선택일 수 있다.

반대 예제를 살펴보자:

```swift
barfoo({
	print(description)
})
```

* 클로저가 non-escaping일 수 있다.
* `description`은 지역 변수를 참조할 수 있으며, 이는 escaping 클로저에서 인스턴스 프로퍼티를 가린 것일 수 있다.

이 두 예제에서 `self` 키워드는 호출된 함수의 시그니처를 확인하지 않고서는 참조 순환 문제에 대해 주의해야 하는지 여부를 확실히 알려주지 않는다. 단지 self가 캡처되었음을 알려줄 뿐이다. 제안하는 방식에서는 `self`가 다시 의미를 갖게 된다: 어떤 것이 지역 프로퍼티이고 어떤 것이 인스턴스 프로퍼티인지 나타낸다.


## 제안하는 해결책

인스턴스 프로퍼티와 함수에 접근할 때 `self`를 사용하지 않는 방식은 두 단계로 적용할 것을 제안한다. Swift 2.x에서는 경고로 시작하고, Xcode가 Fix-It을 제공할 수 있다. 그런 다음 Swift 3에서는 컴파일러 오류로 변경되고, 마이그레이터가 코드를 전환하는 데 도움을 줄 수 있다.

다음 코드는 이전에는 컴파일되었지만, 주석이 달린 줄에서 오류를 발생시킨다:

```swift
class Person {
	var name: String = "David"
	
	func foo() {
		print("Hello \(name)") // 컴파일되지 않음
	}
	
	func bar() {
		foo() // 컴파일되지 않음
	}
}
```

이 코드는 다음과 같이 수정해야 정상적으로 컴파일된다:

```swift
class Person {
	var name: String = "David"
	
	func foo() {
		print("Hello \(self.name)")
	}
	
	func bar() {
		self.foo()
	}
}
```


## 기존 코드에 미치는 영향

이 제안은 기존 코드에 상당한 영향을 미친다. 하지만 마이그레이션 도구와 Xcode의 Fix-It 기능으로 쉽게 해결할 수 있다.


## 고려된 대안들

현재 동작을 유지하는 방법도 하나의 대안이다. 하지만 이 경우 앞서 언급한 단점들이 여전히 존재한다.

다른 대안으로는 컴파일 오류를 경고 수준으로 낮추는 방법이 있다.


## 커뮤니티 반응

* "이 암시적인 'self' 동작 때문에 앱에서 적어도 두 가지 버그를 마주했다. 위험할 뿐만 아니라 추적하기도 어렵다." -- Rudolf Adamkovic, salutis@me.com
* "이러한 이유로 일부 팀은 iVars에 언더스코어를 사용한다. 아쉬운 일이다. 나는 가능한 한 명시적으로 self를 사용한다. 언어가 우리에게 명확함을 요구했으면 좋겠다." -- Dan, robear18@gmail.com
* "이것이 얼마나 많은 Swift 사용자에게 영향을 미치는지 모르겠지만, 나는 색맹이라 지역 변수와 프로퍼티의 구문 색상 구분이 매우 어렵다." -- Tyler Cloutier, cloutiertyler@aol.com
* "+1 함수 인자와 프로퍼티 이름이 같아서 발생한 이상한 문제를 여러 번 겪었다. 모던 Objective-C에서는 거의 이런 문제를 겪지 않았다." -- Colin Cornaby, colin.cornaby@mac.com
* "교육적인 측면에서 self를 필수로 요구하면 학생들이 인스턴스 프로퍼티와 지역 변수를 혼동하지 않는다. 특히 클로저에서 self가 필수일 때 학생들이 혼란스러워한다. 모든 인스턴스 프로퍼티에 self를 필수로 하면 훨씬 명확하고 읽기 쉬워진다." -- Yichen Cao, ycao@me.com
* "이렇게 하면 혼란을 피할 수 있고, 언어 접근 방식을 일관되게 유지하며, 버그를 줄이는 데 도움이 된다. 물론 시적인 하이쿠 코드를 덜 쓸 수 있지만, 중대형 소프트웨어 제품에서는 여러 사람이 작업하고 시간이 지남에 따라 프로젝트 인원이 바뀔 수 있으므로 반드시 나쁜 것만은 아니다." -- Panajev
* "다른 사람들이 이미 말한 이유로 이에 동의하지만, 1년 전만큼 강력하게 동의하지는 않는다. Swift 1이 처음 출시되었을 때 이 문제가 매우 걱정되었지만, 그 이후로 실제로 이런 실수를 한 적이 없다. 아마도 내가 이 문제에 대해 너무 신경을 써서일 것이다." -- Michael Buckley, michael@buckleyisms.com




