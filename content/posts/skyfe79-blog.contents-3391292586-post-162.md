---
title: "Swift 모던 컨커런시 - async/await 이해하기"
date: 2025-09-07T06:51:38Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/understanding-async-await-in-swift/

*이 글은 [Swift의 모던 컨커런시](https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/) 시리즈의 일부다.*

*이 글은 원래 Xcode 13 베타 1 버전으로 예제를 작성했다. 이후 Xcode 13 베타 3 버전에 맞춰 글 내용, 코드 예제, 제공된 샘플 프로젝트를 업데이트했다.*

---

Swift에서 컨커런시를 시작하기 전에 반드시 async/await를 이해해야 한다. 이는 필수적인 과정이다. async/await가 유일한 [컨커런시 옵션](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/)은 아니지만, Apple의 SDK에서 점점 더 많이 활용되고 있다. 서드파티 라이브러리 제공자들도 이를 도입할 것임은 분명하다.

이 글은 async/await에만 집중한다. 이 개념을 이해한 후에는 구조화된 컨커런시, 비구조화 컨커런시, SwiftUI 등 더 고급 주제로 넘어갈 것이다.

콜백 기반 컨커런시를 작성해 본 경험이 있다면, async/await 구현이 기존 Apple 기술과 완전히 다르다는 점을 명심해야 한다. 기존에 알고 있던 동시성 프로그래밍 개념을 완전히 뒤엎는다. 이 글을 읽을 때 이 점을 염두에 두는 것이 중요하다.

이 글에서는 이미지를 다운로드하고 별도의 네트워크 호출로 메타데이터를 가져오는 함수를 작성할 것이다. 콜백 기반 컨커런시로 구현할 때 빠르게 복잡해지는 상황과 async/await가 이를 어떻게 우아하게 해결하는지 보여줄 것이다.


## 개념 복습하기

## 절차적 프로그래밍 다시 보기

네트워크나 I/O 같은 특별한 기능이 필요 없는 일반적인 프로그램을 작성할 때는 코드가 작성된 순서대로 실행된다. 필요한 프로시저를 호출하고, 필요하면 호출자에게 결과를 반환하는 방식이다.

다음 코드를 살펴보자:

```swift
func sayHi() {
    print("Hi")
}

func multiply(_ x: Int, _ y: Int) -> Int {
    x * y
}

func sayBye(result: Int) {
    print("Bye (result)")
}

func performCoolStuff() {
    sayHi()
    let x = 10
    let y = 5
    let result = multiply(x, y)
    sayBye(result: result)
}

// performCoolStuff() 호출
performCoolStuff()
```

`performCoolStuff()`를 호출하면 코드는 다음과 같이 실행된다:

1. 먼저 `sayHi()`를 호출한다
2. `x`와 `y` 두 변수를 선언한다
3. `x`와 `y` 값을 전달해 `multiply`를 호출한다
4. 곱셈 결과를 인자로 `sayBye`를 호출한다

여기서 길을 잃을 일은 없다. 코드는 작성한 순서대로 호출된다. 다른 함수를 호출하는 함수들은 *호출 스택*에 나타난 순서 그대로 배치되며, 값을 반환하면서 다시 호출자에게 제어권을 넘긴다. 함수 호출이 발생하면 `return`을 통해 호출자에게 제어권을 돌려준다. `multiply`를 호출하면 제어권이 해당 함수로 넘어가고, 결과를 반환할 때 `return`을 통해 다시 제어권을 돌려받는다.

절차적 프로그래밍에 대해 깊이 생각할 필요는 없다. 매일 사용하는 방식이며, 항상 예상한 대로 동작한다.


## 콜백 기반 동시성 코드 복습

다른 코드와 병렬로 실행될 수 있는 코드는 조금 더 복잡하다. 다음 예제는 네트워크 호출을 통해 이미지를 다운로드하고, 다른 네트워크 호출로 메타데이터를 가져오는 경우다(이 코드는 새 프로젝트의 뷰 컨트롤러에 복사해 붙여넣어 바로 실행할 수 있다). 다운로드는 메인 스레드의 실행과 동시에 진행된다:

