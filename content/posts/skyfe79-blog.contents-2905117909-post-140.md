---
title: "[SE-0031] inout 선언을 타입 데코레이션에 맞게 조정하기"
date: 2025-03-09T01:09:53Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## `inout` 선언을 타입 데코레이션에 맞게 조정하기

* 제안: [SE-0031](0031-adjusting-inout-declarations.md)
* 작성자: [Joe Groff](https://github.com/jckarter), [Erica Sadun](https://github.com/erica)
* 리뷰 관리자: [Chris Lattner](https://github.com/lattner)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0031-adjusting-inout-declarations-for-type-decoration/1478)
* 구현: [apple/swift#1333](https://github.com/apple/swift/pull/1333)


## 소개

`inout` 키워드는 복사 입력/복사 출력 인자 동작을 나타낸다. 현재 구현에서는 이 키워드가 인자 이름 앞에 위치한다. 우리는 `inout` 키워드를 매개변수 라벨이 아닌 타입을 꾸미기 위해 콜론 오른쪽으로 이동할 것을 제안한다.

*이 주제에 대한 초기 Swift-Evolution 논의는 "[Replace 'inout' with &](https://forums.swift.org/t/pitch-replace-inout-with/652/29)" 스레드에서 이루어졌다.*

[스레드에서 제안으로](https://forums.swift.org/t/proposal-adjusting-inout-declarations-for-type-decoration/1239), [리뷰](https://forums.swift.org/t/review-se-0031-adjusting-inout-declarations-for-type-decoration/1399)


## 동기

Swift 2에서 `inout` 매개변수는 콜론의 라벨 쪽이 아닌 타입 쪽에 위치한다. 이 키워드는 라벨을 수정하는 것이 아니라 타입을 수정한다. 라벨 대신 타입을 꾸미는 것은 다음과 같은 장점을 제공한다:

* `inout` 키워드가 전체 타입 구문에 제대로 통합될 수 있게 한다. 예를 들어:

    ```swift
    (x: inout T) -> U // => (inout T) -> U
    ```

* `inout`으로 라벨링된 인수와의 표기법적 유사성을 피할 수 있다. 예를 들어:

    ```swift
    func foo(inOut x: T) // foo(inOut:), type (T) -> Void
    func foo(inout x: T) // foo(_:), type (inout T) -> Void
    ```

* 이를 이동하면 `inout`을 매개변수 라벨로 사용할 수 있게 된다. 이는 그 자체로 특별히 강력한 동기는 아니지만, 현재 Swift 3에서 `inout`은 매개변수 라벨로 사용할 수 없는 *유일한* 키워드이다. 이 제한을 없애면 언어가 단순해진다.

* Rust의 borrowing과 같은 다른 언어에서의 유사한 패턴과 더 잘 맞는다. 이러한 패턴은 나중에 Swift에 다시 도입될 수 있다.


## 상세 설계

```
parameter → 외부-매개변수-이름 opt로컬-매개변수-이름 : 타입-주석
타입-주석 → inout 타입-주석
```


## 고려한 대안

`@inout`을 사용한 데코레이션(`@inout(T)` 또는 `@inout T`)은 고려되었지만, 최종적으로 제외되었다.




