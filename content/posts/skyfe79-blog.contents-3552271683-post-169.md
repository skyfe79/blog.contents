---
title: "Swift 모던 컨커런시 - 액터 이해하기"
date: 2025-10-25T10:06:22Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/

동시성 작업을 할 때 개발자가 가장 자주 마주치는 문제는 데이터 레이스(data race)다. 한 태스크가 값을 업데이트하는 동시에 다른 태스크가 해당 값을 읽거나, 두 태스크가 동시에 값을 써서 유효하지 않은 상태가 되는 경우가 대표적이다. 데이터 레이스는 쉽게 발생하지만 디버깅하기 어렵다. 데이터 레이스 문제를 해결하기 위한 전용 서적과 패턴도 존재한다.

데이터 레이스는 공유 가능한 가변 상태(mutable state)에서 발생한다. 변경되지 않는 `let` 변수만 사용하면 데이터 레이스 문제를 거의 마주치지 않는다. 하지만 아주 간단한 프로그램도 결국 어느 시점에서는 가변 상태를 가지기 때문에, 모든 것을 불변으로 만드는 것은 실용적이지 않다. 일반적으로 `let`을 최대한 사용하고 값 타입(`struct` 등)을 활용하면 데이터 레이스 문제를 크게 줄일 수 있다.

공유 가변 상태는 동기화가 필요하다. 가장 기본적이면서도 어려운 방법은 락(lock)과 같은 동기화 기법을 사용하는 것이다. 락은 한 번에 하나의 프로세스만 가변 상태를 수정하도록 보장한다. 지난 몇 년간 애플 플랫폼 개발자들은 직렬 디스패치 큐(serial dispatch queue)를 주로 사용해 왔다. 이는 동시성을 다루는 더 높은 수준의 개념이지만, 여전히 많은 코드를 직접 작성해야 한다.

다행히 Swift 5.5와 WWDC2021에서 소개된 새로운 동시성 API 덕분에, 이제 Swift는 가변 상태를 훨씬 쉽게 관리할 수 있다. 이 API는 한 번에 하나의 프로세스만 값 수정을 허용한다. 이 시리즈에서 살펴본 다른 새로운 동시성 API와 마찬가지로 사용하기 쉽지만, 더 많은 제어가 필요할 때는 제한적일 수 있다. 좋은 소식은 액터 API가 대부분의 개발자에게 충분할 만큼 강력하다는 점이다.


# 액터 모델 소개

액터는 변경 가능한 상태에 대한 동기화를 자동으로 제공하며, 자신의 상태를 프로그램의 다른 부분과 격리한다. 즉, 액터를 통하지 않고서는 공유 상태를 수정할 수 없다. 액터가 격리되어 있고 값을 수정하려면 액터와 통신해야 하므로, 액터는 자신의 상태에 대한 접근이 상호 배제적으로 이루어지도록 보장한다. 한 번에 오직 하나의 프로세스만 상태를 수정할 수 있다. 내부적으로 액터는 수동 동기화 작업을 대신 처리하며, 상태를 수정하려는 프로세스들을 '대기열에 넣어' 한 번에 하나씩만 처리되도록 조정한다.


## 구현 세부 사항

Swift에서 `actor`는 `actor` 타입으로 구현된다. `class`, `enum`, `struct`를 정의하는 것과 유사하게 `actor` 키워드를 사용해 선언한다. 액터는 참조 타입으로, `struct`보다는 `class`와 더 유사한 동작을 보인다. 이는 액터가 공유되는 *변경 가능한* 상태를 숨기고 다른 타입이 접근할 수 있도록 하는 특성 때문이다. `actor`와 `class`의 주요 차이점은 액터가 내부적으로 동기화 메커니즘을 구현한다는 점, 데이터가 프로그램의 나머지 부분과 격리된다는 점, 그리고 액터가 상속을 받거나 제공할 수 없지만 프로토콜을 준수하거나 확장할 수 있다는 점이다.

Swift 컴파일러에 깊게 통합된 덕분에, 액터는 동시성 관련 문제로 인해 발생할 수 있는 코드 오류로부터 개발자를 보호한다.

다음 예제를 살펴보자:

```swift
class Counter {
    var count = 0
    func increment() -> Int {
        count += 1
        return count
    }
}

class ViewController: UIViewController {
    
    var tasks = [Task<Void, Never>]()

    override func viewDidLoad() {
        super.viewDidLoad()
        
        let counter = Counter()
        
        tasks += [
            Task.detached {
                print(counter.increment())
            }
        ]

        tasks += [
            Task.detached {
                print(counter.increment())
            }
        ]
    }
}
```

