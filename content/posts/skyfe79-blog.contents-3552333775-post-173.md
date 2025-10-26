---
title: "Swift 모던 컨커런시 - 정리 및 감사의 말"
date: 2025-10-25T11:02:49Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/modern-swift-concurrency-summary-cheatsheet-thanks/

WWDC21 이후, Swift 5.5에 도입된 새로운 컨커런시 기능에 대해 깊이 있게 다뤘다. 다루어진 주제가 많아 이 시리즈를 마무리하며 각 글의 핵심 내용을 요약하는 글을 작성하기로 했다. 필요한 경우 관련 글의 링크를 제공해 더 자세한 정보를 확인할 수 있도록 했다.


# async/await 이해하기

*   `async`와 `await`는 모던 컨커런시 시스템의 가장 핵심적인 키워드다.
*   프로그래밍을 처음 배울 때는 코드가 순차적으로 실행되는(절차적 프로그래밍) 방식에 익숙해진다. 작성한 순서대로 코드가 실행된다.
*   모던 컨커런시를 다루기 전에는, 애플은 클로저 기반의 콜백 방식을 제공했다. 작업이 완료되면 클로저를 통해 알림을 받거나, 오래된 코드에서는 델리게이트 패턴을 사용했다. 콜백과 델리게이트 기반의 컨커런시는 프로그램 실행 순서를 변경할 수 있다. 일련의 코드가 실행되면서 다른 스레드에서 새로운 데이터를 받을 수 있게 해주지만, 시간이 지날수록 이해하기 어려워질 수 있다.
*   async/await를 사용하면 위에서 아래로 순차적으로 실행되는 동시성 코드를 작성할 수 있다. 비동기적으로 호출할 수 있는 함수는 함수 시그니처에 `async`를 표시해야 한다.

```swift
func downloadData() async throws -> CustomData { 
	//...
}
```

*   `async`로 표시된 함수를 호출할 때는 `await` 키워드를 앞에 붙여야 한다.

```swift
func processData() async throws -> CustomData {
   let newData = try await downloadData()
   return newData
}
```

*   코드 실행이 `await` 키워드에 도달하면, 코드 실행이 **일시 중단**될 수 있다. 이때 해당 스레드는 다른 작업을 수행할 수 있게 된다. 시스템이 다른 작업을 할당하는 방식이다. 코드가 일시 중단되면 `await` 아래의 코드는 비동기 작업이 완료될 때까지 실행되지 않는다.
*   코드가 일시 중단된 경우, 비동기 작업이 완료되면 시스템이 다시 코드로 돌아와 실행을 계속한다. 즉, `await` 호출 아래의 모든 코드가 다시 실행을 시작한다.
*   `async/await`는 스레드 일시 중단 덕분에 위에서 아래로 실행되는 절차적 흐름을 유지할 수 있게 해준다.
*   `await` 호출 아래의 코드를 *컨티뉴에이션*이라고 부른다. 델리게이트나 클로저 기반의 컨커런시 코드를 async/await로 변환할 때 이 개념을 알아두면 유용하다.
*   중요한 점은 컨티뉴에이션이 일시 중단된 스레드와 같은 스레드에서 실행되지 않을 수 있다는 것이다. UI를 업데이트해야 한다면 @MainActor에서 해당 코드를 실행해야 한다.
*   `async` 코드는 `async` 컨텍스트에서 실행되어야 한다. `async`로 표시된 함수 내부이거나, 직접 `Task {}`로 컨텍스트를 생성한 경우에 해당한다.

