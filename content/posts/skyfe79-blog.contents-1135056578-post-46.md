---
title: "가장 빠르게 Swift 코드 실행하기"
date: 2022-02-13T02:59:49Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

이 글은 최적화에 대한 글이 아니다.😄 간단한 Swift 코드를 만들고 실행할 때, Xcode Playground 말고 다른 방법으로 Swift 코드를 빠르게 실행해 보는 방법에 관한 이야기다.

## Xcode Playground

Xcode Playground는 정말 좋다. 이미지 리소스도 쉽게 포함할 수 있다. 하지만 매번 Xcode를 실행하고 Playground를 만들어서 테스트하고 실행하는 점이 번거로울 수 있다.

간단한 Swift 코드를 작성할 때는 `VSCode` 와 `swift` 명령어를 사용해서 쉽고 빠르게 Swift 코드를 실행할 수 있다.

## VSCode

Visual Studio Code에서 Swift 코드를 아주 쉽게 작성할 수 있다. Swift 자동 완성도 지원한다. 자동완성에 대한 내용은 [vscode에서 Swift 자동 완성 사용하기](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-973482357-post-18/)을 참고한다.

아래와 같은 코드를 vscode로 작성한 후 터미널에서 `swift` 명렁어로 실행할 수 있다.

```swift
import Foundation

extension Array {
  var isNotEmpty: Bool {
    return !self.isEmpty
  }
}

extension Array where Element: Numeric {
  func customSum() -> Element {
    return self.reduce(0, +)
  }
}

extension Array where Element == String {
  func customJoin(sep: String = "") -> Element {
    return self.reduce("", {$0 + $1 + sep})
  }
}


assert(["A", "B", "C"].customJoin() == "ABC")
assert([].isNotEmpty == false)

print(["A", "B", "C"].customJoin())
```

### swift로 실행하기

```
$ swift ArrayExtentionSample.swift
```

## 스크립트

swift 코드를 스크립트로 실행할 수 있다. 첫 줄에 `swift`를 해쉬뱅으로 설정하면 된다.

```
#!/usr/bin/env swift

import Foundation

extension Array {
  var isNotEmpty: Bool {
    return !self.isEmpty
  }
}

extension Array where Element: Numeric {
  func customSum() -> Element {
    return self.reduce(0, +)
  }
}

extension Array where Element == String {
  func customJoin(sep: String = "") -> Element {
    return self.reduce("", {$0 + $1 + sep})
  }
}


assert(["A", "B", "C"].customJoin() == "ABC")
assert([].isNotEmpty == false)

print(["A", "B", "C"].customJoin())
```

### 스크립트 실행

`chmod` 로 실행 권한을 부여한 다음 실행한다.

```
$ chmod +x ArrayExtentionSample.swift
```

 단 한번만 실행 권을 주면 된다. 그 다음부터는 아래와 같이 실행한다.

```
$ ./ArrayExtensionSample.swift
```