---
title: "Swift 기초"
date: 2023-06-26T06:25:42Z
author: "skyfe79"
draft: false
tags: ["swift","swift5.9"]
---



Swift는 iOS, macOS, watchOS, tvOS 앱 개발을 위한 프로그래밍 언어입니다. C 또는 Objective-C 개발 경험이 있다면, Swift의 많은 부분이 친숙할 것입니다.

Swift는 정수에 대한 Int, 부동 소수점 값에 대한 Double 및 Float, 불리언 값에 대한 Bool, 텍스트 데이터에 대한 String을 비롯한 모든 기본 C 및 Objective-C 타입의 자체 버전을 제공합니다. Swift는 [Collection Types](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes)에서 설명된 Array, Set 및 Dictionary의 세 가지 주요 컬렉션 타입의 강력한 버전도 제공합니다.

C와 마찬가지로, Swift는 값을 저장하고 참조하기 위해 식별 이름으로 변수를 사용합니다. Swift는 값이 변경될 수 없는 변수도 광범위하게 사용합니다. 이러한 변수는 상수로 알려져 있으며 C의 상수보다 훨씬 더 강력합니다. Swift에서는 값을 변경할 필요가 없는 경우에 값을 안전하게 유지하고 명확하게 할 수 있도록 상수를 사용합니다.

친숙한 타입 외에도, Swift는 Objective-C에 없는 튜플과 같은 고급 타입을 소개합니다. 튜플을 사용하면 값의 그룹을 만들고 전달할 수 있습니다. 튜플을 사용하여 함수에서 여러 값을 하나의 복합 값으로 반환할 수 있습니다.

Swift는 값이 없는 경우를 처리하는 optional 타입도 소개합니다. Optional은 "값이 있으며 x와 같다" 또는 "값이 전혀 없다"고 말합니다. 옵셔널은 Objective-C의 포인터에서 nil을 사용하는 것과 유사하지만, 클래스뿐만 아니라 모든 타입에 대해 작동합니다. 옵셔널은 옵셔널 포인터보다 더 안전하고 의미 있는 코드를 작성하는 데 필수적입니다.

Swift는 타입 안전한 언어입니다. 이는 언어가 코드가 작동할 수 있는 값을 명확하게 하는 데 도움이 되는 것을 의미합니다. 코드 일부가 문자열을 필요로 할 경우, 타입 안전성은 실수로 정수를 전달하는 것을 방지합니다. 마찬가지로, 타입 안전성은 non-optional 문자열을 필요로 하는 코드에 optional 문자열을 실수로 전달하지 않도록 방지합니다. 타입 안전성은 개발 프로세스의 최대한 이른 시기에 오류를 발견하고 수정하는 데 도움이 됩니다.

## 상수와 변수

상수와 변수는 특정 타입의 값(숫자 10 또는 문자열 "Hello"와 같은)에 이름(maximumNumberOfLoginAttempts 또는 welcomeMessage와 같은)을 연결합니다. 상수의 값은 한 번 설정되면 변경할 수 없지만, 변수는 나중에 다른 값으로 설정할 수 있습니다.

### 상수와 변수 선언

상수와 변수는 사용되기 전에 선언해야 합니다. 상수는 let 키워드로 선언하고 변수는 var 키워드로 선언합니다. 다음은 사용자가 만든 로그인 시도 수를 추적하는 데 상수와 변수를 사용하는 방법의 예입니다.

```swift
let maximumNumberOfLoginAttempts = 10
var currentLoginAttempt = 0
```

위 코드는 다음과 같이 읽을 수 있습니다.

"최대 로그인 시도 횟수라는 새로운 상수 maximumNumberOfLoginAttempts를 선언하고 값으로 10을 부여합니다. 그 다음, 현재 로그인 시도라는 새로운 변수 currentLoginAttempt를 선언하고 초기 값으로 0을 부여합니다."

이 예제에서 허용된 로그인 시도 최대 횟수는 최대 값이 항상 변하지 않으므로 상수로 선언되었습니다. 현재 로그인 시도 카운터는 로그인에 실패할 때마다 증가해야 하는 값이므로 변수로 선언되었습니다.

쉼표로 구분하여 한 줄에 여러 개의 상수 또는 변수를 선언할 수도 있습니다.
```swift
var x = 0.0, y = 0.0, z = 0.0
```

> 참고
> 
> 코드의 저장된 값이 변경되지 않는 경우 항상 let 키워드로 상수를 선언하세요. 변수는 값이 변경될 수 있는 값을 저장하는 데만 사용하세요.

### 타입 어노테이션(Type Annotations)

상수나 변수를 선언할 때 해당 상수나 변수가 어떤 종류의 값을 저장할 수 있는지 명확하게 하기 위해 타입 어노테이션(Type Annotation)을 제공합니다. 상수나 변수 이름 다음에 콜론(:)을 붙이고, 그 뒤에 사용할 타입 이름을 붙여 타입 어노테이션을 작성할 수 있습니다.

```swift
var welcomeMessage: String
```

위의 코드는 `welcomeMessage`라는 변수에 String 값을 저장할 수 있다는 타입 어노테이션을 제공합니다. 

`:`는 “…의 타입…”을 의미하므로 위의 코드는 다음과 같이 읽을 수 있습니다.

“String 타입의 welcomeMessage 변수를 선언합니다.”

“String 타입”은 “모든 String 값을 저장할 수 있다”는 것을 의미합니다. 즉, 저장할 수 있는 “값의 종류”라는 의미로 생각할 수 있습니다.

이제 `welcomeMessage` 변수는 아무 문자열 값이나 저장할 수 있습니다.

```swift
welcomeMessage = "Hello"
```

다음과 같이 쉼표(,)로 구분해서 한 줄에 여러 개의 관련 변수를 동일한 타입으로 정의할 수도 있습니다. 마지막 변수 이름 바로 뒤에 타입 어노테이션을 작성하면 됩니다.

```swift
var red, green, blue: Double
```