async/await에 대해 더 알고 싶다면 [Swift에서 async/await 이해하기](https://www.andyibanez.com/posts/understanding-async-await-in-swift/) 글을 참고하자.


# 델리게이트와 클로저 기반 코드를 async/await로 변환하기

* 컴파일러가 자동으로 변환해주는 경우가 있다. 클로저를 사용할 것으로 예상되는 메서드를 작성하면, 컴파일러가 이미 `async` 버전을 자동 생성한 경우도 있다.

* 직접 변환할 수도 있다.
* 변환하려면 수동으로 *콘티뉴에이션*을 생성해야 한다. 콘티뉴에이션은 `await` 호출 이후의 모든 작업을 의미한다.
* `withCheckedContinuation` 또는 `withCheckedThrowingContinuation` 함수를 사용해 클로저 기반 호출을 감싸거나, 델리게이트 기반 호출에서 나중에 사용할 콘티뉴에이션 참조를 저장할 수 있다.
* 이 메서드들은 동시성 작업이 완료될 때 명시적으로 호출해야 하는 콘티뉴에이션을 제공한다. `withCheckedThrowingContinuation`의 경우 반환 값을 전달하거나 에러를 던질 수 있다.
* **반드시** 콘티뉴에이션을 정확히 한 번 호출해야 한다. 호출을 잊지 말고, 한 번만 호출해야 한다.
* 다음 코드는 클로저 기반 동시성을 async/await로 변환하는 방법을 보여준다.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    return try await withCheckedThrowingContinuation({
        (continuation: CheckedContinuation<DetailedImage, Error>) in
        downloadImageAndMetadata(imageNumber: imageNumber) { image, error in
            if let image = image {
                continuation.resume(returning: image)
            } else {
                continuation.resume(throwing: error!)
            }
        }
    })
}
```

* 델리게이트 기반 호출을 async/await로 변환하는 것은 조금 더 복잡하지만 불가능하지 않다. `withChecked*Continuation` 호출로 제공된 콘티뉴에이션을 저장하고 적절한 시점에 호출해야 한다.

```swift
class ContactPicker: NSObject, CNContactPickerDelegate {
    private typealias ContactCheckedContinuation = CheckedContinuation<CNContact, Never> // 1

    private unowned var viewController: UIViewController
    private var contactContinuation: ContactCheckedContinuation? // 2
    private var picker: CNContactPickerViewController

    init(viewController: UIViewController) {
        self.viewController = viewController
        picker = CNContactPickerViewController()
        super.init()
        picker.delegate = self
    }

    func pickContact() async -> CNContact { // 3
        viewController.present(picker, animated: true)
        return await withCheckedContinuation({ (continuation: ContactCheckedContinuation) in
            self.contactContinuation = continuation
        })
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        contactContinuation?.resume(returning: contact) // 4
        contactContinuation = nil
        picker.dismiss(animated: true, completion: nil)
    }
}
```

* 델리게이트 기반 동시성뿐만 아니라 동일 스레드에서 작동하는 델리게이트 기반 호출도 이 방법으로 변환할 수 있다. (하지만 과도한 엔지니어링이 되지 않도록 노력할 가치가 있는지 고려해야 한다)

클로저 또는 델리게이트 기반 코드를 `async/await`로 변환하는 방법에 대해 더 알아보려면 [Swift에서 클로저 기반 코드를 async/await로 변환하기](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/) 글을 참고하라.


# 구조화된 동시성(Structured Concurrency)

* 여러 개의 `await` 호출을 연속해서 사용한다고 해서 반드시 동시성이 발생하는 것은 아니다. 아래 코드는 동시성을 활용하지 않는다. 비록 `await` 호출들이 서로 독립적이어서 동시에 실행될 수 있음에도 불구하고 말이다.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    let image = try await downloadImage(imageNumber: imageNumber)
    let metadata = try await downloadMetadata(for: imageNumber)
    return DetailedImage(image: image, metadata: metadata)
}
```

* 구조화된 동시성은 위에서 아래로 읽을 수 있는 동시성 코드 작성을 가능하게 한다. 여러 작업을 쉽게 병렬로 실행할 수 있다.
* 구조화된 동시성에는 두 가지 유형이 있다: `async let` 호출과 태스크 그룹(Task Groups).


## async let을 이용한 동시성 처리

*   `await` 키워드로 호출할 수 있는 작업은 동시에 실행할 수도 있다.
*   이를 위해선 변수 정의 시 `let`이나 `var` 앞에 `async` 키워드를 추가하고, `await` 호출을 제거하면 된다.
*   이후 필요한 시점에 변수에 대해 `await`만 하면 된다.
*   아래 코드는 위와 동일한 기능을 하지만, 두 `async` 작업을 동시에 수행한다.

```swift
func downloadImageAndMetadata(imageNumber: Int) async throws -> DetailedImage {
    async let image = downloadImage(imageNumber: imageNumber)
    async let metadata = downloadMetadata(for: imageNumber)
    return try DetailedImage(image: await image, metadata: await metadata)
}
```

*   `image`와 `metadata`가 비동기 값임에도 불구하고, 함수 반환 전에 값을 `await`하기 때문에 코드 가독성이 여전히 높다.
*   `async let`은 동시에 수행해야 할 작업 수가 명확할 때 이상적이다. 위 예제에서는 `downloadImage`와 `downloadMetadata` 두 가지 작업이 있다.

