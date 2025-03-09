---
title: "[SE-0003] 함수 매개변수에서 var 제거"
date: 2025-03-08T23:41:34Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 함수 매개변수에서 `var` 제거

* 제안: [SE-0003](0003-remove-var-parameters.md)  
* 작성자: [Ashley Garland](https://github.com/bitjammer)  
* 리뷰 관리자: [Joe Pamer](https://github.com/jopamer)  
* 상태: **구현 완료 (Swift 3.0)**  
* 결정 노트: [Rationale](https://forums.swift.org/t/se-0003-removing-var-from-function-parameters-and-pattern-matching/1230)  
* 구현: [apple/swift@8a5ed40](https://github.com/apple/swift/commit/8a5ed405bf1f92ec3fc97fa46e52528d2e8d67d9)


## 참고 사항

이 제안은 초기 형태에서 상당히 변경되었다. 문서 마지막 부분에서 역사적 배경과 변경 이유를 확인할 수 있다.


## 소개

함수 파라미터에 `inout`과 `var`를 사용할 때 의미상 혼란이 발생할 수 있다. 두 경우 모두 값의 변경 가능한 로컬 복사본을 제공하지만, `inout`으로 표시된 파라미터는 자동으로 원본에 다시 기록된다.

함수 파라미터는 기본적으로 불변이다:

```swift
func foo(i: Int) {
  i += 1 // 불가능
}

func foo(var i: Int) {
  i += 1 // 가능하지만, 호출자는 이 변경을 관찰할 수 없다.
}
```

여기서 `x`의 *로컬 복사본*은 변경되지만, 이 변경은 원래 전달된 값에 전파되지 않는다. 따라서 호출자는 이 변경을 직접 관찰할 수 없다. 값 타입에서 이를 발생시키려면 파라미터에 `inout`을 표시해야 한다:

```swift
func doSomethingWithVar(var i: Int) {
  i = 2 // 이 변경은 호출자에게 전달된 Int에 영향을 미치지 않지만, 로컬에서 i를 수정할 수 있다.
}

func doSomethingWithInout(inout i: Int) {
  i = 2 // 이 변경은 호출자에게 전달된 Int에 영향을 미친다.
}

var x = 1
print(x) // 1

doSomethingWithVar(x)
print(x) // 1

doSomethingWithInout(&x)
print(x) // 2
```


## 동기

함수 파라미터에 `var` 어노테이션을 사용하면 코드 한 줄을 줄이는 데는 도움이 되지만, 대부분의 사람들이 기대하는 의미를 가진 `inout`과 혼동을 일으킬 가능성이 크다. 이 값들이 고유한 복사본이며 `inout`의 쓰기 반환(write-back) 의미를 가지지 않는다는 사실을 강조하기 위해, 여기서 `var`을 허용하지 않는 것이 좋다.

요약하면, 이 변경을 추진하는 이유는 다음과 같다:

- 함수 파라미터에서 `var`이 `inout`과 자주 혼동된다.
- `var`이 값 타입에 참조 의미를 부여하는 것으로 오해되기 쉽다.
- 함수 파라미터는 *if-*, *while-*, *guard-*, *for-in-*, *case* 문과 같은 refutable 패턴이 아니다.


## 설계

이 변경은 파서에 사소한 수정을 가하는 것이다. Swift 2.2에서는 사용 중단 경고가 발생하고, Swift 3에서는 오류로 처리된다.


## 기존 코드에 미치는 영향

`var` 사용을 순수하게 기계적으로 마이그레이션하기 위해, 위의 모든 사용 사례에서 불변 복사본을 가리는 임시 변수를 즉시 도입할 수 있다. 예를 들어:

```swift
func foo(i: Int) {
  var i = i
}
```

그러나 변수 가림(shadowing)이 반드시 이상적인 해결책은 아니며, 안티패턴을 나타낼 수도 있다. Swift 사용자들이 이러한 패턴을 사용하는 기존 코드를 재고해 볼 것을 기대하지만, 이 언어 변화에 반드시 대응할 필요는 없다.


## 고려된 대안들

이 제안은 원래 모든 refutable 패턴과 함수 파라미터에 대한 `var` 바인딩 제거를 포함했다.

[원본 SE-0003 제안서](https://github.com/swiftlang/swift-evolution/blob/8cd734260bc60d6d49dbfb48de5632e63bf200cc/proposals/0003-remove-var-parameters-patterns.md)

Refutable 패턴에서 `var` 제거는 Swift 2 코드에서 이미 사용 중인 유효한 뮤테이션 패턴에 부담을 주는 점 때문에 재고되었다. 관련 논의는 swift-evolution 메일링 리스트에서 확인할 수 있다:

[재고에 대한 초기 논의](https://forums.swift.org/t/reconsidering-se-0003-removing-var-from-function-parameters-and-pattern-matching/1159)

최종 결론에 대한 근거도 swift-evolution 리스트에 공유되었다. 여기서 확인할 수 있다:

[제안서 수정에 대한 설명](https://forums.swift.org/t/se-0003-removing-var-from-function-parameters-and-pattern-matching/1230)