> 참고
> 
> 실제로는 대부분의 경우 초기값을 상수나 변수가 선언될 때 바로 제공하므로 타입 추론(Type Inference)을 통해 대부분의 타입 어노테이션을 작성할 필요가 없습니다. [Type Safety and Type Inference](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics#Type-Safety-and-Type-Inference)에서 설명한 것처럼, 위의 `welcomeMessage` 예시에서는 초기값이 제공되지 않았기 때문에 초기값에 따라 타입을 추론하는 대신 타입 어노테이션을 작성해야 합니다.

### 상수와 변수 이름 붙이기

상수와 변수 이름에는 대부분의 문자(Unicode 문자 포함)가 포함될 수 있습니다.

```swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"
```

하지만 이름에 공백 문자, 수학 기호, 화살표, private-use Unicode scalar 값, 라인 및 박스 드로잉 문자를 사용할 수는 없습니다. 또한 숫자로 시작할 수 없지만, 이름 내 다른 위치에는 숫자를 포함시킬 수 있습니다.

한번 특정 타입의 상수 또는 변수를 선언하면, 같은 이름으로 다시 선언하거나 다른 타입의 값을 저장하도록 변경할 수 없습니다. 또한 상수를 변수로 변경하거나 변수를 상수로 변경할 수도 없습니다.

> 참고
> 
> Swift 예약어와 동일한 이름을 가진 상수 또는 변수를 지정해야 할 경우, 이름을 사용할 때 예약어를 역 따옴표`(```)`로 묶어 이스케이프합니다. 그러나 절대적으로 선택할 수 없는 경우를 제외하고는 키워드를 이름으로 사용하지 않는 것이 좋습니다.

호환 가능한 타입의 값이라면 다른 값을 저장할 수 있습니다. 이 예제에서 friendlyWelcome의 값은 "Hello!"에서 "Bonjour!"로 변경됩니다.

```swift
var friendlyWelcome = "Hello!"
friendlyWelcome = "Bonjour!"
// friendlyWelcome는 이제 "Bonjour!"입니다. 
```

변수와 달리, 상수의 값은 설정한 후에 변경할 수 없습니다. 변경하려고 하면 코드가 컴파일될 때 오류가 발생합니다.

```swift
let languageName = "Swift"
languageName = "Swift++"
// 이것은 컴파일 타임 오류입니다: languageName은 변경될 수 없습니다.
```

### 상수와 변수 출력하기

상수와 변수의 현재 값을 출력하려면 `print(_:separator:terminator:)` 함수를 사용할 수 있습니다.

```Swift
print(friendlyWelcome)
// "Bonjour!" 출력
```

`print(_:separator:terminator:)` 함수는 하나 이상의 값을 적절한 출력으로 표시하는 전역 함수입니다. 예를 들어 Xcode에서는 `print(_:separator:terminator:)` 함수가 Xcode의 "콘솔" 팬에 출력합니다. separator와 terminator 매개변수는 기본값이 있으므로 이 함수를 호출할 때 생략할 수 있습니다. 기본적으로 함수는 줄 바꾸기를 추가하여 출력 라인을 종료합니다. 줄 바꿈이 없이 값만 출력하려면 문자열이 비어 있는 terminator를 전달합니다. 예를 들어, `print(someValue, terminator: "")`와 같은 형식으로 입력합니다. 기본 값이 있는 매개변수에 대한 정보는 [Default Parameter Values](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions#Default-Parameter-Values)를 참조하십시오.

Swift는 문자열 보간을 사용하여 상수 또는 변수의 이름을 긴 문자열의 placeholder로 포함시키고 해당 상수 또는 변수의 현재 값으로 변경합니다. 변수 또는 상수 이름을 괄호로 묶고 여는 괄호 앞에 백슬래시로 이스케이프합니다.

```Swift
print("The current value of friendlyWelcome is \(friendlyWelcome)")
// "The current value of friendlyWelcome is Bonjour!" 출력
```

> 참고
> 
> 문자열 보간 옵션에 대한 정보는 [String Interpolation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/stringsandcharacters#String-Interpolation)를 참조하십시오.


## 주석

주석은 코드에 대한 노트나 메모를 작성할 때, 코드가 컴파일될 때 무시돼야 하는 정보를 작성할 때 사용합니다. Swift에서의 주석은 C 언어와 매우 유사합니다. 

### 한 줄 주석

한 줄 주석은 두 개의 슬래시(//)로 시작합니다:

```swift
// 이것은 한 줄 주석입니다.
```

### 여러 줄 주석

여러 줄 주석은 시작과 끝을 나타내는 슬래시(/)와 별표(*)를 사용합니다.

```swift
/* 이것은 여러 줄 주석입니다.
   여러 줄에 걸쳐 작성할 수 있습니다. */
```

C 언어와는 다르게, Swift에서는 중첩 주석이 가능합니다. 중첩된 주석을 만들 때는, 첫 번째 여러 줄 주석 블록 안에서 두 번째 여러 줄 주석을 시작합니다. 그리고 두 번째 주석 블록을 끝내고, 첫 번째 주석 블록을 닫습니다.

```swift
/* 이것은 첫 번째 여러 줄 주석의 시작입니다.
   /* 이것은 두 번째, 중첩된 여러 줄 주석입니다. */
   이것은 첫 번째 여러 줄 주석의 끝입니다. */