`async let`을 활용한 구조적 동시성에 대해 더 알고 싶다면 [Structured Concurrency in Swit: Using async let](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/) 글을 참고하라.


## 그룹 태스크 활용 방법

<!-- -->
* 동시 실행할 작업의 수를 미리 알 수 없는 경우에 그룹 태스크를 사용한다. 예를 들어 웹 서비스에서 가변적인 수의 URL을 가져온 후, 이를 동시에 다운로드하려는 경우에 적합하다.
* 그룹 태스크를 시작할 때는 `withThrowingTaskGroup` 또는 `withTaskGroup` 메서드를 사용한다.
* 위 예제에서는 여러 이미지를 동시에 다운로드하기 위해 태스크 그룹을 생성했다.

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

<!-- -->
* `group` 변수는 다운로드된 데이터를 담고 있다. 이는 `AsyncSequence` 타입이므로 반복문으로 처리하거나 `filter`, `map`, `reduce` 같은 함수를 적용할 수 있다.
* 그룹의 우선순위를 지정할 수 있어, `async let`보다 유연하게 구조화된 동시성을 구현할 수 있다.

```swift
group.async(priority: .userInitiated) {
   //...
}
```

<!-- -->
* 작업이 취소될 상황에 대비하려면 `asyncUnlessCancelled`를 사용한다.

```swift
group.asyncUnlessCancelled(priority: nil) {
   //...
}
```


### 전송 가능 타입(Sendable Types)

* **전송 가능 타입**은 동시성 작업과 잘 호환되는 타입을 의미한다. 이러한 타입을 동시성 컨텍스트에서 사용하면 컴파일러가 오류를 발생시키지 않는다. 또한 `@Sendable` 클로저는 오직 `Sendable` 타입(프로토콜을 따르는 경우)과만 함께 작동한다.
* **`@Sendable` 클로저**는 변경 가능한 변수를 캡처할 수 없다.
* 값 타입(value types), 액터(actors), 클래스, 또는 자체적인 동기화를 구현한 객체만 캡처해야 한다.

Group Tasks와 전송 가능 타입에 대해 더 자세히 알고 싶다면 다음 글을 참고하기 바란다:  
[Swift에서 Task Groups를 활용한 구조화된 동시성](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/)  
[Swift의 새로운 동시성 모델에서 액터 이해하기](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)


## 태스크 트리

구조적 동시성(`async let`과 태스크 그룹 모두에서)의 중요한 개념이 바로 태스크 트리이다.

* `async` 함수는 다른 `async` 태스크를 생성할 수 있다. 이렇게 생성된 태스크는 호출한 태스크의 *자식* 태스크가 된다.
* 자식 태스크는 우선순위, 지역 변수, 취소 상태와 같은 정보를 부모 태스크로부터 상속받는다.
* 부모 태스크는 자식 태스크들이 모두 작업을 완료해야만 자신의 작업을 끝낼 수 있다.
* 태스크 취소는 태스크 트리에 따라 관리되며 협력적인 방식으로 이루어진다. 태스크가 취소되면(직접 `cancel` 또는 `cancellAll`을 호출하거나 오류가 발생한 경우), 트리 내의 태스크들이 즉시 중단되지 않는다. 대신 태스크는 `cancelled`로 표시되지만, 작업을 중단하기 적절한 시점까지 계속 실행된다. 부모 태스크가 취소되면 자식 태스크들도 모두 `cancelled`로 표시된다.
* 태스크의 취소 상태를 확인하고 작업 중단 여부를 결정하려면 오류를 던질 수 있는 태스크에는 `Task.checkCancellation()`을, 오류를 던지지 않는 태스크에는 `Task.isCancelled`를 사용한다.

```swift
func downloadImage(imageNumber: Int) async throws -> UIImage {
    try Task.checkCancellation() // <- 취소된 경우 여기서 오류 발생
    let imageUrl = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/(imageNumber).png")!
    let imageRequest = URLRequest(url: imageUrl)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}
```

태스크 트리에 대해 더 알고 싶다면 [Swift에서 구조적 동시성: async let 사용하기](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/) 글을 참고하자.


# 비구조적 동시성

비구조적 동시성은 작업의 흐름이 정해진 절차를 따르지 않을 때 유용하다. 특이한 실행 흐름을 크게 줄이는 데 도움이 되며, 구조적 동시성보다 더 많은 제어 권한을 제공한다.

