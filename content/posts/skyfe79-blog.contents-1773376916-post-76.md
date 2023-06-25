---
title: "Swift 매크로"
date: 2023-06-25T16:57:33Z
author: "skyfe79"
draft: false
tags: ["swift","swift5.9"]
---

Swift 매크로는 컴파일 시 소스 코드를 변환하여 반복적인 코드 작성을 피하게 해줍니다. 컴파일 시 Swift는 매크로를 확장하여 일반적으로 코드를 빌드하기 전에 소스 코드에 포함된 매크로를 전부 확장합니다.

<img width="407" alt="660052f0a983c0ee4fc02416adaeb6de_MD5" src="https://github.com/skyfe79/blog.contents/assets/309935/cd8561f3-188f-4d6d-8b06-841cb6a85164">

매크로 확장은 항상 추가적인 연산입니다. 매크로는 기존의 코드를 삭제하거나 수정하지 않으며 새로운 코드를 추가합니다.

매크로의 입력과 출력은 항상 Swift 코드의 문법적 유효성을 확인합니다. 또한, 매크로에 전달하는 값과 매크로에서 생성되는 코드의 값은 올바른 자료형을 갖도록 확인됩니다. 또한, 매크로 구현에서 오류가 발생하면 컴파일러가 이를 컴파일 오류로 처리합니다. 이는 매크로를 사용하는 코드를 이해하는 데 도움을 주고, 잘못된 매크로 사용이나 버그가 있는 매크로 구현 등의 문제를 식별하기 쉽게 만듭니다.

Swift에는 두 가지 종류의 매크로가 있습니다.

- 독립형(Freestanding) 매크로: 선언과 상관없이 다른 코드에 포함될 수 있는 매크로입니다.
- 첨부형(Attached) 매크로: 해당 선언에 포함되는 매크로입니다.

첨부형(Attached) 매크로와 독립형(freestanding) 매크로를 사용하는 방법은 약간 다릅니다. 그러나 매크로 확장에 대한 모델은 동일하며, 두 종류의 매크로를 동일한 접근 방식으로 구현합니다. 다음 섹션에서는 두 가지 유형의 매크로에 대해 자세히 설명합니다.

## 독립형 매크로

독립형 매크로는 이름 앞에 `샵`(`#`)을 붙여서 호출할 수 있습니다. 매크로의 이름 뒤 괄호 안에 인자를 전달합니다. 다음은 예시 코드입니다.

```swift
func myFunction() {
    print("현재 실행 중인 함수는 \(#function)입니다.")
    #warning("문제가 있습니다.")
}
```

위의 코드에서 매크로 `#function`은 Swift 표준 라이브러리의 function 매크로를 호출합니다. 이 코드를 컴파일하면, Swift는 해당 매크로의 구현을 호출하여 `#function`을 현재 함수의 이름으로 대체합니다. 이 코드를 실행하고 myFunction()을 호출하면, "현재 실행 중인 함수는 myFunction()입니다."라고 출력됩니다. 두 번째 줄의 `#warning`은 커스텀 컴파일 경고를 생성하기 위해 Swift 표준 라이브러리의 `warning(_:)` 매크로를 호출합니다.

독립형 매크로는 `#function`과 같이 값을 생성할 수도 있으며, `#warning`과 같이 컴파일 시간에 동작을 수행할 수도 있습니다.

## 첨부형 매크로

첨부형 매크로는 이름 앞에 `at`(`@`)를 붙여서 호출할 수 있습니다. 매크로의 이름 뒤 괄호 안에 인자를 전달합니다.

첨부형 매크로는 호출된 선언을 수정합니다. 매크로가 새로운 메서드를 정의하거나 프로토콜을 구현하는 등의 코드를 추가합니다.

다음은 매크로를 사용하지 않은 예시 코드입니다.

```swift
struct SundaeToppings: OptionSet {
    let rawValue: Int
    static let nuts = SundaeToppings(rawValue: 1 << 0)
    static let cherry = SundaeToppings(rawValue: 1 << 1)
    static let fudge = SundaeToppings(rawValue: 1 << 2)
}
```

