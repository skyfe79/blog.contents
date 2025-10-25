---
title: "Swift 모던 컨커런시 - 비구조적 동시성(Unstructured Concurrency) 소개"
date: 2025-09-13T08:24:13Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/


이 글을 읽기 전에 Swift의 구조화된 컨커런시에 대한 이해가 필요하다. 해당 개념이 익숙하지 않다면 이 시리즈의 [Swift 모던 컨커런시 - 구조적 동시성과 async let 사용법](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/)과 [Swift 모던 컨커런시 - Task Groups를 활용한 Swift의 구조화된 동시성](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/) 글을 먼저 읽어보길 권한다.

지금까지는 Swift 5.5에 도입된 새로운 API를 활용해 구조화된 컨커런시를 탐구하는 데 집중했다. 구조화된 컨커런시는 프로그램의 흐름을 선형적으로 유지하면서도 이해하기 쉬운 태스크 계층 구조를 형성하는 데 탁월하다. 태스크 취소를 체계적으로 관리하고, 오류 처리를 동시성 없이 작성한 코드만큼 명확하게 만들어준다. 구조화된 컨커런시는 코드 가독성을 해치지 않으면서도 여러 작업을 동시에 실행할 수 있는 강력한 도구다.


## 비구조적 동시성 소개

구조적 동시성이 매우 유용하지만, 경우에 따라(비록 드물겠지만) 작업에 어떤 구조적 패턴도 적용할 수 없는 상황이 발생한다. 이런 경우에는 더 단순함을 희생하는 대신 작업에 대한 더 큰 제어권을 제공하는 비구조적 동시성을 활용할 수 있다. 다행히 Swift 5.5는 많은 단순성을 포기하지 않으면서도 이를 구현할 수 있는 도구를 제공한다. 예를 들어 사용자에게 이미지 다운로드 기능을 제공하면서도 동시에 다운로드 취소 옵션을 제공할 수 있다.

비구조적 동시성을 사용해야 하는 전형적인 상황은 다음과 같다:

* 비동기 컨텍스트가 아닌 곳에서 작업을 시작하는 경우. 이 경우 작업은 자신의 범위를 벗어나 지속될 수 있다.
* 부모 작업의 어떤 정보도 상속받지 않는 독립적인 작업(Detached tasks)을 실행하는 경우.

이 글에서는 전자의 경우에 초점을 맞춰 설명한다.


## 동기 컨텍스트에서 태스크 실행하기

이전에 이미 다룬 내용이지만, 이번에는 `Task {}` 블록에 대해 깊이 있게 알아본다. `async/await`를 처음 설명할 때 언급했듯이, 태스크를 `await`하려면 `async` 컨텍스트 내에 있어야 한다. 함수 시그니처에 `async`가 포함되어 있다면 아무 문제 없이 `await`를 사용할 수 있다.

문제는 Apple의 SDK가 처음부터 동시성을 지원하도록 설계되지 않았다는 점이다. `UIKit`을 예로 들면, 뷰 컨트롤러 라이프사이클 메서드 중 `viewDidAppear` 같은 메서드는 `async`로 표시되지 않는다. 동시성 작업을 수행하거나 `async` 태스크를 `await`하려면 `Task` 블록을 사용해야 한다.

[Swift의 async/await 이해하기](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)에서 이미 이 방법을 사용했다. 해당 글을 읽지 않았다면, 결국 다음과 같은 코드로 마무리했음을 알 수 있다:

```swift
// MARK: - 정의

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

override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)

    Task {
        if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
            self.imageView.image = imageDetail.image
            self.metadata.text = "(imageDetail.metadata.name) ((imageDetail.metadata.firstAppearance) - (imageDetail.metadata.year))"
        }
    }
}
```

위 코드에는 `await` 가능한 몇 가지 메서드가 있다. 이들을 `viewDidAppear`에서 호출하려 하지만, `viewDidAppear` 함수 시그니처에 `async`가 없으므로 직접 호출할 수 없다. 대신 `async`를 사용해 비동기 컨텍스트를 생성하고, 그 안에서 `await`를 사용해야 한다.