```

중첩된 여러 줄 주석을 사용하면, 이미 여러 줄 주석이 작성된 코드에서 큰 블록의 코드를 쉽고 빠르게 주석 처리할 수 있습니다.

## 세미콜론 (;)

Swift 언어에서는 다른 많은 언어와 달리 한 문장이 끝난 후 세미콜론 (;)을 작성하지 않아도 됩니다. 하지만, 한 줄에 여러 개의 문장을 작성하려면 세미콜론이 필요합니다.

```swift
let cat = "🐱"; print(cat)
// 출력 결과: "🐱"
```


## 정수(Integers) 데이터 타입

정수(Integers)는 소수점이 없는 정수형 숫자를 의미합니다. Swift에서는 부호가 있는(Signed) 정수와 부호가 없는(Unsigned) 정수를 지원하며, 이러한 정수들은 8, 16, 32, 64비트를 지원합니다. 각각의 데이터 타입은 UInt8, Int32와 같은 방식으로 이름이 붙여집니다.

Swift에서는 모든 데이터 타입과 마찬가지로, 정수형 데이터 타입 이름은 모두 대문자로 시작합니다.

### 정수형 데이터 타입의 범위

각각의 정수형 데이터 타입에는 min 속성과 max 속성이 제공됩니다. 이를 통해 해당 정수형 데이터 타입의 최솟값과 최댓값을 알 수 있습니다.

```swift
let minValue = UInt8.min  // minValue의 값은 0이고, UInt8 타입입니다.
let maxValue = UInt8.max  // maxValue의 값은 255이고, UInt8 타입입니다.
```

이러한 속성들의 값은 해당 숫자 타입(예: UInt8)에 해당하는 숫자 타입의 속성이므로, 동일한 타입의 다른 값들과 함께 표현식에서 사용할 수 있습니다.

### Int 데이터 타입

Int는 현재 플랫폼에서 사용되는 네이티브 워드(word) 사이즈와 동일한 크기를 가지는 데이터 타입입니다. 

- 32비트 플랫폼에서는 Int는 Int32와 같은 크기를 가지며,
- 64비트 플랫폼에서는 Int는 Int64와 같은 크기를 가집니다.

보통은 특정한 크기의 정수를 사용할 필요성이 없으므로, 코드에서 정수형 값을 사용할 때는 항상 Int를 사용하는 것이 좋습니다. 이렇게 함으로써 코드 일관성과 상호운용성을 증가시킬 수 있습니다. 심지어 32비트 플랫폼에서도 Int는 -2,147,483,648부터 2,147,483,647 사이의 값을 저장할 수 있는 크기를 가지므로, 많은 정수 범위에 대해서 충분합니다.

### UInt 데이터 타입

Swift에서는 Unsigned(부호가 없는) 정수형 데이터 타입인 UInt를 제공합니다. UInt는 현재 플랫폼에서 사용되는 네이티브 워드(word) 사이즈와 동일한 크기를 가집니다. 

- 32비트 플랫폼에서는 UInt는 UInt32와 같은 크기를 가지며,
- 64비트 플랫폼에서는 UInt는 UInt64와 같은 크기를 가집니다.

> 참고
> 
> UInt 정수형 데이터 타입은 반드시 필요한 경우가 아니라면, Int 데이터 타입을 사용하는 것이 좋습니다. Int 데이터 타입을 사용하는 것은 코드 상호운용성과 서로 다른 숫자 타입 간의 변환이 필요하지 않게 되며, [Type Safety and Type Inference](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics#Type-Safety-and-Type-Inference)에서 소개한 타입 추론(Type Inference)과 일관성을 유지할 수 있습니다. 

## 부동 소수점 수(Floating-Point Number)

부동 소수점 수는 소수 부분을 갖는 수로, 3.14159, 0.1, -273.15 등이 있습니다.

부동 소수점 타입은 정수 타입보다 훨씬 넓은 값의 범위를 나타낼 수 있으며, Int에 저장할 수 없는 매우 크거나 작은 수를 저장할 수 있습니다. Swift는 두 가지 부호 있는 부동 소수점 숫자 타입을 제공합니다.

- Double은 64비트 부동 소수점 수를 나타냅니다.
- Float은 32비트 부동 소수점 수를 나타냅니다.

> 참고
> 
> Double은 적어도 15개의 십진수 자릿수의 정밀도를 가지고 있으며, Float의 정밀도는 최소 6개의 십진수 자릿수가 될 수 있습니다. 사용할 적절한 부동 소수점 타입은 코드에서 다루어야 하는 값의 성격과 범위에 따라 다릅니다. 두 타입 모두 적절한 경우, Double을 사용하는 것이 좋습니다.

## 타입 안전성과 타입 추론

Swift는 타입 안전한 언어입니다. 타입 안전한 언어는 코드에서 사용 가능한 값의 타입을 명확하게 하도록 권장합니다. 예를 들어 코드의 일부가 문자열을 요구하고 있을 때 실수로 Int를 전달할 수 없습니다.

Swift는 타입 안전하기 때문에 코드를 컴파일할 때 타입 체크를 수행하고 일치하지 않는 타입을 오류로 표시합니다. 이를 통해 개발 프로세스에서 가능한 빨리 오류를 감지하고 수정할 수 있습니다.

타입 체크를 통해 다른 유형의 값과 작업할 때 오류를 방지할 수 있습니다. 그러나 모든 상수와 변수의 타입을 지정해야하는 것은 아닙니다. 값의 타입을 지정하지 않으면 Swift는 타입 추론을 사용하여 적절한 타입을 결정합니다. 타입 추론은 컴파일러가 코드를 컴파일할 때 특정 표현식의 타입을 자동으로 추론 할 수 있는 것을 말합니다.

타입 추론 덕분에 Swift는 C 또는 Objective-C와 같은 언어보다 훨씬 적은 타입 선언을 필요로 합니다. 상수와 변수는 여전히 명시적으로 입력됩니다. 그러나 그들의 타입을 지정하는 작업은 대부분 대신해줍니다.

타입 추론은 상수나 변수를 초기값으로 선언할 때 특히 유용합니다. 이를 통해 리터럴 값을 상수나 변수에 할당할 수 있는데, 리터럴 값이란 42나 3.14159와 같이 직접 소스 코드에 나타나는 값입니다.

예를 들어 새로운 상수에 타입을 지정하지 않고 리터럴 값을 42로 할당하면, Swift는 이 상수가 Int 타입이 되길 원한다는 것을 추론할 수 있습니다.

```swift
let meaningOfLife = 42
// meaningOfLife는 Int 타입으로 추론됩니다.
```

마찬가지로 부동 소수점 리터럴의 타입을 지정하지 않으면, Swift는 Double을 생성하려고 시도하는 것으로 추론합니다.

```swift
let pi = 3.14159
// pi는 Double 타입으로 추론됩니다.
```

Swift는 항상 부동 소수점 숫자의 타입으로 Double을 선택합니다.

정수와 부동 소수점 리터럴을 표현식에서 결합하면, Double의 타입이 컨텍스트에서 추론됩니다.

```swift
let anotherPi = 3 + 0.14159
// anotherPi는 Double 타입으로 추론됩니다.
```

숫자 3의 리터럴 값은 명시적인 타입이 없으므로 덧셈의 일부인 부동 소수점 리터럴에서 적절한 Double 타입이 추론됩니다.

## 숫자 리터럴

숫자 리터럴은 다음과 같이 작성될 수 있습니다:

- 접두사가 없는 십진수
- 0b 접두사가 있는 2진수
- 0o 접두사가 있는 8진수
- 0x 접두사가 있는 16진수

다음 모든 정수 리터럴은 10진수 값 17을 가집니다:

```swift
let decimalInteger = 17
let binaryInteger = 0b10001      // 2진수 표기법으로 17
let octalInteger = 0o21          // 8진수 표기법으로 17
let hexadecimalInteger = 0x11    // 16진수 표기법으로 17
```

부동소수점 리터럴은 접두사가 없는 10진수 또는 0x 접두사가 있는 16진수일 수 있습니다. 항상 소수점 양쪽에 숫자(또는 16진수)가 있어야 합니다. 10진수 부동소수점은 대문자 또는 소문자 e로 나타내는 지수를 선택적으로 가질 수 있습니다. 16진수 부동소수점은 대문자 또는 소문자 p로 나타내는 지수를 필수적으로 가집니다.

지수가 x인 10진수의 경우 기수 숫자는 10ˣ으로 곱해집니다.

- 1.25e2는 1.25 x 10² 또는 125.0을 의미합니다.
- 1.25e-2는 1.25 x 10⁻² 또는 0.0125를 의미합니다.

지수가 x인 16진수의 경우 기수 숫자는 2ˣ으로 곱해집니다.

- 0xFp2는 15 x 2² 또는 60.0을 의미합니다.
- 0xFp-2는 15 x 2⁻² 또는 3.75를 의미합니다.

다음 부동소수점 리터럴 모두 10진수 값 12.1875를 가집니다:

```swift
let decimalDouble = 12.1875
let exponentDouble = 1.21875e1
let hexadecimalDouble = 0xC.3p0
```

숫자 리터럴은 가독성을 높이기 위해 추가적인 서식 지정을 포함할 수 있습니다. 정수 및 부동소수점 모두 추가적인 0으로 패딩하여 가독성을 높이고 밑줄을 넣어 가독성을 높일 수 있습니다. 이러한 서식 지정은 리터럴의 기본 값에 영향을 주지 않습니다.

```swift
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```

## 숫자형 타입 변환

코드에서 모든 일반적인 목적의 정수 상수와 변수에 대해 Int 타입을 사용하세요. 비록 그들이 음수 값이 아니더라도요. 일상적인 상황에서 기본 정수 타입을 사용함으로써 정수 상수와 변수는 코드에서 즉시 상호 운용 가능하게 되며, 정수 리터럴 값에 대한 추론된 타입과 일치하게 됩니다.

다른 정수 타입은 명시적으로 필요한 경우, 외부 소스에서 명시적으로 크기가 지정된 데이터 때문에 사용해야만 하는 경우와 성능, 메모리 사용, 기타 필요한 최적화 때문에만 사용하세요. 이러한 상황에서 명시적으로 크기가 지정된 타입을 사용하면 임의 값 오버플로우를 잡을 수 있고 사용되는 데이터의 성격을 암시적으로 문서화할 수 있습니다.

### 정수 변환

정수가 저장될 수 있는 수의 범위는 각 숫자형 타입마다 다릅니다. Int8 상수나 변수는 -128과 127 사이의 숫자를 저장할 수 있지만, UInt8 상수나 변수는 0과 255 사이의 숫자를 저장할 수 있습니다. 크기가 지정된 정수 타입의 상수나 변수에 맞지 않는 숫자는 코드를 컴파일 할 때 오류로 보고 됩니다.

```swift
let cannotBeNegative: UInt8 = -1
// UInt8 은 음수를 저장할 수 없습니다. 따라서 오류가 보고 됩니다.
let tooBig: Int8 = Int8.max + 1
// Int8은 최대 값보다 큰 수를 저장 할 수 없습니다. 그래서 오류가 보고 됩니다.
```

각 숫자 형식이 서로 다른 값의 범위를 저장할 수 있기 때문에 경우에 따라 숫자형 변환이 필요합니다. 이 접근법은 숨겨진 변환 오류를 방지하고 코드에서 형 변환 의도가 명시적으로 표시 되도록 합니다.

하나의 특정 숫자 형식을 다른 형식으로 변환하려면 기존 값으로 새로운 숫자를 초기화합니다. 아래 예제에서 상수 twoThousand는 UInt16 타입이고 상수 one은 UInt8 타입입니다. 바로 덧셈을 할 수 없습니다. 왜냐하면 두 타입이 같지 않기 때문입니다. 대신 `UInt16(one)`을 호출하여 새로운 UInt16을 초기화하고 이 값을 사용합니다.

```swift
let twoThousand: UInt16 = 2_000
let one: UInt8 = 1
let twoThousandAndOne = twoThousand + UInt16(one)
```

이제 덧셈의 양쪽 모두가 UInt16 형이므로 덧셈이 허용됩니다. 출력 상수(twoThousandAndOne)는 두 UInt16 값의 합이므로 유추 타입이 UInt16임을 암시합니다.

`SomeType(ofInitialValue)`은 Swift 타입 초기자를 호출하고 초기값을 전달하는 기본 방법입니다. UInt16에는 UInt8 값 허용하는 초기자가 있기 때문에 이 초기값을 사용하여 새로운 UInt16을 만듭니다. 그러나 여기에는 전달 할 수 있는 타입이 있습니다. UInt16이 초기자를 제공하는 타입이어야 합니다. 기존 타입을 확장하여 새로운 타입(사용자 정의 타입 포함)을 수용 할 수 있는 초기자를 제공하는 방법은 [확장](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions)에서 다룹니다.

### 정수 및 부동 소수점 변환

정수와 부동 소수점 숫자 형식 간의 변환은 명시적으로 수행해야합니다.

```swift
let three = 3
let pointOneFourOneFiveNine = 0.14159
let pi = Double(three) + pointOneFourOneFiveNine
// pi는 3.14159이며 Double 타입으로 추론됩니다.
```

여기에서 상수 three의 값은 Double 형식의 새로운 값을 생성 할 때 사용됩니다. 이렇게 하면 덧셈 양쪽이 같은 타입이 되므로 덧셈 연산을 적용 할 수 있습니다. 이러한 변환이 없으면 덧셈이 허용되지 않습니다.

부동 소수점에서 정수로의 변환도 명시적으로 수행해야 합니다. 정수 타입은 Double이나 Float 값을 사용하여 초기화 될 수 있습니다.

```swift
let integerPi = Int(pi)
// integerPi는 3이며 Int 타입으로 추론됩니다.
```

이러한 방법으로 새 정수 값을 ​​초기화 할 때는 항상 부동 소수점 값이 잘립니다. 이것은 4.75가 4가되며 -3.9가 -3이 된다는 것을 의미합니다.

> 참고
> 
> 숫자 상수와 변수를 결합하는 규칙은 숫자 리터럴과 결합하는 규칙과 다릅니다. 숫자 리터럴은 그 자체로 명시적 타입을 갖지 않기 때문에 리터럴 값 3을 리터럴 값 0.14159에 직접 더할 수 있습니다. 그리고 숫자 리터럴은 컴파일러에 의해 평가되는 시점에만 추론됩니다. 즉, `let pi = 3 + 0.14159` 덧셈은 가능하며 pi는 Double로 추론됩니다.

## 타입 별칭(Type Aliases)

타입 별칭(Type Aliases)은 기존 타입의 대체 이름을 정의하는 것입니다. `typealias` 키워드를 사용하여 타입 별칭을 정의할 수 있습니다.

타입 별칭은 외부 소스에서 특정 크기의 데이터를 다루는 상황과 같이, 기존 타입을 더 맥락에 맞는 이름으로 참조하고자 할 때 유용합니다.

```swift
typealias AudioSample = UInt16
```

위 코드에서 `AudioSample`을 `UInt16`의 별칭(alias)으로 정의 했습니다. 이제 별칭을 사용해서 기존 타입을 사용할 수 있습니다.

```swift
var maxAmplitudeFound = AudioSample.min 
// maxAmplitudeFound는 0이 됩니다.
```

위 코드에서는 `AudioSample.min`이 실제로는 `UInt16.min`을 호출하기 때문에, `maxAmplitudeFound` 변수의 초기값으로 0이 할당됩니다.

## 부울(Boolean)

Swift는 Bool이라는 기본 부울 타입을 가지고 있습니다. 부울 값은 참 또는 거짓일 수밖에 없으므로 논리적이라고 합니다. Swift는 두 개의 부울 상수(true와 false)를 제공합니다:

```swift
let orangesAreOrange = true
let turnipsAreDelicious = false
```

orangesAreOrange과 turnipsAreDelicious의 타입은 부울값으로부터 추론됩니다. 위 코드와 같이 true나 false로 상수나 변수를 초기화 할 경우, bool로 선언하지 않아도 됩니다. 이처럼 타입 추론 기능은 이미 알려진 값들로 상수나 변수를 초기화 할 때, Swift 코드를 더 간결하고 가독성이 좋아지도록 돕습니다.

부울 값은 특히 if문과 같은 조건문에서 유용합니다.

```swift
if turnipsAreDelicious {
    print("Mmm, tasty turnips!")
} else {
    print("Eww, turnips are horrible.")
}
// 출력: "Eww, turnips are horrible."
```

위 코드에서는 부울 값 turnipsAreDelicious가 참이 아니므로 else 블록이 실행됩니다. 조건문에 대한 자세한 내용은 [Control Flow](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow)에서 다루고 있습니다.

Swift의 타입 안정성은 부울 값이 아닌 다른 값을 지정할 수 없도록 합니다. 다음 예제는 컴파일 시간 오류를 보고합니다.

```swift
let i = 1
if i {
    // 이 예제는 컴파일되지 않으며 오류를 보고합니다.
}
```

하지만 아래와 같은 예제는 유효합니다.

```swift
let i = 1
if i == 1 {
    // 이 예제는 컴파일됩니다.
}
```

`i == 1` 비교 결과는 부울 타입이므로 이 두 번째 예제는 타입 검사를 통과합니다. `i == 1`과 같은 비교는 [Basic Operators](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators)에서 설명됩니다.

Swift에서 다른 타입 안정성 예제와 마찬가지로, 이러한 접근법은 우연한 오류를 피하도록 도와주며, 특정 코드의 의도가 항상 명확하게 보장됩니다.

## 튜플(Tuple)

튜플은 여러 값을 하나의 복합 값으로 묶은 것입니다. 튜플 내의 값은 어떤 유형이든 될 수 있으며, 서로 같은 유형일 필요는 없습니다.

예를 들어, (404, "Not Found")는 HTTP 상태 코드를 설명하는 튜플입니다. HTTP 상태 코드는 웹 페이지를 요청할 때 웹 서버가 반환하는 특수한 값입니다. 요청한 웹 페이지가 없는 경우 404 Not Found 상태 코드가 반환됩니다.

```swift
let http404Error = (404, "Not Found")
// http404Error의 유형은 (Int, String)이며, (404, "Not Found")와 동일합니다.
```

(404, "Not Found") 튜플은 Int와 String을 하나로 묶어 HTTP 상태 코드를 두 개의 별도 값, 숫자와 사람이 읽기 쉬운 설명,으로 나타냅니다. 이를 "유형이 (Int, String)인 튜플"로 설명할 수 있습니다.

당신은 어떤 유형의 순열이든 튜플로 만들 수 있으며, 당신이 원하는 만큼 많은 다른 유형을 포함시킬 수 있습니다. (Int, Int, Int) 또는 (String, Bool) 또는 당신이 필요로 하는 모든 순열은 만들 수 있습니다.

튜플의 내용을 별도의 상수 또는 변수로 분해하여 일반적으로 액세스할 수 있습니다.

```swift
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// "The status code is 404" 를 출력합니다.
print("The status message is \(statusMessage)")
// "The status message is Not Found" 를 출력합니다.
```

튜플의 일부 값만 필요한 경우 튜플을 분해할 때 밑줄 `(_)`로 해당 튜플의 일부를 무시할 수 있습니다.

```swift
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// "The status code is 404" 를 출력합니다.
```

또한 인덱스 번호를 사용하여 튜플의 개별 요소 값에 액세스할 수 있습니다.

```swift
print("The status code is \(http404Error.0)")
// "The status code is 404" 를 출력합니다.
print("The status message is \(http404Error.1)")
// "The status message is Not Found" 를 출력합니다.
```

튜플을 정의할 때 튜플의 개별 요소에 이름을 정할 수 있습니다.

```swift
let http200Status = (statusCode: 200, description: "OK")
```

요소의 이름을 정한 경우 요소 이름을 사용하여 해당 요소의 값을 액세스할 수 있습니다.

```swift
print("The status code is \(http200Status.statusCode)")
// "The status code is 200" 을 출력합니다.
print("The status message is \(http200Status.description)")
// "The status message is OK" 를 출력합니다.
```

튜플은 함수의 반환 값으로 특히 유용합니다. 웹 페이지를 검색하려는 함수는 페이지 검색의 성공 또는 실패를 설명하기 위해 (Int, String) 튜플 유형을 반환할 수 있습니다. 서로 다른 유형의 두 가지 별개의 값으로 이루어진 튜플을 반환함으로써 함수는 한 유형의 단일 값만 반환할 수 있는 경우보다 더 유용한 정보를 제공합니다. 자세한 내용은 "[다중 반환 값을 갖는 함수](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions#Functions-with-Multiple-Return-Values)"를 참조하십시오.

> 참고
> 
> 튜플은 관련된 단순한 값의 그룹에 유용합니다. 그러나 복잡한 데이터 구조를 만드는 데에는 적합하지 않습니다. 데이터 구조가 더 복잡할 경우에는 튜플 대신 클래스 또는 구조체로 모델링하는 것이 좋습니다. 자세한 내용은 "[구조체와 클래스](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures)"를 참조하십시오.

## 옵셔널 (Optionals)

옵셔널은 값이 없을 때 사용됩니다. 옵셔널은 두 가지 가능성을 나타냅니다: 값이 존재하면 옵셔널을 언래핑하여 그 값을 액세스할 수 있고, 값이 없으면 값에 액세스할 수 없습니다.

> 참고
> 
> 옵셔널의 개념은 C나 Objective-C에 존재하지 않습니다. Objective-C에서 가장 비슷한 개념은 객체를 반환하는 메소드에서 nil을 반환하는 것입니다. 여기서 nil은 "유효한 객체가 없음"을 의미하기 때문에 객체에만 적용되며, 구조체, 기본 C 타입 또는 열거형 값에는 적용되지 않습니다. 이러한 유형의 경우 Objective-C 메소드는 일반적으로 특별한 값(예: NSNotFound)을 반환하여 값이 없음을 나타냅니다. 이 접근 방식은 메소드의 호출자가 특수한 값을 테스트해야 한다는 것을 가정하며, 기억해야 한다는 것입니다. Swift의 옵셔널을 사용하면 특별한 상수가 필요하지 않고 모든 유형의 값이 없음을 나타낼 수 있습니다.

다음은 값이 없을 때 옵셔널을 처리하는 방법의 예입니다. Swift의 Int 유형에는 String 값을 Int 값으로 변환하려는 초기자가 있습니다. 그러나 모든 문자열을 정수로 변환할 수는 없습니다. 문자열 "123"은 숫자값 123으로 변환할 수 있지만 "hello, world" 문자열에는 변환 가능한 숫자 값이 없습니다.

아래 예제는 String을 Int로 변환하기 위해 초기자를 사용합니다.

```swift
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)
// convertedNumber는 Int? 또는 옵셔널 Int로 추론됩니다.
```

초기자가 실패할 수 있으므로, 옵셔널 Int가 반환됩니다. 옵셔널 Int는 Int가 아니라 Int?로 작성됩니다. 물음표는 그 안에 포함된 값이 선택적임을 나타내며, 일부 Int 값이 포함될 수도 있고, 전혀 포함되지 않을 수도 있습니다. (Bool 값이나 String 값과 같은 다른 값을 포함할 수 없습니다. Int이거나 없거나 입니다.)

### nil

nil 특별 값을 사용하여 옵셔널 변수를 값 없는 상태로 설정할 수 있습니다.

```swift
var serverResponseCode: Int? = 404
// serverResponseCode에는 실제 Int 값인 404가 포함됩니다.
serverResponseCode = nil
// serverResponseCode에는 이제 값이 없습니다.
```

> 참고
> 
> 옵셔널이 아닌 상수 및 변수에 nil을 사용할 수 없습니다. 코드의 상수나 변수가 특정 조건에서 값이 없는 상태에서 작동해야 하는 경우 항상 적절한 유형의 옵셔널 값으로 선언하세요.


기본 값을 제공하지 않고 옵셔널 변수를 정의하면 해당 변수는 자동으로 nil로 설정됩니다.

```swift
var surveyAnswer: String?
// surveyAnswer는 자동으로 nil로 설정됩니다.
```

> 참고
> 
> Objective-C의 nil은 Swift의 nil과 동일하지 않습니다. Objective-C에서 nil은 존재하지 않는 객체를 가리키는 포인터입니다. Swift에서 nil은 특정 유형의 값이 없음을 나타납니다. Swift옵셔널은 객체 유형뿐만 아니라 모든 유형을 nil로 설정할 수 있습니다.

### If 문과 강제 언래핑

옵셔널 변수를 nil 값과 비교하여 if 문을 사용하여 값의 존재 유무를 확인할 수 있습니다. 이 비교는 "equal to" 연산자(`==`) 또는 "not equal to" 연산자(`!=`)를 사용하여 수행합니다.

옵셔널 변수에 값이 있으면 nil이 아닌 것으로 평가됩니다.:

```swift
if convertedNumber != nil {
    print("convertedNumber contains some integer value.")
}
// 출력 결과 : "convertedNumber contains some integer value."
```

옵셔널에 값이 있다는 것을 확실히 확인했다면, 옵셔널 변수 이름 뒤에 느낌표(!)를 추가하여 해당 값을 직접적으로 접근할 수 있습니다. 느낌표는 "이 옵셔널에 정말로 값이 있어, 해당 값을 사용해주세요"라고 알려주는 것과 같습니다. 이를 옵셔널의 강제 언래핑이라고 합니다.

```swift
if convertedNumber != nil {
    print("convertedNumber has an integer value of \(convertedNumber!).")
}
// 출력 결과: "convertedNumber has an integer value of 123."
```
if 문에 대한 자세한 내용은 [Control Flow](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow)를 참조하세요.

> 참고
> 
> 존재하지 않는 옵셔널 값을 언래핑하려고 하면 런타임 오류가 발생합니다. 반드시 강제 언래핑하기 전에 옵셔널에 값이 있는지 확인하십시오.

### 옵셔널 바인딩

옵셔널 바인딩은 옵셔널이 값을 포함하는지 확인하고, 값을 포함하면 해당 값을 일시적인 상수나 변수로 사용할 수 있게 하는 기능입니다. if문과 while문을 사용하여 옵셔널 내부의 값이 있는지를 확인하고, 그 값을 상수나 변수로 추출하여 단일 작업의 일부로 사용할 수 있습니다. if문과 while문은 제어 흐름(Control Flow) 섹션에서 더 자세하게 다루고 있습니다.

if문에 옵셔널 바인딩을 적용하는 방법은 다음과 같습니다:

```swift
if let <#constantName#> = <#someOptional#> {
   <#statements#>
}
```

옵셔널 섹션에서의 possibleNumber 예제를 강제 언랩핑 대신 옵셔널 바인딩을 사용하여 다시 작성할 수 있습니다:

```swift
if let actualNumber = Int(possibleNumber) {
    print("해당 스트링 \"\(possibleNumber)\"의 정수 값은 \(actualNumber)입니다.")
} else {
    print("해당 스트링 \"\(possibleNumber)\"이(가) 정수로 변환될 수 없습니다.")
}
// 결과: "해당 스트링 "123"의 정수 값은 123입니다."
```

이 코드는 다음과 같이 읽을 수 있습니다:

“Int(possibleNumber)가 옵셔널 값이 존재한다면, 그 값을 actualNumber라는 새 상수에 입력하세요.”

변환에 성공하면 actualNumber 상수는 if문의 첫 번째 브랜치에서 사용하게 됩니다. 이미 옵셔널에 포함된 값으로 초기화되어 있으므로, ! 접미사를 사용하여 해당 값을 사용할 필요가 없습니다. 이 예에서는 actualNumber를 사용하여 변환 결과를 출력하기만 합니다.

값을 추출한 후 원본 옵셔널 상수나 변수에 대한 참조가 필요하지 않은 경우, 같은 이름을 새로운 상수나 변수에 사용할 수 있습니다:

```swift
let myNumber = Int(possibleNumber)
// myNumber는 옵셔널 정수입니다.
if let myNumber = myNumber {
    // myNumber는 옵셔널이 아닌 정수입니다.
    print("내 숫자는 \(myNumber)입니다.")
}
// 출력: "내 숫자는 123입니다."
```

이 코드는 기존 예제와 동일하게 myNumber가 값을 담고 있는지 확인하고 있습니다. myNumber가 값이 있으면, 해당 값을 가지는 새로운 상수의 이름을 myNumber로 지정합니다. if문의 본문 안에서 myNumber를 쓰면, 새로 생성된 옵셔널이 아닌 상수를 참조하게 됩니다. if문의 시작 전과 끝에서 myNumber를 사용하면 옵셔널 정수 상수를 참조하게 됩니다.

이러한 코드가 흔하기 때문에, 옵셔널 값을 풀기 위해 상수나 변수의 이름만을 작성하여 간략하게 쓸 수 있습니다. 새로 풀린 상수나 변수는 옵셔널 값과 동일한 이름을 갖게 됩니다.

```swift
if let myNumber {
    print("내 숫자는 \(myNumber)입니다.")
}
// 출력: "내 숫자는 123입니다."
```

상수와 변수 모두에게 옵셔널 바인딩을 사용할 수 있습니다. if구문의 첫 번째 브랜치에서 myNumber의 값을 수정하고 싶다면, `if var myNumbe`r와 같이 작성하여, 옵셔널 내부의 값을 상수 대신 변수로 사용할 수 있습니다. if문 본문 내에서 myNumber를 수정하면, 해당 지역 변수에만 영향을 끼치며, 기존 옵셔널 상수나 변수에는 영향을 끼치지 않습니다.

if문 안에, 쉼표로 구분하여 여러 개의 옵셔널 바인딩과 부울 조건을 사용할 수 있습니다. 옵셔널 바인딩 중 하나라도 nil이거나 부울 조건 중 하나라도 false라면, 전체 if문의 조건은 false로 간주됩니다. 다음 if문들은 모두 동일합니다:

```swift
if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
    print("\(firstNumber) < \(secondNumber) < 100")
}
// 출력: "4 < 42 < 100"


