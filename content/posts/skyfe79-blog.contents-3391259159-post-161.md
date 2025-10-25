---
title: "Swift 모던 컨커런시: 시작하기"
date: 2025-09-07T06:09:47Z
author: "skyfe79"
draft: false
tags: ["swift","아마추어 번역"]
---

> https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/

## 서론

*본 시리즈는 원래 Xcode 13 베타 1 버전을 사용해 예제를 만들며 작성되었다. 시리즈 내 글, 코드 샘플, 제공된 샘플 프로젝트는 Xcode 13 베타 3 버전으로 업데이트되었다.*

이 튜토리얼 시리즈는 WWDC2021에서 애플이 소개한 새로운 async/await API에 초점을 맞추고 있다. 아직 총 몇 개의 글로 구성될지 확정하지 못했지만, 앞으로 몇 주 동안 계속해서 게시될 예정이다.

WWDC2021 세션 동영상들은 이 새로운 API를 설명하는 데 훌륭하지만, 초보자나 오랜 경력의 개발자 모두에게 여전히 부담스러울 수 있다고 생각한다. 이 시리즈의 목적은 새로운 컨커런시 API를 단계별로 설명하며, 각 글마다 몇 가지 개념을 다루어 독자들이 이 API에 대한 이해도를 높일 수 있도록 돕는 것이다. 필요한 경우 애플에서 제공한 코드 조각을 사용하거나 수정할 수도 있다. 이런 외부 코드는 명시적으로 표기할 것이다.

시리즈 전반에 걸쳐 소개할 지식은 WWDC2021 세션(각 글마다 관련 세션 링크를 제공할 예정), 직접 실험해 본 경험, 그리고 다른 출처에서 얻은 내용을 바탕으로 한다. 새로운 await/async API에 대한 모든 것을 알고 있다고 주장하지 않으며, Evolution 제안서가 WWDC2021 이전에 승인되었지만, 본 시리즈는 WWDC2021 이전 제안서를 탐구하지 않은 상태에서 얻은 지식을 바탕으로 작성하고 있다.

따라서 부정확한 내용을 발견하면 꼭 지적해 주길 바란다. 이 시리즈가 최대한 명확하고 정확할 수 있도록 하는 것이 매우 중요하다. 오타나 이상한 점을 발견하면 이메일이나 트위터로 알려주기 바란다.

새로운 API를 탐구하기 전에, 현재의 컨커런시 구현 방식과 그 문제점에 대해 먼저 이야기해보자. 이 소개 글을 마칠 때쯤이면 독자들은 새로운 API에 시간을 투자할 가치가 있다는 확신을 갖게 될 것이다.


## 동시성과 기존 구현 방식의 문제점


WWDC2021에서 애플은 개발자들이 앱에 동시성을 구현할 수 있는 새로운 방법을 소개했다. 이 방식을 'async/await API'라고 부를 것이다. 이 두 단어가 핵심 개념이기 때문이다.

개발자들은 종종 자신도 모르게 동시성을 사용해 왔다. iOS SDK에서 클로저를 인자로 받는 대부분의 메서드가 비동기 호출이다. iOS 개발 경험이 있다면 UI 코드가 메인 스레드에서 실행된다는 사실을 알고 있을 것이다. UI 조작이 이곳에서 이뤄지기 때문에, 어떤 작업이 너무 오래 걸리면 시스템이 앱이 멈췄다고 판단해 강제 종료할 수 있다. 사용자들이 앱이 멈춘 상태임을 인지하기 전에 말이다. 일반적으로 소프트웨어에서 동시성이 필요한 이유는 *동시* 작업을 생성해 어떤 일을 처리하거나 속도를 높이기 위해서다. 애플 기술에서도 동시성 필요성은 같지만, 메인 스레드가 차단되지 않도록 주의해야 한다는 점이 추가된다.

메인 스레드를 멈출 수 있는 호출은 애플 SDK 전체에 퍼져 있다. 그래서 애플은 작업을 다른 스레드로 위임하고 메인 스레드를 자유롭게 유지할 수 있는 다양한 도구를 제공한다.

