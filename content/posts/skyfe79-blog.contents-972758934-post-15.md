---
title: "Swift 유닛테스트 환경 쉽게 만들기"
date: 2021-08-17T14:34:42Z
draft: false
tags: ["swift"]
---

Swift 로 UnitTest 작성 환경이 필요할 때, 가장 빠른 방법이 무엇일까? 
코딩테스트나 간단한 알고리즘 구현시에 유닛테스트를 만들어 진행하면 편리한데, IDE를 실행하여 프로젝트를 만들거나 플레이그라운드를 만드는 것은 뭔가 번잡하게 느껴졌다.



## Swift Package Manager

Swift Package Manager를 사용하면 쉽게 UnitTest 환경을 만들 수 있다. 아래처럼 테스팅을 위한 폴더를 만든 다음 Swift Package 로 초기화한다.

```bash
$ mkdir collection-test
$ cd collection-test
$ swift package init 
```

```
Creating library package: collection-test
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/collection-test/collection_test.swift
Creating Tests/
Creating Tests/collection-testTests/
Creating Tests/collection-testTests/collection_testTests.swift
```



## 테스트 실행

Swift Package로 초기화를 하면 테스트가 이미 만들어져 있다. 아래와 같이 만들어진 테스트를 실험할 수 있다.

```
$ swift test
```

```
[5/5] Linking collection-testPackageTests

* Build Completed!
Test Suite 'All tests' started at 2021-08-17 23:36:38.860
Test Suite 'collection-testPackageTests.xctest' started at 2021-08-17 23:36:38.861
Test Suite 'collection_testTests' started at 2021-08-17 23:36:38.861
Test Case '-[collection_testTests.collection_testTests testExample]' started.
Test Case '-[collection_testTests.collection_testTests testExample]' passed (0.003 seconds).
Test Suite 'collection_testTests' passed at 2021-08-17 23:36:38.864.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.003 (0.003) seconds
Test Suite 'collection-testPackageTests.xctest' passed at 2021-08-17 23:36:38.864.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.003 (0.003) seconds
Test Suite 'All tests' passed at 2021-08-17 23:36:38.864.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.003 (0.004) seconds
```


## 패키지 폴더 구조

Swift Package Manager가 초기화한 패키지 폴더 구조는 아래와 같다.

```
.
├── Package.swift
├── README.md
├── Sources
│   └── collection-test
│       └── collection_test.swift
└── Tests
    └── collection-testTests
        └── collection_testTests.swift
```

## iOS 관련 유닛테스트

만약, 디펜던시로 iOS 용 라이브러리를 사용한다면 테스트를 iOS 개발 환경에서 실행해야 한다. 이를 위해서 스킴과 개발 플랫폼을 설정하여 테스트를 실행할 수 있다. 이 때는 `swift` 대신에 `xcodebuild`를 사용한다.

### Scheme 목록 확인

```
$ xcodebuild -list
```

```
Command line invocation:
    /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -list

User defaults from command line:
    IDEPackageSupportUseBuiltinSCM = YES

Resolve Package Graph

Resolved source packages:
  collection-test: /Users/burt/github/temp/collection-test

Information about workspace "collection-test":
    Schemes:
        collection-test
```

`collection-test` 스킴을 확인할 수 있다.

### iOS 개발 플랫폼에서 테스트 실행하기

```
$ xcodebuild -scheme collection-test test -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 12'
```

```
Command line invocation:
    /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -scheme collection-test test -sdk iphonesimulator -destination "platform=iOS Simulator,name=iPhone 12"

User defaults from command line:
    IDEPackageSupportUseBuiltinSCM = YES

Build settings from command line:
    SDKROOT = iphonesimulator14.5

Resolve Package Graph

Resolved source packages:
  collection-test: /Users/burt/github/temp/collection-test

...

Testing started
Test Suite 'All tests' started at 2021-08-17 23:43:15.537
Test Suite 'collection-testTests.xctest' started at 2021-08-17 23:43:15.538
Test Suite 'collection_testTests' started at 2021-08-17 23:43:15.538
Test Case '-[collection_testTests.collection_testTests testExample]' started.
Test Case '-[collection_testTests.collection_testTests testExample]' passed (0.002 seconds).
Test Suite 'collection_testTests' passed at 2021-08-17 23:43:15.541.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.002 (0.003) seconds
Test Suite 'collection-testTests.xctest' passed at 2021-08-17 23:43:15.541.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.002 (0.003) seconds
Test Suite 'All tests' passed at 2021-08-17 23:43:15.542.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.002 (0.004) seconds
2021-08-17 23:43:15.808 xcodebuild[21690:392957] [MT] IDETestOperationsObserverDebug: 7.114 elapsed -- Testing started completed.
2021-08-17 23:43:15.808 xcodebuild[21690:392957] [MT] IDETestOperationsObserverDebug: 0.000 sec, +0.000 sec -- start
2021-08-17 23:43:15.808 xcodebuild[21690:392957] [MT] IDETestOperationsObserverDebug: 7.114 sec, +7.114 sec -- end

Test session results, code coverage, and logs:
	/Users/burt/Library/Developer/Xcode/DerivedData/collection-test-ghhizvljwobqsmcyznxghxnvhbyv/Logs/Test/Test-collection-test-2021.08.17_23-43-05-+0900.xcresult

** TEST SUCCEEDED **
```




