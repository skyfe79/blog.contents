---
title: "Swift 기본 연산자"
date: 2023-06-29T14:02:36Z
author: "skyfe79"
draft: false
tags: ["swift","swift5.9"]
---

연산자는 값을 확인, 변경 또는 결합하는 데 사용하는 특수 기호 또는 구문입니다. 예를 들어, 덧셈 연산자 (+)는 두 숫자를 더합니다. let i = 1 + 2 와 같이 사용할 수 있으며, 논리 AND 연산자 (&&)는 두 부울 값을 결합합니다. enteredDoorCode && passedRetinaScan와 같이 사용할 수 있습니다.

Swift는 C와 같은 언어에서 이미 알고 있을 수 있는 연산자를 지원하며, 일반적인 코딩 오류를 제거하기 위해 여러 기능을 개선했습니다. 할당 연산자(`=`)는 값을 반환하지 않아서, 등호 연산자(`==`)를 사용할 자리에 실수로 할당 연산자(`=`) 사용하지 않도록 방지합니다. 산술 연산자(`+,-,*,/,%, ...`)는 값의 오버플로를 검사하고 허용범위를 초과하는 숫자를 사용할 때 예기치 않은 결과를 방지하기 위해 값을 거부합니다. Swift에서 제공하는 오버플로 연산자를 사용하여 값의오버플로 동작을 선택할 수 있습니다. 이에 대한 자세한 내용은 [Overflow Operators](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/advancedoperators#Overflow-Operators)를 참고하십시오.

또한, `a..<b` 및 `a...b`와 같이 C에서 찾을 수 없는 범위 연산자를 제공하여 값의 범위를 표현할 수 있습니다.

이 장에서는 Swift에서 일반적으로 사용하는 연산자를 설명합니다. 고급 연산자는 Swift의 고급 연산자를 다루며, 자신만의 사용자 정의 연산자를 정의하고 사용자 정의 타입을 위한 표준 연산자를 구현하는 방법을 설명합니다.

## 용어

연산자는 단항(unary), 이항(binary) 또는 삼항(ternary)입니다.

- 단항 연산자는 하나의 대상에 연산을 적용합니다 (예 : `-a`). 단항 접두사 연산자는 대상 바로 앞에 나타납니다 (예 : `!b`) 및 단항 후위 연산자는 대상 바로 뒤에 나타납니다 (예 : `c!`).
- 이항 연산자는 두 대상(예: `2 + 3`)에 연산을 적용하며, 이들 두 대상 사이에 나타나므로 중위(infix) 연산자입니다.
- 삼항 연산자는 세 개의 대상에 연산을 적용하며. C처럼 Swift에는 삼항 조건 연산자 (`a ? b : c`)가 하나뿐입니다.

연산자가 영향을 미치는 값을 피연산자라고 합니다.. 식 `1 + 2`에서 `+` 기호는 중위 연산자이며 1과 2가 피연산자입니다.

## 할당 연산자

할당 연산자(`a = b`)는 a를 b의 값으로 초기화하거나 업데이트합니다.

```swift
let b = 10
var a = 5
a = b
// a는 이제 10입니다.
```

할당 연산의 오른쪽에 여러 값으로 구성된 튜플이 있는 경우, 요소를 한꺼번에 여러 개의 상수 또는 변수로 분해할 수 있습니다.

```swift
let (x, y) = (1, 2)
// x는 1이고 y는 2입니다.
```

C와 Objective-C의 할당 연산자와는 달리, Swift의 할당 연산자는 값 자체를 반환하지 않습니다. 다음 문은 유효하지 않습니다.

```swift
if x = y {
    // 이 문은 유효하지 않습니다. x = y는 값을 반환하지 않기 때문입니다.
}
```

이 기능은 등호 연산자(`==`)를 사용할 자리에 실수로 할당 연산자(`=`)를 사용하는 것을 방지하여 코드 오류를 피할 수 있도록 도와줍니다.

## 산술 연산자

Swift는 모든 숫자 유형에 대해 네 가지 표준 산술 연산자를 지원합니다.

- 덧셈 (`+`)
- 뺄셈 (`-`)
* 곱셈 (`*`)
- 나눗셈 (`/`)
```swift
1 + 2       // 3
5 - 3       // 2
2 * 3       // 6
10.0 / 2.5  // 4.0
```

C와 Objective-C의 산술 연산자와는 다르게 Swift의 산술 연산자는 기본적으로 값의 오버플로를 허용하지 않습니다. Swift의 오버플로 연산자(예 : `a &+ b`)를 사용하여 값의 오버플로를 허용할 수 있습니다. [오버플로 연산자](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/advancedoperators#Overflow-Operators)를 참조하세요.

또한 덧셈 연산자는 문자열 연결을 지원합니다.

```swift
"hello, " + "world"  // "hello, world"
```

### 나머지 연산자

나머지 연산자(`a % b`)는 b가 a안에 몇 번 들어갈 수 있는지 계산하고 남은 값을 반환합니다(우린 이 값을 나머지지라고 부릅니다).

> 참고
> 
> 나머지 연산자(%)는 다른 언어에서 모듈로 연산자로도 알려져 있습니다. 그러나 Swift에서 음수 값에 대한 동작 방식을 생각해보면, 엄밀히 말해 모듈로 연산이 아니라 나머지 연산입니다.

나머지 연산자가 작동하는 방식은 다음과 같습니다. `9 % 4`를 계산하려면, 먼저 9 안에 4가 몇 개 들어갈 수 있는지 계산합니다.


![ba4b4c97815a40752fa3d7501da4242f_MD5](https://github.com/skyfe79/blog.contents/assets/309935/dad3b89c-67cd-4073-83e2-7a65b042d3ec)

여기서 9 안에는 4가 2 개 들어가며, 남은 값은 1입니다(주황색으로 표시함).

Swift에서는 다음과 같이 작성합니다.

```swift
9 % 4    // 1을 반환
```

`a % b`의 답을 결정하기 위해, `%` 연산자는 다음 식을 계산하고 반환합니다.

```swift
a = (b x 일부 배수) + 나머지
```

여기서 일부 배수는 a 안에 들어갈 수 있는 b 배수의 갯수입니다.

이 식에 9와 4를 삽입하면 다음과 같습니다.

```swift
9 = (4 x 2) + 1
```

음수 a에 대해 나머지를 계산할 때도 같은 방법이 적용됩니다.

```swift
-9 % 4   // -1을 반환
```

이 식에 -9와 4를 삽입하면 다음과 같습니다.

```swift
-9 = (4 x -2) + -1
```

나머지 값은 -1입니다.

음수 b의 경우 b의 부호는 무시됩니다. 따라서 `a % b`와 `a % -b`는 항상 같은 답을 반환합니다.

### 단항 마이너스 연산자

숫자 값의 부호를 바꾸려면 접두사로 `-`를 사용하는 단항 마이너스 연산자를 사용할 수 있습니다.

```swift
let three = 3
let minusThree = -three       // minusThree는 -3과 같습니다.
let plusThree = -minusThree   // plusThree는 3 또는 "마이너스 마이너스 three"과 같습니다.
```

단항 마이너스 연산자(`-`)는 연산 대상 값 바로 앞에 공백 없이 놓입니다.

### 단항 더하기 연산자

단항 더하기 연산자인 (`+`)는 연산하는 값에 아무런 변경 없이 그대로 반환합니다:

```swift
let minusSix = -6
let alsoMinusSix = +minusSix  // alsoMinusSix는 -6과 같습니다.
```

단항 더하기 연산자는 실제로 아무런 작업을 수행하지 않지만, 음수에 대한 단항 마이너스 연산자와 함께 양수 숫자에 대한 대칭성을 제공하는 코드에서 사용할 수 있습니다.

## 복합 할당 연산자

C 언어와 마찬가지로, Swift는 할당(=)과 다른 작업을 결합한 복합 할당 연산자를 제공합니다. 그 중 하나는 덧셈 할당 연산자(`+=`)입니다:

```swift
var a = 1
a += 2
// a는 이제 3입니다
```

표현식 `a += 2`는 `a = a + 2`의 줄임말입니다. 이를 통해 덧셈과 할당이 동시에 수행되는 하나의 연산자로 결합됩니다.

> 참고
> 
> 복합 할당 연산자는 값을 반환하지 않습니다. 예를 들어, `let b = a += 2`와 같이 작성할 수 없습니다.

Swift 표준 라이브러리에서 제공하는 연산자에 대한 정보는 [연산자 선언](https://developer.apple.com/documentation/swift/operator_declarations)을 참조하십시오.

## 비교 연산자

Swift는 다음과 같은 비교 연산자를 지원합니다:

- `같다(==)`
- `다르다(!=)`
- `크다(>)`
- `작다(<)`
- `크거나 같다(>=)`
- `작거나 같다(<=)`

> 참고
> 
> Swift는 두 개의 Identity(동일) 연산자 (`===` 및 `!==`)를 제공합니다. 두 객체 참조가 동일한 객체 인스턴스를 가리키는지 테스트할 때 사용됩니다. 더 자세한 정보는 [Identity(동일) 연산자 문서](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures#Identity-Operators)를 참조하십시오. 참고로 Equality는 동등으로 번역합니다.

각 비교 연산자는 비교문이 참인지 아닌지를 나타내는 Bool 값을 반환합니다.

```Swift
1 == 1   // 1은 1과 같으므로 true가 반환됩니다
2 != 1   // 2는 1과 같지 않으므로 true가 반환됩니다
2 > 1    // 2는 1보다 크므로 true가 반환됩니다
1 < 2    // 1은 2보다 작으므로 true가 반환됩니다
1 >= 1   // 1은 1보다 크거나 같으므로 true가 반환됩니다
2 <= 1   // 2는 1보다 작거나 같지 않으므로 false가 반환됩니다
```

비교 연산자는 if 문과 같은 조건문에서 자주 사용됩니다.

```Swift
let name = "world"
if name == "world" {
    print("hello, world")
} else {
    print("I'm sorry \(name), but I don't recognize you")
}
// 출력: "hello, world" - name이 "world"와 동일하므로 true가 됩니다.
```

if 문에 대한 자세한 내용은 [제어문](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow)을 참조하십시오.

튜플은 동일한 타입과 같은 개수의 값이 있는 경우에만 비교할 수 있습니다. 튜플은 왼쪽에서 오른쪽으로 한 번에 하나씩 값이 비교되며, 두 개의 값이 동일하지 않을 때까지 계속 비교합니다. 이 두 값을 비교하고 그 비교 결과에 따라 튜플 비교의 전체 결과가 결정됩니다. 모든 요소가 동일하다면, 튜플 자체도 동일합니다. 예를 들어:

```Swift
(1, "zebra") < (2, "apple")   // 1은 2보다 작으므로 true이고, "zebra"과 "apple"은 비교되지 않습니다.
(3, "apple") < (3, "bird")    // 3은 3과 같고 "apple"은 "bird"보다 작으므로 true가 반환됩니다
(4, "dog") == (4, "dog")      // 4는 4와 같고 "dog"는 "dog"와 같으므로 true가 반환됩니다.
```

위 예제에서는 첫 번째 줄에서 왼쪽에서 오른쪽으로 비교하는 작동 방식을 볼 수 있습니다. "zebra"가 "apple"보다 작지 않더라도 비교는 튜플의 첫 번째 요소에 의해 이미 결정되었으므로 (1, "zebra")는 (2, "apple")보다 작게 됩니다. 그러나 튜플의 첫 번째 요소가 동일한 경우 두 번째 요소를 비교합니다. 이것이 두 번째와 세 번째 줄에서 발생하는 일입니다.

튜플을 비교하려면 해당 튜플의 각 값에 비교 연산자를 적용할 수 있는 경우에만 비교 연산자를 사용할 수 있습니다. 예를 들어 아래 코드에서처럼 두 개의 (String, Int) 유형의 튜플을 비교할 수 있습니다. 왜냐하면 String 및 Int 값 모두 `<` 연산자를 사용하여 비교할 수 있기 때문입니다. 그러나 (String, Bool) 유형의 두 튜플은 `<` 연산자를 사용하여 비교할 수 없습니다. 왜냐하면 `<` 연산자를 Bool 값에 적용할 수 없기 때문입니다.

```Swift
("blue", -1) < ("purple", 1)        // 결과: true
("blue", false) < ("purple", true)  // 불가능: <는 Boolean 값을 비교할 수 없습니다.
```

> 참고
> 
> Swift 표준 라이브러리에는 일곱 개 미만의 원소를 갖는 튜플을 비교하는 튜플 비교 연산자자 포함되어 있습니다. 일곱 개 이상의 원소를 갖는 튜플을 비교하려면 비교 연산자를 직접 구현해야합니다.

## 삼항 조건 연산자

삼항 조건 연산자는 `질문 ? 답1 : 답2` 형식을 가진 특수 연산자입니다. 이는 질문이 true 또는 false인지에 따라 두 표현식 중 하나를 평가하는 단축키 역할을 합니다. 만약 질문이 true라면, 답1을 평가하고 해당 값이 반환됩니다. 그렇지 않다면, 답2를 평가하고 해당 값이 반환됩니다.

삼항 조건 연산자는 아래와 같은 코드의 약식입니다:

```swift
if question {
    answer1
} else {
    answer2
}
```

다음은 삼항 조건 연산자를 사용한 예시입니다. 이 예시에서는 테이블 행의 높이를 계산합니다. 해당 행이 헤더를 가지고 있다면 콘텐츠 높이보다 50 포인트 더 높고, 헤더가 없다면 20 포인트만 더 높아야 합니다:

```swift
let contentHeight = 40
let hasHeader = true
let rowHeight = contentHeight + (hasHeader ? 50 : 20)
// rowHeight는 90과 같습니다.
```

위 코드를 if문으로 동일하게 작성해 보면 아래와 같습니다.:

```swift
let contentHeight = 40
let hasHeader = true
let rowHeight: Int
if hasHeader {
    rowHeight = contentHeight + 50
} else {
    rowHeight = contentHeight + 20
}
// rowHeight는 90과 같습니다.
```

첫번째 예제에서 사용된 삼항 조건 연산자는 코드 한 줄로 rowHeight 값을 올바르게 설정할 수 있어 두번째 예제 코드보다 간결합니다.

삼항 조건 연산자는 두 개의 표현식 중 어떤 것을 고려할지 결정하는데 효율적인 단축 방법을 제공합니다. 그러나 삼항 조건 연산자의 간결함은 남용될 경우 코드 읽기를 어렵게 만들 수 있으므로 주의해야 합니다. 특히, 여러 개의 삼항 조건 연산자를 하나의 복합문으로 결합하는 것을 피해야 합니다.

## Nil-Coalescing Operator

Nil 병합 연산자(`??`)는 옵셔널 a가 값이 있는 경우에는 a를 언랩핑(값풀기, unwrapping)하고, a가 nil인 경우에는 기본값 b를 반환합니다. 표현식 a는 항상 옵셔널 타입이어야 하며, 표현식 b는 a 안에 저장된 타입과 일치해야 합니다.

Nil 병합 연산자는 다음 코드를 간략화한 것입니다:

```swift
a != nil ? a! : b
```

위 코드는 삼항 조건 연산자와 강제 언랩핑(a!)를 사용하여 a가 nil이 아닌 경우 a 안에 감싸진 값을 얻고, 그렇지 않은 경우 b를 반환합니다. Nil 병합 연산자는 위 삼항식 조건 확인과 언랩핑를 간결하고 가독성 있게 표현합니다.

> 참고
> 
> a의 값이 nil이 아닌 경우 b의 값은 평가되지 않습니다. 이를 단축 평가라고 합니다.

아래 예제는 nil 병합 연산자를 사용하여 옵셔널인 사용자 정의 색상 이름과 기본 색상 이름 중에서 하나를 선택합니다:

```swift
let defaultColorName = "red"
var userDefinedColorName: String?   // 기본값은 nil입니다.

var colorNameToUse = userDefinedColorName ?? defaultColorName
// userDefinedColorName은 nil이므로 colorNameToUse는 기본값인 "red"로 설정됩니다.
```

userDefinedColorName 변수는 옵셔널 String으로 정의되며, 기본값으로 nil이 할당됩니다. userDefinedColorName은 옵셔널 타입이므로 nil 병합 연산자를 사용하여 값의 존재 여부를 판단할 수 있습니다. 위 예제는 이 연산자를 사용하여 String 변수인 colorNameToUse의 초기값을 결정합니다. userDefinedColorName이 nil이기 때문에 표현식 `userDefinedColorName ?? defaultColorName`은 defaultColorName인 "red"를 반환합니다.

만약 userDefinedColorName에 nil이 아닌 값을 할당하고 nil 병합 연산자를 다시 평가하면, 기본값 대신  userDefinedColorName 이 담고 있는 값을 사용합니다.:

```swift
userDefinedColorName = "green"
colorNameToUse = userDefinedColorName ?? defaultColorName
// userDefinedColorName은 nil이 아니므로 colorNameToUse는 "green"으로 설정됩니다.
```

## 범위 연산자

Swift에는 값의 범위를 표현하는 여러 가지 범위 연산자가 있습니다.

### 닫힌 범위 연산자

닫힌 범위 연산자 `(a...b)`는 a에서 b까지의 범위를 정의하며, 값 a와 b를 포함합니다. a의 값은 b보다 작아야 합니다.

닫힌 범위 연산자는 `for-in` 루프와 같이 모든 값을 사용하는 범위를 반복할 때 유용합니다:

```swift
for index in 1...5 {
    print("\(index) 곱하기 5는 \(index * 5)입니다.")
}
// 1 곱하기 5는 5입니다.
// 2 곱하기 5는 10입니다.
// 3 곱하기 5는 15입니다.
// 4 곱하기 5는 20입니다.
// 5 곱하기 5는 25입니다.
```

for-in 루프에 대한 자세한 내용은 [제어 흐름](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow)을 참조하세요.

### 반열린 범위 연산자(Half-Open Range Operator)

반열린 범위 연산자`(a..<b)`는 b를 포함하지 않고, a부터 b까지의 범위를 정의합니다. 첫 번째 값은 포함하지만 마지막 값은 포함하지 않기 때문에 반열린이라고 합니다. 닫힌 범위 연산자와 마찬가지로, a의 값은 b보다 작아야 합니다. 만약 a의 값이 b와 같다면, 아무 것도 포함하고 있지 않는 빈 범위가 됩니다.

반열린 범위는 배열처럼 0부터 시작하는 리스트와 함께 작업할 때 특히 유용합니다. 리스트의 길이까지(포함하지 않고) 계산하는 것에 유용합니다.:

```swift
let names = ["Anna", "Alex", "Brian", "Jack"]
let count = names.count
for i in 0..<count {
    print("Person \(i + 1) is called \(names[i])")
}
// Person 1 is called Anna
// Person 2 is called Alex
// Person 3 is called Brian
// Person 4 is called Jack
```

배열에는 네 개의 항목이 있지만, `0..<count`는 마지막 항목의 인덱스인 3까지만 계산합니다. 이는 반열린 범위이기 때문입니다. 배열에 대한 자세한 내용은 [배열](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes#Arrays)을 참조하세요.

### 한쪽 범위

닫힌 범위 연산자에는 한 방향으로 가능한 범위 끝까지 이어지는 범위 형태가 있습니다. 예를 들어, 배열에서 인덱스 2부터 배열 끝까지의 모든 요소를 포함하는 범위입니다. 이 경우 범위 연산자의 한쪽 값을 생략할 수 있습니다. 연산자는 한쪽에만 값을 가지기 때문에 이러한 종류의 범위를 한쪽 범위라고 합니다. 예를 들어:

```swift
for name in names[2...] {
    print(name)
}
// Brian
// Jack

for name in names[...2] {
    print(name)
}
// Anna
// Alex
// Brian
```

반열린 범위 연산자에도 해당하는 최종 값을 가진 한쪽 범위 형태가 있습니다. 양쪽에 값을 포함할 때와 마찬가지로 최종 값은 범위에 포함되지 않습니다. 예를 들어:

```swift
for name in names[..<2] {
    print(name)
}
// Anna
// Alex
```

한쪽 범위는 서브스크립트(`[]`) 뿐만 아니라 다른 경우에도 사용할 수 있습니다. 다만, 첫 번째 값이 생략된 한쪽 범위로 반복할 수는 없습니다. 왜냐하면 반복이 어디에서 시작해야 하는지 명확하지 않기 때문입니다. 그러나 마지막 값을 생략한 한쪽 범위로 반복은 가능합니다. 이때 범위가 무한히 이어지기 때문에 명시적인 종료 조건을 반드시 추가해야 합니다. 또한 아래 코드에서 보여지듯이 한쪽 범위가 특정 값을 포함하고 있는지 확인할 수도 있습니다.

```swift
let range = ...5
range.contains(7)   // false
range.contains(4)   // true
range.contains(-1)  // true
```

## 논리 연산자

논리 연산자들은 참과 거짓, 부울 논리 값을 수정하거나 결합합니다. Swift는 C 기반 언어에서 찾을 수 있는 세 가지 표준 논리 연산자를 지원합니다:

- 논리 NOT (`!a`)
- 논리 AND (`a && b`)
- 논리 OR (`a || b`)


### 논리 NOT 연산자

논리 NOT 연산자(`!a`)는 불리언 값을 반전시켜서 true가 false로, false가 true로 변환합니다.

논리 NOT 연산자는 접두사 연산자이며, 공백 없이 바로 그 뒤에 있는 값을 조작합니다. 다음 예시에서 볼 수 있듯이 "a가 아니다"라고 읽을 수 있습니다:

```swift
let allowedEntry = false
if !allowedEntry {
    print("ACCESS DENIED")
}
// "ACCESS DENIED"가 출력됩니다.
```

`if !allowedEntry` 문은 "만약 허용되지 않았다면"으로 해석할 수 있습니다. 다음 줄은 "허용되지 않았다면"이 참일 때만 실행됩니다. 즉, allowedEntry가 false인 경우입니다.

이 예시처럼, 불리언 상수와 변수의 이름을 신중하게 선택하여 코드의 가독성과 간결성을 유지하면, 부정문 또는 혼동스러운 논리문을 피할 수 있습니다.

### 논리 AND 연산자

논리 AND 연산자 (`a && b`)는 두 값이 모두 참일 때 전체 식이 참이 되는 논리식을 만듭니다.

두 값 중 하나라도 false이면 전체 식도 false가 됩니다. 사실, 첫 번째 값이 false이면 두 번째 값은 평가되지 않습니다. 왜냐하면 두 번째 값이 전체 식이 true가 되도록 할 수 없기 때문입니다. 이를 일컬어 "단락 평가(short-circuit evaluation)"라고 합니다.

다음 예제는 두 개의 Bool 값이 고려되며, 두 값이 모두 true인 경우에만 접근을 허용합니다:

```swift
let enteredDoorCode = true
let passedRetinaScan = false
if enteredDoorCode && passedRetinaScan {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// "ACCESS DENIED"을 출력합니다
```

### 논리 OR 연산자

논리 OR 연산자 (`a || b`)는 두 인접한 파이프 문자로 이루어진 중위 연산자입니다. 이 연산자는 두 값 중 하나만 true이면 전체 표현식이 true가 되는 논리식을 생성하는 데 사용됩니다.

위에 설명된 것처럼 논리 AND 연산자와 마찬가지로 논리 OR 연산자는 단릭 평가를 사용하여 표현식을 고려합니다. 만약 논리 OR 식의 왼쪽 부분이 true이면, 오른쪽 부분은 평가되지 않습니다. 왜냐하면 오른쪽 부분은 전체 표현식의 결과를 변경할 수 없기 때문입니다.

아래의 예시에서 첫 번째 Bool 값인 hasDoorKey는 false이지만, 두 번째 값인 knowsOverridePassword는 true입니다. 하나의 값이 true이기 때문에 전체 표현식도 true로 평가되며, 접근이 허용됩니다:

```swift
let hasDoorKey = false
let knowsOverridePassword = true
if hasDoorKey || knowsOverridePassword {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// 출력: "Welcome!"
```

### 논리 연산자 조합

여러 논리 연산자를 조합하여 긴 복합 표현식을 만들 수 있습니다:

```swift
if enteredDoorCode && passedRetinaScan || hasDoorKey || knowsOverridePassword {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// 출력: "Welcome!"
```

이 예제에서는 여러 개의 &&와 || 연산자를 사용하여 긴 복합 표현식을 생성합니다. 그러나 &&와 || 연산자는 여전히 두 개의 값에 대해서만 작동하므로 이는 사실상 세 개의 작은 표현식이 연결된 것입니다. 예제를 다음과 같이 해석할 수 있습니다:

올바른 출입코드를 입력하고 망막 스캔에 통과하였거나 유효한 출입 키를 가지고 있거나 비상용 재정 비밀번호를 알고 있는 경우라면, 접근 허용.

입력된 출입코드, 망막 스캔 통과 여부, 출입 키 보유 여부를 기반으로 하면, 첫 두 개의 하위 표현식은 거짓입니다. 그러나 비상용 재정 비밀번호는 알려져 있으므로 전체적인 복합 표현식은 참으로 평가됩니다.

> 참고
> 
> Swift의 논리 연산자 &&와 ||은 좌결합성을 가지기 때문에 여러 논리 연산자가 있는 복합 표현식을 계산할 때 가장 왼쪽의 표현식을 먼저 평가합니다.

### 명시적 괄호

복잡한 표현식의 의도를 쉽게 파악하기 위해 괄호를 사용하는게 유용합니다. 앞서 언급한 도어 액세스 예시에서 복합 표현식의 첫 번째 부분에 괄호를 추가하여 명시적으로 의도를 표시할 수 있습니다.

```swift
if (enteredDoorCode && passedRetinaScan) || hasDoorKey || knowsOverridePassword {
    print("Welcome!")
} else {
    print("ACCESS DENIED")
}
// 출력: "Welcome!"
```

괄호를 추가함으로써 첫 번째 두 값이 전체 논리에서 별개의 가능한 상태로 간주된다는 것을 명확하게 나타낼 수 있습니다. 복합 표현식의 출력은 변경되지 않지만, 독자에게 전반적인 의도가 더욱 명확해집니다. 가독성은 항상 간결함보다 우선되며, 의도가 명확해지는 데 도움이 되는 경우에는 괄호를 사용해야 합니다.