비구조적 동시성을 구현하는 두 가지 방법이 있다: `Task` 호출과 `Task.detached`를 이용한 분리된 태스크.


## 태스크(Task)

* `Task {}`를 사용하면 실제로 동시성 태스크를 실행한다. 이는 비동기(async)와 동기(sync) 세계 사이의 '다리' 역할을 한다.
* 변수에 저장해 필요할 때 수동으로 취소할 수 있다.
* 특정 우선순위로 시작할 수도 있다.

비정형 동시성(Unstructured Concurrency)과 태스크에 대해 더 알고 싶다면 [Swift의 비정형 동시성 소개](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/) 글을 참고한다.


## 독립 실행형 태스크

* `Task.detached {}`로 실행한다.
* 다른 종류의 태스크와 달리, 부모 태스크로부터 아무것도 상속받지 않는다. 심지어 우선순위도 물려받지 않는다.
* 실행된 컨텍스트와 완전히 독립적으로 동작한다.

독립 실행형 태스크에 대해 더 알아보려면 [Swift에서 독립 실행형 태스크를 이용한 비정형 동시성 처리](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/) 글을 참고한다.


# 액터 모델

* 액터는 참조 타입으로, 자신의 상태를 프로그램의 다른 부분과 격리한다. 이는 데이터 경쟁을 방지하는 완벽한 메커니즘이다.
* 액터는 내부적으로 접근 동기화를 제공한다. 이로 인해 데이터 경쟁이 발생하지 않는다.
* 액터 상태를 직접 수정할 수 없다. 액터를 변경하는 모든 호출은 반드시 액터 자체를 통해서만 가능하다.
* 액터가 제공하는 모든 메서드는 `await` 호출을 통해 접근해야 한다. 명시적으로 표시하지 않아도 이 규칙이 적용된다.
* 격리할 필요가 없거나 격리할 수 없는 프로퍼티는 `nonisolated`로 표시할 수 있다.
* 액터 재진입(여러 번 진입)을 고려해 설계해야 한다. 상태가 변경되기 때문에 추가적인 고려사항이 생길 수 있다. 예를 들어 이미지를 다운로드하고 캐시하는 액터는 빠르게 연속으로 진입되면 같은 이미지를 두 번 다운로드할 수도 있다.

[Swift의 새로운 동시성 모델에서 액터 이해하기](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)


# @MainActor와 글로벌 액터


*   서로 다른 파일과 타입에 걸쳐 글로벌 액터를 정의할 수 있다. 특정 액터에서 실행되도록 클래스를 표시하면 모든 코드가 동일한 스레드에서 실행된다.
*   `@globalActor` 속성으로 글로벌 액터를 선언하고, 해당 액터 이름 앞에 `@`를 붙여 사용한다. 아래 예제에서는 `MediaActor`라는 액터를 만들고, 이 액터에서 실행되는 `videogames` 변수를 생성한다.

```swift
@globalActor
struct MediaActor {
  actor ActorType { }

  static let shared: ActorType = ActorType()
}

struct Videogame {
    let id = UUID()
    let name: String
    let releaseYear: Int
    let developer: String
}

@MediaActor var videogames: [Videogame] = []
```

*   `@MainActor`는 메인 스레드에서 실행되는 Swift가 제공하는 특별한 글로벌 액터다. 뷰 컨트롤러, 뷰 모델 등 메인 스레드에서 강제로 실행하고 싶은 코드에 `@MainActor`를 표시할 수 있다. 클래스에 액터를 표시하면 모든 프로퍼티와 메서드가 동일한 액터에서 실행된다. 아래 예제에서는 뷰 컨트롤러에 `@MainActor` 속성을 추가해 모든 코드가 메인 스레드에서 실행되도록 보장한다.

```swift
@MainActor
class GameLibraryViewController: UIViewController {
	//...
	nonisolated var fetchVideogameTypes() -> [VideogameType] { ... }
	//...
}
```

특정 메서드의 액터를 재정의할 수도 있다.

```swift
@MainActor
class GameLibraryViewController: UIViewController {
   @MediaActor func doThisInAnotherActor() {}
}
```

