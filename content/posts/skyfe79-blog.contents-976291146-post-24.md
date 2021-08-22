---
title: "Swift로 Command Line 앱 만들기 #2"
date: 2021-08-22T05:37:10Z
draft: false
tags: ["swift"]
---

[Swift로 Command Line 앱 만들기 #1]() 글에서 배운 내용을 바탕으로 간단한 앱을 만들어 보자. 다음 두 가지 명령어를 지원하는 `random` 앱을 만들 것이다.

-  숫자를 입력하면 1부터 입력한 숫자 범위 내에서 임의로 숫자를 출력하는 명령어
-  입력한 문자열 리스트 중에서 n개를 임의로 선택해 출력하는 명령어


## 프로젝트 생성

`random` 이름으로 프로젝트를 생성한다.

```
$ mkdir random && cd random
$ swift package init --type executable
Creating executable package: random
Creating Package.swift
Creating README.md
Creating .gitignore
Creating Sources/
Creating Sources/random/main.swift
Creating Tests/
Creating Tests/randomTests/
Creating Tests/randomTests/randomTests.swift
```

## 디펜던시 추가

`random` 프로젝트는 명령어 파싱을 위해서 [swift-argument-parser](https://github.com/apple/swift-argument-parser) 패키지를 사용할 것이다. `Package.swift` 파일을 열고 아래와 같이 패키지 정보와 디펜던시 정보를 입력한다.

```swift
// swift-tools-version:5.3
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "random",
    dependencies: [
        // Dependencies declare other packages that this package depends on.
        .package(url: "https://github.com/apple/swift-argument-parser", from: "0.4.4"),
    ],
    targets: [
        // Targets are the basic building blocks of a package. A target can define a module or a test suite.
        // Targets can depend on other targets in this package, and on products in packages this package depends on.
        .target(
            name: "random",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
            ]),
        .testTarget(
            name: "randomTests",
            dependencies: ["random"]),
    ]
)
```

### 패키지 설치

디펜던시로 추가한 외부 패키지를 설치하기 위해서 빌드한다.

```
$ swift build
Fetching https://github.com/apple/swift-argument-parser from cache
Cloning https://github.com/apple/swift-argument-parser
Resolving https://github.com/apple/swift-argument-parser at 0.4.4
[39/39] Linking random

* Build Completed!
```

## main.swift 구현

`main.swift` 파일을 열고 원하는 구현을 시작한다. [swift-argument-parser](https://github.com/apple/swift-argument-parser)는 `ParsableCommand` 프로토콜을 구현하는 타입을 명령어로 판단한다. 디자인 패턴 중 [Command 패턴](https://en.wikipedia.org/wiki/Command_pattern)을 따른다고 생각하면 쉽다. 

```swift
/// A type that can be executed as part of a nested tree of commands.
public protocol ParsableCommand: ParsableArguments {
  /// Configuration for this command, including subcommands and custom help
  /// text.
  static var configuration: CommandConfiguration { get }
  
  /// *For internal use only:* The name for the command on the command line.
  ///
  /// This is generated from the configuration, if given, or from the type
  /// name if not. This is a customization point so that a WrappedParsable
  /// can pass through the wrapped type's name.
  static var _commandName: String { get }
  
  /// Runs this command.
  ///
  /// After implementing this method, you can run your command-line
  /// application by calling the static `main()` method on the root type.
  /// This method has a default implementation that prints help text
  /// for this command.
  mutating func run() throws
}
```

[Command 패턴](https://en.wikipedia.org/wiki/Command_pattern)의 `execute()` 메서드가 `ParsableCommand` 프로토콜의 `run()` 메서드라고 생각하면 된다.

### random 명령어 구현

우선 random 명령어를 구현해 보자. `ParsableCommand` 프로토콜을 따르는 `Random` 구조체를 생성한다.

```swift
struct Random: ParsableCommand {
  
}
```

`ParsableCommand` 프로토콜을 준수하기 위해서는 `run()` 메서드를 구현해야 한다.

```swift
struct Random: ParsableCommand {
  func run() throws {
  
  }
}

Random.main()
```

`Random.main()`은 일종의 프로그램 시작점이라고 볼 수 있다. `run()` 메서드를 실행해 준다. 

```swift
extension ParsableCommand {
  ...
  ...
  public static func main(_ arguments: [String]?) {
    do {
      var command = try parseAsRoot(arguments)
      try command.run()
    } catch {
      exit(withError: error)
    }
  }

  public static func main() {
    self.main(nil)
  }
}
```


위 상태에서 실행해 보자.

```
$ swift run random
```

`run()` 함수에 아무 내용도 없어 아무 것도 출력되지 않는다. 그렇다면 아래처럼 실행해 보자.

```
$ swift run random --help
USAGE: random

OPTIONS:
  -h, --help              Show help information.
```

도움말이 출력되는 것을 확인할 수 있다. `ParsableCommand` 프로토콜을 따르면 `ParsableCommand` 확장에 의해서 기본적으로 제공되는 기능이다.


위 도움말에 명령어를 소개하는 글을 추가해 보자. 명령어에 구성(CommandConfiguration)을 적용하여 설명을 추가할 수 있다. 

```swift
struct Random: ParsableCommand {
  static let configuration = CommandConfiguration(abstract: "1부터 입력한 숫자 범위에서 임의 수를 출력합니다.")
  func run() throws {
  
  }
}
```

```
$ swift run random --help
OVERVIEW: 1부터 입력한 숫자 범위에서 임의 수를 출력합니다.

USAGE: random

OPTIONS:
  -h, --help              Show help information.
```

`CommandConfiguration`는 다양한 기능을 제공하는데 자세한 내용은 [swift-argument-parser문서](https://github.com/apple/swift-argument-parser/tree/main/Documentation)를 참고한다.

이제 `run()` 메서드를 구현해 보자. 임의의 숫자를 출력하기 위해서는 사용자로부터 숫자 n을 받아야 한다. 그래야 [1...10] 범위에서 임의 숫자를 출력할 수 있다. 이를 위해서 `@Argument` 속성 랩퍼를 사용하여 인자를 선언한다.

```swift
struct Random: ParsableCommand {
  static let configuration = CommandConfiguration(abstract: "1부터 입력한 숫자 범위에서 임의 수를 출력합니다.")

  @Argument(help: "1 이상 숫자를 입력하세요.")
  var n: Int = 1

  func run() throws {
    print(n)
  }
}
```

`@Argument` 속성 래퍼에 도움말을 설정할 수 있다. 기본 값은 속성 초기값이 된다.

다시 도움말을 실행해 보자.

```
$ swift run random --help
OVERVIEW: 1부터 입력한 숫자 범위에서 임의 수를 출력합니다.

USAGE: random [<n>]

ARGUMENTS:
  <n>                     1 이상 숫자를 입력하세요. (default: 1)

OPTIONS:
  -h, --help              Show help information.
```

이제 n을 주어 실행해 보자.

```
$ swift run random
1
```

`n`값이 없으면 기본값 1이 출력됨을 확인할 수 있다. n값에 10을 입력해 보자.

```
$ swift run random 10
10
```

이제 `run()` 메서드를 구현해 보자.

```swift
import ArgumentParser

struct Random: ParsableCommand {
  static let configuration = CommandConfiguration(abstract: "1부터 입력한 숫자 범위에서 임의 수를 출력합니다.")

  @Argument(help: "1 이상 숫자를 입력하세요.")
  var n: Int = 1

  func run() throws {
    let result = Int.random(in: 1...n)
    print(result)
  }
}

Random.main()
```

실행해 보자.

```
$ swift run random 100
58
$ swift run random 100
99
```

원하는 대로 잘 실행되는 것을 확인할 수 있다. 만약 사용자가 1 미만의 숫자를 입력할 때는 어떻게 될까? 이를 미리 막을 수 있을까?  `ParsableCommand`는 `validate()` 메서드로 인자의 유효성을 검사할 수 있도록 지원하고 있다.

```swift
import ArgumentParser

struct Random: ParsableCommand {
  static let configuration = CommandConfiguration(abstract: "1부터 입력한 숫자 범위에서 임의 수를 출력합니다.")

  @Argument(help: "1 이상 숫자를 입력하세요.")
  var n: Int = 1

  func validate() throws {
    guard n >= 1 else {
      throw ValidationError("1 이상 숫자를 입력하셔야 합니다.")
    }
  }

  func run() throws {
    let result = Int.random(in: 1...n)
    print(result)
  }
}

Random.main()
```

0을 주어 입력해 보자.

```
$ swift run random 0
Error: 1 이상 숫자를 입력하셔야 합니다.
Usage: random [<n>]
  See 'random --help' for more information.
```

`validate()` 메서드에 의해서 오류로 처리되고 오류 문구가 출력된 후 실행을 멈추는 것을 확인할 수 있다.

## 여러 개의 명령어 만들기

여러 개의 명령어를 만들고 싶을 때는 어떻게 할까? [swift-argument-parser](https://github.com/apple/swift-argument-parser)는 서브커맨드라는 개념으로 이를 지원한다. 

`CommandConfiguration`에는 `subcommands`에 `ParsableCommand` 배열을 담을 수 있다. 아래와 같이 Random을 변경해 보자.

```swift
struct Random: ParsableCommand {
  static let configuration = CommandConfiguration(abstract: "1부터 입력한 숫자 범위에서 임의 수를 출력합니다.", subcommands: [
    Number.self,
    Pick.self
  ])
}
```

하위 명령어로 Number와 Pick을 등록했다. 주로 Random의 확장으로 하위 명령어를 정의한다. 기존에 정의했던 `Random` 의 내용을 `struct Number`로 옮겨 보자.

```swift
extension Random {
  struct Number: ParsableCommand {
    static let configuration = CommandConfiguration(commandName: "number", abstract: "1부터 입력한 숫자 범위에서 임의 수를 출력합니다.")

    @Argument()
    var n: Int

    func validate() throws {
      guard n >= 1 else {
        throw ValidationError("1 이상 숫자를 입력하셔야 합니다.")
      }
    }

    func run() throws {
      print(Int.random(in: 1...n))
    }
  }
}
```



추가로 Pick 명령어도 구현해 보자.

```swift
extension Random {
  // > random pick --count 3 A B C D E F G H I
  struct Pick: ParsableCommand {
    static let configuration = CommandConfiguration(commandName: "pick", abstract: "입력한 리스트에서 임의 원소를 count개 뽑아 출력합니다.")
    
    @Option(help: "선택할 원소의 갯수")
    var count: Int = 1

    @Argument
    var elements: [String]

    func validate() throws {
      guard !elements.isEmpty else {
        throw ValidationError("최소 1개의 원소를 입력해야 합니다.")
      }
    }

    func run() throws {
      let picks = elements.shuffled().prefix(count)
      print(picks.joined(separator: "\n"))
    }
  }
}
```

이제 실행해 보자.

```
$ swift run random --help
OVERVIEW: 1부터 입력한 숫자 범위에서 임의 수를 출력합니다.

USAGE: random <subcommand>

OPTIONS:
  -h, --help              Show help information.

SUBCOMMANDS:
  number                  1부터 입력한 숫자 범위에서 임의 수를 출력합니다.
  pick                    입력한 리스트에서 임의 원소를 count개 뽑아 출력합니다.

  See 'random help <subcommand>' for detailed help.
```

하위 명령어 도움말도 따로 볼 수 있다.

```
$ swift run random help pick
OVERVIEW: 입력한 리스트에서 임의 원소를 count개 뽑아 출력합니다.

USAGE: random pick [--count <count>] [<elements> ...]

ARGUMENTS:
  <elements>

OPTIONS:
  --count <count>         선택할 원소의 갯수 (default: 1)
  -h, --help              Show help information.
```

```
$ swift run random number 100
77
```

```
$ swift run random pick --count 3 A B C D E F G H I
G
F
B
```

```
$ swift run random pick A B C
C
```

[swift-argument-parser](https://github.com/apple/swift-argument-parser) 패키지를 사용해 복잡한 명령어와 인자를 다룰 수 있는 커맨드라인 앱을 쉽게 만들 수 있다.

