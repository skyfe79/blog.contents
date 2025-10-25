---
title: "Swift 모던 컨커런시 - 클로저 기반 코드를 async/await로 변환하기"
date: 2025-09-07T07:00:15Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/

*이 글은 [Swift 모던 컨커런시](https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/) 시리즈의 일부이다.*

*이 글은 원래 Xcode 13 베타 1 버전으로 작성되었으며, Xcode 13 베타 3 버전에 맞춰 내용, 코드 예제, 샘플 프로젝트가 업데이트되었다.*

---

이 글을 더 잘 이해하려면 async/await 개념에 익숙해야 한다. 아직 잘 모른다면 이 시리즈의 첫 번째 글을 먼저 읽어보자: [Swift에서 async/await 이해하기](https://www.andyibanez.com/posts/understanding-async-await-in-swift/).

원래 이 글을 독립적인 글로 작성할지, 아니면 'Swift의 async/await 소개' 글에 추가할지 고민했다. 결국 정보 과부하를 막고 개념을 더 쉽게 이해할 수 있도록 이전 글을 짧게 정리하기로 결정했다.

지난 주에는 async/await에 대해 길게 논의했다. 콜백 방식과 비교하며 설명했고, async/await가 얼마나 유용한지 여러 예제를 통해 보여줬다.

이제 실제 동시성 프로그래밍까지 한 걸음 남았다. 다음 주에 다룰 *구조화된 동시성*으로 넘어가기 전에, 클로저 기반과 델리게이트 기반 코드를 async/await로 변환하는 방법을 소개하려 한다. 이 글은 프로젝트에 async/await를 점진적으로 도입할 수 있도록 필요한 모든 도구를 제공하는 것이 목표다.

라이브러리 개발자라면, 모든 클로저 기반 API에 대해 async/await 버전을 제공할 수 있다. 이렇게 하면 자신의 코드에서 사용할 뿐만 아니라 사용자에게도 async/await 기능을 제공할 수 있다.

라이브러리 개발자가 아니더라도, 실제 앱을 운영 중이라면 콜백으로 알림을 주는 비동기 코드를 사용하고 있을 것이다. 이런 프로젝트를 마이그레이션하려면 비동기 메서드의 `async` 버전을 구현하면 된다. 서드파티 라이브러리가 아직 async/await 버전을 제공하지 않는다면, 직접 구현해서 사용할 수도 있다.


## 연속성(Continuation) 이해하기

이 글의 [첫 번째 파트](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)를 읽었다면 연속성이 무엇인지 기억할 수 있지만, 계속하기 전에 간단히 복습해 보자.

연속성이란 기본적으로 비동기 호출 이후에 발생하는 작업을 의미한다. async/await를 사용할 때 연속성은 쉽게 이해할 수 있다. `await` 호출 아래에 있는 모든 코드가 연속성이다.

다음 예제를 살펴보자:

```swift
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

이 예제에서 `await` 키워드는 다른 스레드에서 데이터 다운로드 작업을 트리거할 수 있다. `await` 아래에 있는 모든 코드(즉, `guard`로 시작하는 줄부터)가 *연속성*이다.

연속성은 async/await API에만 국한되지 않는다. 클로저 기반 비동기 API를 사용할 때는 완료 핸들러 내부에서 호출되는 모든 코드가 연속성이다.

```swift
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
```

이 코드는 위 예제의 클로저 버전이다. 다시 한번 연속성은 `guard`에서 시작한다. 주요 차이점은 완료 핸들러 버전은 코드 흐름을 따라가기가 더 어렵다는 것이다.


# 명시적 연속 작업(Continuation) 소개

Swift는 콜백 기반 코드를 async/await로 변환하는 데 사용할 수 있는 메서드를 제공한다. `withCheckedContinuation`과 `withCheckedThrowingContinuation`이 바로 그것이다. 두 메서드의 차이점은 후자가 에러를 던질 수 있는 코드에 사용된다는 점이다. 이 메서드를 *명시적 연속 작업*이라고 부른다.

앞서 선언한 `downloadMetadata(for:)` 메서드의 완료 핸들러 버전이 있다고 가정해 보자:

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

// MARK: - 함수

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
```

이 코드의 원작자가 아니고 소스 코드를 직접 수정할 수 없는 상황이라고 가정하자. 이 메서드를 async/await로 마이그레이션하려면 `downloadImageAndMetadata(for:imageNumber:completionHandler)` 호출을 `withCheckedThrowingContinuation` 메서드로 감싸는 것이 가장 간단한 방법이다.

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

이 함수의 마법은 `withCheckedThrowingContinuation` 내부에서 일어난다. 이 함수는 `CheckedContinuation<T, E> where E: Error` 객체를 제공하며, 우리가 호출해야 할 메서드를 갖고 있다. 이 예제에서 원본 `downloadImageWithMetadata`는 `DetailedImage`나 에러를 전달하므로, 받은 결과에 따라 적절한 `resume` 메서드를 호출해야 한다. 만약 이 메서드가 `Result<DetailedImage, Error>`를 반환했다면 `.resume(with:)`를 호출해 `result`를 바로 전달할 수 있었을 것이다.

연속 작업은 **반드시 한 번만 호출**해야 한다. 따라서 `withCheckedThrowingContinuation` 내 모든 분기에서 연속 작업을 호출해야 한다. `.resume` 호출을 잊으면 문제가 발생할 수 있다. 다행히 Swift가 이를 알려준다.

**참고**: *적어도 그렇게 동작해야 한다. 이 글은 [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/?time=1733) 세션의 마지막 부분을 바탕으로 작성했다. 최소한 Beta 1 버전에서는 `resume`을 호출하지 않는 분기가 있는 코드도 동작했다.*

이렇게 클로저 기반 코드를 더 깔끔한 형태로 변환했다! 이 함수의 `async/await` 버전을 사용하는 것은 매우 간단하다:

```swift
Task {
    if let imageDetail = try? await downloadImageAndMetadata(imageNumber: 1) {
        self.imageView.image = imageDetail.image
        self.metadata.text = "(imageDetail.metadata.name) ((imageDetail.metadata.firstAppearance) - (imageDetail.metadata.year))"
    }
}
```

이를 실제로 실행해 볼 수 있는 샘플 프로젝트는 [여기](https://www.andyibanez.com/archives/CheckedContinuations.zip)에서 다운로드할 수 있다.


## 델리게이트 기반 코드를 async/await로 변환하기

지금까지 콜백 기반 코드를 async/await로 변환하는 방법을 살펴봤다. 이번에는 델리게이트 기반 코드도 같은 방식으로 변환할 수 있다는 점을 알아본다. 델리게이트 기반 API는 대부분 콜백으로 대체되었지만, 특히 이벤트 기반 API(블루투스, 위치 서비스 등)에서는 여전히 흔히 마주칠 수 있다. 따라서 이런 경우에도 async/await를 적용할 수 있다는 점을 알아두면 유용하다.

사용자가 ViewController에서 연락처를 선택할 수 있는 UIKit 앱을 예로 들어보자. 가장 간단한 형태는 다음과 같다:

```swift
class ViewController: UIViewController, CNContactPickerDelegate {

    @IBOutlet weak var contactNameLabel: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()
        // 뷰 로드 후 추가 설정
    }

    @IBAction func chooseContactTouchUpInside(_ sender: Any) {
        showContactPicker()
    }

    func showContactPicker() {
        let picker = CNContactPickerViewController()
        picker.delegate = self
        present(picker, animated: true)
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        self.contactNameLabel.text = contact.givenName
        picker.dismiss(animated: true, completion: nil)
    }

}
```

"연락처 선택" 버튼을 누르면 `showContactPicker`가 호출되어 실제 피커가 표시된다. 사용자가 연락처를 선택하면 시스템이 `contactPicker(_:contact)` 메서드를 통해 이벤트를 알려준다.

하지만 더 나은 방법이 있다. 모든 연락처 관련 기능을 감싸는 객체를 만들 수 있다. 그런 다음 사용자가 연락처를 선택했을 때 알려주는 `async` 메서드를 생성한다. 이렇게 하면 프로그램의 선형성을 유지하고 더 쉽게 따라갈 수 있는 흐름을 만들 수 있다.

`ContactPicker`는 다음과 같이 선언한다:

```swift
@MainActor
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

여기서 이해해야 할 핵심 사항은:

1. `CheckedContinuation<CNContact, Never>`를 타입 앨리어스로 정의해 쉽게 참조할 수 있게 한다. 오류가 발생하지 않으므로 에러 파라미터는 `Never`다.
2. `private var contactContinuation: ContactCheckedContinuation?`는 컨티뉴에이션에 대한 참조를 보관한다. 이 컨티뉴에이션은 `withCheckedContinuation` 핸들러에서 제공된다. 한 번만 호출되도록 하기 위해 옵셔널로 선언하고 첫 호출 후 nil로 설정한다.
3. `pickContact`는 `async` 메서드로 `CNContact`를 반환한다. 여기서 `withCheckedContinuation`을 호출한다.
4. 연락처가 선택되면 컨티뉴에이션의 `resume`을 호출한다.

이를 사용하는 방법은 다음과 같다:

```swift
@IBAction func chooseContactTouchUpInside(_ sender: Any) {
    async {
        let contactPicker = ContactPicker(viewController: self)
        let contact = await contactPicker.pickContact()
        self.contactNameLabel.text = contact.givenName
    }
}
```

하지만 이 구현에는 결함이 있다. `ContactsUI` 프레임워크를 사용해본 적이 있다면 알아차렸을 것이다.

표시되는 UI는 사용자에게 연락처를 선택하지 않고 취소할 수 있는 옵션을 제공한다. 앞서 컨티뉴에이션을 다룰 때 정확히 한 번만 호출해야 한다고 언급했다. 위 프로그램에서는 `contactPickerDidCancel(_)` 메서드를 구현하지 않아 사용자가 취소할 때 컨티뉴에이션이 호출되지 않는다.

이 문제를 해결하는 두 가지 방법이 있다: 사용자가 취소할 때 오류를 던지거나 nil 연락처를 전달할 수 있다. 이 경우 오류를 던지는 것은 적절하지 않으므로 nil 연락처를 전달하도록 코드를 수정한다.

```swift
class ContactPicker: NSObject, CNContactPickerDelegate {
    private typealias ContactCheckedContinuation = CheckedContinuation<CNContact?, Never>

    private unowned var viewController: UIViewController
    private var contactContinuation: ContactCheckedContinuation?
    private var picker: CNContactPickerViewController

    init(viewController: UIViewController) {
        self.viewController = viewController
        picker = CNContactPickerViewController()
        super.init()
        picker.delegate = self
    }

    func pickContact() async -> CNContact? {
        return await withCheckedContinuation({ (continuation: ContactCheckedContinuation) in
            self.contactContinuation = continuation
            viewController.present(picker, animated: true)
        })
    }

    func contactPicker(_ picker: CNContactPickerViewController, didSelect contact: CNContact) {
        contactContinuation?.resume(returning: contact)
        contactContinuation = nil
        picker.dismiss(animated: true, completion: nil)
    }

    func contactPickerDidCancel(_ picker: CNContactPickerViewController) {
        contactContinuation?.resume(returning: nil)
        contactContinuation = nil
    }
}

//...

// ViewController 내부

@IBAction func chooseContactTouchUpInside(_ sender: Any) {
    async {
        let contactPicker = ContactPicker(viewController: self)
        let contact = await contactPicker.pickContact()
        self.contactNameLabel.text = contact?.givenName
    }
}
```

이제 훨씬 더 좋아졌다. 모든 가능한 경로에서 `resume`을 호출하고 프로그램은 항상 유효한 상태를 유지한다. 코드는 더 길어졌지만 장기적으로 볼 때 선형성을 유지하기 위한 추가 작업이 프로그램 구조에 도움이 될 것이다.

연락처 피커 앱의 전체 버전은 [여기](https://www.andyibanez.com/archives/AsyncAwaitContactPicker.zip)에서 다운로드할 수 있다. 단순한 버튼과 레이블로 구성된 UIKit 앱으로, 선택한 연락처의 이름을 보여준다. 이 글이 더 잘 이해되는 데 도움이 되길 바란다.


## 요약

이 글에서는 콜백 기반 코드나 델리게이트 기반 코드를 `async/await`로 전환하는 방법을 살펴봤다. 체크드 컨티뉴에이션을 활용하는 방법을 배웠고, 컨티뉴에이션이 실제로 무엇인지에 대한 개념도 확립했다.

이제 여러분은 `async/await`의 핵심 개념을 모두 이해했다. 실제 동시성 작업을 다룰 준비가 되었으며, 다음 주에는 *구조적 동시성*을 시작으로 본격적인 동시성 주제를 다룰 예정이다. 여러 작업을 병렬로 실행하는 방법과 그 결과를 처리하는 방식을 배우게 될 것이다.