```swift
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

func sayHi() {
    print("Hi")
}

func multiply(_ x: Int, _ y: Int) -> Int {
    x * y
}

func sayBye(result: Int) {
    print("Bye (result)")
}

func downloadImageAndMetadata(
    imageNumber: Int,
    completionHandler: @escaping (_ image: DetailedImage?, _ error: Error?) -> Void
) {
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/(imageNumber).png")!
    let imageTask = URLSession.shared.dataTask(with: imageUrl) { data, response, error in
        guard let data = data, let image = UIImage(data: data), (response as? HTTPURLResponse)?.statusCode == 200 else {
            completionHandler(nil, ImageDownloadError.badImage)
            return
        }
        let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/(imageNumber).json")!
        let metadataTask = URLSession.shared.dataTask(with: metadataUrl) { data, response, error in
            guard let data = data, let metadata = try? JSONDecoder().decode(ImageMetadata.self, from: data),  (response as? HTTPURLResponse)?.statusCode == 200 else {
                completionHandler(nil, ImageDownloadError.invalidMetadata)
                return
            }
            let detailedImage = DetailedImage(image: image, metadata: metadata)
            completionHandler(detailedImage, nil)
        }
        metadataTask.resume()
    }
    imageTask.resume()
}

func performMessyStuff() {
    sayHi()
    let x = 10
    downloadImageAndMetadata(imageNumber: 1) { image, error in
        DispatchQueue.main.async {
            print("We got results")
        }
    }
    let y = 5
    let result = multiply(x, y)
    sayBye(result: result)
}

performMessyStuff()
```

**참고**: *애플은 WWDC2021의 [Meet async/await in Swift 세션](https://developer.apple.com/videos/play/wwdc2021/10132/)에서 비슷한 예제를 사용했다. 이 예제는 그 내용을 기반으로 했지만, 직접 컴파일 가능한 버전을 만들었다.*

실행 흐름은 다음과 같다:

1. `sayHi()` 메서드가 정상적으로 호출된다.
2. 변수 `x`를 생성하고 값을 할당한다.
3. `downloadImageAndMetadata`가 호출되며, 내부적으로 실행에 필요한 첫 번째 변수(`imageUrl`)를 설정한다.
4. `dataTask`를 저장할 변수를 동기적으로 생성하고, 다운로드 완료 후 호출될 완료 핸들러를 제공한다.
5. 다운로드를 시작하기 위해 작업에 `resume()`을 호출한다.
6. 완료 핸들러의 내용은 즉시 실행되지 않는다. 다운로드가 진행되는 동안 프로그램은 실행을 계속한다.
7. 프로그램은 `"We got results"`를 출력할 수도 있고 안 할 수도 있다. 네트워크 다운로드의 경우 항상 시간이 걸리지만, 더 빠른 비동기 작업이라면 이 시점에 호출될 수 있다. 프로그램은 변수 `y`를 생성한다.
8. 두 다운로드가 모두 성공적으로 완료되면 프로그램은 `"We got results"`를 출력할 수 있다. 그렇지 않으면 `result` 변수를 생성하고 `multiply`를 호출하는데, 이는 다운로드보다 먼저 끝날 수도 있고 아닐 수도 있다.
9. 다운로드가 성공적으로 완료되면 프로그램은 `"We got results"`를 출력하고, 그렇지 않으면 `sayBye`를 호출한다.
10. 위 과정 중 어느 시점에서든 이미지 작업 다운로드가 완료된 후 메타데이터 다운로드 작업이 시작될 수 있다.

이 실행 흐름은 지저분한데, 네트워크에서 데이터를 다운로드하는 작업이 비동기적이며 모든 작업이 다른 곳에서 일어나기 때문이다. 다운로드가 진행되는 동안 메인 스레드에서는 어떤 일이든 발생할 수 있다. 콘솔에 출력되는 내용은 실행마다 다를 수 있다. 다운로드는 메인 스레드에서 다른 스레드로 분기되지만, 프로그램은 메인 스레드의 코드를 문제없이 계속 실행한다. 이는 절차적으로 생각하기 어렵게 만드는데, 작업이 완료됐을 때 알려주는 `completionHandler`에 의존해야 하기 때문이다. 메인 스레드에서 수행할 수 있지만 이미지나 메타데이터에 의존하는 작업이 있다면, 완료 핸들러 내에서 모든 작업을 처리해야 한다(필요할 때마다 `DispatchQueue.main.async`로 메인 스레드에 작업을 다시 넘기면서).

콜백 기반 비동기 코드에서는 완료 핸들러가 실행될 때마다 제어권이 반환된다.

예상할 수 있듯이, 이러한 호출은 점점 더 복잡해지고 중첩될 수 있다.


## async/await 소개

async/await를 간단히 설명한다면 이렇게 표현할 수 있다:

> async/await는 절차적 프로그래밍과 콜백 기반 클로저의 혼합 형태다.

이해를 돕기 위해 두 가지 기본 개념을 기억하자:

1. 절차적 코드는 위에서 아래로 순차적으로 실행된다. 제어 흐름은 `return`을 통해 호출자에게 돌아간다.
2. 콜백 기반 비동기 처리에서는 비동기 작업을 생성하지만, 현재 스레드는 작업이 진행 중이더라도 계속 실행된다. 제어 흐름은 완료 핸들러를 통해 호출자에게 돌아간다.

이제 `async`와 `await` 키워드를 각각 살펴보자.


## 비동기 처리

`async` 키워드는 두 가지 용도로 사용한다:

*   컴파일러에게 특정 코드가 비동기임을 알려준다.
*   병렬로 비동기 작업을 생성한다.

함수를 `async`로 표시하려면, 함수의 닫는 괄호 다음과 반환 화살표 앞에 키워드를 추가한다:

```swift
func downloadImage(id: Int) async -> UIImage? { ... }
```

또는:

```swift
func downloadImage(id: Int) async throws -> UIImage { ... }
```

여기서 큰 장점을 바로 확인할 수 있다. 완료 핸들러(completion handler)가 사라지고, 함수 시그니처만으로도 목적이 명확해졌다. 한 눈에 비동기 여부와 반환 타입을 파악할 수 있다.

`async` 코드는 *동시성* 컨텍스트 내에서만 실행할 수 있다. 즉, 다른 `async` 함수 내부에서 호출하거나 `Task {}`를 통해 수동으로 디스패치해야 한다. `Task {}`에 대해서는 잠시 후에 알아볼 것이다.


## await 동작 원리

`await` 키워드는 비동기 프로그래밍의 핵심이다. 프로그램이 `await`을 만나면 함수 실행을 일시 중단할 수 있다. 실제로 중단할지는 시스템이 결정한다.

함수가 중단되면 제어권은 호출자에게 반환되지 않고 시스템으로 넘어간다. 시스템은 해당 스레드를 활용해 다른 작업을 수행하다가 중단된 함수가 완료되면 실행을 재개한다. `await` 아래에 있는 코드는 완료될 때까지 실행되지 않는다. 시스템이 어떤 작업을 우선 실행할지 결정하고, `await`된 작업이 끝나면 제어권을 다시 돌려준다.

이를 교통 신호등에 비유할 수 있다. 빨간불을 만나면 대부분 정지하겠지만, 새벽 4시처럼 차량이 전혀 없는 상황에서는 그냥 지나갈 수도 있다.

**중요한 점은 `await`이 함수 실행을 중단하기로 결정하면, 시스템이 재개하라는 신호를 보낼 때까지 아래 코드가 실행되지 않는다는 것이다. 그동안 시스템은 해당 스레드로 다른 작업을 수행한다.**

모든 `async` 함수 호출은 반드시 `await`으로 표시해야 한다.

이해를 돕기 위해 `downloadImageAndMetadata` 함수를 `async/await`을 사용해 다시 작성해보자.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {

        // 이미지 다운로드 시도
        let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/(imageNumber).png")!
        let imageRequest = URLRequest(url: imageUrl)
        let (imageData, imageResponse) = try await URLSession.shared.data(for: imageRequest)
        guard let image = UIImage(data: imageData), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.badImage
        }

        // 문제 없으면 메타데이터 다운로드 진행
        let metadataUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part1/(imageNumber).json")!
        let metadataRequest = URLRequest(url: metadataUrl)
        let (metadataData, metadataResponse) = try await URLSession.shared.data(for: metadataRequest)
        guard (metadataResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.invalidMetadata
        }

        let detailedImage = DetailedImage(image: image, metadata: try JSONDecoder().decode(ImageMetadata.self, from: metadataData))

        return detailedImage
    }
