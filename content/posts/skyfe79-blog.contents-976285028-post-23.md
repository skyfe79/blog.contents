---
title: "Swift로 Command Line 앱 만들기 #1"
date: 2021-08-22T04:54:02Z
draft: false
tags: ["swift"]
---

[Swift로 스크립트 작성하기](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976039728-post-22/) 글에서 스크립트를 통해 원하는 작업을 할 수 있었다. 하지만 스크립트는 외부 패키지를 활용하는 점이 어려웠다. 

Command Line 앱을 만들면 쉽게 외부 패키지를 포함하여 앱을 만들 수 있다. 또한 바이너리로 앱을 배포할 수 있다. 

## 프로젝트 생성

원하는 폴더를 만들고 프로젝트를 생성한다. 예로 `greet` 이라는 프로젝트를 만들어 보자.

```
$ mkdir greet && cd greet
$ swift package init --type executable
```

```
Creating executable package: greet
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/greet/main.swift
Creating Tests/
Creating Tests/greetTests/
Creating Tests/greetTests/greetTests.swift
```

`--type executable` 옵션을 주어 타입을 `executable`로 정해 주는 것이 중요하다. 이렇게 하면 Swift Package가 초기화 되어 관련 파일들이 생성된다. 패키지 빌드로 생성되는 앱의 바이너리 이름은 `greet`이 된다. 만약 이 이름을 바꾸고 싶다면 패키지 초기화시에 아래 처럼 `--name` 옵션을 주어 변경할 수 있다.

```
$ swift package init --name Greet --type executable
```


## 패키지 폴더 구조

패키지를 초기화 하면 아래와 같은 폴더 구조가 만들어진다.

```
$ tree
.
├── Package.swift
├── README.md
├── Sources
│   └── greet
│       └── main.swift
└── Tests
    └── greetTests
        └── greetTests.swift
```

### Package.swift

패키지를 설명하는 파일이다. 

- 패키지의 이름
- 패키지 빌드 타켓
- 패키지 테스트 타켓
- 외부 패키지 디펜던시 설정 

등을 관리한다.

```swift
// swift-tools-version:5.3
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "greet",
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        // .package(url: /* package url */, from: "1.0.0"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "greet",
            dependencies: []),
        .testTarget(
            name: "greetTests",
            dependencies: ["greet"]),
    ]
)
```