이렇게 하는 것의 의미는 흥미롭다. 첫째, `Task {}`는 명시적인 태스크를 생성한다. 둘째, 새로운 태스크를 시작하므로 `Task {}` 블록 외부의 코드는 블록 내부와 동시에 실행된다.

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // 뷰 로드 후 추가 설정
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        async {
            if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
                print("이미지 다운로드 완료")
            }
        }

        print("비동기 블록과 함께 실행 계속")
    }
}
```

이 코드를 실행하면 출력 결과가 다음과 같다:

```swift
비동기 블록과 함께 실행 계속
이미지 다운로드 완료
```

네트워크에서 무언가를 다운로드하는 것보다 선형 코드 실행이 훨씬 빠르므로, 이 출력 결과는 항상 동일하게 나타난다. 여러 개의 `Task {}` 블록을 사용하면 각각 비동기 태스크가 시작된다는 점도 쉽게 이해할 수 있다.

마지막으로 가장 흥미로운 부분은, 이렇게 `Task`를 사용하면 `Task<T, Error>` 타입의 핸들을 반환한다는 점이다. 이 핸들을 저장해두면 나중에 명시적으로 태스크를 취소하거나 결과를 기다리는 등의 작업을 수행할 수 있다.

이것이 바로 "비구조화된" 태스크의 핵심이다. 한 곳에서 태스크를 시작하고, 전혀 관련 없는 다른 곳에서 취소할 수 있다.

예를 들어 버튼 탭으로 다운로드를 시작할 수 있다:

```swift
// 전체 코드는 글 마지막에서 확인할 수 있다.
class ViewController: UIViewController {
// ...
var downloadAndShowTask: Task<Void, Never>? {
    didSet {
        if downloadAndShowTask == nil {
            triggerButton.setTitle("다운로드", for: .normal)
        } else {
            triggerButton.setTitle("취소", for: .normal)
        }
    }
}

func downloadAndShowRandomImage() {
    let imageNumber = Int.random(in: 1...3)
    downloadAndShowTask = async {
        do {
            let imageMetadata = try await downloadImageAndMetadata(imageNumber: imageNumber)
            imageView.image = imageMetadata.image
            let metadata = imageMetadata.metadata
            metadataLabel.text = "(metadata.name) ((metadata.firstAppearance) - (metadata.year))"
        } catch {
            showErrorAlert(for: error)
        }
        downloadAndShowTask = nil
    }
}

// ViewController 내부
@IBAction func triggerButtonTouchUpInside(_ sender: Any) {
    if downloadTask == nil {
        // 실행 중인 태스크가 없으므로 다운로드 시작
        Task {
            await downloadAndShowRandomImage()
        }
    } else {
        // 실행 중인 태스크가 있으므로 취소
        cancelDownload()
    }
}

// ...
}
```

사용자가 원할 때 다운로드를 취소할 수도 있다:

```swift
func cancelDownload() {
    downloadAndShowTask?.cancel()
}
```

전체 프로그램에는 `triggerButton`이 포함되어 있으며, `downloadAndShowTask` 값이 변경될 때마다 버튼 레이블이 바뀐다. 값이 nil이면 실행 중인 태스크가 없으므로 버튼으로 이미지를 다운로드한다. 그렇지 않으면 버튼으로 작업을 취소한다.

`downloadAndShowTask`는 `Task<Void, Never>` 타입이다. 태스크 자체는 아무것도 반환하지 않고 오류도 던지지 않기 때문이다. 버튼은 이미지를 다운로드하고 레이블을 설정한다.

이미지를 다운로드하지만 직접 처리하지 않으려면, 태스크가 특정 값을 반환하도록 정의할 수 있다.

다음 예제는 더 복잡하지만, `Task {}` 비구조화 태스크의 유연성을 잘 보여준다.

먼저 뷰 컨트롤러 선언에 `@MainActor`를 추가한다. 메인 스레드 외부에서 뷰 컨트롤러 값에 접근할 가능성이 있기 때문이다.

```swift
@MainActor
class ViewController: UIViewController //...
```

다음으로 `downloadAndShowTask`를 `downloadTask`로 변경하고 시그니처를 `Task<DetailedImage, Error>`로 수정한다. 이렇게 하면 `DetailedImage`를 `await`하거나 필요시 태스크 내부에서 오류를 던질 수 있다.

```swift
var downloadTask: Task<DetailedImage, Error>? {
    didSet {
        if downloadTask == nil {
            triggerButton.setTitle("다운로드", for: .normal)
        } else {
            triggerButton.setTitle("취소", for: .normal)
        }
    }
}
```

새로운 메서드 `beginDownloadingRandomImage`를 생성한다. 이 메서드는 이미지 다운로드를 시작하고 `downloadTask` 핸들에 저장한다. 아울렛을 업데이트하는 코드도 함께 작성한다.

```swift
func beginDownloadingRandomImage() {
    let imageNumber = Int.random(in: 1...3)
    downloadTask = Task {
        return try await downloadImageAndMetadata(imageNumber: imageNumber)
    }
}