```

긴 함수이지만 콜백 지옥 버전보다 훨씬 명확하다. 주요 실행 흐름을 살펴보자:

1. 프로그램은 `imageUrl`과 `imageRequest`를 순차적으로 생성한다.
2. 비동기 호출 `URLSession.shared.data(for:)`에 도달한다.
3. 함수를 중단할지 계속할지 결정한다. 네트워크 작업 특성상 중단할 가능성이 높지만 항상 그렇다고 가정해서는 안 된다. 여기서는 함수가 중단된다고 가정한다.
4. 제어권이 시스템으로 넘어간다.
5. 시스템은 다운로드가 완료될 때까지 이 작업과 관련 없는 다른 작업을 수행할 수 있다.
6. 첫 번째 `await` 아래 코드는 실행되지 않는다. guard 문에 도달하지 않고, 메타데이터 관련 변수를 생성하지 않으며, `await`된 함수가 완료될 때까지 아무 작업도 하지 않는다.
7. 시간이 지난 후 시스템은 `await`된 함수가 완료되면 제어권을 돌려준다.
8. guard 문에 도달해 필요한 경우 오류를 발생시킨다.
9. 프로그램은 2-8단계를 메타데이터 작업에 대해 반복한다.
10. 새로운 `DetailedImage`를 반환한다.

보듯이 매우 직관적인 흐름이며, `await`이 나머지 실행을 시스템이 재개할 때까지 중단하는 방식은 절차적 프로그래밍과 매우 유사하게 동작한다.

이 함수를 여러 함수로 분리할 수도 있다:

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
```