## 디펜던시 추가하기

`Package.swift` 파일을 열고 `dependencies` 에 원하는 디펜던시를 추가한다. 이 글에서는 애플의 `swift-collections` 라이브러리를 설치해 본다.

```swift
// swift-tools-version:5.3
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "collection-test",
    products: [
        // Products define the executables and libraries a package produces, and make them visible to other packages.
        .library(
            name: "collection-test",
            targets: ["collection-test"]),
    ],
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        .package(url: "https://github.com/apple/swift-collections.git", from: "0.0.5"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "collection-test",
            dependencies: []),
        .testTarget(
            name: "collection-testTests",
            dependencies: [
                "collection-test",
                // 중요!
                .product(name: "Collections", package: "swift-collections")
            ]),
    ]
)
```

`.testTarget` 에 `.product` 로 명확하게 `swift-collections`를 선언해 주는 것이 중요하다. 이렇게 하지 않으면 test타겟에서 `swift-collections`를 찾을 수 없다.. 디펜던시를 추가하고 `swift build` 또는 `swift test`를 실행하여 디펜던시를 설치한다.

```
$ swift test
```

```
Fetching https://github.com/apple/swift-collections
Cloning https://github.com/apple/swift-collections
Resolving https://github.com/apple/swift-collections at 0.0.5
warning: dependency 'swift-collections' is not used by any target
[1/1] Planning build

* Build Completed!Test Suite 'All tests' started at 2021-08-17 23:49:32.667
Test Suite 'collection-testPackageTests.xctest' started at 2021-08-17 23:49:32.668
Test Suite 'collection_testTests' started at 2021-08-17 23:49:32.668
Test Case '-[collection_testTests.collection_testTests testExample]' started.
Test Case '-[collection_testTests.collection_testTests testExample]' passed (0.004 seconds).
Test Suite 'collection_testTests' passed at 2021-08-17 23:49:32.673.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.004 (0.005) seconds
Test Suite 'collection-testPackageTests.xctest' passed at 2021-08-17 23:49:32.673.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.004 (0.005) seconds
Test Suite 'All tests' passed at 2021-08-17 23:49:32.673.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.004 (0.005) seconds
```

## 테스트 작성하기

방금 설치한 `swift-collections`를 테스트해 보자. 이미 만들어진 테스트 파일 중에 `collection_testTests.swift` 파일을 열고 아래와 같이 테스트를 작성한다.

```swift
import XCTest
@testable import collection_test
import Collections

final class collection_testTests: XCTestCase {
    func testExample() {
        // This is an example of a functional test case.
        // Use XCTAssert and related functions to verify your tests produce the correct
        // results.
        XCTAssertEqual(collection_test().text, "Hello, World!")
    }

    func testDeque() {
        var deque: Deque<String> = ["Ted", "Rebecca"]
        deque.prepend("Keeley")
        deque.append("Nathan")
        XCTAssertEqual(deque, ["Keeley", "Ted", "Rebecca", "Nathan"])
        XCTAssertEqual(deque.popLast(), "Nathan")
        XCTAssertEqual(deque[1], "Ted")
        XCTAssertEqual(deque.count, 3)

        deque.remove(at: 1)
        XCTAssertEqual(deque.count, 2)
    }
}
```


## 테스트 실행

작성한 테스트를 실행한다.

```
$ swift test
```

```
[3/3] Linking collection-testPackageTests

* Build Completed!
Test Suite 'All tests' started at 2021-08-18 00:20:07.819
Test Suite 'collection-testPackageTests.xctest' started at 2021-08-18 00:20:07.820
Test Suite 'collection_testTests' started at 2021-08-18 00:20:07.820
Test Case '-[collection_testTests.collection_testTests testDeque]' started.
Test Case '-[collection_testTests.collection_testTests testDeque]' passed (0.002 seconds).
Test Case '-[collection_testTests.collection_testTests testExample]' started.
Test Case '-[collection_testTests.collection_testTests testExample]' passed (0.000 seconds).
Test Suite 'collection_testTests' passed at 2021-08-18 00:20:07.822.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.003 (0.003) seconds
Test Suite 'collection-testPackageTests.xctest' passed at 2021-08-18 00:20:07.822.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.003 (0.003) seconds
Test Suite 'All tests' passed at 2021-08-18 00:20:07.822.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.003 (0.003) seconds
```

테스트가 잘 실행 됨을 확인할 수 있다.