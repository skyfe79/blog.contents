---
title: "Swift 모던 컨커런시 - 구조적 동시성과 async let 사용법"
date: 2025-09-13T07:42:53Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/

*이 글은 [Swift 모던 컨커런시](https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/) 시리즈의 일부다.*

*원문은 Xcode 13 beta 1을 기준으로 작성되었으며, 이후 Xcode 13 beta 3에 맞춰 내용과 코드 샘플을 업데이트했다.*

* * *

*이 글을 이해하려면 async/await 개념을 미리 알고 있어야 한다. 해당 개념이 익숙하지 않다면 시리즈 첫 번째 글인 [Swift에서 async/await 이해하기](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)를 먼저 읽어보길 권한다.*

`async/await`는 Swift의 새로운 동시성 시스템에서 가장 중요한 개념이다. async/await를 이해하면 깔끔한 문법과 직관적인 코드로 여러 작업을 병렬로 수행할 수 있다.

실제로 [이를 구현하는 여러 방법](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/)이 있지만, Apple이 WWDC2021에서 Swift 5.5와 함께 소개한 방식이 가장 안전하다. 특별한 요구사항이 없는 한 거의 항상 이 방식을 사용하게 될 것이다.


## 구조화된 동시성 소개

이전 글에서 우리는 콜백 기반 코드가 동시성 컨텍스트에서 사용될 때 관리하기 어려워질 수 있다는 점을 논의했다. 이를 해결하기 위해 애플은 `async/await` 키워드를 도입했으며, 이는 코드의 선형적 흐름을 유지하면서 동시성 코드를 작성할 수 있게 해준다. 이 코드는 위에서 아래로 읽을 수 있다. 하지만 [Swift에서 async/await 이해하기](https://www.andyibanez.com/posts/understanding-async-await-in-swift/) 글에서 언급했듯이, async/await를 사용한다고 해서 반드시 여러 작업을 동시에 수행하는 것은 아니다(호출하는 작업이 내부적으로 그렇게 할 수는 있다). 이제 우리는 병렬로 코드를 실행하는 방법을 배우기 시작할 것이며, *구조화된 동시성* 개념부터 살펴볼 것이다.

구조화된 동시성의 아이디어는 구조화된 프로그래밍과 같은 개념에서 비롯되었다. 우리는 대부분의 시간을 구조화된 코드를 작성하며 보내므로, 이를 의식하지 않는다. 구조화된 코드는 위에서 아래로 읽을 수 있으며, 출력이 예측 가능하고 코드가 정확한 순서대로 실행된다. 변수를 사용할 때는 선언된 블록 내에서 잘 정의된 수명을 가진다. 콜백 기반 동시성에서는 메인 스레드가 계속 실행되는 동안 다른 스레드나 컨텍스트에서 작업을 시작하므로, 프로그램을 실행할 때마다 출력이 달라질 수 있다. Objective-C를 작성하는 경우 블록 내에서 변수를 수정하려면 `__block`으로 처리해야 한다. 이는 원하는 결과를 얻기 위해 모든 것이 어떤 순서로든 발생할 수 있는 코드의 미로를 생성한다.

이제 다음 함수들을 살펴보자:

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}

func downloadImage(imageNumber: Int) async throws -> UIImage {
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/(imageNumber).png")!
    let imageRequest = URLRequest(url: imageUrl)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}

func downloadMetadata(for id: Int) async throws -> ImageMetadata {
    let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/(id).json")!
    let metadataRequest = URLRequest(url: metadataUrl)
    let (data, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
    guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.invalidMetadata
    }

    return try JSONDecoder().decode(ImageMetadata.self, from: data)
}

//...

struct ImageMetadata: Codable {
    let name: String
    let firstAppearance: String
    let year: Int
}

struct DetailedImage {
    let image: UIImage
    let metadata: ImageMetadata
}

enum ImageDownloadError: Error {
    case badImage
    case invalidMetadata
}
```

`downloadImageAndMetadata`는 이미지와 해당 메타데이터를 다운로드하여 `DetailedImage` 객체로 감싸는 함수다. 다운로드를 수행하기 위해 이미지 자체를 다운로드하는 `downloadImage` 함수와 메타데이터를 다운로드하는 `downloadMetadata` 함수를 호출한다. `downloadImageAndMetadata` 함수를 좀 더 자세히 살펴보자.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}
```