위의 코드에서 SundaeToppings 옵션 세트의 각 옵션에는 반복적이고 수동적으로 이니셜라이저를 호출하는 부분이 있습니다. 새로운 옵션을 추가할 때 잘못된 숫자를 입력하는 등의 실수를 할 수 있습니다.

다음은 매크로를 사용하여 위의 코드를 개선한 예시입니다.

```swift
@OptionSet<Int>
struct SundaeToppings {
    private enum Options: Int {
        case nuts
        case cherry
        case fudge
    }
}
```

위의 코드에서 SundaeToppings는 Swift 표준 라이브러리의 @OptionSet 매크로를 호출합니다. 이 매크로는 비공개 열거형의 각 케이스 목록을 읽고, 각 옵션에 대한 상수 목록을 생성하고, OptionSet 프로토콜에 대한 구현을 추가합니다.

비교를 위해 @OptionSet 매크로의 확장된 버전을 살펴보겠습니다. 이 코드는 직접 작성하지 않으며, 매크로의 확장을 표시하고 싶을 때만 볼 수 있습니다.

```swift
struct SundaeToppings {
    private enum Options: Int {
        case nuts
        case cherry
        case fudge
    }


    typealias RawValue = Int
    var rawValue: RawValue
    init() { self.rawValue = 0 }
    init(rawValue: RawValue) { self.rawValue = rawValue }
    static let nuts: Self = Self(rawValue: 1 << Options.nuts.rawValue)
    static let cherry: Self = Self(rawValue: 1 << Options.cherry.rawValue)
    static let fudge: Self = Self(rawValue: 1 << Options.fudge.rawValue)
}
extension SundaeToppings: OptionSet { }
```

비공개 열거형 이후의 모든 코드는 @OptionSet 매크로에서 생성됩니다. @OptionSet 매크로를 사용하여 생성된 모든 정적 변수를 포함한 SundaeToppings 버전은 이전에 수동으로 작성한 버전보다 읽기 쉽고 유지보수하기 쉽습니다.

## 매크로 선언

대부분의 Swift 코드에서는 함수 또는 타입과 같은 기호를 구현할 때 별도의 선언이 필요하지 않지만, 매크로의 경우 선언과 구현이 별도로 있습니다. 매크로의 선언에는 이름, 매개변수, 사용 가능한 위치 및 생성할 코드 유형이 포함됩니다. 매크로의 구현에는 Swift 코드를 생성하는 코드가 포함됩니다.

매크로 선언은 macro 키워드로 시작합니다. 다음은 이전 예시에서 사용한 @OptionSet 매크로의 일부 선언입니다.

```swift
public macro OptionSet<RawType>() =
    #externalMacro(module: "SwiftMacros", type: "OptionSetMacro")
```

첫 번째 줄은 매크로의 이름과 인자를 지정합니다. 이름은 OptionSet이며, 인자를 사용하지 않습니다. 두 번째 줄은 Swift 표준 라이브러리의 `externalMacro(module:type:)` 매크로를 사용하여 매크로의 구현 위치를 Swift에 알려줍니다. 이 경우 SwiftMacros 모듈에 OptionSetMacro라는 이름의 유형이 매크로를 구현합니다.

첨부형 매크로의 이름은 클래스와 구조체와 같은 이름과 동일한 upper camel case를 사용합니다. 독립형 매크로는 변수와 함수와 같은 이름과 동일한 lower camel case를 사용합니다.

> 참고: 매크로는 항상 public으로 선언됩니다. 매크로를 선언하는 코드가 매크로를 사용하는 코드와 다른 모듈에 있기 때문에 비공개 매크로를 적용할 수 있는 위치가 없습니다.

매크로 선언은 매크로의 역할과 관련된 정보를 제공합니다. 역할은 macro 선언의 속성에서 작성하는 위치입니다. 다음은 @OptionSet 선언의 일부 및 해당 역할에 대한 속성입니다.

```swift
@attached(member)
@attached(conformance)
public macro OptionSet<RawType>() =
    #externalMacro(module: "SwiftMacros", type: "OptionSetMacro")
```

