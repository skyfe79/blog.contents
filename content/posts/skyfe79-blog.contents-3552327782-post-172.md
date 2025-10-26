---
title: "Swift 모던 컨커런시 - AsyncSequence 사용하기"
date: 2025-10-25T10:54:31Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/using-asyncsequence-in-swift/

WWDC2021에서 소개된 새로운 동시성 API와 함께 등장한 `AsyncSequence`는 데이터를 비동기적으로 처리할 수 있는 컬렉션 프로토콜이다. 이 프로토콜을 사용하면 루프 내에서 데이터를 수신할 수 있을 뿐만 아니라 `filter`, `map`, `reduce` 같은 고차 함수도 비동기적으로 적용할 수 있다. 새로운 데이터가 도착할 때마다 `await`를 통해 기다릴 수 있는 특징을 갖는다.


## AsyncSequence 소개

시퀀스(sequence)로써 우리는 다른 시퀀스와 동일한 작업들을 수행할 수 있다. 고차 함수를 적용하는 것 외에도, 시퀀스 내부를 검색하거나 요소의 개수를 세는 등 다양한 작업이 가능하다.

중요한 점은 이러한 시퀀스의 기본 동작 방식을 이해하는 것이다.

`await`는 일시 중단을 의미한다는 점을 기억하자. 코드 실행 중 `await` 호출을 만나면, 해당 작업이 다른 곳에서 처리되는 동안 현재 코드의 실행이 멈춘다. 비동기 작업이 완료되면 컴파일러는 `await` 호출 이후의 코드를 실행하기 시작한다.

`AsyncSequence`도 기본적으로 동일한 동작 방식을 가지지만, 한 가지 핵심적인 차이점이 존재한다.

원격 서버에 다음과 같은 파일이 있다고 가정해보자:

```
// videogames.csv
The Legend of Zelda: Ocarina of Time|1998|10
The Legend of Zelda: Majora's Mask|2000|10
The Legend of Zelda: The Wind Waker|2003|10
Tales of Vesperia|2008|8
Tales of Graces|2011|9
Tales of the Abyss|2006|10
Tales of Xillia|2013|10
```

편의를 위해 이 파일을 [여기](https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part11/videogames.csv)에서 찾을 수 있다.


## AsyncSequence 활용하기

파일을 한 줄씩 읽어 처리하는 방법은 매우 간단하다:

```swift
struct Videogame {
    let title: String
    let year: Int?
    let score: Int?
    
    init(rawLine: String) {
        let splat = rawLine.split(separator: "|")
        self.title = String(splat[0])
        self.year = Int(splat[1])
        self.score = Int(splat[2])
    }
}

//...

func loadVideogames() async {
    let url = URL(string: "https://www.andyibanez.com/fairesepages.github.io/tutorials/async-await/part11/videogames.csv")!
    
    var videogames: [Videogame] = []
    
    do {
        for try await rawVg in url.lines {
            if rawVg.contains("|") {
                // 유효한 게임 데이터
                videogames += [Videogame(rawLine: rawVg)]
            }
        }
    } catch {
        // 에러 처리
    }
}
```

`lines`는 `AsyncSequence`다. URL에서 파일의 새로운 줄을 가져올 때마다 한 줄씩 처리된다. 이는 배열이나 특정 컬렉션 타입이 아니다. 시간이 지남에 따라 값을 전달하는 추상적인 개념이다. 우리만의 `AsyncSequence`를 만들 수도 있다.

하지만 `AsyncSequence`가 더 흥미로워지는 건 코드를 더 합리적으로 리팩토링할 수 있을 때다.

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

여기서 주목할 점은 `AsyncSequence`를 이런 방식으로 사용할 때(여러 호출을 연결해 컬렉션을 변환하는 경우), 시퀀스가 자동으로 시작되지 않는다는 것이다. `for` 루프를 추가하지 않으면 시퀀스가 시작되지 않아 아무것도 보이지 않는다. 이는 `.count`를 호출해 요소 수를 가져올 수 없다는 제약을 의미한다. 또한 `dropLast()` 같은 일부 메서드가 누락된 것도 확인할 수 있다.

