---
title: "Swift 모던 컨커런시 - @TaskLocal 프로퍼티 래퍼로 태스크 간 데이터 공유하기"
date: 2025-10-25T10:43:45Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model/

이번 튜토리얼 시리즈에서 우리는 동시성과 관련된 다양한 주제를 탐구했다. 동시성이 작동하는 기본 원리부터 분리된 독릭 태스크(Detached Tasks)를 활용한 복잡한 작업 처리 방법까지 배웠다.

특히 흥미로운 주제 중 하나는 태스크 트리다([Swift의 구조화된 동시성: async let 사용법](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/) 참조). 태스크 트리는 다른 태스크 내에서 여러 태스크를 호출할 때 생성되며(Detached Tasks 제외), 트리 내 태스크는 우선순위와 컨텍스트 같은 부모 태스크의 정보를 상속받는다.

태스크가 컨텍스트 정보를 공유할 수 있다면, 다른 데이터도 공유할 수 있으면 좋지 않을까? 바로 이를 가능하게 하는 것이 [@TaskLocal](https://developer.apple.com/documentation/swift/tasklocal) 프로퍼티 래퍼다. 이 글에서는 이 프로퍼티 래퍼를 사용해 다양한 태스크 간에 데이터를 공유하는 방법을 살펴본다.

**참고**: *`@MainActor`를 제외한 [글로벌 액터](https://www.andyibanez.com/posts/mainactor-and-global-actors-in-swift/)와 마찬가지로, `@TaskLocal`을 처음 본 것은 Xcode 13 Beta 3 릴리스 노트에서였다. 이 기능이 이전부터 존재했지만 문서화되지 않았는지, 아니면 완전히 새로운 기능인지는 확실하지 않다.*


## @TaskLocal 프로퍼티 래퍼 소개

`TaskLocal` 값은 태스크 컨텍스트 내에서 읽고 쓸 수 있다. 이 값은 암묵적으로 공유되며, 태스크가 생성한 모든 하위 태스크(`async let` 또는 그룹 태스크 포함)에서 접근할 수 있다.


### @TaskLocal 사용법

이 프로퍼티 래퍼를 사용하려면 `@TaskLocal`로 표시된 프로퍼티가 반드시 정적(static)이어야 한다. 옵셔널이거나 기본값을 가질 수 있다.

값을 읽을 때는 특별한 작업이 필요 없다. 어디서든 값을 사용할 수 있지만, 부모 비동기 작업에서 미리 설정하지 않은 경우 nil 또는 할당한 기본값이 반환된다.

```swift
class ViewController: UIViewController {
    @TaskLocal static var currentVideogame: Videogame?
    // ...
}
```

위 코드에서 `TaskLocal` 프로퍼티 `currentVideogame`을 생성했다.

값을 읽는 방법은 다음과 같다:

```swift
// ViewController 외부
func expensiveVidegameOperation() async {
    if let vg = await ViewController.currentVideogame {
        print("We are processing (vg.title)")
    }
}
```

`currentVideogame`을 직접 수정하려고 하면(`ViewController` 내부에서도) 컴파일러가 허용하지 않는다. 이는 읽기 전용 프로퍼티이기 때문이다.

값을 "할당"하려면 "바인딩"해야 한다. 이를 위해 `TaskLocal`의 프로젝트된 값에 접근하면 `withValue`라는 바인딩 메서드를 사용할 수 있다.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    // 뷰 로드 후 추가 설정
    
    let vg = Videogame(title: "The Legend of Zelda: Ocarina of Time", year: 1998)
    Self.$currentVideogame.withValue(vg) {
        // 여기서 LocalValue를 사용하는 비동기 작업 실행 가능
    }
}
```

이 예제에서는 `vg`를 `currentVideogame` 태스크 값에 바인딩한다. 이후 생성된 모든 태스크는 태스크 트리의 일부인 동안 이 값에 접근할 수 있다.

다음 예제를 살펴보자:

```swift
class ViewController: UIViewController {
    @TaskLocal static var currentVideogame: Videogame?

    override func viewDidLoad() {
        super.viewDidLoad()
        // 뷰 로드 후 추가 설정
        
        let vg = Videogame(title: "The Legend of Zelda: Ocarina of Time", year: 1998)
        Self.$currentVideogame.withValue(vg) {
            Task {
                await expensiveVidegameOperation()
                Task {
                    await expensiveVidegameOperation()
                    Task.detached {
                        await expensiveVidegameOperation()
                    }
                }
            }
        }
    }
}
```

위 코드에서는 `Videogame`을 바인딩한 후 일부 태스크를 시작한다. `expensiveVideogameOperation`을 호출하는 `Task`를 시작하면 `We are processing The Legend of Zelda: Ocarina of Time`을 출력한다. `await` 후 현재 태스크의 자식인 또 다른 `Task`를 시작한다. `expensiveVidegameOperation`을 호출하면 동일한 부모에 접근할 수 있으므로 동일한 메시지가 출력된다. 분리된(detached) 태스크를 시작할 때 더 흥미로운 점이 나타난다. 분리된 태스크에서 `expensiveVidegameOperation`을 호출하면 `No videogame found in the task hierarchy!`가 출력된다. 분리된 태스크는 완전히 독립적이며 부모가 없기 때문이다(다른 태스크의 부모가 될 수는 있지만 분리된 태스크로 시작하지 않은 경우에만 해당).

분리된 태스크 내에서 다른 비디오 게임을 자유롭게 바인딩하고 다른 태스크를 시작하여 해당 값에 접근할 수 있다:

```swift
Task.detached {
    await expensiveVidegameOperation()
    let anotherVg = Videogame(title: "Tales of the Abyss", year: 2005)
    await Self.$currentVideogame.withValue(anotherVg) {
        await expensiveVidegameOperation()
    }
}
```

`currentVideogame`으로 값을 바인딩하기 전에 `await`를 사용한다. 컴파일러의 마법인지는 확실하지 않지만, 태스크 내부에서는 반드시 await를 사용해야 한다. `TaskLocal` 값은 여러 스레드에서 동시에 접근할 수 있으므로, 값을 쓰면 경쟁 조건을 방지할 수 있다.


# 결론

작업 계층 구조 내 모든 하위 작업에 값을 공유해야 하는 경우가 있을 수 있다. 이럴 때는 `@TaskLocal` 프로퍼티 래퍼를 사용하면 된다. 이 값은 모든 하위 작업에 공유되지만, 분리된(detached) 작업의 특성상 해당 작업들에는 값이 전달되지 않는다.