다운로드는 순차적으로 이루어지며, 이는 대부분의 경우 원하는 동작이다. 함수는 먼저 이미지를 다운로드하고, 그 다음에 메타데이터를 다운로드한다. 이는 많은 경우에 유용하지만, 작업들이 서로 의존성을 가지지 않아 동시에 실행할 수 있는 경우도 있다.

이 예제에서 이미지 다운로드와 메타데이터 다운로드는 독립적인 작업이므로, 두 작업을 동시에 실행하여 더 빨리 완료할 수 있다.

진행하기 전에, 클로저 기반 코드로 이를 어떻게 구현할지 생각해보자. 먼저 두 개의 `URLSession` 데이터 태스크를 각각의 완료 핸들러와 함께 시작해야 한다. 하지만 그 다음은? 이 작업의 완료를 어떻게 조율할 것인가? 이미지가 먼저 다운로드되면 어떻게 되는가? 메타데이터가 먼저 완료되면 어떻게 되는가? 최종 완료 핸들러가 호출되도록 접근을 어떻게 "잠그고" 보장할 것인가?

사실 순수한 클로저 기반 코드(심지어 델리게이트 기반 코드에서도)로 이 작업을 수행하는 것은 매우 빠르게 지저분해진다. 그리고 우리는 단지 *두 개*의 작업을 동시에 처리하는 것에 대해 이야기하고 있다!

Swift에서는 구조화된 동시성을 다루는 두 가지 방법이 있다:

*   `async let`
*   태스크 그룹

이 글은 `async let`에 국한될 것이며, 태스크 그룹은 추후 글에서 다룰 것이다.


## 태스크의 이해

태스크는 Swift에서 코드를 병렬로 실행하는 핵심 메커니즘이다. 각 태스크는 새로운 비동기 실행 컨텍스트를 제공하며, 다른 태스크와 동시에 실행될 수 있다. 안전하고 효율적인 상황에서는 자동으로 병렬 실행된다.

현재 `downloadImageAndMetadata` 함수는 실제로 태스크를 생성하지 않는다. 두 다운로드 작업 모두 `await`로 처리되기 때문에 병렬 실행되지 않는다. 이 문제를 해결해 볼 것이다.

Swift의 새로운 동시성 기능은 언어에 깊이 통합되어 있다. 동시성 코드를 작성할 때 컴파일러가 흔한 동시성 버그를 방지해 준다. 초보 개발자에게는 컴파일 오류로 표시되어 답답할 수 있지만, 사실 Swift는 코드가 잘못된 동작을 하지 않도록 보호하는 역할을 한다. 동시성은 해결하기 매우 어려운 문제이며, 운영체제 관련 서적을 읽어본 사람이라면 개발자가 안전한 동시성 코드를 작성하기 위해 사용할 수 있는 다양한 패턴이 있다는 것을 알 것이다. 하지만 이를 수동으로 구현하는 것은 어렵고 오류가 발생하기 쉬우며, 컨텍스트에 따라 테스트도 어려울 수 있다. 컴파일 타임에 이러한 검사를 수행할 수 있다는 것은 큰 안전 장치다.

함수를 `async`로 표시한다고 해서 자동으로 새로운 태스크가 생성되는 것은 아니다. 기본적으로 컴파일러는 `async`로 표시된 함수가 호출될 때마다 `await`될 것으로 예상한다. 태스크 생성은 자동 프로세스가 아니며, 우리는 컴파일러에 동시성 코드 실행을 요청할 수 있지만 실제로 이를 허용할지는 컴파일러의 판단에 달려 있다. 태스크는 항상 명시적으로 생성해야 한다.

구조화된 동시성은 단순성과 유연성 사이의 균형에 관한 개념이다. 대부분의 동시성 작업을 이러한 제약 조건 내에서 처리할 수 있지만, 더 많은 유연성이 필요하다면 안전성이 낮은 대신 더 많은 제어권을 제공하는 저수준 API를 사용할 수 있다. 자세한 내용은 [Apple 플랫폼의 멀티스레딩 옵션](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/) 글을 참고하면 된다.


## async let 소개

`async let`은 *동시성 바인딩*이라고도 하며, 병렬로 작업을 실행한다.

```swift
async let result = //... 비동기 함수 호출 (await 없이)
```

Swift가 `async let`을 만나면 등호 오른쪽의 함수가 동시에 실행되기 시작한다. 즉, `await` 호출이 프로그램 실행을 일시 중단하는 반면, `async let`은 작업을 시작한 후 값이 필요할 때까지 아래 코드를 계속 실행한다.