시퀀스는 새 값을 생성해야 할 때 `await`된다. 이 예제에서는 각각의 새로운 게임 줄이 `await`를 트리거한다. 새 값이 방출될 때마다 코드가 일시 중단되고, 스레드는 새 값을 생성하거나 완료되거나 오류가 발생할 때까지 다른 작업을 수행한다.

일반적인 반복이므로 루프 내에서 `break`와 `continue`를 사용할 수 있다.

```swift
for try await videogame in videogames {
    if videogame.score == 10 {
        continue
    }
    print("(videogame.title) ((videogame.year ?? 0))")
}
```

이 예제에서는 완벽한 점수를 받은 게임을 출력하지 않도록 `continue` 문을 추가했다. 물론 `videogames`에 이 제약 조건을 추가하는 필터를 적용하는 방법도 있다.

```swift
let videogames =
    url
    .lines
    .filter { $0.contains("|") }
    .map { Videogame(rawLine: $0) }
    .filter { $0.score != 10 } // 여기에 필터 적용

do {
    for try await videogame in videogames {
        print("(videogame.title) ((videogame.year ?? 0))")
    }
} catch {
    
}
```

또 다른 흥미로운 점은 이 특정 사례에서 네트워크를 통해 데이터를 전달하는 `AsyncSequence`를 사용한다는 것이다. 로컬 파일에서도 사용할 수 있다.

Apple은 SDK 전반에 걸쳐 `AsyncSequence`를 사용하는 여러 API를 추가했다:

* `FileHandle.standardInput.bytes.lines`: 커맨드라인이나 다른 소스에서 입력을 받는 데 사용할 수 있다.
* URL은 `lines`와 `bytes` 모두 접근할 수 있다. 줄 단위가 아닌 원시 데이터를 읽으려면 `resourceBytes` 프로퍼티를 호출한다.
* `URLSession`에는 `bytes(from:)` 메서드가 있어 네트워크에서 바이트 단위로 데이터를 다운로드할 수 있다.
* `NotificationCenter`에는 이제 지정된 타입의 새 알림을 `await`할 수 있는 API가 있다. 이에 대한 글을 쓸 계획이다.


## AsyncStream 활용하기

기존에 콜백이나 델리게이트를 통해 특정 이벤트의 업데이트를 지속적으로 전달하는 코드가 있을 수 있다. 예를 들어 `CoreLocation`을 사용해 사용자의 위치를 실시간으로 받아오는 경우, 새로운 위치 정보가 생길 때마다 처리하는 코드가 필요하다.

`AsyncStream`을 사용하면 한 번에 여러 곳에서 결과를 전달하는 이런 코드를 간소화할 수 있다. [Swift에서 클로저 기반 코드를 async/await로 변환하기](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/)와 유사하게, '실시간' 또는 '스트리밍' 코드를 합리적인 비동기 시퀀스로 변환할 수 있다.

이를 보여주기 위해 먼저 CoreLocation 델리게이트 메서드를 감싸는 작은 래퍼를 만들 것이다. 이는 권한 상태에 대한 연속(continuation)을 생성하고, 위치 이벤트에 대한 스트림을 설정하는 좋은 예시가 될 것이다.

