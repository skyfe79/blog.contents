---
title: "Swift Type Eraser Wrapper 패턴 이해하기"
date: 2022-01-29T05:36:29Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

Swift 언어를 잘 활용하기 위해서는 Protocol을 잘 다뤄야 한다. 특히 라이브러리를 제작하고 배포할 때 더 그러하다. Protocol을 사용하면 자주 마주치는 컴파일 오류가 있다.

```swift
Protocol 'Somehting' can only be used as a generic constraint because it has Self or associated type requirements
```

Self에 제약이 걸려 있거나 Associated type(이하 연관 타입)이 필요한 프로토콜을 사용할 때이다. Self 제약과 연관 타입이 없는 프로토콜을 사용하는 일은 어렵지 않다.

## Self 제약과 연관 타입이 없는 Protocol

`Person` 이라고 하는 Protocol을 아래와 같이 정의하자.

```swift
protocol Person {
    var name: String { get set }
    var age: Int { get set }
    func greeting() -> String
}
```

Person 프로토콜은 Self 제약과 연관 타입이 없기 때문에 아래처럼 변수 타입, 컬렉션 타입 그리고 함수의 인자 타입과 반환 타입으로 사용할 수 있다.

```swift
struct KoreanPerson: Person {
    var name: String
    var age: Int
    
    func greeting() -> String {
        return "안녕하세요. 제 이름은 \(name)이고 나이는 \(age)"
    }
}

struct EnglishPerson: Person {
    var name: String
    var age: Int
    
    func greeting() -> String {
        return "Hello, My name is \(name) and I'm \(age) years old."
    }
}

let someone: Person = KoreanPerson(name: "김철수", age: 31)

var people: [Person] = [
	KoreanPerson(name: "김철수", age: 31),
	EnglishPerson(name: "Berry Chan", age: 21)
]

func meet(person: Person) {
    print(person.greeting())
}

func lastPerson(people: [Person]) -> Person? {
    return people.last
}
```

하지만 Protocol에 Self 제약과 연관 타입이 필요해지면 상황은 돌변한다.


## Self 제약이 있는 Protocol

`Person` 프로토콜을 아래와 같이 정의해 보자.

```swift
protocol Person {
    var name: String { get set }
    var age: Int { get set }
    func greeting() -> String
    func meet(other: Self)
}
```

그러면 아래 오류가 발생한다.

```swift
Protocol 'Person' can only be used as a generic constraint because it has Self or associated type requirements
```

이 오류가 발생하는 이유는 `Self` 는 Protocol을 채택한 최종 타입을 의미하는데 컴파일 타임에는 Self 가 무엇인지 알 수 없기 때문이다. 

위 `Person` 선언을 아래와 같이 해 주는 것이 좋다.

```swift
protocol Person {
    var name: String { get set }
    var age: Int { get set }
    func greeting() -> String
    func meet(other: Person)
}
```

## Associated Type이 있는 Protocol

연관타입은 Generic Struct, Generic Enum, Generic Class, Generic Functions 처럼 일반적인 프로토콜을 정의하기 위해 사용한다. 다만 프로토콜은 아래처럼 Generic 선언을 지원하지 않는다.

```swift
// 지원하지 않는 코드
protocol Container<Item> { 
	...
}
```

대신 연관 타입을 사용하여 선언해야 한다.

```swift
protocol Container {
	associatedtype Item
}
```

위처럼 연관 타입을 가진 프로토콜도 변수 타입, 컬렉션 타입 그리고 함수의 인자 타입과 반환 타입으로 사용할 수 없다. 사용하면 오류가 발생한다.

```swift
let containers: [Container] = []

// Error
Protocol 'Container' can only be used as a generic constraint because it has Self or associated type requirements
```

연관타입이 특정 타입으로 특징되어야 컴파일타임에 변수의 할당 크기 등을 정할 수 있는데, 위 예제에서는 `Container` 의 연관 타입인 `Item` 이 어떤 타입인지 정해지지 않았기 때문에 오류가 발생한다.