다음 예제를 살펴보자:

```swift
func downloadImageConcurrentlyWhilePrinting(imageNumber: Int) async throws -> UIImage {
    print("첫 번째 출력")
    print("이미지 다운로드 시작")
    async let image = downloadImage(imageNumber: imageNumber)
    print("이미지가 준비되기 전에도 출력 가능")
    print("계속 출력")
    return try await image
}
```

모든 `print` 문이 거의 즉시 실행되는 것을 확인할 수 있다. 이는 `async let`이 `downloadImage`를 다른 작업으로 실행했기 때문이다. `async let` 호출 전의 두 `print` 문은 예상대로 실행된다. 나머지 print 문도 `downloadImage`가 `await` 호출이 아니므로 즉시 출력된다. `return try await image`에 도달하면 이미지 다운로드가 완료될 때까지(또는 오류가 발생할 때까지) 반환문에서 프로그램이 일시 중단된다.

이것은 코드를 동시에 실행할 수 있는 메커니즘 중 하나이므로, 여러 `async let` 호출을 동시에 사용할 수 있으며 시스템은 가능한 경우 이를 병렬로 실행한다.

이제 `downloadImageAndMetadata` 함수를 수정해 이미지와 메타데이터를 동시에 다운로드할 수 있다.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}
```

**참고**: *이 글의 대부분은 [Explore structured concurrency in Swift](https://developer.apple.com/videos/play/wwdc2021/10134/) 세션을 기반으로 하며, 비슷한 예제를 사용하지만 직접 실행해 볼 수 있다.*

`let` 앞에 `async`를 추가하고 `await` 키워드를 값이 존재할 것으로 예상되는 위치로 이동함으로써, 구조화된 흐름을 유지하면서 여러 작업을 동시에 수행할 수 있다. 매우 간결하다!

이것이 새로운 async/await API로 코드를 동시에 실행하는 방법이다. 그러나 아직 끝나지 않았다. 마지막으로 매우 중요한 개념인 Task Tree를 살펴볼 것이다.


### 태스크 트리


구조화된 동시성(Structured Concurrency)은 **태스크 트리**라는 개념을 활용한다. 태스크 트리는 구조화된 동시성 코드가 실행되는 계층 구조로, 태스크의 취소, 우선순위, 로컬 변수 같은 속성에 영향을 미친다. 한 비동기 함수에서 다른 함수로 이동할 때도 동일한 태스크가 새로운 호출을 실행한다.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}
```

`downloadImageAndMetadata` 함수를 호출하면 부모 태스크의 모든 속성을 상속받는다. `async let`을 호출할 때마다 새로운 태스크가 생성되는데, 이 태스크는 현재 함수가 실행 중인 태스크의 **자식 태스크**가 된다.

