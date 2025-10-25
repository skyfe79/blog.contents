---
title: "Swift 모던 컨커런시 - Task Groups를 활용한 Swift의 구조화된 동시성"
date: 2025-09-13T07:57:00Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/

이 글을 읽기 전에 구조화된 컨커런시와 `async let`에 대한 이해가 필요하다. 해당 개념에 익숙하지 않다면 이 시리즈의 세 번째 글인 [Swift 모던 컨커런시 - 구조적 동시성과 async let 사용법](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/)을 먼저 읽어보길 권한다.

태스크 그룹은 Swift에서 구조화된 컨커런시를 구현하는 두 번째 방법이다. `async let`을 살펴볼 때 한 가지 제약 사항을 발견했는데, 바로 동시에 실행할 태스크의 수를 유연하게 조정할 수 없다는 점이다. 예를 들어 반복문 안에서 여러 태스크를 실행하려고 하면 각 결과를 `await`해야 하기 때문이다. 이로 인해 여러 이미지를 동시에 다운로드하는 등의 작업이 제한된다.

Swift는 이러한 제약을 해결하기 위해 태스크 그룹이라는 기능을 제공한다.


## 태스크 그룹

태스크 그룹은 `async let`보다 더 유연하면서도 구조화된 동시성의 단순함을 유지한다. 

태스크 그룹은 동적인 동시성을 제공하기 위해 설계된 구조화된 동시성의 한 형태다. 이를 사용하면 여러 태스크를 *그룹*으로 묶어 동시에 실행할 수 있다.

태스크 그룹을 시작하는 두 가지 방법이 있다:

* `withThrowingTaskGroup` 호출
* `withTaskGroup` 호출

이 글 시리즈에서 여러 번 살펴봤듯이, 오류를 발생시킬 수 있는 태스크와 그렇지 않은 태스크에 대해 각각 다른 버전이 존재한다. 그룹에 추가된 태스크는 그룹이 정의된 블록 범위를 벗어날 수 없다. 자식 태스크가 그룹에 추가되면 즉시 어떤 순서로든 실행이 시작되므로, 자식 태스크 간에 의존성이 없도록 코드를 설계해야 한다. 그룹이 범위를 벗어나면 내부의 모든 태스크 완료가 암시적으로 `await` 된다.

구조화된 동시성을 사용하면 그룹 내부에 `async let` 태스크를 생성할 수 있고, 반대로 `async let` 호출 내부에 태스크 그룹을 시작할 수도 있다.

다음과 같이 태스크 그룹 내에서 변수를 수정하려고 시도하면:

```swift
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    var imagesMetadata: [DetailedImage] = []
    try await withThrowingTaskGroup(of: Void.self, body: { group in
        for image in images {
            group.async {
                async let image = downloadImageAndMetadata(imageNumber: image)
                imagesMetadata +=  [try await image]
            }
        }
    })
    return imagesMetadata
}
```