위 파일은 `Swift`언어로 작성된 파일이다. `Package`가 장황해 보이지만 사실은 `Package()` 함수이다. 자세한 내용은 [PackageDescription API](https://docs.swift.org/package-manager/PackageDescription/index.html)에서 확인할 수 있다.

### greetTest.swift

테스트 작성 파일이다. 테스트 파일을 보면 `testExample()` 함수 내용이 흥미롭다.

```swift
import XCTest
import class Foundation.Bundle

final class greetTests: XCTestCase {
    func testExample() throws {
        // This is an example of a functional test case.
        // Use XCTAssert and related functions to verify your tests produce the correct
        // results.

        // Some of the APIs that we use below are available in macOS 10.13 and above.
        guard #available(macOS 10.13, *) else {
            return
        }

        // Mac Catalyst won't have `Process`, but it is supported for executables.
        #if !targetEnvironment(macCatalyst)

        let fooBinary = productsDirectory.appendingPathComponent("greet")

        let process = Process()
        process.executableURL = fooBinary

        let pipe = Pipe()
        process.standardOutput = pipe

        try process.run()
        process.waitUntilExit()

        let data = pipe.fileHandleForReading.readDataToEndOfFile()
        let output = String(data: data, encoding: .utf8)

        XCTAssertEqual(output, "Hello, world!\n")
        #endif
    }

    /// Returns path to the built products directory.
    var productsDirectory: URL {
      #if os(macOS)
        for bundle in Bundle.allBundles where bundle.bundlePath.hasSuffix(".xctest") {
            return bundle.bundleURL.deletingLastPathComponent()
        }
        fatalError("couldn't find the products directory")
      #else
        return Bundle.main.bundleURL
      #endif
    }
}

```

[Swift로 스크립트 작성하기](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976039728-post-22/) 글에서 보았던 `Process`와 `Pipe`를 사용하여 출력을 테스트하기 때문이다. [Swift로 스크립트 작성하기](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-976039728-post-22/) 글에서 표준 출력을 파이프에 연결하여 출력을 캡쳐할 수 있음을 배웠는데 그 내용을 활용하여 출력을 테스트 하는 것이다.

그 이외에도 `var productsDirectory: URL` 속성을 보면 빌드한 프로덕트의 디렉토리 위치를 얻는 과정도 볼 만하다.

### main.swift

만들려고 하는 패키지의 실제 구현 내용을 담는 파일이다. 현재는 아래와 같이 화면에 `Hello, world!`를 출력하는 내용만 있다.

```swift
print("Hello, world!")
```

## 빌드하기

### Debug 빌드

패키지를 빌드하는 방법은 쉽다. `swift build` 를 통해 패키지를 빌드 할 수 있다.

```
$ swift build
[3/3] Linking greet

* Build Completed!
```

`swift build`를 하면 `Debug` 모드로 빌드가 된다. 그 결과는 `.build` 폴더에 담긴다. `.build` 폴더의 구조는 아래와 같다.

```
$ cd .build
$ tree
.
├── artifacts
├── checkouts
├── debug -> x86_64-apple-macosx/debug
├── debug.yaml
├── manifest.db
├── manifest.db-shm
├── manifest.db-wal
├── manifest.db.lock
├── repositories
├── workspace-state.json
└── x86_64-apple-macosx
    ├── build.db
    └── debug
        ├── ModuleCache
        │   ├── 17YAVJCLXMW5D
        │   │   └── SwiftShims-2DA6NLEWJC11R.pcm
        │   ├── 3LL7Y84ZXZ8SS
        │   │   ├── SwiftShims-2DA6NLEWJC11R.pcm
        │   │   └── SwiftShims-2DA6NLEWJC11R.pcm.timestamp
        │   ├── AppKit-2M0Z5MR5C3IY8.swiftmodule
        │   ├── CloudKit-2X0UL7GLNE100.swiftmodule
        │   ├── Combine-3S6DCRYSSQVV.swiftmodule
        │   ├── CoreData-QEDF00OV85KC.swiftmodule
        │   ├── CoreFoundation-1LFTOIU4SXW4O.swiftmodule
        │   ├── CoreGraphics-2B15X0KCALZVO.swiftmodule
        │   ├── CoreImage-YK43G7LD15ON.swiftmodule
        │   ├── CoreLocation-3845USDBJ9Z1G.swiftmodule
        │   ├── Darwin-3IZFOTRZIXH6J.swiftmodule
        │   ├── Dispatch-IHOPE9XROUN0.swiftmodule
        │   ├── F655JRRZ38DK
        │   │   ├── AppKit-1LWHB1MWS5AWP.pcm
        │   │   ├── AppKit-1LWHB1MWS5AWP.pcm.timestamp
        │   │   ├── ApplicationServices-XV6KNRUFW5PV.pcm
        │   │   ├── ApplicationServices-XV6KNRUFW5PV.pcm.timestamp
        │   │   ├── CFNetwork-WOPYBVYTVV3P.pcm
        │   │   ├── CFNetwork-WOPYBVYTVV3P.pcm.timestamp
        │   │   ├── CloudKit-2WJEQIT331HGH.pcm
        │   │   ├── CloudKit-2WJEQIT331HGH.pcm.timestamp
        │   │   ├── ColorSync-3E49XPWC4LN6N.pcm
        │   │   ├── ColorSync-3E49XPWC4LN6N.pcm.timestamp
        │   │   ├── CoreData-1DHIL9VVBSYDQ.pcm
        │   │   ├── CoreData-1DHIL9VVBSYDQ.pcm.timestamp
        │   │   ├── CoreFoundation-RZX25862PY17.pcm
        │   │   ├── CoreFoundation-RZX25862PY17.pcm.timestamp
        │   │   ├── CoreGraphics-MC4FPA2MN9QR.pcm
        │   │   ├── CoreGraphics-MC4FPA2MN9QR.pcm.timestamp
        │   │   ├── CoreImage-2H41H5QLPV8GG.pcm
        │   │   ├── CoreImage-2H41H5QLPV8GG.pcm.timestamp
        │   │   ├── CoreLocation-L7TM93RJXN1K.pcm
        │   │   ├── CoreLocation-L7TM93RJXN1K.pcm.timestamp
        │   │   ├── CoreServices-199P4UTAUTZ7C.pcm
        │   │   ├── CoreServices-199P4UTAUTZ7C.pcm.timestamp
        │   │   ├── CoreText-4BX5WGXMXE7W.pcm
        │   │   ├── CoreText-4BX5WGXMXE7W.pcm.timestamp
        │   │   ├── CoreVideo-J9HZM7HTOIXH.pcm
        │   │   ├── CoreVideo-J9HZM7HTOIXH.pcm.timestamp
        │   │   ├── Darwin-1IVCWVLR6MT9T.pcm
        │   │   ├── Darwin-1IVCWVLR6MT9T.pcm.timestamp
        │   │   ├── DiskArbitration-1YS8ZT252YZXQ.pcm
        │   │   ├── DiskArbitration-1YS8ZT252YZXQ.pcm.timestamp
        │   │   ├── Dispatch-2M9AOUJY3TW9V.pcm
        │   │   ├── Dispatch-2M9AOUJY3TW9V.pcm.timestamp
        │   │   ├── Foundation-2FJBXN8U6QRTS.pcm
        │   │   ├── Foundation-2FJBXN8U6QRTS.pcm.timestamp
        │   │   ├── IOKit-PMP3SCKMT7TH.pcm
        │   │   ├── IOKit-PMP3SCKMT7TH.pcm.timestamp
        │   │   ├── IOSurface-GCVS5NOA7EOF.pcm
        │   │   ├── IOSurface-GCVS5NOA7EOF.pcm.timestamp
        │   │   ├── ImageIO-3MTK4VGYXNJP8.pcm
        │   │   ├── ImageIO-3MTK4VGYXNJP8.pcm.timestamp
        │   │   ├── MachO-194CE0O93KDBK.pcm
        │   │   ├── MachO-194CE0O93KDBK.pcm.timestamp
        │   │   ├── Metal-LZGKTAAK5A7P.pcm
        │   │   ├── Metal-LZGKTAAK5A7P.pcm.timestamp
        │   │   ├── ObjectiveC-1A3ZNHZR9RRLF.pcm
        │   │   ├── ObjectiveC-1A3ZNHZR9RRLF.pcm.timestamp
        │   │   ├── OpenGL-2SRSU83KOCQ0D.pcm
        │   │   ├── OpenGL-2SRSU83KOCQ0D.pcm.timestamp
        │   │   ├── QuartzCore-1LT8XAPDT2LBF.pcm
        │   │   ├── QuartzCore-1LT8XAPDT2LBF.pcm.timestamp
        │   │   ├── Security-2RB2R7QDU33DQ.pcm
        │   │   ├── Security-2RB2R7QDU33DQ.pcm.timestamp
        │   │   ├── SwiftOverlayShims-2DA6NLEWJC11R.pcm
        │   │   ├── SwiftOverlayShims-2DA6NLEWJC11R.pcm.timestamp
        │   │   ├── SwiftShims-2DA6NLEWJC11R.pcm
        │   │   ├── SwiftShims-2DA6NLEWJC11R.pcm.timestamp
        │   │   ├── XCTest-2NCTUXJWAFHPE.pcm
        │   │   ├── XCTest-2NCTUXJWAFHPE.pcm.timestamp
        │   │   ├── XPC-3LIAJY90PG03M.pcm
        │   │   ├── XPC-3LIAJY90PG03M.pcm.timestamp
        │   │   ├── _Builtin_intrinsics-3UL74VXOKN0LJ.pcm
        │   │   ├── _Builtin_intrinsics-3UL74VXOKN0LJ.pcm.timestamp
        │   │   ├── _Builtin_stddef_max_align_t-3UL74VXOKN0LJ.pcm
        │   │   ├── _Builtin_stddef_max_align_t-3UL74VXOKN0LJ.pcm.timestamp
        │   │   ├── launch-1IVCWVLR6MT9T.pcm
        │   │   ├── launch-1IVCWVLR6MT9T.pcm.timestamp
        │   │   ├── libkern-1IVCWVLR6MT9T.pcm
        │   │   ├── libkern-1IVCWVLR6MT9T.pcm.timestamp
        │   │   ├── os_object-1IVCWVLR6MT9T.pcm
        │   │   ├── os_object-1IVCWVLR6MT9T.pcm.timestamp
        │   │   ├── os_workgroup-1IVCWVLR6MT9T.pcm
        │   │   ├── os_workgroup-1IVCWVLR6MT9T.pcm.timestamp
        │   │   ├── ptrauth-3UL74VXOKN0LJ.pcm
        │   │   └── ptrauth-3UL74VXOKN0LJ.pcm.timestamp
        │   ├── Foundation-3J9R2BG9D4PM.swiftmodule
        │   ├── IOKit-GYJ6NOVDK08V.swiftmodule
        │   ├── Metal-3E3C1ZH1F85NS.swiftmodule
        │   ├── ObjectiveC-12RVOEHAV272N.swiftmodule
        │   ├── QuartzCore-1ZAPSYNBC9FNF.swiftmodule
        │   ├── Swift-2RFHNMR9ZKBFU.swiftmodule
        │   ├── SwiftOnoneSupport-3F3T2J7XMABH4.swiftmodule
        │   ├── XPC-2O19RN3ATZY6H.swiftmodule
        │   └── modules.timestamp
        ├── description.json
        ├── greet
        ├── greet.build
        │   ├── greet.swiftdoc
        │   ├── greet.swiftmodule
        │   ├── greet.swiftsourceinfo
        │   ├── main.d
        │   ├── main.swift.o
        │   ├── main.swiftdeps
        │   ├── main~partial.swiftdoc
        │   ├── main~partial.swiftmodule
        │   ├── main~partial.swiftsourceinfo
        │   ├── master.swiftdeps
        │   └── output-file-map.json
        ├── greet.product
        │   └── Objects.LinkFileList
        ├── greetPackageTests.product
        │   └── Objects.LinkFileList
        └── index
            ├── db
            │   └── v13
            │       └── p2901--fdfc5d
            │           ├── data.mdb
            │           └── lock.mdb
            └── store
                └── v5
                    ├── records
                    │   ├── 05
                    │   │   └── main.swift-2DELS97E22C05
                    │   ├── 0B
                    │   │   └── x86_64-apple-macos.swiftinterface_Playground-2BDEY5WTUT30B
                    │   ├── 2M
                    │   │   └── x86_64-apple-macos.swiftinterface_String-2UUUVO9HJ1P2M
                    │   ├── 2R
                    │   │   └── x86_64-apple-macos.swiftinterface_Hashing-1HGEMNXLU3D2R
                    │   ├── 2V
                    │   │   └── x86_64-apple-macos.swiftinterface_Collection-2UGGKO436Z12V
                    │   ├── 3S
                    │   │   └── x86_64-apple-macos.swiftinterface_Assert-V65YNMTP6C3S
                    │   ├── 7B
                    │   │   └── x86_64-apple-macos.swiftinterface_Collection_Lazy_Views-3JAGWJWUSOZ7B
                    │   ├── 9I
                    │   │   └── x86_64-apple-macos.swiftinterface_Misc-5INJHXUAMQ9I
                    │   ├── B0
                    │   │   └── x86_64-apple-macos.swiftinterface-3G6V8LBMQNRB0
                    │   ├── BJ
                    │   │   └── x86_64-apple-macos.swiftinterface_Collection_Type-erased-2GDE9N2H0B8BJ
                    │   ├── GR
                    │   │   └── x86_64-apple-macos.swiftinterface_C-3FECARLS9UGR
                    │   ├── L7
                    │   │   └── x86_64-apple-macos.swiftinterface_Pointer-38N9GLOANQ8L7
                    │   ├── MD
                    │   │   └── x86_64-apple-macos.swiftinterface_Collection_HashedCollections-2UH3II70A2LMD
                    │   ├── MN
                    │   │   └── x86_64-apple-macos.swiftinterface_Math_Vector-PJN9ZL0MXYMN
                    │   ├── PD
                    │   │   └── x86_64-apple-macos.swiftinterface_Math_Floating-3L3PLMQZFG8PD
                    │   ├── PP
                    │   │   └── x86_64-apple-macos.swiftinterface_Bool-1AFBUWMD84EPP
                    │   ├── PT
                    │   │   └── x86_64-apple-macos.swiftinterface_Reflection-2N5N7DVE9TKPT
                    │   ├── PZ
                    │   │   └── x86_64-apple-macos.swiftinterface_KeyPaths-H0WDHRNCUMPZ
                    │   ├── S1
                    │   │   └── x86_64-apple-macos.swiftinterface_Math-1AQNC3D9CLDS1
                    │   ├── SH
                    │   │   └── x86_64-apple-macos.swiftinterface_Collection_Array-241X93Y2MIDSH
                    │   ├── TX
                    │   │   └── x86_64-apple-macos.swiftinterface_Math_Integers-3ESHE1N3VZITX
                    │   ├── V2
                    │   │   └── x86_64-apple-macos.swiftinterface_Optional-1LDU2VVQQV8V2
                    │   ├── VA
                    │   │   └── x86_64-apple-macos.swiftinterface_Result-EF33XPFC6OVA
                    │   └── WP
                    │       └── x86_64-apple-macos.swiftinterface_Protocols-22Z80D2N4Q7WP
                    └── units
                        ├── main.swift.o-3F2FGE5STP6RF
                        └── x86_64-apple-macos.swiftinterface-3760W9GA66NLA

45 directories, 146 files
```

빌드 결과물은 `debug` 폴더에 있다.

```
$ cd debug
$ l
total 176
drwxr-xr-x   9 burt  staff   288B  8 22 14:20 .
drwxr-xr-x   4 burt  staff   128B  8 22 14:20 ..
drwxr-x---  24 burt  staff   768B  8 22 14:20 ModuleCache
-rw-r--r--   1 burt  staff   5.7K  8 22 14:20 description.json
-rwxr-xr-x   1 burt  staff    76K  8 22 14:20 greet
drwxr-xr-x  13 burt  staff   416B  8 22 14:20 greet.build
drwxr-xr-x   3 burt  staff    96B  8 22 14:00 greet.product
drwxr-xr-x   3 burt  staff    96B  8 22 14:00 greetPackageTests.product
drwxr-x---   4 burt  staff   128B  8 22 14:00 index
```

`greet` 실행파일을 확인할 수 있다. 실행해 보면 아래와 같이 `Hello, world!`가 출력된다.

```
$ ./greet 
Hello, world!
```

### Release 빌드

`Release` 모드 빌드는 `-c release` 옵션을 붙여주면 된다.

```
$ swift build -c release 
[2/2] Linking greet

* Build Completed!
```

빌드 결과물은 `.build`폴더의 `release` 폴더에 담긴다.

```
$ cd .build/release
$ ./greet
Hello, world!
```

## 실행하기

`Debug` 및 `Release`로 빌드하면 빌드 결과물이 담긴 해당 폴더에서 실행 파일을 실행해야 결과를 확인할 수 있다. 매번 이렇게 결과를 확인하는 것은 번거롭다. 그래서 Swift 패키지 매니저는 바로 실행해 볼 수 있도록 하는 명령어를 제공한다.

```
$ swift run
[3/3] Linking greet

* Build Completed!
Hello, world!
```

`swift run`을 실행하면 먼저 빌드한 후 빌드 결과물을 실행해 주기 때문에 매번 실행 파일을 실행해야 하는 번거로움을 제거해 준다.

## 테스트하기

`swift test` 명령어로 테스트를 수행할 수 있다.

```
$ swift test
[3/3] Linking greetPackageTests

* Build Completed!
Test Suite 'All tests' started at 2021-08-22 14:33:28.473
Test Suite 'greetPackageTests.xctest' started at 2021-08-22 14:33:28.473
Test Suite 'greetTests' started at 2021-08-22 14:33:28.473
Test Case '-[greetTests.greetTests testExample]' started.
Test Case '-[greetTests.greetTests testExample]' passed (0.073 seconds).
Test Suite 'greetTests' passed at 2021-08-22 14:33:28.547.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.073 (0.073) seconds
Test Suite 'greetPackageTests.xctest' passed at 2021-08-22 14:33:28.547.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.073 (0.074) seconds
Test Suite 'All tests' passed at 2021-08-22 14:33:28.547.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.073 (0.074) seconds
```