함수를 `async`로 표시하기만 하면 이런 구현이 가능하다.

주의할 점은 이 코드가 선형적으로 실행된다는 것이다. 이미지와 메타데이터가 동시에 다운로드되지 않는다. 먼저 이미지를 다운로드하고, 그 다음 메타데이터를 다운로드한다. 둘을 동시에 다운로드하는 방법도 있지만, 이 글은 실제 동시성에 대한 내용이 아니다. 구조화된 동시성을 배울 때 두 작업을 동시에 수행하는 방법을 살펴볼 것이다.

함수 중단을 직접 확인하려면 `await` 코드 앞뒤에 print 문을 추가해보자. 다운로드 작업이 중단되고 시스템이 다른 작업을 수행한 후 제어권을 돌려주는 과정에서 print 문이 천천히 실행되는 것을 볼 수 있다.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    print("이미지 다운로드 시작")
    let image = try await downloadImage(imageNumber: imageNumber)
    print("이미지 다운로드 완료")
    print("메타데이터 다운로드 시작")
    let metadata = try await downloadMetadata(for: imageNumber)
    print("메타데이터 다운로드 완료")
    return DetailedImage(image: image, metadata: metadata)
}
```

인터넷 속도가 너무 빨라 print 문이 잘 보이지 않는다면 Apple이 제공하는 `Task.sleep` 메서드를 사용할 수 있다. 이 함수는 주어진 시간 동안 스레드를 일시 중지하는 역할만 하며, async/await을 실험하는 데 유용하다.

*   **참고**: *Xcode 13 Beta 1 기준으로 `Task.sleep`이 크래시를 일으키는 것 같다. `await Task.sleep(2 * 1_000_000_000)`.*

`await`에 대한 마지막 중요한 점: `await` 위쪽 코드를 실행한 스레드와 아래쪽 코드(일반적으로 연속(continuation)이라고 함)를 실행할 스레드가 반드시 같지 않다는 것이다. 이는 UI 작업을 다룰 때 중요한 영향을 미친다. ViewController처럼 메인 스레드가 필요한 컨텍스트에서 `await`을 사용한다면, `await`이 표시된 함수에 `@MainActor` 속성을 추가하거나 전체 클래스 선언에 이 속성을 추가해야 한다. Swift의 새로운 동시성 모델이 어떻게 작동하는지 자세히 알고 싶다면 [Swift 동시성: 내부 구조](https://developer.apple.com/videos/play/wwdc2021/10254/) WWDC2021 세션을 참고하자.


## 동기와 비동기 세계를 잇는 'Task'의 역할

`Task`를 사용하면 동기와 비동기 세계 사이에 '다리'를 놓을 수 있다. 왜 이 기능이 필요한지 다음 코드 예제로 살펴보자:

```swift
func performDownload() {
    let imageDetail = try? await downloadMetadata(for: 1)
}
```

컴파일러는 이 코드가 잘못 실행되는 것을 방지하기 위해 다음과 같은 오류를 표시한다:

> 동시성을 지원하지 않는 함수 내에서 'async' 호출이 발생했습니다. 'performDownload()' 함수에 'async'를 추가해 비동기로 만들어 주세요

컴파일러는 `performDownload` 함수를 async로 표시하라고 제안한다.

```swift
func performDownload() async {
    let imageDetail = try? await downloadMetadata(for: 1)
}
```

하지만 이 방법이 항상 가능한 것은 아니다. 만약 `performDownload`가 뷰 컨트롤러나 비동기 컨텍스트를 제공할 수 없는 다른 곳에 위치한다면 어떻게 해야 할까?

이 문제를 해결하기 위해 `Task {}`를 사용해 동기 함수를 비동기 세계와 연결할 수 있다.

```swift
func performDownload() {
    Task {
        let imageDetail = try? await downloadMetadata(for: 1)
    }
}
```

명시적으로 비동기 컨텍스트를 생성했기 때문에, 이제 동기 컨텍스트에서도 문제 없이 performDownload를 호출할 수 있다.


## 비동기 속성 접근

더 나아가, 읽기 전용 속성에도 `await` 키워드를 사용할 수 있다.

다음과 같은 래퍼 객체가 있다고 가정해보자:

```swift
struct Character {
    let id: Int
}
```

`downloadImageAndMetadata`를 호출해 이미지와 메타데이터를 가져올 수 있지만, 이 객체에 계산 속성(computed property)을 추가해 이미지와 메타데이터를 각각 독립적으로 가져올 수도 있다.

```swift
struct Character {
    let id: Int

