---
title: "[SE-0015] 튜플 비교 연산자"
date: 2025-03-09T00:28:20Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 튜플 비교 연산자

* 제안: [SE-0015](0015-tuple-comparison-operators.md)
* 저자: [Lily Ballard](https://github.com/lilyball)
* 검토 관리자: [Dave Abrahams](https://github.com/dabrahams)
* 상태: **구현 완료 (Swift 2.2)**
* 결정 노트: [Rationale](https://forums.swift.org/t/review-add-a-lazy-flatmap-for-sequences-of-optionals/695/4)
* 구현: [apple/swift#408](https://github.com/apple/swift/pull/408)


## 소개

특정 크기까지 튜플에 비교 연산자를 구현한다.

[Swift Evolution 토론](https://forums.swift.org/t/proposal-implement-and-for-tuples-where-possible-up-to-some-high-arity/251), [리뷰](https://forums.swift.org/t/review-add-a-lazy-flatmap-for-sequences-of-optionals/695)

참고: 리뷰는 처음에 잘못된 스레드와 제목으로 시작했으나 이후 수정되었다.


## 동기

비교 가능한 값들로 이루어진 튜플을 비교하려고 할 때, 튜플이 일반적인 비교 연산자를 지원하지 않는다는 사실이 불편하다. 동등 비교 가능한 값들로 이루어진 튜플에 대해 `==`와 `!=` 연산자를 정의하는 것은 매우 명확하다. 또한 순서 비교 연산자(사전식 비교)도 합리적으로 명확하게 정의할 수 있다.

튜플을 비교할 수 있는 기능은 단순히 튜플 비교를 넘어서, 튜플과 유사한 구조체에 대한 비교 연산자를 구현하는 작업도 더 쉽게 만든다. 해당 연산자는 구조체의 프로퍼티들을 담은 튜플을 비교하는 방식으로 간단히 구현할 수 있다.


## 제안하는 해결책

Swift 표준 라이브러리는 특정 크기까지의 모든 튜플에 대해 비교 연산자의 일반적인 구현을 제공해야 한다. 이 크기는 편의성(모든 튜플이 이를 지원)과 코드 크기(모든 정의가 표준 라이브러리의 크기를 증가시킴) 사이의 균형을 고려해 선택해야 한다.

Swift가 프로토콜에 대한 조건부 준수를 지원하게 되고, 만약 Swift가 튜플 확장을 지원하게 된다면, 선택된 크기까지의 튜플은 `Equatable` 및 `Comparable` 프로토콜을 조건부로 준수하도록 선언되어야 한다.

Swift가 가변 타입 매개변수를 지원하게 된다면, 심각한 코드 크기 문제가 없다는 가정 하에 가변 타입을 기반으로 연산자(및 프로토콜 준수)를 재정의하는 방안을 고려해야 한다.


## 상세 설계

실제 정의는 gyb를 통해 생성된다. 여기서 제안하는 차수(arity)는 6이며, 이는 대부분의 튜플에 충분히 큰 크기지만(내가 원하는 만큼 크지는 않음), 코드 크기가 급격히 증가하지 않도록 적절한 수준이다. 차수 6에 대한 이 제안을 구현한 후, Ninja-ReleaseAssert 빌드에서 `libswiftCore.dylib`(macosx와 iphoneos 모두)의 코드 크기가 43.6KiB 증가했으며, 이는 1.4%의 증가율에 해당한다.

생성된 정의는 다음과 같다(차수 3의 경우):

```swift
@warn_unused_result
public func == <A: Equatable, B: Equatable, C: Equatable>(lhs: (A,B,C), rhs: (A,B,C)) -> Bool {
  return lhs.0 == rhs.0 && lhs.1 == rhs.1 && lhs.2 == rhs.2
}

@warn_unused_result
public func != <A: Equatable, B: Equatable, C: Equatable>(lhs: (A,B,C), rhs: (A,B,C)) -> Bool {
  return lhs.0 != rhs.0 || lhs.1 != rhs.1 || lhs.2 != rhs.2
}

@warn_unused_result
public func < <A: Comparable, B: Comparable, C: Comparable>(lhs: (A,B,C), rhs: (A,B,C)) -> Bool {
  if lhs.0 != rhs.0 { return lhs.0 < rhs.0 }
  if lhs.1 != rhs.1 { return lhs.1 < rhs.1 }
  return lhs.2 < rhs.2
}
@warn_unused_result
public func <= <A: Comparable, B: Comparable, C: Comparable>(lhs: (A,B,C), rhs: (A,B,C)) -> Bool {
  if lhs.0 != rhs.0 { return lhs.0 < rhs.0 }
  if lhs.1 != rhs.1 { return lhs.1 < rhs.1 }
  return lhs.2 <= rhs.2
}
@warn_unused_result
public func > <A: Comparable, B: Comparable, C: Comparable>(lhs: (A,B,C), rhs: (A,B,C)) -> Bool {
  if lhs.0 != rhs.0 { return lhs.0 > rhs.0 }
  if lhs.1 != rhs.1 { return lhs.1 > rhs.1 }
  return lhs.2 > rhs.2
}
@warn_unused_result
public func >= <A: Comparable, B: Comparable, C: Comparable>(lhs: (A,B,C), rhs: (A,B,C)) -> Bool {
  if lhs.0 != rhs.0 { return lhs.0 > rhs.0 }
  if lhs.1 != rhs.1 { return lhs.1 > rhs.1 }
  return lhs.2 >= rhs.2
}
```


## 기존 코드에 미치는 영향

기존 코드에는 아무런 영향을 미치지 않는다.


## 고려한 대안

튜플(tuple)의 인자 개수(arity)를 최대 12까지로 설정하여 Ninja-ReleaseAssert 빌드를 테스트했다. 이 경우 코드 크기가 171KiB 증가했으며, 이는 전체 크기의 5.5%에 해당한다. 다른 인자 개수에 대해서는 아직 테스트하지 않았다.