## Associated Type + Generic Type

이럴 경우, 연관 타입을 가진 프로토콜을 제네릭 구체 타입이 채택하면 컴파일 오류를 없앨 수 있다.

```swift
protocol Container {
    associatedtype Item
    var items: [Item] { get set }
    var count: Int { get }
    mutating func push(item: Item)
    mutating func pop() -> Item?
}

extension Container {
    var count: Int {
        return items.count
    }
}
```

위 프로토콜을 제네릭 구체 타입이 채택한다.

```swift
struct StackContainer<Item>: Container {
    var items: [Item] = []
    mutating func push(item: Item) {
        items.append(item)
    }
    mutating func pop() -> Item? {
        return items.popLast()
    }
}
```

아래처럼 사용할 수 있다.

```swift
var containers: [StackContainer<Int>] = [
    StackContainer()
]

containers[0].push(item: 1)
containers[0].push(item: 2)
containers[0].push(item: 3)

print(containers[0].count)
```

이렇게 사용할 수 있는 이유는 `<Int>` 로 제네릭 타입을 `Int` 타입으로 결정하면 `Container`의 연관 타입인 `Item` 타입이 `Int` 타입으로 결정되어 컴파일이 가능하기 때문이다. 그런데 우리는 StackContainer에 Int 말고 다른 타입도 담고 싶을 수도 있다. 

```swift
var anyContainer: StackContainer<Any> = StackContainer()
anyContainer.push(item: 1)
anyContainer.push(item: "Hello")
anyContainer.push(item: 12.5)
```

제네릭 타입을 `Any` 타입으로 정하여 위처럼 구현하는 것은 쉽다. 하지만 anyContainer에 담은 값을 실제로 사용할 때 문제가 발생한다. `Any` 타입은 타입 정보를 지우기 때문에 아래와 같이 타입 캐스팅이 반드시 필요하다. 

```swift
if let item = anyContainer.pop() {
	// item의 타입을 모른다.
	// 그래서 아래와 같이 타입 캐스팅이 필요하다.
	switch item {
    case is Int:
        print("Hi, I'm Int. and value is \(item)")
    case is String:
        print("Hi, I'm String. and value is \(item)")
    case is Double:
        print("Hi, I'm Double. and value is \(item)")
    default:
        print("I don't know what am I")
    }
}
```

그리고 `StackContainer<Int>`, `StackContainer<Double>` 등을 하나의 배열에 담아 사용할 경우도 있다.

```swift
var containers = [
    StackContainer<Int>(),
    StackContainer<Double>()
]
```

하지만 위 코드는 오류가 발생한다. `containers` 의 타입을 정할 수 없기 때문이다. 컴파일러는 `containers` 타입이 모호하니 `[Any]` 로 명확하게 타입을 선언하는 것을 제안한다.

```swift
Heterogeneous collection literal could only be inferred to '[Any]'; add explicit type annotation if this is intentional
```

앱을 개발하다 보면 실제로 `containers` 같은 변수가 필요한 경우가 많다. 예를 들어 UICollectionView의 Cell을 그리기 위해서 Model을 사용할 때, 여러 Model을 담을 수 있는 컬렉션이 필요하다.

```swift
protocol Model {
    associatedtype Data
    var data: Data { get set }
}

struct CellModel<Data>: Model {
    var data: Data = []
}

var dataCollection = [
    CellModel<String>(),
    CellModel<Int>()
    ...
]
```

이 때, dataCollection을 `var dataCollection: [Any]` 로 선언하면 위에서 보았듯이 값이 필요할 때 타입 캐스팅이 필요해 진다.  `var dataCollection: [Any]`를 사용하지 않고 이 문제를 해결 할 수 있는 방법이 있을까?  여러 제네릭 타입을 가진 `CellModel<T>` 을 하나의 배열에 담을 수 있고 배열 원소는 타입 정보(속성과 메서드)를 잃지 않는 타입을 만들 수 있을까?