*(애플은 [Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/) WWDC2021 세션에서 유사한 예제를 사용했다)*

*또한 원래는 이 예제들을 위한 플레이그라운드 샘플을 제공하려 했으나, Xcode 13 Beta 4에서는 작동하지 않아 이 글 마지막에 표준 iOS 프로젝트로 대체한다*

이 예제에서는 분리된 태스크 내에서 카운터 변수를 증가시키려 한다. 코드가 예상대로 동작하도록 보장하는 락킹 메커니즘 또는 동기화가 없다. 시스템은 두 번 모두 0으로 증가할 수 있으며, 출력되는 값은 실행마다 크게 달라질 수 있다.

`Counter`를 클래스 대신 `actor`로 변경하면 출력이 항상 "1, 2"가 되도록 보장할 수 있다.

```swift
actor Counter {
    var count = 0
    func increment() -> Int {
        count += 1
        return count
    }
}
```

하지만 이 변경만으로는 충분하지 않다. 컴파일하고 실행하려고 하면 출력하려는 두 곳 모두에서 다음 오류가 발생한다:

```swift
Expression is 'async' but is not marked with 'await'
```

이것은 동시성이 컴파일러 수준에서 얼마나 깊게 구현되어 버그가 있는 동시성 코드로부터 개발자를 보호하는지 잘 보여준다. 직접 안전한 동시성 코드를 작성하는 데 수시간, 수일, 수개월 또는 수년을 들이지 않아도 된다. 컴파일러 통합이 정말 마음에 드는데, 이는 이 시리즈에서 살펴본 모든 개념이 하나로 모인다는 것을 보여주기 때문이다. 컴파일러는 지금까지 배운 모든 것을 이해하는 데 도움을 준다.

이 오류를 수정하려면 `increment()`를 호출할 때 `await`를 추가하면 된다.

```swift
print(await counter.increment())
```

액터의 모든 공개 인터페이스는 자동으로 소비자에게 비동기적으로 제공된다. 이는 `await` 키워드를 사용해 액터와 안전하게 상호작용할 수 있게 해주며, 코드가 액터 내부로 들어가 작업을 수행할 수 있을 때까지 실행을 일시 중단한다.