위 선언에서 @attached 속성은 각 매크로 역할마다 한 번씩 나타납니다. 첫 번째 사용인 @attached(member)는 매크로가 적용된 타입에 새 멤버를 추가한다는 것을 나타냅니다. @OptionSet 매크로는 OptionSet 프로토콜에서 요구하는 `init(rawValue:)` 이니셜라이저와 멤버를 추가합니다. 두 번째 사용인 @attached(conformance)는 @OptionSet이 하나 이상의 프로토콜 적합성을 추가한다는 것을 나타냅니다. @OptionSet 매크로는 프로토콜 적합성을 추가하기 위해 매크로가 적용된 타입을 확장합니다.

독립형 매크로의 경우, 역할을 지정하기 위해 @freestanding 속성을 사용합니다.

```swift
@freestanding(expression)
public macro line<T: ExpressibleByIntegerLiteral>() -> T =
    /* 매크로 구현 위치... */
```

위의 `#line` 매크로는 expression 역할을 갖고 있습니다. expression 매크로는 값을 생성하거나 컴파일 시간에 워닝을 생성하는 등 컴파일 시간에 동작을 수행합니다.

매크로 선언은 생성되는 심볼의 이름에 대한 정보도 제공합니다. 매크로 선언이 심볼의 목록을 제공하면 해당 이름을 사용하는 선언만 생성되어 명시적으로 제공된 이름을 사용하여 생성된 코드를 이해하고 디버깅할 수 있습니다. 다음은 @OptionSet의 전체 선언입니다.

```swift
@attached(member, names: named(RawValue), named(rawValue),
    named(`init`), arbitrary)
@attached(conformance)
public macro OptionSet<RawType>() =
    #externalMacro(module: "SwiftMacros", type: "OptionSetMacro")
```

위 선언에서 @attached(member) 매크로는 매크로에서 생성되는 심볼 이름 뒤에 named: 레이블에 주어진 인자를 포함합니다. @OptionSet 매크로는 RawValue, rawValue, init이라는 이름의 심볼에 대한 선언을 추가합니다. 이러한 이름이 미리 알려져 있기 때문에 매크로 선언에서 명시적으로 나열됩니다.

매크로 선언은 또한 이름 목록 뒤에 arbitrary라는 임의의 이름을 포함하므로 매크로를 사용할 때까지 이름을 알 수 없는 선언을 생성할 수 있습니다. 예를 들어, @OptionSet 매크로가 SundaeToppings에 적용될 때, nuts, cherry, fudge와 같은 열거형 케이스에 해당하는 타입 프로퍼티를 생성합니다.

