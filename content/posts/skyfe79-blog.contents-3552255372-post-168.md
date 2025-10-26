---
title: "Swift 모던 컨커런시 - Swift에서 분리된 독립 태스크를 활용한 비구조적 동시성"
date: 2025-10-25T09:56:35Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/

*이 글을 이해하려면 비동기 태스크에 대한 사전 지식이 필요합니다. 비동기 태스크 개념이 익숙하지 않다면, 본 시리즈의 'Swift에서 비구조적 동시성 소개' 글을 먼저 읽어보세요.*

이번 시리즈를 통해 우리는 `async/await`의 개념을 살펴봤다. 또한 `async let`과 `Group Tasks`를 이용한 구조적 동시성의 기초를 다졌다. 구조적 동시성이 유용하지만 모든 상황을 커버할 수는 없다는 점도 확인했으며, 이에 따라 비구조적 동시성의 존재를 언급하고 `Task {}` 블록을 활용하는 방법을 탐구했다.

이번 글에서는 Swift 5.5이 제공하는 가장 유연한 방법인 분리된 독립 태스크(Detached tasks)를 사용해 비구조적 동시성을 구현하는 최종적인 방법을 알아볼 것이다.


## 독립 태스크(Detached Tasks) 소개

새로운 `async/await` 시스템에서 제공하는 동시성 옵션 중 독립 태스크(Detached Tasks)는 가장 유연한 기능이다. 어디서든 시작할 수 있고, 생명주기에 제약이 없으며, `Task.Handle`을 통해 수동으로 취소하거나 기다릴 수 있다. 부모 태스크로부터 아무것도 상속받지 않는 유일한 태스크 타입이다. 심지어 우선순위도 상속받지 않는다. 독립 태스크는 실행 컨텍스트와 완전히 독립적이다.

독립 태스크는 부모 태스크와 전혀 연관 없는 작업을 수행할 때 유용하다. 대표적인 예로 네트워크에서 이미지를 다운로드한 후 디스크에 캐싱하는 경우가 있다(애플의 WWDC2021 [Explore Structured Concurrency in Swift](https://developer.apple.com/videos/play/wwdc2021/10134/) 발표에서 소개된 예제). 캐싱 작업은 독립 태스크로 실행할 수 있다. 이미지를 얻은 후에는 다운로드 태스크가 취소되더라도 캐싱 작업까지 취소될 필요가 없기 때문이다.

```swift
func storeImageInDisk(image: UIImage) async {
    guard
        let imageData = image.pngData(),
        let cachesUrl = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask).first 
    else { 
       return 
    }

    let imageUrl = cachesUrl.appendingPathComponent(UUID().uuidString)
    try? imageData.write(to: imageUrl)
}
```

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    Task.detached(priority: .background) {
        await storeImageInDisk(image: image)
    }
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}
```

`storeImageInDisk` 태스크를 생성한 후, `downloadImageAndMetadata` 함수 내에서 `Task.detached`로 호출한다. 이미지 다운로드가 완료되면 바로 캐싱을 시도한다.

`Task {}`를 이해했다면 `Task.detached {}`도 쉽게 이해할 수 있다. 독립 태스크를 시작할 때는 우선순위를 지정할 수 있다. 여기서는 `background` 우선순위를 사용했다. 사용자와 직접 관련 없는 작업이므로 높은 우선순위가 필요하지 않기 때문이다. `.userInitiated`는 사용자가 직접 요청한 작업에 적합하다.

독립 태스크는 비구조적이므로 `Task.detached`는 `Task` 핸들을 반환한다. 이 핸들로 언제든지 태스크를 취소할 수 있다. `Task.detached`는 시작한 태스크와 독립적이지만, 그 안에서 시작된 다른 태스크들은 여전히 `Task.detached`에 의존한다. 따라서 `Task.detached` 태스크를 취소하면 모든 자식 태스크도 `cancelled`로 표시된다. 단, `Task.detached` 내부에서 또 다른 `Task.detached`를 실행한 경우는 예외다.


## 요약

`Task.detached`는 `Task {}`를 이해하면 어렵지 않다. 두 방식은 거의 동일하게 동작한다. 주요 차이점은 `Task.detached`가 부모 컨텍스트로부터 아무것도 상속받지 않는다는 점이다. 둘 다 수동으로 취소할 수 있으며, 독립적인 작업을 원하는 시점에 실행할 때 유용하다.