if let firstNumber = Int("4") {
    if let secondNumber = Int("42") {
        if firstNumber < secondNumber && secondNumber < 100 {
            print("\(firstNumber) < \(secondNumber) < 100")
        }
    }
}
// 출력: "4 < 42 < 100"
```

> 참고
> 
> if문의 옵셔널 바인딩 상수와 변수는 if문 안에서만 사용할 수 있습니다. 반면, guard문의 상수와 변수는 [Early Exit](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow#Early-Exit) 섹션에서 설명한대로 guard문 이후의 코드에서도 사용할 수 있습니다.

### 암시적으로 해제된 옵셔널(Implicitly Unwrapped Optionals)

앞서 말했듯이, 옵셔널은 "값이 없을 수 있다"는 것을 나타내는 상수나 변수입니다. 옵셔널은 값이 존재하는지 if문으로 확인할 수 있으며, 옵셔널 바인딩을 사용하여 옵셔널 값에 접근할 수 있습니다.

때로는 프로그램의 구조에서 옵셔널 값이 처음 설정된 후 항상 존재할 것을 알 수 있는 경우가 있습니다. 이 경우 옵셔널의 값을 확인하고 매번 접근할 때마다 값을 해제하는 의무를 없애는 것이 편리합니다.

이러한 종류의 옵셔널은 암시적으로 해제된 옵셔널로 정의됩니다. 암시적으로 해제된 옵셔널은 선택적으로 값을 확인할 필요가 없으므로 항상 값이 존재할 수 있다고 안전하게 가정할 수 있습니다. 

암시적으로 해제된 옵셔널을 만드는 방법은 물음표(String?) 대신 느낌표(String!)를 사용하여 타입 뒤에 붙입니다. 옵셔널 이름을 사용할 때 느낌표 대신 옵셔널의 타입 뒤에 느낌표를 사용합니다.

Swift에서 암시적으로 해제된 옵셔널의 주요 사용법은 클래스 초기화에서 설명한 [Unowned References and Implicitly Unwrapped Optional Properties](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting#Unowned-References-and-Implicitly-Unwrapped-Optional-Properties)입니다. 

암시적으로 해제된 옵셔널은 내부적으로 일반적인 옵셔널입니다. 하지만 해제할 필요 없이 비-옵셔널 값처럼 사용할 수 있습니다. 다음 예에서는 명시적인 String을 사용하여 래핑된 값에 접근할 때 옵셔널 문자열과 암시적으로 해제된 문자열의 동작 차이가 나타납니다.

``` swift
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // 느낌표가 필요