매크로 역할의 전체 목록 및 해당하는 SwiftSystem 프로토콜에 대한 자세한 내용은 [Attributes](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes)의 [attached](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes#attached) 및 [freestanding](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes#freestanding)을 참조하세요.

## 매크로 확장

매크로가 사용된 코드를 빌드할 때 컴파일러는 매크로의 구현을 호출하여 매크로를 확장합니다.

<img width="711" alt="b949f527d25f4eda090f9854bdb1070c_MD5" src="https://github.com/skyfe79/blog.contents/assets/309935/a9e14640-7ee0-4cd1-8f25-8dc75f67cbeb">

구체적으로, Swift는 다음과 같은 방식으로 매크로를 확장합니다.

1. 컴파일러는 코드를 읽어들여 구문의 내부 표현을 만들어냅니다.
2. 컴파일러는 일부 내부 표현을 매크로 구현에 전송하고, 매크로를 확장합니다.
3. 컴파일러는 매크로 호출을 해당 확장된 형태로 대체합니다.
4. 컴파일러는 확장된 소스 코드를 사용하여 계속해서 컴파일을 진행합니다.

구체적인 단계를 살펴보기 위해 다음 코드를 생각해 봅시다.

```swift
let magicNumber = #fourCharacterCode("ABCD")
```

위의 코드에서 `#fourCharacterCode` 매크로는 네 개의 글자로 이루어진 문자열을 받아 해당 문자열의 ASCII 값을 조합하여 32비트 부호 없는 정수로 반환합니다. 이와 같은 정수는 파일 형식과 같은 데이터를 식별하기 위해 사용됩니다. 이 값은 메모리 절약적이면서 디버거에서 볼 수 있는 가독성을 제공합니다. 매크로 구현 방법은 아래의 매크로 구현 섹션에서 설명하겠습니다.

위의 코드에서 매크로를 확장하기 위해 컴파일러는 Swift 파일을 읽고 해당 코드에 대한 AST(추상 구문 트리)의 인메모리 표현을 만듭니다. AST는 코드의 구조를 명시적으로 나타내어 해당 구조와 상호작용하는 코드(예: 컴파일러 또는 매크로 구현)를 작성하기 쉽게 만듭니다. 다음은 위의 코드의 약간 단순화된 AST 표현입니다.

<img width="309" alt="f47f5786bbc410ca6e177cbd5d97192d_MD5" src="https://github.com/skyfe79/blog.contents/assets/309935/469b7ac9-3e07-4101-b095-f79aefbae76d">

위의 다이어그램은 AST가 메모리에서 어떻게 표현되는지를 보여줍니다. AST의 각 요소는 소스 코드의 부분에 해당합니다. "Constant declaration" AST 요소는 두 개의 하위 요소를 가지며, 상수의 이름과 값 부분을 나타냅니다. "Macro call" 요소는 매크로의 이름과 매크로에 전달되는 인자 목록을 나타내는 하위 요소를 가지고 있습니다.

AST를 구성하는 과정 중에 컴파일러는 소스 코드가 유효한 Swift 코드인지 확인합니다. 예를 들어, `#fourCharacterCode`는 하나의 인자를 받으며, 그 인자는 문자열이어야 합니다. 정수 인자를 전달하려고 하거나 문자열 리터럴의 끝에 따옴표(")를 빼먹으면 여기서 오류가 발생합니다.

컴파일러는 소스 코드에서 매크로를 호출하는 위치를 찾고, 해당 매크로 구현을 포함하는 외부 이진 파일을 로드합니다. 각 매크로 호출마다 컴파일러는 AST의 일부를 해당 매크로 구현에 전달합니다. 다음은 해당 부분 AST를 나타내는 표현입니다.

<img width="249" alt="3473ff8bb26c5ebe092caa930bc80ade_MD5" src="https://github.com/skyfe79/blog.contents/assets/309935/36d540ef-b077-4d1e-9ffc-b22f6684b881">

위의 표현은 `#fourCharacterCode` 매크로의 구현이 매크로에 전달된 부분 AST로 읽힌다는 것을 보여줍니다. 매크로의 구현은 전달된 부분 AST만을 기반으로 작동합니다. 즉, 매크로는 앞뒤로 오는 코드와 상관없이 항상 동일한 방식으로 확장됩니다. 이 제한은 매크로 확장을 이해하기 쉽게 만들어주며, 매크로가 변경되지 않은 경우 Swift가 확장되지 않은 매크로를 피하여 코드 빌드를 더 빠르게 만듭니다.

Swift는 매크로 작성자가 다른 입력을 읽지 않도록 매크로를 구현하는 코드에 제한을 둡니다.

- 매크로 구현에 전달되는 AST에는 매크로를 나타내는 AST 요소만 포함됩니다. 다른 AST 요소는 전달되지 않습니다.
- 매크로 구현은 파일 시스템이나 네트워크에 액세스할 수 없는 격리된 환경에서 실행됩니다.

이러한 안전장치 외에도, 매크로의 작성자는 매크로의 입력 이외의 것을 읽거나 수정하지 않아야 합니다. 예를 들어, 매크로의 확장은 현재 시간과 같은 상황에 의존해서는 안 됩니다.

`#fourCharacterCode`의 구현은 확장된 코드를 포함하는 새로운 AST를 생성합니다. 아래는 해당 코드가 컴파일러에게 반환하는 내용입니다:

<img width="191" alt="8d0a4ff618b68f34bd9231673b6031cb_MD5" src="https://github.com/skyfe79/blog.contents/assets/309935/881a5ac4-e780-404f-884d-33ae74bfdcef">

컴파일러는 이 확장된 내용을 받으면, 매크로 호출을 포함하는 AST 요소를 매크로 확장 요소를 포함하는 요소로 대체합니다. 매크로 확장 후 컴파일러는 여전히 문법적으로 올바른 Swift 코드이고 모든 타입이 올바른지 다시 한 번 확인합니다. 이렇게 하면 일반적으로 컴파일할 수 있는 최종 AST가 생성됩니다.

<img width="227" alt="8d3126814047ae244aa0d9966b031a52_MD5" src="https://github.com/skyfe79/blog.contents/assets/309935/f2ffb988-dd35-4442-9102-2d1990dd271d">

이 AST는 다음과 같은 Swfit 코드에 해당합니다:

```swift
let magicNumber = 1145258561
```

이 예제에서 입력 소스 코드에는 하나의 매크로만 있지만, 실제 프로그램에서는 여러 매크로 인스턴스와 다른 매크로 호출이 여러개 있을 수 있습니다. 컴파일러는 매크로를 하나씩 확장합니다.

하나의 매크로가 다른 매크로 안에 포함되어 있는 경우, 외부 매크로가 먼저 확장됩니다. 이렇게 함으로써 내부 매크로가 확장되기 전에 외부 매크로에서 내부 매크로를 수정할 수 있습니다.

## 매크로 구현

매크로를 구현하려면 매크로의 확장을 수행하는 타입과 매크로를 선언하기 위한 라이브러리 두 가지 구성 요소가 필요합니다. 이 두 부분은 매크로 및 매크로 라이브러리의 빌드 과정 중에 별도로 구성됩니다. 같이 개발 중인 경우에도 매크로 및 해당 클라이언트를 별도의 컴파일 단계에서 실행되도록 만들어야 합니다.

Swift Package Manager를 사용하여 새로운 매크로를 생성하려면 `swift package init --type macro` 명령을 실행하면 됩니다. 이렇게 하면 매크로 구현과 선언에 대한 템플릿을 비롯한 여러 파일이 생성됩니다.

기존 프로젝트에 매크로를 추가하려면 매크로 구현을 위한 대상(target)과 매크로 라이브러리를 위한 대상(target)을 추가해야 합니다. 예를 들어, 다음과 비슷한 코드를 Package.swift 파일에 추가할 수 있습니다. 필요에 따라 이름을 변경하세요.

```swift
targets: [
    // Source transformations performed by the macro implementation.
    .macro(
        name: "MyProjectMacros",
        dependencies: [
            .product(name: "SwiftSyntaxMacros", package: "swift-syntax"),
            .product(name: "SwiftCompilerPlugin", package: "swift-syntax")
        ]
    ),


    // Library exposing a macro as part of its API.
    .target(name: "MyProject", dependencies: ["MyProjectMacros"]),
]
```

위 코드는 두 가지 대상을 정의합니다. MyProjectMacros는 매크로의 구현을 포함하고, MyProject는 해당 매크로를 사용할 수 있게 됩니다.

매크로의 구현은 [SwiftSyntax](http://github.com/apple/swift-syntax/) 모듈을 사용하여 구문에 구조적인 방식으로 상호작용하기 때문에 SwiftSyntax에 종속성을 추가해야 합니다. Swift 패키지 관리자로 매크로 패키지를 생성한 경우Package.swift 파일에 이미 SwiftSyntax 패키지 종속성이 설정되어 있습니다. 기존 프로젝트에 매크로를 추가하는 경우 Package.swift 파일에서 SwiftSyntax 종속성을 추가해야 합니다.

```swift
dependencies: [
    .package(url: "https://github.com/apple/swift-syntax.git", from: "some-tag"),
],
```

위의 코드에서 some-tag 자리에 사용하려는 SwiftSyntax 버전의 Git 태그로 변경합니다.

매크로의 역할에 따라 매크로 구현이 준수해야 하는 SwiftSystem 프로토콜이 있습니다. 예를 들어, 이전 섹션에서 사용한 `#fourCharacterCode`를 생각해 봅시다. 다음은 해당 매크로를 구현하는 구조체입니다.

```swift
public struct FourCharacterCode: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        guard let argument = node.argumentList.first?.expression,
              let segments = argument.as(StringLiteralExprSyntax.self)?.segments,
              segments.count == 1,
              case .stringSegment(let literalSegment)? = segments.first
        else {
            throw CustomError.message("정적 문자열이 필요합니다.")
        }


        let string = literalSegment.content.text
        guard let result = fourCharacterCode(for: string) else {
            throw CustomError.message("잘못된 네 글자 코드입니다.")
        }


        return "\(raw: result)"
    }
}


private func fourCharacterCode(for characters: String) -> UInt32? {
    guard characters.count == 4 else { return nil }


    var result: UInt32 = 0
    for character in characters {
        result = result << 8
        guard let asciiValue = character.asciiValue else { return nil }
        result += UInt32(asciiValue)
    }
    return result.bigEndian
}
enum CustomError: Error { case message(String) }
```

`#fourCharacterCode` 매크로는 표현식을 생성하는 독립형 매크로이므로, FourCharacterCode 타입은 ExpressionMacro 프로토콜을 준수합니다. ExpressionMacro 프로토콜은 AST를 확장하는 `expansion(of:in:)` 메서드를 요구합니다. [첨부형 매크로(attached)](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes#attached) 및 [독립형 매크로(freestanding)](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes#freestanding)에서 매크로 역할과 해당하는 SwiftSystem 프로토콜의 전체 목록을 보려면 [Attributes](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/attributes)를 참조하세요.

`#fourCharacterCode` 매크로를 확장하기 위해 Swift는 해당 매크로 구현을 포함하는 라이브러리에 소스 코드에 대한 AST를 보냅니다. 라이브러리 내부에서 Swift는 `FourCharacterCode.expansion(of:in:)`를 호출하며, AST와 컨텍스트를 메서드 인자로 전달합니다. `expansion(of:in:)`의 구현은 `#fourCharacterCode` 매크로 호출에 전달된 AST에서 문자열을 추출하고 해당 문자열의 정수 리터럴 값을 계산합니다.

앞의 예시에서 첫 번째 가드 블록은 AST에서 문자열 리터럴을 추출하고 해당 AST 요소를 literalSegment에 할당합니다. 두 번째 가드 블록은  `fourCharacterCode(for:)` private 함수를 호출합니다. 이러한 블록은 매크로를 잘못 사용한 경우 에러를 던집니다. 예를 들어, `#fourCharacterCode("AB" + "CD")`와 같이 매크로를 호출하면 "정적 문자열이 필요합니다"라는 에러가 표시됩니다.

`expansion(of:in:)` 메서드는 ExprSyntax의 인스턴스를 반환합니다. 이는 SwiftSyntax에서 표현식을 나타내는 유형으로 StringLiteralConvertible 프로토콜을 준수합니다. 따라서 매크로 구현은 가벼운 문법으로 문자열 리터럴을 사용하여 결과를 생성합니다. 매크로에서 반환하는 SwiftSyntax 유형은 모두 StringLiteralConvertible 프로토콜을 준수하므로 모든 종류의 매크로를 구현할 때 이 접근 방식을 사용할 수 있습니다.

## 매크로 개발 및 디버깅

매크로는 테스트를 사용하여 개발하기에 적합한 기능입니다. 매크로는 외부 상태에 의존하지 않으며, 외부 상태에 변경을 가하지 않으면서 하나의 AST를 다른 AST로 변환하기 때문입니다. 또한 문자열 리터럴로부터 구문 노드를 생성할 수 있으므로 테스트 입력을 설정하는 것을 단순화합니다. 또한 AST의 description 속성을 읽어서 예상 값과 비교하는 데 사용할 수 있습니다. 다음은 이전 섹션에서 설명한 `#fourCharacterCode` 매크로의 테스트 예시입니다.

```swift
let source: SourceFileSyntax =
    """
    let abcd = #fourCharacterCode("ABCD")
    """


let file = BasicMacroExpansionContext.KnownSourceFile(
    moduleName: "MyModule",
    fullFilePath: "test.swift"
)


let context = BasicMacroExpansionContext(sourceFiles: [source: file])


let transformedSF = source.expand(
    macros: ["fourCharacterCode": FourCharacterCode.self],
    in: context
)


let expectedDescription =
    """
    let abcd = 1145258561
    """


precondition(transformedSF.description == expectedDescription)
```

위의 예시에서는 precondition을 사용하여 매크로를 테스트하지만, 테스트 프레임워크를 사용할 수도 있습니다.