    var metadata: ImageMetadata {
        get async throws {
            let metadata = try await downloadMetadata(for: id)
            return metadata
        }
    }

    var image: UIImage {
        get async throws {
            return try await downloadImage(imageNumber: id)
        }
    }
}
```

이렇게 작성한 속성은 다음과 같이 사용할 수 있다:

```swift
let metadata = try? await character.metadata
```


## 요약

`async/await`에 대한 긴 설명이었지만, 포함된 예제와 논의를 통해 이 개념이 어떻게 작동하는지 이해하는 데 도움이 되었기를 바란다. `async/await`는 새로운 동시성 시스템의 핵심이므로 제대로 파악해야 한다. 앞으로의 글은 이렇게 길지 않을 것이다. 기본 개념을 다루는 것은 세부 사항을 놓치지 않도록 신경 써야 하기 때문에 많은 노력이 필요하다. 이 글이 여러분에게 유용하길 바란다.

UIKit 프로젝트에서 다운로드한 이미지와 메타데이터를 활용한 샘플 프로젝트를 만들었다. [여기](https://www.andyibanez.com/archives/AsyncAwaitIntro.zip)에서 다운로드할 수 있다.

프로그램을 실행하면 다음과 같이 콘텐츠를 다운로드하여 표시한다:

<img width="640" height="1136" alt="async/await 튜토리얼 1 결과" src="https://github.com/user-attachments/assets/252f2016-47ae-4571-b704-3118495ad886" />

`viewDidAppear` 메서드에서 다음 코드를 확인할 수 있다:

```swift
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)

    // MARK: METHOD 1 - Async/Await 사용

    Task {
        if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
            self.imageView.image = imageDetail.image
            self.metadata.text = "(imageDetail.metadata.name) ((imageDetail.metadata.firstAppearance) - (imageDetail.metadata.year))"
        }
    }

    // MARK: METHOD 2 - async 프로퍼티 사용

//        Task {
//            let character = Character(id: 1)
//            if
//                let metadata = try? await character.metadata,
//                let image = try? await character.image{
//                imageView.image = image
//                self.metadata.text = "(metadata.name) ((metadata.firstAppearance) - (metadata.year))"
//            }
//        }

    // MARK: Method 3 - 콜백 사용

//        downloadImageAndMetadata(imageNumber: 1) { imageDetail, error in
//            DispatchQueue.main.async {
//                if let imageDetail = imageDetail {
//                    self.imageView.image = imageDetail.image
//                    self.metadata.text =  "(imageDetail.metadata.name) ((imageDetail.metadata.firstAppearance) - (imageDetail.metadata.year))"
//                }
//            }
//        }
}
```

`MARK: - Method x` 아래의 내용을 주석 처리하거나 해제하여 다양한 데이터 수집 방법으로 아웃렛을 채울 수 있다. 이를 통해 Swift에서 `async/await`가 어떻게 작동하는지 더 잘 이해할 수 있을 것이다.

앞서 언급한 두 가지 핵심 사항을 다시 짚어보자:

> 1. 절차적 코드는 위에서 아래로 실행된다. 제어는 `return`을 통해 호출자에게 돌아간다.
> 2. 콜백 기반 동시성은 비동기 작업을 생성하지만, 해당 작업이 실행 중이더라도 현재 스레드의 실행을 문제없이 계속한다. 제어는 완료 핸들러를 통해 호출자에게 돌아간다.

이제 한 가지를 추가해 정리할 수 있다:

> 3. `async/await`는 절차적 프로그래밍처럼 순서대로 실행된다. `await` 호출을 만나면 작업이 일시 중단되고 호출자 대신 시스템에 제어권을 넘긴다. 콜백 기반 동시성과 달리, 작업이 완료될 때까지 아래 문장의 실행을 계속하지 않는다. 시스템은 스레드를 활용해 다른 작업을 수행하다가, 해당 함수로 다시 돌아올 시점이 되면 순차적으로 실행을 재개한다.

이제 [Swift에서 클로저 기반 코드를 async/await로 변환하기](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/) 시리즈의 세 번째 글을 통해 컨티뉴에이션, 명시적 컨티뉴에이션, 클로저 및 델리게이트 기반 코드를 `async/await`로 연결하는 방법을 배울 준비가 되었다.