앞으로 나아가기 전에, 이 새로운 API가 표준으로 자리 잡을 것이라는 점은 확실하지만 동시성을 위한 유일한 방법은 아니다. async/await API가 요구사항을 충족하지 못한다면 [다른 옵션에 관한 전체 글](https://www.andyibanez.com/posts/multithreading-options-on-apple-platforms/)을 참고하면 된다.


## API 소비자를 위한 콜백 기반 동시성 처리

`URLSession` API를 예로 들어보자. WWDC2021 이전에는 네트워크 호출이 필요할 때 다음과 같은 방식을 사용했다:

```swift
// ... (1)

let task = URLSession.shared.dataTask(with: ...) { data, response, error in
    // ... (2)
}

task.resume()

// ... (3)
```

콜백 클로저 내부의 코드(중괄호 `{}` 안의 모든 내용)는 다운로드가 완료된 후 *비동기적으로* 호출된다. 호출 순서는 보장되지 않으며, `(1)` 이후에 실행된다는 것만 알 수 있다.

위 코드 조각에서 `(1)`은 네트워크 호출 전에 실행되는 코드다. 하지만 `(2)`는 즉시 실행되지 않는다. 대신 프로그램은 `(3)`으로 계속 실행될 *수도* 있고, 다운로드가 완료된 상태라면 `(2)`가 실행될 수도 있다. `(2)`와 `(3)`의 실행 순서는 보장되지 않는다. 이 예제에서는 네트워크 호출이 프로그램의 선형 실행보다 느리다는 것이 "당연해" 보이지만, 네트워크를 사용하지 않는 API 중에도 본질적으로 비동기적인 경우가 많다는 점을 명심하자.

이러한 콜백 방식은 여전히 유효하고, 이러한 레거시 API는 사라지지 않을 것이다. 하지만 응답을 기반으로 JSON을 파싱하거나 추가 네트워크 호출이 필요하다면 어떻게 될까? 이는 고통스러운 작업이 되며, 소위 *파멸의 피라미드*에 직면하게 된다.

```swift
let task = URLSession.shared.dataTask(with: ...) { data, response, error in
    let taskThatNeedsPreviousResponse = URLSession.shared.dataTask(with: ...) { data, response, error in
        let evenMoreNestedNetworking = URLSession.shared.dataTask(with: ...) { data, response, error in
            /// 여기서야 비로소 추가 작업을 수행할 수 있음
        }
        evenMoreNestedNetworking.resume()
    }
    taskThatNeedsPreviousResponse.resume()
}

task.resume()
```

호출이 중첩될수록 가독성 문제가 발생한다. 각 "피라미드 층"을 별도의 함수로 분리할 수는 있지만, 이는 스코프를 오염시키는 임시 방편일 뿐 근본적인 해결책이 되지 못한다.


## API 디자이너를 위한 콜백 기반 동시성 처리

이미지를 다운로드하고 썸네일 크기로 리사이즈하는 함수를 만들어야 한다고 가정해 보자. 콜백을 사용하면 다음과 같은 코드를 작성하게 될 수 있다.

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else {
                return // (1)
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    return // (2)
                }
                completion(thumbnail, nil)
            }
        }
    }
    task.resume()
}
```

*(이 코드는 Apple의 [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/) 세션에서 그대로 가져왔다)*

먼저 눈에 띄는 점은 이 코드가 상당히 길고 복잡하다는 것이다. 이미지 다운로드와 리사이즈 모두 비동기 호출로 이루어지며, 각 단계에서 콜백을 처리해야 한다.

이 코드에는 버그도 존재하며 발견하기 어려울 수 있다. 콜백 기반 함수를 작성할 때는 어떤 일이 발생하든지 반드시 전달받은 콜백을 호출해야 한다. 위 예제에서 `(1)`과 `(2)`로 표시된 부분은 콜백을 호출하지 않고 종료되는 경우다. 이렇게 되면 API를 호출한 측은 영원히 응답을 기다리게 된다. 다행히도 최소한 스레드는 블로킹되지 않는다.

첫 번째 문제는 작업 완료 시 콜백을 직접 호출해야 한다는 점이다. 간단한 함수라면 괜찮지만, 다양한 예외 상황을 고려해야 할 때는 관리가 어려워진다.

콜백 기반 API의 또 다른 문제점은 반환 타입과 오류 정보가 모두 클로저에 포함된다는 것이다. 이로 인해 명확한 반환 타입과 오류 발생 가능성을 표현하는 깔끔한 API를 만들 수 없다. 오류를 던지는(throw) 개념이 없기 때문에 콜백을 통해 오류를 전달해야 한다. 정적 타입 검사는 여전히 작동하지만 추상화 레이어가 추가되어 자동 완성 기능도 덜 유용해진다. 게다가 API 사용자가 오류를 무시할 수도 있는데, 항상 나쁜 것은 아니지만 특정 상황에서는 반드시 처리해야 하는 경우도 있다.


## Combine?


Combine 프레임워크는 파이프라인을 활용해 앞서 언급한 문제들을 우아하게 해결한다. 하지만 이 시리즈에서는 Apple의 리액티브 프레임워크에 대해 많이 다루지 않을 것이다. 이에는 몇 가지 이유가 있다.

첫째, Combine의 미래가 불확실하다고 판단한다. 개인적으로 이 프레임워크를 좋아하지만, WWDC2021 초반에는 [새로운 API들이 Combine을 대체할 수 있을지 회의적이었다](https://twitter.com/AndyIbanezK/status/1402463420931317762?s=20). 하지만 관련 세션을 더 본 후 생각이 바뀌었다.

둘째, Combine이 충분히 활용되지 않고 있다고 느낀다. 2019년에 소개된 Combine은 SwiftUI의 존재에 큰 영향을 미쳤다. 하지만 출시 후 몇 년 동안 크게 채택되지 않은 것 같다. 콜백 기반 코드를 대체하기 위해 사용된다는 증거도 없으며, 커뮤니티 자료의 부족(일부 훌륭한 자료는 존재함)과 프레임워크 업데이트의 부재로 보아 Apple의 계획이 명확해질 때까지 시간을 투자하지 않는 것이 현명할 수 있다.

이 튜토리얼 시리즈에서는 관련이 없는 한 Combine을 자주 언급하지 않을 것이다. 일반적으로 더 이상 콜백 기반 코드를 대체할 후보로 보지 않는다. 비동기 코드를 Future로 감싸는 것을 매우 좋아했지만 말이다.


## 새로운 사고방식

아래 글들을 본격적으로 살펴보기 전에, 기존의 컨커런시 개념을 일단 접어두길 권한다. async/await 구현 방식은 기존과 완전히 다르기 때문이다. 이 기능을 제대로 이해하려면 먼저 새로운 사고방식을 받아들여야 한다. async/await을 먼저 이해하면 나머지 도구들은 더 쉽게 이해할 수 있다.

기존 지식이 아예 쓸모없다는 뜻은 아니다. 오히려 그 반대다. 하지만 더 쉽게 작성할 수 있는 컨커런시 코드를 구현하려면 지난 수십 년간 애플 플랫폼에서 컨커런시를 어떻게 다뤘는지 다시 생각해볼 필요가 있다. 사실 async/await은 비동기 코드를 처음 접하는 사람들이 이해하기 더 쉬울 수 있다. 절차적 프로그래밍과 매우 유사하기 때문이다.


## 바로 시작해 보자

아래 목차는 이 시리즈의 글들을 보여준다. 각 글은 독립적으로 구성되어 있어, 마지막 글만 필요하다면 앞부분을 읽지 않아도 된다. 하지만 비동기/await 개념이 처음이라면 순서대로 모두 읽는 것이 좋다.

대부분의 글에는 직접 실행해 볼 수 있는 코드 예제가 포함되어 있다. 학습에 도움이 되도록 자유롭게 복사해 사용하거나, 제공되는 샘플 프로젝트를 다운로드하면 된다.


## 목차

1. [Swift에서 async/await 이해하기](https://www.andyibanez.com/posts/understanding-async-await-in-swift/)
2. [Swift에서 클로저 기반 코드를 async/await로 변환하기](https://www.andyibanez.com/posts/converting-closure-based-code-into-async-await-in-swift/)
3. [Swift의 구조적 동시성: async let 사용법](https://www.andyibanez.com/posts/structured-concurrency-in-swift-using-async-let/)
4. [Swift에서 Task Group을 활용한 구조적 동시성](https://www.andyibanez.com/posts/structured-concurrency-with-group-tasks-in-swift/)
5. [Swift의 비구조적 동시성 소개](https://www.andyibanez.com/posts/introduction-to-unstructured-concurrency-in-swift/)
6. [Swift에서 Detached Task를 사용한 비구조적 동시성](https://www.andyibanez.com/posts/unstructured-concurrency-with-detached-tasks-in-swift/)
7. [Swift의 새로운 동시성 모델에서 Actor 이해하기](https://www.andyibanez.com/posts/understanding-actors-in-the-new-concurrency-model-in-swift/)
8. [Swift에서 @MainActor와 Global Actors](https://www.andyibanez.com/posts/mainactor-and-global-actors-in-swift/)
9. [Swift의 새로운 동시성 모델에서 @TaskLocal 프로퍼티 래퍼로 Task 간 데이터 공유하기](https://www.andyibanez.com/posts/modern-concurrency-in-swift-introduction/posts/sharing-data-across-tasks-tasklocal-new-swift-concurrency-model)
10. [Swift에서 AsyncSequence 사용하기](https://www.andyibanez.com/posts/using-asyncsequence-in-swift/)
11. [모던 Swift 동시성 요약, 치트시트, 그리고 감사의 말](https://www.andyibanez.com/posts/modern-swift-concurrency-summary-cheatsheet-thanks/)




