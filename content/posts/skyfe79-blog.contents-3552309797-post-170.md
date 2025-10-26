---
title: "Swift 모던 컨커런시 - @MainActor와 글로벌 액터"
date: 2025-10-25T10:32:32Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/mainactor-and-global-actors-in-swift/

최근에 [액터](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)의 개념과 사용 방법에 대해 알아봤다. 액터는 자신의 프로퍼티에 대한 접근을 제어하여 여러 프로세스가 동시에 데이터를 수정하지 못하도록 막는다. 이렇게 하면 데이터 손상을 방지할 수 있다.


# 메인 스레드의 중요성

애플 플랫폼에서 개발 경험이 많든 적든, 메인 스레드에 대해 들어본 적이 있을 것이다. 메인 스레드는 UI 코드를 실행하는 핵심적인 역할을 담당한다. 애플 플랫폼에서는 메인 스레드 외부에서 UI를 업데이트할 수 없다. 일반적으로 비동기로 실행되는 프로세스들은 작업 중인 스레드에서 값을 반환하지만, 결과는 반드시 메인 스레드로 전달해야 한다.

모던 컨커런시 시스템 이전에는 `DispatchQueue.main.async`를 호출해 완료 블록을 전달하는 방식으로 처리했다. 이 블록은 메인 스레드에서 실행되므로 UI 업데이트가 안전하게 이루어졌다. 하지만 모든 작업을 메인 스레드에서 처리하면 안 된다. 메인 스레드가 과도하게 바쁘면 사용자에게 성능 저하가 눈에 띄게 나타나고, 앱이 응답하지 않는 상태가 지속되면 시스템이 일정 시간 후 강제로 종료시킨다.

새로운 컨커런시 시스템은 다양한 스레드 간에 작업을 전환하고, 일시 중단했다가 다른 스레드에서 재개하는 등 동적인 동작을 한다. 따라서 메인 스레드를 업데이트하기 위한 새로운 메커니즘이 필요해졌다. 이 역할을 하는 것이 바로 특별한 종류의 액터인 `@MainActor`다.


## 메인 액터 소개

`@MainActor`로 표기되는 주요 액터는 메인 스레드를 나타낸다. 이 액터는 모든 동기화 작업을 메인 디스패치 큐에서 수행한다. 이 액터는 SwiftUI, AppKit, UIKit, watchKit 등 애플 프레임워크 전반에서 발견할 수 있기 때문에 '특별'하다. 메인 스레드에서 실행해야 하는 장소는 엄청나게 많으며, 각 프레임워크 내에서 메인 스레드 동기화가 필요한 개별 UI 클래스까지 고려하면 그 수는 더욱 늘어난다. 모든 뷰나 뷰 컨트롤러는 메인 스레드에서 동작해야 하므로, 어디서든 `@MainActor`에 접근할 필요성이 크게 증가한다.

`@MainActor`를 사용하려면 메서드나 클래스 정의에 해당 속성을 추가해야 한다. 함수에 `@MainActor`를 추가하면 해당 함수는 항상 메인 스레드에서 실행된다.

```swift
@MainActor func fetchGames() {

}
```

위 예제에서 `fetchGames`는 항상 메인 액터에서 실행된다. 이 방식은 직관적이며, 이후에 코드를 보는 개발자들이 이 코드가 메인 스레드에서 실행되어야 함을 명확히 알 수 있어 추측 작업을 줄이고 더 명시적인 코드를 작성하는 데 도움이 된다.

메인 스레드가 아닌 곳에서 `@MainActor` 메서드를 호출하려면 await를 사용해야 한다.

```swift
await fetchGames()
```

클래스와 같은 더 큰 정의에 `@MainActor` 속성을 추가하면 모든 프로퍼티와 메서드가 `MainActor`가 된다. 개별 메서드는 `nonisolated` 키워드를 사용해 메인 액터의 일부가 아니도록 선택할 수 있다.

```swift
@MainActor
class GameLibraryViewController: UIViewController {
	//...
	nonisolated var fetchVideogameTypes() -> [VideogameType] { ... }
	//...
}
```

`MainActor`는 매우 중요한 개념이며, 이를 올바르게 사용하는 법을 배우면 애플이 제공하는 UI 프레임워크에서 모던 컨커런시 시스템을 더 쉽게 적용할 수 있다. 다행히 사용법은 직관적이며, 신경 써야 할 마법 같은 동작이나 숨겨진 기능이 없다.


# 글로벌 액터

앞서 `MainActor`가 '특별한' 종류의 액터라고 설명했다. 실제로 그렇지만, 유일한 특별한 액터는 아니다. `@MainActor`는 사실 글로벌 액터(Global Actor)라는 범주에 속한다.

UI 컴포넌트는 문자 그대로 어디에나 존재한다. 다양한 프레임워크에서 사용되며, 여러 파일과 임포트 경로에 걸쳐 분산되어 있다. UI 작업에 `MainActor`를 적용하려면, 필요한 모든 곳에서 사용할 수 있는 범용 액터가 필요하다. 이름 그대로 글로벌 액터는 전역적으로 선언되며, 사용을 원하는 객체는 간단히 `@MainActor class MainActor에서_실행할_클래스` 같은 방식으로 속성을 추가하면 된다.

Xcode 13 베타 3부터는 사용자 정의 글로벌 액터를 만들 수 있다.