*(여기서 잠시 멈춰 Swift의 새로운 동시성 시스템의 가장 기본적인 구성 요소인 `async/await`를 이해했는지 생각해보는 것이 좋다. 복습이 필요하다면 이 시리즈의 [Understanding async/await in Swift](https://www.andyibanez.com/posts/understanding-async-await-in-swift/) 글을 읽어보자.)*

프로퍼티(이 경우 `count`)에 직접 접근하려고 할 때 몇 가지 함의가 있다는 점에 유의하자. 먼저 읽기 전용 접근은 가능하지만 비동기 컨텍스트를 통해야 한다. 따라서 다음은 작동하지 않는다:

```swift
print(counter.count)
```

컴파일러는 다음과 같이 알려줄 것이다:

```swift
Actor-isolated property 'count' can only be referenced from inside the actor
```

이는 메서드와 마찬가지로 프로퍼티도 getter를 `async`로 노출하기 때문이다.

```swift
async {
    let count = await counter.count
    print("count is (count)")
}
```

마지막으로, 액터 자체를 통하지 않고는 액터의 공유 상태를 수정할 수 없다는 점을 기억하자. 이는 액터가 값을 수정할 메서드를 노출해야 함을 의미한다. 액터의 프로퍼티를 직접 수정할 수 없다.

```swift
counter.count = 3
```

```swift
Actor-isolated property 'count' can only be mutated from inside the actor
```


## 액터 내부 동작

액터는 외부 호출자에게 비동기 코드를 노출하며, 관련된 모든 것을 `async`로 표시한다. 하지만 액터 내부에서는 모든 호출이 동기적으로 처리된다. 이는 액터 내부에서 더 자연스러운 코드 작성이 가능하도록 도와주며, 이상한 실행 순서를 걱정할 필요가 없게 해준다.

다음 메서드를 `Counter`에 추가하면 직접 확인할 수 있다.

```swift
func reset() {
    while count > 0 {
        count -= 1
    }
    print("Done resetting")
}
```

그리고 새로운 함수 `foo`를 생성한 후 내부에 `reset`을 입력하면, 자동 완성 기능이 `reset()`으로 제안하는 것을 볼 수 있다.

<img width="826" height="454" alt="액터 내부에서 reset() 호출" src="https://github.com/user-attachments/assets/09e6c865-0c25-473b-a829-9d48cb36e2c5" />

반면 외부에서 `reset`을 호출하면 `reset()` 메서드 시그니처에 `async`가 표시된다. 

<img width="678" height="168" alt="액터 외부에서 reset() 호출" src="https://github.com/user-attachments/assets/53a620e2-1570-4056-8aa0-7354e0ec458d" />

액터 내부에서 호출되는 모든 것은 동기적임을 알 수 있다(`async` 키워드가 없기 때문). 하지만 동일한 메서드를 외부에서 호출하면 `async`가 된다. 액터의 동기 코드는 *항상* 중단 없이 완료까지 실행된다. 액터의 프로퍼티나 메서드에 await를 사용할 수 없지만, 액터가 다른 액터나 외부의 비동기 메서드를 호출하는 것은 제한되지 않는다.


# 액터 재진입성

액터는 자체 상태를 다른 액터와 격리하지만, 일반적으로 다른 액터나 코드베이스와 상호작용한다. 이로 인해 예기치 않은 동작이 발생할 수 있다. 다음 예시를 살펴보자:

```swift
enum ImageDownloadError: Error {
    case badImage
}

func downloadImage(url: URL) async throws -> UIImage {
    let imageRequest = URLRequest(url: url)
    let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
    guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
        throw ImageDownloadError.badImage
    }
    return image
}

actor ImageDownloader {
    private var cache: [URL: UIImage] = [:]
    
    func image(from url: URL) async throws -> UIImage {
        if let image = cache[url] {
            return image
        }
        
        let image = try await downloadImage(url: url)
        cache[url] = image
        return image
    }
    
    private func downloadImage(url: URL) async throws -> UIImage {
        let imageRequest = URLRequest(url: url)
        let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
        guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.badImage
        }
        return image
    }
}
```

*(이 코드는 Apple의 WWDC2021 세션 [Protect mutable state with Swift actors](https://developer.apple.com/videos/play/wwdc2021/10133/)에서 발표한 ImageDownloader 코드와 유사하지만, 실행 가능한 샘플로 만들었다.)*

이미지를 다운로드하고 캐시하는 ImageDownloader 액터가 있다. `if let`으로 이미지가 캐시되었는지 확인하고, 캐시된 이미지가 있으면 반환한다. 그렇지 않으면 이미지를 다운로드한 후 캐시에 저장하고 새로 다운로드한 이미지를 반환한다. 하지만 이 코드를 두 번 호출하면 어떤 일이 발생할까?

다음은 위의 `ImageDownloader` 액터를 사용하는 예시 코드다:

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    Task.detached {
        await self.downloadImages()
    }
}

//...

func downloadImages() async {
    let downloader = ImageDownloader()
    let imageURL = URL(string:  "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part3/3.png")!
    async let downloadedImage = downloader.image(from: imageURL)
    async let sameDownloadedImage = downloader.image(from: imageURL)
    var images = [UIImage?]()
    images += [try? await downloadedImage]
    images += [try? await sameDownloadedImage]
}
```

**중요 사항:** Xcode 13 Beta 4(및 Beta 3까지)에서는 동일한 `Task`에서 `async let`을 통해 액터에 두 번 진입할 때 교착 상태가 발생하는 버그가 있다. Apple은 이 문제를 인지하고 있으며 이후 베타 버전에서 수정될 예정이다. 이 버그가 수정될 때까지는 여러 `async let` 바인딩을 동시에 사용할 때 `Task.detached`를 사용하는 것이 해결책이다. 최종 출시 버전에서는 이 버그가 수정될 수 있으므로, 일반 `Task`와 `Task.detached` 호출의 용도가 다르다는 점을 유의해야 한다.

두 개의 `async let` 호출로 액터에 진입한다. 첫 번째 호출(`downloadedImage`)은 액터에 진입해 `downloadImages`의 `await` 호출을 만날 때까지 실행된다. 그리고 일시 중단되면 두 번째 호출인 `sameDownloadedImage`가 실행을 시작한다. `downloadedImage`가 `await`에 도달했지만 일시 중단된 상태이므로 아직 이미지를 다운로드하지 않았다. 따라서 캐시에 이미지가 없어 `sameDownloadedImage`도 메모리에서 가져오는 대신 이미지를 다시 다운로드한다. 운이 나쁘면 서버가 동일한 URL의 이미지를 업데이트했을 수 있으므로 `downloadedImage`와 `sameDownloadedImage`가 서로 다른 이미지를 다운로드할 수도 있다!

문제는 `await` 호출 이후의 프로그램 상태를 가정한다는 점이다. "이미지를 다운로드하고 캐시하면 이후 접근하는 모든 요청은 캐시된 버전을 가져올 것"이라고 가정하지만, 실제로는 동시에 액터에 접근하는 여러 호출이 있을 수 있으므로 이 코드로는 이를 보장할 수 없다. 결과적으로 동일한 이미지를 두 번 다운로드하는 버그가 발생한다.

이 문제를 해결하려면 액터가 각 다운로드 상태를 유지하고, 이미지 다운로드를 시도하기 전에 먼저 이 상태를 확인하는 방식을 사용할 수 있다:

```swift
actor ImageDownloader {
    private enum ImageStatus {
        case downloading(_ task: Task<UIImage, Error>)
        case downloaded(_ image: UIImage)
    }
    
    private var cache: [URL: ImageStatus] = [:]
    
    func image(from url: URL) async throws -> UIImage {
        if let imageStatus = cache[url] {
            switch imageStatus {
            case .downloading(let task):
                return try await task.value
            case .downloaded(let image):
                return image
            }
        }
        
        let task = Task {
            try await downloadImage(url: url)
        }
        
        cache[url] = .downloading(task)
        
        do {
            let image = try await task.value
            cache[url] = .downloaded(image)
            return image
        } catch {
            // 오류 발생 시 URL을 캐시에서 제거하고
            // 원래 오류를 다시 던진다.
            cache.removeValue(forKey: url)
            throw error
        }
    }
    
    private func downloadImage(url: URL) async throws -> UIImage {
        let imageRequest = URLRequest(url: url)
        let (data, imageResponse) = try await URLSession.shared.data(for: imageRequest)
        guard let image = UIImage(data: data), (imageResponse as? HTTPURLResponse)?.statusCode == 200 else {
            throw ImageDownloadError.badImage
        }
        return image
    }
}
```

*이 코드는 Apple의 WWDC2021 세션 [Protect mutable state with Swift actors](https://developer.apple.com/wwdc21/10133)에서 제공한 코드와 유사하다.*

이 코드는 복잡해 보이지만 매우 직관적이다(이 직관성이 새로운 동시성 API의 강점이다!). 먼저 현재 URL의 상태를 저장할 enum을 선언한다. URL이 처음 다운로드되면 `.downloading` 상태로 캐시에 추가한다. 동시에 동일한 URL로 액터에 호출이 발생하면 이미지가 캐시에 있음을 확인하고 다시 다운로드하는 대신 `await`로 기다린다. 이후의 호출은 이미 다운로드된 이미지를 보게 되므로 즉시 반환된다. 이미지가 처음이자 마지막으로 다운로드되면 `.downloaded` 상태로 캐시된다.

액터 재진입성은 교착 상태를 방지하고 진행을 보장하지만, 동시성과 직접적 관련이 없는 버그(예: 동일한 이미지를 여러 번 다운로드)를 방지하기 위해 가정을 확인해야 한다. 액터 재진입 개념을 잘 활용하기 위한 몇 가지 포인트는 다음과 같다:

* 동기 코드에서 상태를 변경한다. 캐시를 동일한 태스크에서 변경하고 다른 곳에서는 업데이트를 시도하지 않는다.
* `await` 이후에 상태가 언제든지 변경될 수 있음을 인지한다. 필요한 경우 상태를 수동으로 확인해 변경 사항에 대응할 수 있다.


# 액터 격리

액터의 핵심은 격리에 있다. 액터의 주된 목적은 자신의 상태를 외부로부터 분리하는 것이다. 이렇게 함으로써 액터는 자신의 프로퍼티에 대한 접근을 제어할 수 있으며, 여러 쓰기 작업이 동시에 발생해 프로그램이 예상치 못한 상태가 되는 것을 방지한다.

불변 프로퍼티는 언제든지 접근할 수 있다.

```swift
actor DollMaker {
    let id: Int
    var dolls: [Doll] = []
    
    init(id: Int) {
        self.id = id
    }
}

extension DollMaker: Equatable {
    static func ==(_ lhs: DollMaker, rhs: DollMaker) -> Bool {
        lhs.id == rhs.id
    }
}
```

위 코드에서 `==` 연산자는 두 타입을 비교하는 `static` 메서드다. `static`은 이 메서드가 액터 "외부"에 있음을 의미한다(즉, `self` 인스턴스가 없다). 메서드 내부에서 오직 불변 상태만 접근한다는 사실과 결합하면, 컴파일러는 이 작업이 안전함을 알 수 있다.

```swift
extension DollMaker: Hashable {
    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}
```

반면 이 코드는 문제가 될 수 있다. `id` 필드만 참조하지만, 이 메서드는 인스턴스 메서드다. 격리되려면 `async`로 선언해야 한다. 다행히 이 경우에는 메서드를 명시적으로 `nonisolated`로 표시해 컴파일러에게 이 메서드가 격리되지 않음을 알릴 수 있다. 컴파일러는 이 메서드를 액터 "외부"에 있는 것으로 간주하고, 내부에서 오직 불변 프로퍼티만 접근한다면 문제없이 처리한다. 만약 hasher가 `id` 대신 `dolls` 프로퍼티를 사용했다면, `dolls`는 가변이므로 이 방식은 작동하지 않을 것이다.


# Sendable 타입의 이해

컨커런시 모델에서는 `Sendable` 타입이라는 개념을 도입한다. `Sendable` 타입은 동시에 공유해도 안전한 타입을 의미한다. 다음은 `Sendable` 타입의 대표적인 예시다:

* 값 타입(예: 구조체)
* 액터 타입

클래스도 `Sendable`이 될 수 있지만, 불변(immutable) 상태이거나 내부적으로 동기화 메커니즘을 제공하는 경우에 한한다. `Sendable` 클래스는 특별한 경우에 해당한다.

컨커런트 코드에서는 `Sendable` 타입을 사용해 통신하는 것이 권장된다. 향후 Swift는 컴파일 타임에 함수 간 비-`Sendable` 타입 공유 여부를 검사할 수 있을 것으로 기대되지만, Xcode 13 Beta 4 기준으로는 아직 구현되지 않았다.


## Sendable 프로토콜

여러분도 예상했겠지만, 타입을 `Sendable`로 만들려면 `Sendable` 프로토콜을 준수하면 된다. 단순히 프로토콜을 준수한다고 선언하기만 해도 Swift 컴파일러가 많은 작업을 대신 처리해준다.

다음 예제를 살펴보자:

```swift
struct Videogame: Sendable {
    var title: String
}

struct VideogameMaker: Sendable {
    var name: String
    var games: [Videogame]
}
```

이 코드는 문제 없이 컴파일된다. `VideogameMaker`와 `Videogame` 모두 `Sendable`을 준수하기 때문이다.

구조체의 경우 `Sendable`을 명시적으로 준수하지 않아도 작동한다:

```swift
struct Videogame {
    var title: String
}

struct VideogameMaker: Sendable {
    var name: String
    var games: [Videogame]
}
```

하지만 클래스는 상황이 다르다.

```swift
class Videogame {
    var title: String
    
    init(title: String) {
        self.title = title
    }
}

struct VideogameMaker: Sendable {
    var name: String
    var games: [Videogame]
}
```

이 경우 다음과 같은 에러가 발생한다:

```swift
'Sendable'을 준수하는 구조체 'VideogameMaker'의 저장 프로퍼티 'games'가 비-전송 가능 타입 '[Videogame]'을 가지고 있습니다
```


## 전송 가능 타입과 제네릭

제네릭 타입은 모든 프로퍼티가 `Sendable`일 때만 `Sendable`로 선언할 수 있다.

```swift
struct Pair<T, U> {
    var first: T
    var second: T
}

extension Pair: Sendable where T: Sendable, U: Sendable {}
```


## 전송 가능 함수

액터 간 전달이 가능한 함수는 `@Sendable`로 표시할 수 있다.

클로저의 경우 `@Sendable`로 표시하면 몇 가지 제한이 생긴다. 주변 스코프의 변경 가능한 변수를 캡처할 수 없으며, 캡처하는 모든 값은 `Sendable`을 준수해야 한다. 또한 클로저가 비동기적이면서 액터 분리(isolated) 상태일 수 없다.


# 마무리

이미지 다운로드 샘플 프로젝트는 [여기](https://www.andyibanez.com/archives/Actors.zip)에서 다운로드할 수 있다.

이번 글에서는 액터의 개념과 사용 방법을 살펴봤다. 액터는 자신의 상태를 격리하며, 모든 프로퍼티 쓰기 접근은 반드시 액터 자체를 통해 이루어져야 한다는 점을 배웠다. 상태를 독립적으로 관리함으로써 액터는 동시성 안전성을 보장한다.

또한 `Sendable` 타입의 중요성과 Swift의 새로운 동시성 시스템에서의 역할도 함께 알아봤다. Sendable 타입은 동시성 코드 작성을 위한 컴파일 타임 검사를 제공한다. 정적 검사 기능 덕분에 동시성 모델을 위반하거나 버그를 유발하는 코드를 작성하기 어렵다.