let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString // 느낌표가 필요 없음
```

암시적으로 해제된 옵셔널은 필요할 때 옵셔널이 강제로 해제되도록 허용하는 것으로 생각할 수 있습니다. 암시적으로 해제된 옵셔널 값을 사용할 때, Swift는 먼저 일반적인 옵셔널 값으로 사용하려고 시도하고, 옵셔널로 사용할 수 없는 경우 Swift는 값을 강제로 해제합니다. 위의 코드에서 암시적 문자열 변수인 assumeString은 암시적 문자열 변수 implicitString에 값을 할당하기 전에 값이 강제로 해제됩니다. 이는 implicitString이 String이라는 명시적이고 옵셔널이 아닌 유형을 가지고 있기 때문입니다. 아래의 코드에서 optionalString은 명시적 유형이 없으므로 일반적인 옵셔널입니다.

```swift
let optionalString = assumedString
// optionalString의 타입은 "String?"이며, assumedString은 강제로 해제되어있지 않습니다.
```

암시적으로 해제된 옵셔널이 nil일 때, 래핑된 값을 가져오려고하면 런타임 에러가 발생합니다. 값이 없는 일반적인 옵셔널에 느낌표를 붙일 때와 정확히 동일한 결과가 나타납니다.

암시적으로 해제된 옵셔널이 nil인지 아닌지 검사하는 방법은 일반적인 옵셔널을 검사하는 방법과 동일합니다.

```swift
if assumedString != nil {
    print(assumedString!)
}
// "An implicitly unwrapped optional string." 출력
```

암시적으로 해제된 옵셔널도 옵셔널 바인딩과 함께 사용하여 값을 확인하고 해제할 수 있습니다.

```swift
if let definiteString = assumedString {
    print(definiteString)
}
// "An implicitly unwrapped optional string." 출력
```

> 중요
> 
> 나중에 변수가 nil이 되는 경우 암시적으로 해제된 옵셔널을 사용하지 마세요. 변수의 수명동안 nil 값을 확인해야하는 경우 항상 일반적인 옵셔널 타입을 사용하세요.

## 오류 처리

프로그램이 실행 중에 발생할 수 있는 오류 조건에 대응하려면 오류 처리를 사용합니다.

값의 존재 또는 부재를 사용하여 함수의 성공 또는 실패를 전달할 수 있는 옵셔널과 달리 오류 처리는 실패의 근본적인 원인을 결정하고 필요한 경우 오류를 프로그램의 다른 부분으로 전파할 수 있습니다.

함수가 오류 조건을 만나면 오류를 throw합니다. 그 함수의 호출자는 이 오류를 catch하여 적절하게 대응할 수 있습니다.

```swift
func canThrowAnError() throws {
    // 이 함수는 오류를 throw할 수도, 안 할 수도 있습니다.
}
```

함수가 오류를 throw할 수 있음을 나타내기 위해서는 선언에 throws 키워드를 포함해야 합니다. 오류를 throw할 수 있는 함수를 호출할 때는 try 키워드를 표현식 앞에 삽입해야 합니다.

Swfit는 오류를 자동으로 현재 범위에서 빠져나가서 catch 절에 의해 처리될 때까지 전파합니다.

```swift
do {
    try canThrowAnError()
    // 오류가 발생하지 않았습니다.
} catch {
    // 오류가 발생했습니다.
}
```

do 문은 새로운 포함 범위를 생성하여 하나 이상의 catch 절로 오류를 전파할 수 있도록 합니다.

다음은 오류 처리를 사용하여 다른 오류 조건에 대응하는 방법의 예시입니다.

```swift
func makeASandwich() throws {
    // ...
}