**참고:** *Xcode 13 베타 3 릴리스 노트에서 처음으로 글로벌 액터의 존재와 사용법이 공개됐다. 이전 베타 버전이나 WWDC2021의 컨커런시 관련 세션에서는 언급되지 않았다. 초기 Xcode 13 베타에서 사용 가능했는지는 확인되지 않았지만, 이런 작은 호기심을 공유하고 싶어 언급한다.*


## 글로벌 액터 생성하기

글로벌 액터를 생성하는 방법은 다음과 같다:

```swift
@globalActor
struct MediaActor {
  actor ActorType { }

  static let shared: ActorType = ActorType()
}
```

여기서 `MediaActor`는 우리가 지정한 이름이다. 그런 다음 이 액터를 사용하려는 타입, 메서드, 모듈은 `@MainActor`와 마찬가지로 선언 앞에 해당 이름을 붙여 사용할 수 있다.

여러 곳에서 동시에 읽고 쓸 수 있는 전역 배열이 있다고 가정해보자. 이 전역 변수에 `@MediaActor`를 적용하면 모든 연산이 동일한 스레드에서 실행되어 액터가 필요에 따라 상태를 동기화한다.

다음 예제에서는 전역 `videogames` 배열을 생성하고 다양한 위치에서 업데이트할 것이다.

먼저 `GlobalState` 파일을 생성하고 글로벌 액터, 글로벌 변수, `Videogame` 구조체를 선언한다:

```swift
// GlobalState.swift

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

**중요 참고**: *이런 방식으로 전역 변수를 사용하는 것을 권장하지 않으며, 더 나은 추상화 방법이 존재한다. 이 프로젝트는 글로벌 액터를 배우기 위한 목적으로만 사용해야 한다.*

다음으로, 모든 것이 기본적으로 메인 액터에서 실행될 뷰 컨트롤러를 생성한다:

```swift
// ViewController.swift

@MainActor
class ViewController: UIViewController {
    
    @MediaActor
    func addRandomVideogames() {
        let zeldaOot = Videogame(name: "The Legend of Zelda: Ocarina of Time", releaseYear: 1998, developer: "Nintendo")
        let xillia = Videogame(name: "Tales of Xillia", releaseYear: 2013, developer: "Bandai Namco")
        let legendOfHeroes = Videogame(name: "The Legend of Heroes: A Tear of Vermilion", releaseYear: 2004, developer: "Nihon Falcom")
        
        videogames += [zeldaOot, xillia, legendOfHeroes]
    }
    
    @MediaActor
    func removeRandomvideogame() {
        if let randomElement = videogames.randomElement() {
            videogames.removeAll { $0.id == randomElement.id }
        }
        
    }
    
    @MediaActor
    func getRandomGame() -> Videogame? {
        return videogames.randomElement()
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        
        Task {
            await addRandomVideogames()
            await removeRandomvideogame()
            if let randomGame = await getRandomGame() {
                print("Random game: (randomGame.name)")
            }
        }
    }
}
```

이 예제를 선택한 이유는 뷰 컨트롤러 자체가 `@MainActor`에서 실행되고, 모든 프로퍼티와 메서드도 기본적으로 여기서 실행되기 때문이다. 하지만 전역 `videogames` 변수와 상호작용하려면 이 메서드들을 `MediaActor`에서 실행해야 한다. 뷰 컨트롤러의 세 가지 연산(`addRandomVideogames()`, `removeRandomvideogame`, `getRandomGame()`) 모두 `videogames`와 동일한 액터에서 실행되어야 하므로, 간단히 `MediaActor`로 표시하면 된다.

`@MainActor`에서 `@MediaActor` 데이터에 접근해야 할 때는 메서드가 암시적으로 `async`로 표시되므로, `await`를 사용해야 한다.

지금까지 `@MainActor`는 여러 다른 위치에 존재한다. 두 개의 다른 파일뿐만 아니라 다양한 선언을 통해 접근할 수 있다. 마지막으로 `Functions.swift` 파일을 생성하고 `@MediaActor`에서 실행될 함수를 추가한다:

```swift
// Functions.swift

@MediaActor
func showAvailableGames() async {
    for game in videogames {
        print("(game.name)")
    }
}
```

이제 끝이다! 여러분만의 글로벌 액터를 구현하는 것이 얼마나 간단한지 확인할 수 있다.


# 마무리

`@MainActor`는 글로벌 액터(Global Actor)다. 모든 UI 코드는 메인 액터에서 실행된다. 다른 스레드에서 실행될 수 있는 코드를 메인 스레드에서 실행해야 할 경우, 해당 메서드를 `@MainActor`로 표시하면 메인 스레드에서 데이터를 받을 수 있다.

글로벌 액터는 물리적으로 다른 파일에 선언된 코드나 다양한 선언들 간에 동기화가 필요할 때 유용하다. 서로 다른 파일과 타입 간의 상태를 동기화해야 한다면 직접 글로벌 액터를 생성할 수 있다. 글로벌 액터를 선언하는 건 간단하며, 해당 액터에서 실행되길 원하는 선언들은 간단히 `@` 속성으로 표시하면 된다.

[여기](https://www.andyibanez.com/archives/GlobalActors.zip)에서 커스텀 글로벌 액터를 활용한 샘플 프로젝트를 확인할 수 있다.