[Swift의 @MainActor와 글로벌 액터](https://www.andyibanez.com/posts/mainactor-and-global-actors-in-swift/)


# @TaskLocal을 활용한 태스크 간 데이터 공유

*   `@TaskLocal` 프로퍼티 래퍼를 사용하면 로컬 태스크 간에 데이터를 공유할 수 있다.
*   동일한 태스크 트리에 속한 태스크들만 이 데이터를 상속받는다. 특정 태스크 내에서 시작된 분리된(detached) 태스크는 상속하지 않는다.

```swift
class ViewController: UIViewController {
    @TaskLocal static var currentVideogame: Videogame?
    // ...
}
```

*   오직 정적(static) 프로퍼티만 이 프로퍼티 래퍼를 사용할 수 있다.
*   값을 쓰려면 반드시 값과 바인딩해야 한다.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // 뷰 로드 후 추가 설정 수행
    
    let vg = Videogame(title: "The Legend of Zelda: Ocarina of Time", year: 1998)
    Self.$currentVideogame.withValue(vg) {
        // 여기서 LocalValue를 사용하는 비동기 태스크를 실행할 수 있다
    }
}
```

*   값을 읽을 때는 `await` 호출이 필요하다.

```swift
func expensiveVidegameOperation() async {
    if let vg = await ViewController.currentVideogame {
        print("We are processing (vg.title)")
    }
}
```

[새로운 Swift 동시성 모델에서 @TaskLocal 프로퍼티 래퍼로 태스크 간 데이터 공유하기](https://www.andyibanez.com/posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model/)


# AsyncSequence와 AsyncStream

* `AsyncSequence`는 시간에 따라 값을 받을 수 있게 해주며, 루프 안에서 `await`로 기다리거나 `filter`, `map`, `reduce` 같은 함수를 적용할 수 있다.

```swift
func loadVideogames() async {
    let url = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part11/videogames.csv")!
    
    let videogames =
        url
        .lines
        .filter { $0.contains("|") }
        .map { Videogame(rawLine: $0) }
    
    do {
        for try await videogame in videogames {
            print("(videogame.title) ((videogame.year ?? 0))")
        }
    } catch {
        
    }
}
```

* 이 시퀀스는 루프에 넣기 전까지는 "시작"하지 않는다. 고차 함수를 적용하는 것은 단순히 `await for` 루프에서 받을 값을 제한할 뿐이다.
* WWDC21에서 `NSNotificationCenter` API를 포함한 여러 API가 이를 지원하도록 업데이트되었다.
* `AsyncStream` 객체는 어딘가에서 오는 값 스트림을 가져와 `for await` 루프에서 사용할 수 있는 형태로 변환하는 데 사용할 수 있다.
* 예를 들어, 델리게이트에서 실시간으로 GPS 업데이트를 받는 경우 이를 감싸서 루프 안에서 새 좌표를 받을 수 있다.

[Swift에서 AsyncSequence 사용하기](https://www.andyibanez.com/posts/using-asyncsequence-in-swift/)


# 감사의 말

이 시리즈의 글들은 제가 2019년에 웹사이트를 재개장한 이후 가장 많이 방문된 페이지 중 하나가 되었습니다. 그 덕분에 커뮤니티 구성원들로부터 많은 피드백을 받을 수 있었습니다.

오타나 어색한 문장 표현에 대해 제게 알려주신 모든 분들께 감사 인사를 드리고 싶습니다. 여러분의 의견과 코멘트를 바탕으로 글의 질을 높이기 위해 많은 노력을 기울였습니다. 여러분 덕분에 이 시리즈의 완성도가 크게 향상되었습니다.

정말 많은 이메일을 받았는데, 연락 주신 분들이 너무 많아서 일일이 이름을 언급하기 어려울 정도입니다. 제 블로그의 질을 높이는 데 도움을 주신 모든 분들께 진심으로 감사드립니다. 또한 답장을 드리지 못한 분들께는 사과의 말씀을 전합니다. 이메일이 너무 많아서 누구에게 답장을 보냈는지 놓치는 경우가 있었습니다.

특별히 한 분을 이름으로 언급하고 싶은데, 이 분은 시리즈의 모든 글을 꼼꼼히 검토하고 매우 상세한 개선 사항을 담은 긴 이메일을 보내주셨습니다. 이 분의 이메일을 받을 때마다 수정 사항을 반영하는 데 많은 시간을 할애했지만, 그 모든 노력은 결실을 맺었습니다. 이 시리즈는 제가 가장 자랑스러워하는 작품 중 하나가 되었습니다. 그 분은 [Dennis Birch](https://twitter.com/dennisbirch2)입니다. Dennis 덕분에 이 글 시리즈가 제가 가장 애정하는 작품 중 하나가 될 수 있었습니다. 정말 큰 감사를 드립니다.




