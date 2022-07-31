---
title: "Swift 스크립트에서 로컬 파일 읽기"
date: 2022-02-13T12:09:39Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

sync 방식으로 파일을 읽는 방법은 많다. 제일 간단한 방법은 `String(contentOfFile:)` 을 사용하는 것이다. Swift 5.5 에 추가된 async/await 를 적용하면 비동기로 파일을 읽을 수 있다. 

현재 폴더에서 ReadLocalFile.swift 를 생성한다.

```
$ code ReadLocalFile.swift
```

스크립트로 실행하기 위해서 ReadLocalFile.swift 최상단에 해쉬뱅을 적는다.

```
#!/usr/bin/env swift
```

## sync

많은 방법이 있지만 제일 간단한 방법은 `String(contentOfFile:encoding:)`을 사용하는 것이다.

```swift
import Foundation

let fileLocation = "./ReadLocalFile.swift"
if let fileContent = try? String(contentsOfFile: fileLocation, encoding: .utf8) {
  print(fileContent)
}
```

## async/await

Swift 5.5부터 지원하는 async/await를 활용하여 FileHandler, URL 등은 macOS 12.0 이상부터 사용할 수 있는 AsyncSequence 속성을 제공한다. 지금 작성하고 있는 것은 macOS에서 실행하는 스크립트이기 때문에  iOS가 아닌 macOS 지원 버전을 확인해야 한다. 

그 중 하나인 FileHandler의 bytes 를 적용하면 `for await` 를 사용하여 비동기 방식으로 줄 단위로 읽을 수 있다.  주의할 점은 FileHandler.bytes와 URL.lines 는 macOS 12.0 이상부터  지원하기 때문에 macOS 12.0 미만에서 실행할 때는 #available로 반드시 버전 검사를 해야 한다.

```swift
if #available(macOS 12.0, *), let fileHandle = FileHandle(forReadingAtPath: fileLocation) {
  Task {
    for try await line in fileHandle.bytes {
      print(line)
    }
  }
}
```

URL은 lines 라는 AsyncSequence 속성을 제공한다.

```swift
if #available(macOS 12.0, *) {
  let url = URL(fileURLWithPath: fileLocation)
  Task {
    for try await line in url.lines {
      print(line)
    }
  }
}
```

## 실행하기

```
$ chmod +x ReadLocalFile.swift
$ ./ReadLocalFile.swift
```

위 방법으로 실행하면 아래와 같이 `ReadLocalFile.swift`를 읽어 출력한다. macOS 12.0 이상에서는 3번 반복하여 출력하고 그렇지 않으면 단 1회만 출력된다.

```
#!/usr/bin/env swift

import Foundation

let fileLocation = "./ReadLocalFile.swift"
if let fileContent = try? String(contentsOfFile: fileLocation, encoding: .utf8) {
  print(fileContent)
}

if #available(macOS 12.0, *), let fileHandle = FileHandle(forReadingAtPath: fileLocation) {
  Task {
    for try await line in fileHandle.bytes {
      print(line)
    }
  }
}

if #available(macOS 12.0, *) {
  let url = URL(fileURLWithPath: fileLocation)
  Task {
    for try await line in url.lines {
      print(line)
    }
  }
}


...
...
...

```