![async let 다이어그램](https://www.andyibanez.com/img/async_let_diagram.png)

`downloadImageAndMetadata` 함수는 두 개의 자식 태스크(이미지 다운로드와 메타데이터 다운로드)를 생성할 수 있으며, 이 코드들은 동시에 실행될 수 있다.

`downloadImageAndMetadata`는 실행 중인 태스크의 속성을 상속받고, `downloadImage`와 `downloadMetadata`는 다시 `downloadImageAndMetadata`의 속성을 상속받는다. 

중요한 점은 태스크가 실행 중인 함수의 자식이 아니라는 것이다. 단지 수명이 함수에 연결될 뿐이다.

태스크 트리는 중요한 규칙을 강제한다: **부모 태스크는 모든 자식 태스크가 작업을 완료한 후에만 자신의 작업을 끝낼 수 있다.**

이 규칙은 `await` 호출이 계속 진행할 수 있는 신호를 받기 전까지 실행을 허용하지 않는 것에서 확인할 수 있다. `downloadImage`와 `downloadMetadata`는 오류를 발생시키거나 값을 반환할 수 있지만, 어느 경우든 이들이 작업을 완료해야만 해당 코드가 실행을 계속할 수 있다.

일반적으로 `downloadImageAndMetadata`는 `downloadImage`와 `downloadMetadata`가 모두 성공적으로 완료된다. 하지만 둘 중 하나가 오류를 발생시키고 다른 하나는 문제없이 완료되면 어떻게 될까?

코드가 구조화되어 있고 위에서 아래로 실행되기 때문에 직관적으로 알 수 있다. 둘 중 하나가 오류를 발생시키면 `downloadImageAndMetadata`도 동일한 오류를 발생시킨다. 하지만 다른 태스크의 실행에는 어떤 영향을 미칠까? 예를 들어 `downloadMetadata`가 실패하고 `downloadImage`가 큰 이미지를 다운로드 중이라면 어떻게 될까?

태스크가 실패하면 Swift는 남은 자식 태스크를 `취소됨`으로 표시한다. 이 예제에서는 `downloadMetadata`가 실패했기 때문에 `downloadImage`가 취소된다. 태스크를 취소로 표시한다고 해서 실제로 태스크가 즉시 취소되는 것은 아니다. 단지 해당 태스크의 결과가 더 이상 필요하지 않음을 알릴 뿐이다. 모든 자식 태스크와 그 하위 태스크들은 부모가 취소되면 함께 취소된다.

그렇다면 태스크는 실제로 언제 실행을 중지할까? 구조화된 태스크의 멋진 특징은 **취소가 협력적**이라는 점이다. 태스크는 즉시 중지되지 않는다. 대신 적절하다고 판단되는 시점에 중지한다. 네트워크 호출 중이라면 취소 알림을 받는 즉시 중지하는 것이 적절하지 않을 수 있다.

태스크는 명시적으로 취소 여부를 확인해야 한다. 어디서든 취소를 확인할 수 있으며, 이는 특히 오랜 시간이 걸리는 태스크를 설계할 때 취소를 고려해야 함을 의미한다.

취소를 확인하는 방법은 두 가지다. 첫째, 함수가 `throws`로 표시된 경우 `try Task.checkCancellation()`을 호출한다. 둘째, `throw` 컨텍스트 내에서 실행되지 않는 태스크에서는 `Task.isCancelled`를 사용해 불리언 값을 확인할 수 있다.

```swift
func downloadImage(imageNumber: Int) async throws -> UIImage {
    try Task.checkCancellation()
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/(imageNumber).png")!
    let imageRequest = URLRequest(url: imageUrl)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}

func downloadMetadata(for id: Int) async throws -> ImageMetadata {
    try Task.checkCancellation()
    let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/(id).json")!
    let metadataRequest = URLRequest(url: metadataUrl)
    let (data, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
    guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.invalidMetadata
    }

    return try JSONDecoder().decode(ImageMetadata.self, from: data)
}

func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}

// 새로운 함수
func downloadMultipleImagesWithMetadata(images: Int...) async throws -> [DetailedImage]{
    var imagesMetadata: [DetailedImage] = []
    for image in images {
        print(image)
        async let image = downloadImageAndMetadata(imageNumber: image)
        imagesMetadata +=  [try await image]
    }
    return imagesMetadata
}
```

위 예제에서는 `downloadImage`와 `downloadMetadata` 시작 부분에 취소 확인을 추가했다. 또한 여러 이미지를 다운로드하는 함수도 추가했다(동시성은 아님 - 태스크 그룹에 대해 다룰 때 가변적인 수의 동시 태스크를 수행하는 방법을 배울 것이다). **어떤** 이미지나 메타데이터 다운로드가 실패하면 자식 태스크에 취소 알림이 전달되고, 취소할 기회가 있다면(즉, 아직 이미지나 메타데이터 다운로드를 시작하지 않았다면) 실행을 중지한다.


## 요약

이번에는 새로운 async/await API를 활용해 실제 동시 실행(concurrent execution)의 세계를 탐험했다. 구조화된 동시성(structured concurrency)의 개념과 `async let`을 이용해 구현하는 방법을 배웠다. 또한 태스크 트리(task tree)와 협력적 취소(cooperative cancellation)의 동작 원리를 이해했다.

새로 만든 `downloadMultipleImagesWithMetadata` 함수가 세 이미지를 동시에 다운로드하지 않는 이유를 눈치챘을 것이다. 배열에 결과를 추가하기 전에 `await`로 대기해야 하기 때문이다. 이 문제는 태스크 그룹(Task Groups)을 다룰 때 변수 개수의 동시 작업을 실행하는 방법을 배우면서 해결할 것이다.

이 글의 내용을 차분히 분석해 보길 바란다. 언제나처럼 [샘플 프로젝트](https://www.andyibanez.com/archives/AsyncAwaitConcurrent.zip)를 통해 이 글의 개념들을 직접 실험해 볼 수 있다.