do {
    try makeASandwich()
    eatASandwich()
} catch SandwichError.outOfCleanDishes {
    washDishes()
} catch SandwichError.missingIngredients(let ingredients) {
    buyGroceries(ingredients)
}
```

이 예시에서 makeASandwich() 함수는 깨끗한 접시가 없거나 어떤 재료가 누락되면 오류를 throw합니다. makeASandwich() 함수가 오류를 throw할 수 있다는 것을 고려하여 함수 호출은 try 표현식으로 래핑됩니다. 함수 호출을 do 문으로 래핑함으로써 throw된 어떤 오류든 해당 catch 절로 전달됩니다.

오류가 throw되지 않으면 eatASandwich() 함수가 호출됩니다. 만약 오류가 발생하고 SandwichError.outOfCleanDishes 경우와 일치하면 washDishes() 함수가 호출됩니다. 만약 오류가 발생하고 SandwichError.missingIngredients 경우와 일치하면 catch 패턴에 의해 캡처된 관련 `[String]` 값과 함께 `buyGroceries(_:)` 함수가 호출됩니다.

오류를 throw하고 catch하는 방법과 오류 전달에 대한 자세한 내용은 "[Error Handling](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling)"을 참조하세요.

## Assertions와 Preconditions

Assertions와 Preconditions은 런타임에서 발생하는 체크입니다. 이들은 코드를 실행하기 전 필수적인 조건이 충족되었는지 확인하는데 사용됩니다. 만약 Assertion이나 Precondition의 불리언 조건이 참이면 코드 실행은 일반적으로 계속 진행됩니다. 그러나 이 조건이 거짓이라면 프로그램의 현재 상태는 유효하지 않기 때문에 실행이 종료되고 앱이 종료됩니다.

Assertions와 Preconditions은 코딩할 때 시작했던 가정과 예상을 표현하는 데 사용됩니다. 이를 코드의 일부로 포함시켜 에러를 찾고, 생산환경에서 문제를 감지하는 데 도움이 됩니다.

Assertions와 Preconditions는 런타임에서 예상을 검증하는데 사용됩니다. Error Handling에서 다루었던 에러 상황들과 달리, Assertions와 Preconditions은 복구 가능하거나 예상 가능한 에러에 대해서는 사용되지 않습니다. 왜냐하면 실패한 Assertion이나 Precondition은 프로그램의 상태가 유효하지 않음을 나타내기 때문에 실패한 Assertion을 잡을 방법이 없기 때문입니다.

Assertions와 Preconditions를 사용하는 것은 유효하지 않은 조건이 발생하지 않도록 코드를 디자인하는 것에 대한 대체수단은 아닙니다. 그러나 유효한 데이터와 상태를 강제적으로 적용함으로써, 앱이 유효하지 않은 상태에 도달했을 때 더 예측 가능하게 종료되며, 문제를 디버깅하는 데 도움이 됩니다. 일단 유효하지 않은 상태가 감지되면 실행을 중단함으로써 그 상태가 야기한 손상도 제한할 수 있습니다.

Assertion과 Precondition의 차이는 체크되는 시점입니다. Assertion은 디버그 빌드에서만 체크되지만 Precondition은 디버그와 프로덕션 빌드 모두에서 체크됩니다. 프로덕션 빌드에서는 Assertion 내부의 조건이 평가되지 않습니다. 이는 개발 중에 원하는 만큼 많은 Assertion을 사용할 수 있으며, 프로덕션에서의 성능에 영향을 미치지 않습니다.

### Assertions로 디버깅하기

Assertion은 Swift 표준 라이브러리의 `assert(_:_:file:line:)` 함수를 호출하여 작성합니다. true 또는 false를 반환하는 표현식과 조건이 false인 경우 표시 할 메시지를 전달합니다. 

다음은 코드 예시입니다.

```swift
let age = -3
assert(age >= 0, "A person's age can't be less than zero.")
```

위 코드 예제에서, 만약 `age >= 0`이 참이라면, 그것은 즉, age의 값이 0 이상인 경우가 됩니다. age가 음수인 경우, 즉 위의 코드 예제와 같은 경우, `age >= 0`이 거짓이되며 Assertion은 실패하여 애플리케이션이 종료됩니다.

Assertion 메시지를 생략할 수도 있습니다. 예를 들어, 조건이 이미 체크된 경우 Assertion 메시지는 반복될 뿐입니다.

```swift
assert(age >= 0)
```

코드가 이미 조건을 확인하는 경우에는 Assertion이 실패했음을 나타내기 위해 `assertionFailure(_:file:line:)` 함수를 사용합니다.

```swift
if age > 10 {
    print("You can ride the roller-coaster or the ferris wheel.")
} else if age >= 0 {
    print("You can ride the ferris wheel.")
} else {
    assertionFailure("A person's age can't be less than zero.")
}
```

### Preconditions 강제하기

Precondition은 조건이 거짓일 가능성이 있지만 코드를 계속 실행하려면 반드시 참이여야 할 때 사용합니다. 예를 들어, subscript가 범위를 벗어나지 않았는지 또는 함수에 유효한 값이 전달되었는지 확인하려면 Precondition을 사용합니다.

Precondition은 `precondition(_:_:file:line:)` 함수를 호출하여 작성합니다. true 또는 false를 반환하는 표현식과 조건이 false인 경우 표시 할 메시지를 전달합니다. 

다음은 코드 예시입니다.

```swift
// subscript의 구현에서...
precondition(index > 0, "Index must be greater than zero.")
```

예를 들어 switch의 default 케이스가 사용되었지만 모든 유효한 입력 데이터가 switch의 다른 케이스 중 하나에서 처리되었어야 하는 경우와 같이 오류가 발생했음을 나타내기 위해 `preconditionFailure(_:file:line:)` 함수를 호출할 수도 있습니다.

> 참고
> 
> `-Ounchecked` 모드에서 컴파일하는 경우, Precondition은 체크되지 않습니다. 컴파일러는 Precondition이 항상 참임을 가정하고 코드를 최적화합니다. 그러나 `fatalError(_:file:line:)` 함수는 최적화 설정에 관계없이 항상 실행을 중단합니다.
> 
> 초기 개발 또는 프로토타이핑 단계에서, `fatalError("Unimplemented")`를 스텁으로 구현해 두면 되며, 이런 경우 Assertion이나 Precondition과 달리 fatal error는 최적화되지 않기 때문에 스텁 구현 실행 시 항상 실행이 종료됩니다.