```swift
@MainActor
class LocationUpdater: NSObject, CLLocationManagerDelegate {
    private(set) var authorizationStatus: CLAuthorizationStatus
    
    private let locationManager: CLLocationManager
    
    // 사용자 위치 추적 권한을 비동기적으로 요청하기 위한 연속
    private var permissionContinuation: CheckedContinuation<CLAuthorizationStatus, Never>?
    
    var locationHandler: ([CLLocation]) -> Void = { _ in }
    
    override init() {
        locationManager = CLLocationManager()
        authorizationStatus = locationManager.authorizationStatus
        
        super.init()
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
    }
    
    func start() {
        locationManager.startUpdatingLocation()
    }
    
    func stop() {
        locationManager.stopUpdatingLocation()
    }
    
    func requestPermission() async -> CLAuthorizationStatus {
        locationManager.requestWhenInUseAuthorization()
        return await withCheckedContinuation { continuation in
            permissionContinuation = continuation
        }
    }
    
    // MARK: - Location Delegate
    
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        locationHandler(locations)
    }

    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
        permissionContinuation?.resume(returning: authorizationStatus)
    }
}
```

이 `LocationUpdater` 클래스는 `permissionContinuation` 연속을 통해 사용자에게 권한을 요청하는 기능을 `async`/await로 제공한다. 개발자는 다음과 같이 이 코드를 호출할 수 있다:

```swift
let authorizationsStatus = await updater.requestPermission()
```

이 코드는 내부적으로 두 개의 다른 메서드를 거쳐 결과를 얻더라도 한 줄로 상태를 반환한다. 연속이 어떻게 동작하는지 기억나지 않거나 모르는 경우, [Swift에서 클로저 기반 코드를 async/await로 변환하기](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/) 글을 참고하면 된다.

`var locationHandler: ([CLLocation]) -> Void = { _ in }` 프로퍼티는 델리게이트를 추가로 구현하지 않고도 위치 이벤트를 받을 수 있게 해주는 클로저다. 이를 `AsyncStream`으로 감싸서 위치 이벤트를 실시간으로 받을 수 있으며, 루프에서 받고 나중에 시퀀스 함수를 사용해 이 배열을 변형할 수도 있다:

```swift
func beginTracking() async {
    await requestPermission()
    if authorizationsStatus == .authorizedWhenInUse {
        for await location in locationEvents() {
            print(location.speed)
        }
    }
}

func locationEvents() -> AsyncStream<CLLocation> {
    let locations = AsyncStream(CLLocation.self) { continuation in
        updater.locationHandler = { locations in
            locations.forEach {
                continuation.yield($0)
            }
        }
        updater.start()
    }
    return locations
}
```

`locationEvents`가 바로 우리의 `AsyncSequence`다.

여기서 중요한 점은 연속이 중지될 때를 알기 위해 리스닝할 수 있다는 것이다. 수동으로 중지해야 하는 시퀀스가 있거나 이벤트 수신 후 정리가 필요한 경우 유용하게 구현할 수 있다. 그 메서드는 다음과 같다:

```swift
continuation.onTermination = { _ in}
```

안타깝게도 이 메서드를 구현하려면 스트리밍 타입(이 경우 `CLLocation`)이 `@Sendable`이어야 한다. CLLocation은 Sendable이 아니므로 여기서 사용할 수 없다. `@Sendable`에 대해 더 알고 싶다면 [Swift의 새로운 동시성 모델에서 액터 이해하기](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/) 글의 "**The Sendable Type**" 섹션을 참고하라. `location` 프로퍼티 하나만 있는 래퍼 타입을 만들어 해결하려 했지만 작동하지 않았다. 현재로서는 `CLLocation`과 동일한 프로퍼티를 가진 구조체를 만드는 것 외에 `AsyncStream`과 CoreLocation을 함께 사용할 최선의 방법을 알 수 없다.


# 결론

`AsyncSequence`는 실시간으로 발생하는 이벤트를 기다릴 수 있게 해준다. 네트워크 이벤트든 시스템 이벤트든 상관없이, `AsyncSequence`는 코드를 더 읽기 쉽고 작성하기 쉽게 단순화하는 데 도움이 된다. `AsyncStream`을 사용하면 지속적인 이벤트 발생기를 `AsyncSequence`로 감쌀 수 있으며, 루프 안에서 이벤트를 받을 수 있다. 그리고 필터링, 매핑, 리듀스 등 표준 컬렉션 연산을 수행할 수 있다.