func showImageInfo(imageMetadata: DetailedImage) {
    imageView.image = imageMetadata.image
    let metadata = imageMetadata.metadata
    metadataLabel.text = "(metadata.name) ((metadata.firstAppearance) - (metadata.year))"
}
```

`downloadAndShowRandomImage` 구현을 업데이트해 두 새 함수를 활용한다.

```swift
func downloadAndShowRandomImage() async {
    beginDownloadingRandomImage()
    do {
        if let image = try await downloadTask?.value {
            showImageInfo(imageMetadata: image)
        }
    } catch {
        showErrorAlert(for: error)
    }
    downloadTask = nil
}
```

이제 이 메서드는 `beginDownloadingImage`를 호출해 `downloadTask`에 값을 할당한다. 그리고 `downloadTask?.value`를 호출한다. `.value`는 다운로드가 완료되면 이미지를 반환하므로 `await`가 필요하다.

`cancelDownload`는 항상 동일하다. 언제든지 다운로드를 취소(또는 시작)할 수 있다.

이렇게 생성된 태스크는 우선순위, 로컬 값, 액터를 상속받는다. 태스크는 자신의 범위를 넘어서도 존재할 수 있으므로 수명을 더 잘 제어할 수 있다.


## 요약

`async`의 실제 동작 방식을 살펴봤다. 이를 통해 명시적으로 생성하고 나중에 수동으로 취소할 수 있는 작업을 만들었다. `Task {}` 블록을 사용하면 구조화되지 않은 동시성 작업을 처리할 수 있다. 이는 작업을 더 세밀하게 제어해야 할 때 유용하다. 필요할 때 작업을 취소할 수 있는 기능은 특히 오래 실행되는 작업이 포함된 경우 사용자 경험을 개선하는 데 도움이 된다. 이러한 작업은 원래 정의된 범위를 벗어날 수 있으며, 이는 구조화되지 않은 동시성을 생성한다는 개념을 강조한다.

`Task {}`를 더 잘 이해하기 위해 마지막 코드 조각이 포함된 [작은 프로젝트](https://www.andyibanez.com/archives/UnstructuredConcurrencyIntro.zip)를 살펴볼 수 있다. 이 프로그램에는 다운로드 중일 때 '취소' 버튼으로 전환되는 '다운로드' 버튼이 있다. 버튼을 빠르게 탭하면 이미지 다운로드 기회를 주지 않고 작업을 명시적으로 취소했음을 알리는 "취소됨" 알림이 표시된다.

<img width="640" height="1136" alt="Image" src="https://github.com/user-attachments/assets/0fe66068-2ec3-43cd-b355-069af908b993" />

<img width="640" height="1136" alt="Image" src="https://github.com/user-attachments/assets/012c2606-79e2-4351-b124-98663f00345f" />