## Type Eraser Wrapper

Type Eraser Wrapper 패턴을 사용하면 위에서 보았던 문제를 어느정도 해결할 수 있다. 즉,  `var dataCollection: [Any]` 처럼 선언하여 아무 `CellModel` 을 추가할 수 있지만 실제로 사용할 때는 타입 캐스팅 없이 사용할 수 있다. 

```swift
var dataCollection: [AnyCellModel<Int>] = [
    AnyCellModel(IntCellModel1()),
    AnyCellModel(IntCellModel2())
]

// 또는

var dataCollection: [AnyCellModel] = [
    AnyCellModel(IntCellModel)
    AnyCellMidel(StringCellModel)
]
```

위 코드를 보면 하나는 제네릭 타입이 있는 Type Eraser Wrapper이고 다른 하나는 제네릭 타입이 없는 Type Eraser Wrapper 이다. 글을 진행하면서 왜 이렇게 두 경우로 나뉘는지도 설명한다.

개인적으로 Type Eraser Wrapper 패턴을 잘 알아야 Swift를 좀 더 고급지게 사용할 수 있게 된다고 생각한다. 그리고 Swift 및 iOS 프래임워크는 이미 Type Eraser Wrapper 타입이 많이 있다. [Type-Erasing Wrappers](https://developer.apple.com/documentation/swift/swift_standard_library/collections/supporting_types)  문서를 보면 아래와 같은 타입이 있다.

- AnySequence
- AnyCollection
- AnyBidirectionalCollection
- AnyRandomAccessCollection
- AnyIterator
- AnyIndex
- AnyHashable

그리고 Combine의 [AnyPublisher](https://developer.apple.com/documentation/combine/anypublisher/) 와 [AnyCancellable](https://developer.apple.com/documentation/combine/anycancellable)도 자주 사용하는 Type Eraser Wrapper 다. 

이 글에서는 연관 타입을 갖는 `Component` 프로토콜을 정의하고 여러 `Component`를 컬렉션에 담을 수 있도록 `AnyComponent` Type Eraser Wrapper 를 만들 것이다. 우선 Component 프로토콜을 정의해 보자.

```swift
protocol Component {
    associatedtype Content
		var id: String { get }
    func contentView() -> Content
    func layoutSize() -> CGSize
}
```

아래 그림은 참고 문서에 기재한 [Breaking Down Type Eraser in Swift](https://bignerdranch.com/blog/breaking-down-type-erasure-in-swift/) 글에서 나오는 그림을 위 Component에 맞춰 그린 것으로 Swift 표준 라이브러리에서 Type Eraser Wrapper 를 만들 때 사용하는 패턴을 그림을 표현한 것이다.

<img width="950" alt="swift-type-eraser-wrapper" src="https://user-images.githubusercontent.com/309935/151649032-9c7e4c3a-45a9-4ba2-a402-e8678cca5303.png">

그림에서 녹색 사각형으로 표시한 3개의 타입이 Type Eraser Wrapper 패턴을 구성한다. 보기에는 복잡해 보이지만 내용은 간단하다. 축약하여 설명하면 다음과 같다.

### Abstract Base

- Any 타입에서 지워지는 타입의 정보(속성과 메서드)를 가지고 있다.
- 추상클래스로 구현한다.

### Private Box

- AbstractBase를 상속받는 클래스다.
- Component 프로토콜을 채택하여 구현한 구체 인스턴스를 담는 상자다.
- 위 그림에서 예를 들면 LabelComponent를 담게 되는 상자다.

### Public Wrapper

- Protocol을 채택한다. 위 그림에서는 Component를 채택한다.
- 내부에 Private Box를 가지고 있다.
- Private Box에 구체 인스턴스를 담는다.
- Component 인터페이스 구현을 box를 통해 구현하다.

Component 프로토콜의 Type Eraser Wrapper인 AnyComponent 를 구현해 보자. 제일 먼저 정의해야 할 것은 Abstract Base 클래스를 구현하는 것이다. 이 Base 클래스 역할을 Concrete Component를 담는 Box를 가리키기 위한 부모 포인터 역할을 한다. 

```swift
private class AnyComponentBase<Content>: Component {
    var id: String { fatalError("implement it!") }
    
    func contentView() -> Content {
        fatalError("implement it!")
    }
    
    func layoutSize() -> CGSize {
        fatalError("implement it!")
    }
}
```

Swift는 Abstract Class를 언어에서 지원하지 않기 때문에 fatalError() 함수를 사용하여 Abstract Class를 흉내낸다. 

그 다음으로 할 일은 Concrete Component를 담을 Private Box 클래스를 구현하는 것이다.

```swift
private class AnyComponentBox<ConcreteComponent: Component>: AnyComponentBase<ConcreteComponent.Content> {
    
    var concrete: ConcreteComponent
    
    init(_ concrete: ConcreteComponent) {
        self.concrete = concrete
    }
    
    override var id: String {
        return concrete.id
    }
    
    override func contentView() -> ConcreteComponent.Content {
        return concrete.contentView()
    }
    
    override func layoutSize() -> CGSize {
        return concrete.layoutSize()
    }
}
```

위 코드에서 AnyComponentBox는 AnyComponentBase를 상속하며 제네릭 타입으로 Component 프로토콜을 채택한 ConcreateComponent를 받는다. AnyComponentBase는 ConcreateComponent의 Content를 사용한다. AnyComponentBox는 추상 클래스인 AnyComponentBase의 메서드를 오버라이드하고 내부 구현을 ConcreateComponent에 위임한다.

마지막으로 개발자가 직접 사용할 Public Wrapper인 AnyComponent를 구현해 보자.

```swift
public class AnyComponent<Content>: Component {
    // Content를 담는 컴포넌트 박스를 표현해야 하므로
    // Abstract class 포인터로 Box를 가리키게 한다.
    private let box: AnyComponentBase<Content>
    
    init<ConcreteComponent: Component>(_ concrete: ConcreteComponent) where ConcreteComponent.Content == Content {
        self.box = AnyComponentBox(concrete);
    }
    
    var id: String {
        return box.id
    }

    func contentView() -> Content {
        return box.contentView()
    }

    func layoutSize() -> CGSize {
        return box.layoutSize()
    }
}
```

AnyComponent의 구현 내용은 AnyComponentBox와 비슷하다. 차이점은 AnyComponent는 Component 프로토콜을 채택하고 AnyComponentBox는 추상클래스인 AnyComponentBase를 상속한다는 점이 다르다. 

AnyComponent는 제네릭 생성자를 사용해 Concrete Component를 생성 인자로 받는다. 이 때 주의할 점은 ConcreteComponent의 Content와 AnyComponent 의 제네릭 타입인 Content가 같아야 한다. 그래야 AnyComponentBox로 랩핑 할 수 있다.

AnyComponent 생성자에서 AnyComponentBox로 ConcreteComponent를 랩핑하고 인스턴스를 AnyComponentBase 클래스 포인터로 가리켜 유지(retain)한다. 이 포인터를 사용해 메서드 구현을 AnyComponentBox에 위임한다.

## AnyComponent 사용해 보기

지금까지 구현한 AnyComponent를 사용해 보기 위해서 LabelComponent와 ButtonComponent를 구현해 보자. 

```swift
struct LabelComponent: Component {
    var id: String {
        return "MyLabel"
    }
    func contentView() -> UIView {
        return UILabel(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 100, height: 50)
    }
}

struct ButtonComponent: Component {
    var id: String {
        return "MyButton"
    }
    func contentView() -> UIView {
        return UIButton(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 200, height: 100)
    }
}
```

LabelComponent와 ButtonComponent를 하나의 배열에 담기 위해서 어떻게 해야할까?

```swift
let components = [
    LabelComponent(),
    ButtonComponent()
]
```

위처럼 선언하면 `Heterogeneous collection literal could only be inferred to '[Any]'; add explicit type annotation if this is intentional`오류가 발생한다. 타입을 명확하게 선언하기 위해서 `[Component]` 로 선언 해 보자. (참고: 여기서 `[Any]` 로 선언하는 것은 타입 정보를 잃기 때문에 우리가 원하는 선언이 아니다.)

```swift
let components: [Component] = [
    LabelComponent(),
    ButtonComponent()
]
```

 위처럼 선언하면 이제는 이런 오류가 발생한다. `Protocol 'Component' can only be used as a generic constraint because it has Self or associated type requirements` 즉, Self 제약 또는 연관 타입이 있는 Component 프로토콜을 사용해 타입을 선언할 수 없다고 한다. 그 이유는 이전에 설명했지만 컴파일 타임에 연관 타입의 크기를 알 수 없기 때문에 컴파일러가 컴파일 할 수 없기 때문이다.

바로 이런 상황이 Type Eraser Wrapper인 AnyComponent가 필요한 시점이다. AnyComponent를 사용해서 다시 선언해 보자.

```swift
let components: [AnyComponent<UIView>] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
]
```

위처럼 선언하면 오류나 경고 없이 잘 컴파일된다! 실제로 사용해 보자.

```swift
components.forEach { component in
    print(component.id)
    print(component.contentView())
    print(component.layoutSize())
}
```

결과는 아래와 같다.

```swift
MyLabel
<UILabel: 0x7f7f5df0ce60; frame = (0 0; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600001151c20>>
(100.0, 50.0)

MyButton
<UIButton: 0x7f7f5df10be0; frame = (0 0; 0 0); opaque = NO; layer = <CALayer: 0x60000326c360>>
(200.0, 100.0)
```

사용하는 코드를 다시 살펴보자.

```swift
components.forEach { component in
    print(component.id)
    print(component.contentView())
    print(component.layoutSize())
}
```

위 코드에서 중요하게 살펴볼 점은 타입 정보를 잃지 않았다는 점이다. 만약 Any로 선언했다고 한다면, 아래와 같이 코드를 작성해야 했을 것이다.

```swift
let components: [Any] = [
    LabelComponent(),
    ButtonComponent()
]

components.forEach { component in
    switch component {
    case let labelComponent as LabelComponent:
        print(labelComponent.id)
        print(labelComponent.contentView())
        print(labelComponent.layoutSize())
        print()
    case let buttonComponent as ButtonComponent:
        print(buttonComponent.id)
        print(buttonComponent.contentView())
        print(buttonComponent.layoutSize())
        print()
    default:
        print("I don't know what am I")
    }
}
```

만약 새로운 컴포넌트를 정의한다고 가정해 보자. 

```swift
struct SwitchComponent: Component {
    var id: String {
        return "MySwitch"
    }
    func contentView() -> UIView {
        return UISwitch(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 80, height: 40)
    }
}

let components: [Any] = [
    LabelComponent(),
    ButtonComponent(),
    SwitchComponent()
]

components.forEach { component in
    switch component {
    case let labelComponent as LabelComponent:
        print(labelComponent.id)
        print(labelComponent.contentView())
        print(labelComponent.layoutSize())
        print()
    case let buttonComponent as ButtonComponent:
        print(buttonComponent.id)
        print(buttonComponent.contentView())
        print(buttonComponent.layoutSize())
        print()
    default:
        print("I don't know what am I")
    }
}
```

실행하면 `SwitchComponent` 에 대해서는 `I don't know what am I` 가 출력된다. 

```swift
MyLabel
<UILabel: 0x7fa4aff0afd0; frame = (0 0; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x6000000c1950>>
(100.0, 50.0)

MyButton
<UIButton: 0x7fa47ff04e50; frame = (0 0; 0 0); opaque = NO; layer = <CALayer: 0x6000023e3760>>
(200.0, 100.0)

I don't know what am I
```

즉, `[Any]` 를 사용하면 새로운 컴포너트가 추가될 때마다 switch 문에 case를 추가해야 한다. 하지만 AnyCompoent를 사용하면 그렇지 않다.

```swift
let components: [AnyComponent<UIView>] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
    AnyComponent(SwitchComponent())
]

components.forEach { component in
    print(component.id)
    print(component.contentView())
    print(component.layoutSize())
    print()
}
```

결과는 아래와 같다.

```swift
MyLabel
<UILabel: 0x7fdada70e5f0; frame = (0 0; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600002c45f90>>
(100.0, 50.0)

MyButton
<UIButton: 0x7fdaeb0060d0; frame = (0 0; 0 0); opaque = NO; layer = <CALayer: 0x600000f56800>>
(200.0, 100.0)

MySwitch
<UISwitch: 0x7fdaeb0073b0; frame = (0 0; 51 31); gestureRecognizers = <NSArray: 0x600000146700>; layer = <CALayer: 0x600000f55a40>>
(80.0, 40.0)
```

AnyComponent는 타입 정보를 기억하고 있기 때문에 이런 차이가 발생한다.

## 이렇게 하면 어떻게 될까?

위에서 구현한 ContentType은 사실 모두 UIView다. 이것을 아래처럼 바꿔보면 어떻게 될까?

```swift
struct LabelComponent: Component {
    var id: String {
        return "MyLabel"
    }
    func contentView() -> UILabel {
        return UILabel(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 100, height: 50)
    }
}

struct ButtonComponent: Component {
    var id: String {
        return "MyButton"
    }
    func contentView() -> UIButton {
        return UIButton(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 200, height: 100)
    }
}

struct SwitchComponent: Component {
    var id: String {
        return "MySwitch"
    }
    func contentView() -> UISwitch {
        return UISwitch(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 80, height: 40)
    }
}

let components: [AnyComponent<UIView>] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
    AnyComponent(SwitchComponent())
]
```

이럴 경우 AnyComponent<UIView>의 ContentType인 UIView와 AnyComponent(LabelComponent())의 UILabel이 다르고 AnyComponent(ButtonComponent())의 UIButton 그리고 AnyComponent(SwitchComponent())의 UISwitch와도 다르기 때문에 오류가 발생한다. 

```swift
Initializer 'init(_:)' requires the types 'UIView' and 'UILabel' be equivalent
Initializer 'init(_:)' requires the types 'UIView' and 'UIButton' be equivalent
Initializer 'init(_:)' requires the types 'UIView' and 'UISwitch' be equivalent
```

아래처럼 할 수 없을까?

```swift
let components: [AnyComponent] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
    AnyComponent(SwitchComponent())
]
```

## 제네릭 타입 없이 Type Eraser Wrapper 구현하기.

아이디어는 아래 코드에서 components 의 ContentType이 모두 UIView라는 것에서 출발한다. 

```swift
let components: [AnyComponent<UIView>] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
    AnyComponent(SwitchComponent())
]
```

이것을 UIView가 아닌 Any로 변경하는 것이다. 물론 이렇게 하면 Component의 contentView() 를 사용할 때 아래와 같은 타입 캐스팅이 필요하다.

```swift
if let contentView = components[0].contentView() as? UIView {
	container.addSubview(contentView)
}
```

따라서 구현 방법과 사용법에 따른 차이일 뿐 어느 방법이 더 낫다는 것은 아니다. 필요에 따라 선택하여 구현하고 사용하면 된다. 

위에서는 클래스 기반으로 Type Eraser Wrapper를 구현했었는데 이제는 구조체 기반으로 구현해보자. 구조체로 구현했을 때 차이점이 있다. 구조체는 상속을 지원하지 않기 때문에 Abstract Base 클래스를 프로토콜로 정의해야 한다.

우선 Component 프로토콜을 정의한다. 연관 타입을 가지고 있는 것은 이전과 동일하다.

```swift
protocol Component {
    associatedtype Content
    var id: String { get }
    func contentView() -> Content
    func layoutSize() -> CGSize
}
```

이제 AnyComponentBase 프로토콜을 정의하자. Component 프로토콜을 상속하지 않고 Shape 만 비슷하게 정의해야 한다. 그리고 Component의 ContentType을 AnyComponentBase에서는 Any로 변경해야 한다.

```swift
private protocol AnyComponentBase {
    var id: String { get }
    func contentView() -> Any
    func layoutSize() -> CGSize
}
```

AnyComponentBase를 정의한 다음 Concrete Component를 담을 Box를 AnyComponentBase 를 채택하여 구현한다.

```swift
private struct AnyComponentBox<ConcreteComponent: Component>: AnyComponentBase {
    var concrete: ConcreteComponent
    
    var id: String {
        return concrete.id
    }
    
    func contentView() -> Any {
        return concrete.contentView()
    }
    
    func layoutSize() -> CGSize {
        return concrete.layoutSize()
    }
    
    init(_ concrete: ConcreteComponent) {
        self.concrete = concrete
    }
}
```

구현내용은 아주 간단하다. 클래스 기반의 AnyComponentBox보다 더 간결하기 때문에 따로 설명은 하지 않겠다.

실제로 사용할 Public Type Eraser Wrapper인 AnyComponent를 구현해 보자.

```swift
public struct AnyComponent: Component {
    private let box: AnyComponentBase
    init<ConcreteComponent: Component>(_ concrete: ConcreteComponent) {
        if let anyComponent = concrete as? AnyComponent {
            self = anyComponent
        } else {
            box = AnyComponentBox(concrete)
        }
    }
    
    var id: String {
        return box.id
    }
    
    func contentView() -> Any {
        return box.contentView()
    }
    
    func layoutSize() -> CGSize {
        return box.layoutSize()
    }
}
```

생성자에서 주의해 볼 내용이 있다. 바로 concrete 가 AnyComponent일 때를 처리해 주는 것이다. 실제로 아래와 같이 사용할 수 있기 때문에 처리해 주어야 한다.

```swift
let components: [AnyComponent] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
    AnyComponent(AnyComponent(SwitchComponent()))
]
```

`AnyComponent(AnyComponent(SwitchComponent()))` 로 선언하면 위 초기자에 의해서 `AnyComponent(SwitchComponent())` 가 `self` 가 되어 겹겹이 쌓이는 문제를 방지한다.

이제 컴포넌트를 구현해 보자. 

```swift
struct LabelComponent: Component {
    var id: String {
        return "MyLabel"
    }
    func contentView() -> UILabel {
        return UILabel(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 100, height: 50)
    }
}

struct ButtonComponent: Component {
    var id: String {
        return "MyButton"
    }
    func contentView() -> UIButton {
        return UIButton(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 200, height: 100)
    }
}

struct SwitchComponent: Component {
    var id: String {
        return "MySwitch"
    }
    func contentView() -> UISwitch {
        return UISwitch(frame: .zero)
    }
    func layoutSize() -> CGSize {
        return CGSize(width: 80, height: 40)
    }
}
```

`contentType()` 메서드의 반환값이 UIView가 아닌 UILabel, UIButton 그리고 UISwitch로 각자 다르다. 이제 실제로 사용해 보자.

```swift
let components: [AnyComponent] = [
    AnyComponent(LabelComponent()),
    AnyComponent(ButtonComponent()),
    AnyComponent(SwitchComponent())
]

components.forEach { component in
    print(component.id)
    print(component.contentView())
    print(component.layoutSize())
    print()
}
```

결과는 아래와 같다.

```swift
MyLabel
<UILabel: 0x7fcde8105a50; frame = (0 0; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600001d78780>>
(100.0, 50.0)

MyButton
<UIButton: 0x7fcde800f350; frame = (0 0; 0 0); opaque = NO; layer = <CALayer: 0x600003e5d940>>
(200.0, 100.0)

MySwitch
<UISwitch: 0x7fcde81077c0; frame = (0 0; 51 31); gestureRecognizers = <NSArray: 0x6000030752c0>; layer = <CALayer: 0x600003e7cd00>>
(80.0, 40.0)
```

아래처럼 선언하고 겹겹이 쌓이는 문제도 해결되는지 확인해 보자.

```swift
let components: [AnyComponent] = [
    AnyComponent(AnyComponent(LabelComponent())),
    AnyComponent(AnyComponent(AnyComponent(ButtonComponent()))),
    AnyComponent(AnyComponent(AnyComponent(AnyComponent(SwitchComponent()))))
]

components.forEach { component in
    print(component.id)
    print(component.contentView())
    print(component.layoutSize())
    print()
}
```

```swift
MyLabel
<UILabel: 0x7fd4370063b0; frame = (0 0; 0 0); userInteractionEnabled = NO; layer = <_UILabelLayer: 0x6000016fd630>>
(100.0, 50.0)

MyButton
<UIButton: 0x7fd42740cf80; frame = (0 0; 0 0); opaque = NO; layer = <CALayer: 0x6000035e0620>>
(200.0, 100.0)

MySwitch
<UISwitch: 0x7fd43700fab0; frame = (0 0; 51 31); gestureRecognizers = <NSArray: 0x600003be2940>; layer = <CALayer: 0x6000035ce5e0>>
(80.0, 40.0)
```

원하는 결과가 나오는 것을 알 수 있다. 위 같은 처리가 필요한 이유는 map 함수 등을 사용할 때 위처럼 겹겹이 쌓이는 경우가 발생하기 때문이다. Compoent를 AnyComponent로 map 하는 로직이 있다고 가정해 보자.

```swift
Component -> map -> AnyComponent
```

그런데 Component 자리에 AnyComponent가 올 경우, 위 로직은 아래와 같이 된다.

```swift
AnyComponent -> map -> AnyComponent(AnyComponent)
```

그래서 생성자에서 겹겹이 쌓이는 문제를 해결해 위와 같은 경우에도 아래와 같이 되도록 한 것이다.

```swift
AnyComponent -> map -> AnyComponent
```

flatMap의 기능을 생성자에 넣은 것이라 이해하면 된다.

## eraseToAnyComponent()

Swift Combine의 eraseToAnyPublisher를 구현해 보자. Component에 확장 함수로 구현할 수 있다.

```swift
extension Component {
    func eraseToAnyComponent() -> AnyComponent {
        return AnyComponent(self)
    }
}
```

아래처럼 사용할 수 있다.

```swift
let components2 = [
    LabelComponent().eraseToAnyComponent(),
    ButtonComponent().eraseToAnyComponent(),
    SwitchComponent().eraseToAnyComponent()
]

components2.forEach { component in
    print(component.id)
    print(component.contentView())
    print(component.layoutSize())
    print()
}
```

당연히 함수 반환 타입 등에서도 쉽게 사용할 수 있다.

```swift
func component() -> AnyComponent {
    return LabelComoponent().eraseToAnyComponent()
}
```

## 마무리

지금까지 Swift의 프로토콜, 제네릭 그리고 이 둘을 활용한 Type Eraser Wrapper 패턴을 알아 보았다. Type Eraser Wrapper 패턴은 Swift 표준 라이브러리에서도 넓게 사용되는 패턴이므로 잘 이해해 놓으면 여러모로 도움이 된다. 부족한 글이지만 Type Eraser Wrapper 패턴을 소개하고 이해하는데 도움이 되길 바란다.

## 참고글

- **Breaking Down Type Erasure in Swift**
    - [https://bignerdranch.com/blog/breaking-down-type-erasure-in-swift/](https://bignerdranch.com/blog/breaking-down-type-erasure-in-swift/)

## 예제 코드

예제코드는 아래 URL에서 받을 수 있습니다.
- https://github.com/skyfe79/SwiftTypeEraserWrapper