*이 코드는 'Beginning Concurrency in Swift: Structured Concurrency and async-let'에 작성된 코드의 변형이다. 예제는 [Explore structured concurrency in Swift](https://developer.apple.com/videos/play/wwdc2021/10134/) WWDC2021 강연을 기반으로 한다.*

컴파일러는 `imagesMetadata`가 여러 태스크에 의해 동시에 안전하지 않게 접근될 가능성을 감지한다. 이는 여러 변수가 동시에 쓰기를 시도할 때 데이터 손상으로 이어질 수 있다. 다행히 새로운 동시성 API가 Swift에 깊게 통합되어 있어, 컴파일러가 정적으로 일부 검사를 수행하고 이러한 데이터 경쟁을 미리 방지할 수 있다.

이 코드를 컴파일하려고 하면 컴파일러는 다음과 같은 오류를 발생시킨다:

> Mutation of captured var ‘imagesMetadata’ in concurrently-executing code

그렇다면 Swift는 어떻게 이런 검사를 수행할 수 있을까?


## @Sendable 클로저 타입

데이터 경쟁 안전성을 도입하기 위해 Swift는 **@Sendable 클로저** 개념을 구현했다.

Task를 생성할 때 본문은 `@Sendable` 클로저이며, 이 클로저는 다음과 같은 특성을 가진다:

* 변경 가능한 변수를 캡처할 수 없다.
* 값 타입(value types), 액터(actors), 클래스, 또는 자체 동기화를 구현한 객체만 캡처해야 한다. 액터에 대해서는 추후 글에서 다룰 예정이다.

이 지식을 바탕으로 앞서 살펴본 Task Group을 수정할 수 있다. `withThrowingTaskGroup`이나 `withTaskGroup`으로 Task Group을 생성할 때, task group은 동시성 작업이 생성할 반환 타입을 매개변수로 받는다.

```swift
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    try await withThrowingTaskGroup(of: DetailedImage.self, body: { group in
        for image in images {
            group.async {
                async let image = downloadImageAndMetadata(imageNumber: image)
                return try await image
            }
        }
    })
}
```

이 메서드 구현은 아직 완성되지 않았지만 단계별로 몇 가지 중요한 수정이 이루어졌다:

* `withThrowingTaskGroup`의 `of` 매개변수는 이제 `DetailedImage`를 받도록 지정했다.
* 배열에 추가하는 대신, `group.async`는 이제 루프가 실행될 때마다 `await`된 `DetailedImage`를 반환한다.

본질적으로 우리는 `DetailedImage`로 그룹을 "채우고" 있으며, 결국 이를 반환할 것이다(오류가 발생하지 않는 한). 오류가 발생하면 자식 작업들이 취소되고 작업들은 중단되어야 한다. Swift의 동시성 시작: 구조화된 동시성과 async-let에서 언급했듯이, 코드 설계 시 취소 가능성을 고려하는 것은 개발자의 책임이지만 구조화된 동시성에서는 단 한 줄의 호출로 처리할 수 있다.

`group` 변수는 `ThrowingTaskGroup<DetailedImage, Error>` 타입이다. 놀랍게도 이는 컬렉션이다! `filter`, `map`, `reduce` 같은 함수형 프로그래밍을 적용하거나 반복할 수 있다.

```swift
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    var imagesMetadata: [DetailedImage] = []
    try await withThrowingTaskGroup(of: DetailedImage.self, body: { group in
        for image in images {
            group.async {
                async let image = downloadImageAndMetadata(imageNumber: image)
                return try await image
            }
        }
        for try await image in group {
            imagesMetadata += [image]
        }
    })
    return imagesMetadata
}
```

`for try await` 부분은 헷갈릴 수 있다(의도적인 말장난 아님). 하지만 Swift 5.5에서 소개된 모든 새로운 동시성 API와 함께, 우리는 새로운 `AsyncSequence` 타입을 갖게 되었다. 이 프로토콜은 시간이 지남에 따라 값을 수신할 타입들이 구현한다. `downloadMultipleImagesWithMetadata` 함수에서는 `group.async`를 사용해 동적으로 여러 `DetailedImage` 다운로드를 시작한다. 다운로드가 끝나면 `for in` 루프에 하나씩 전달되며, 이를 통해 루프 내에서 변수를 안전하게 수정할 수 있다.

for 루프의 `await`은 다른 `await` 호출과 동일하게 동작한다는 점에 유의하자. 실행은 해당 지점에 도달하면 일시 중단되고, 새로운 이미지가 전달되면 실행이 재개된다. 이는 `for-in` 루프 아래에 있는 어떤 코드도 그룹의 모든 요소가 전달될 때까지 실행되지 않는다는 의미이므로 중요하다. 세 개의 이미지를 다운로드하는 경우, 세 이미지는 동시에 다운로드될 수 있지만 for 루프는 한 번에 하나씩만 제공한다. 상당한 수의 이미지를 다운로드하는 경우, `for-in` 아래의 코드를 실행하기 전에 모든 이미지가 다운로드되어야 한다. 또한 오류가 발생하면 `for-in`이 실행을 중단할 수 있다(또는 첫 번째 다운로드가 실패하면 실행조차 되지 않을 수 있다).

`AsyncSequence`에 대해서는 추후 글에서 깊이 있게 다룰 예정이다.

Task Group을 다룰 때는 실제로 더 많은 유연성이 있다. 예를 들어 주어진 우선순위로 비동기 작업을 시작할 수 있다:

```swift
group.async(priority: .userInitiated) {
//...
}
```

여기서 `priority`는 [`Task.Priority`](https://developer.apple.com/documentation/swift/task/priority?changes=__6_8) 타입이다. 이는 취소를 다룰 때 더 유연한 제어를 제공하며, `asyncUnlessCancelled` 메서드도 있어 선택적으로 우선순위를 전달할 수 있다.

```swift
group.asyncUnlessCancelled(priority: nil) {
   //...
}
```

마지막으로 그룹에서 `cancellAll()`을 호출할 수도 있다. 취소는 트리 아래로 전파될 것이다.

`async let`과 비교했을 때 약간의 차이가 있다는 점에 유의하자. 그룹이 정상적으로 종료되어 스코프를 벗어나면 작업의 취소는 암시적이지 않다. 대신 단지 await될 뿐이다. 이는 다른 작업들이 완료될 시간을 주고 [Fork-Join 모델](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model)을 표현하기 위함이다. 본질적으로 "분할 정복" 방식인 이 모델은 우리의 경우 가능한 한 많은 자식 작업으로 여러 이미지를 다운로드하는 것이다.


## 요약

이 글에서는 Task Groups를 사용해 Swift에서 구조화된 동시성(Structured Concurrency)을 구현하는 또 다른 방법을 배웠다. Task Groups는 루프 안에서 여러 이미지를 다운로드해야 하는 경우와 같이 동적인 동시성 작업을 실행할 때 유용하다. `AsyncSequence`에 대해서도 간략히 언급했으며, Task Groups와 함께 사용해 시간 경과에 따라 결과를 전달하는 방법을 살펴봤다.

이번 시리즈의 전통대로, [여기](https://www.andyibanez.com/archives/AsyncAwaitGroupTask.zip)에서 다운로드할 수 있는 작은 예제 프로젝트를 준비했다. UI는 처음 몇 개 프로젝트와 동일하지만, `downloadMultipleImagesWithMetadata`의 컴파일 가능 버전을 직접 실행해보고 실험할 수 있다.

이 글을 통해 Swift에서 구조화된 동시성을 구현하는 두 가지 방법을 모두 살펴봤다:

* 기본적인 동시성 작업이 필요할 때는 `async let`을 사용한다.
* 동적으로 변하는 수의 동시성 작업을 처리해야 할 때는 Task Group을 사용한다.

다음 글에서는 비구조화된 동시성(Unstructured Concurrency)에 대해 알아볼 예정이다. 이름만 들으면 다소 복잡해 보이지만, Swift에서는 구조화된 동시성만큼 쉽게 다룰 수 있다.




