---
title: "[SE-0006] 표준 라이브러리에 API 가이드라인 적용하기"
date: 2025-03-08T23:53:41Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 표준 라이브러리에 API 가이드라인 적용하기

* 제안: [SE-0006](0006-apply-api-guidelines-to-the-standard-library.md)
* 작성자: [Dave Abrahams](https://github.com/dabrahams), [Dmitri Gribenko](https://github.com/gribozavr), [Maxim Moiseev](https://github.com/moiseev)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-with-modifications-se-0006-apply-api-guidelines-to-the-standard-library/1667)


## 리뷰어 노트

이 리뷰는 서로 밀접하게 연관된 세 가지 리뷰 중 하나로, 동시에 진행된다:

* [SE-0023 API Design Guidelines](0023-api-guidelines.md)
  ([리뷰](https://forums.swift.org/t/review-se-0023-api-design-guidelines/1162))
* [SE-0006 Apply API Guidelines to the Standard Library](0006-apply-api-guidelines-to-the-standard-library.md)
  ([리뷰](https://forums.swift.org/t/review-se-0006-apply-api-guidelines-to-the-standard-library/1163))
* [SE-0005 Better Translation of Objective-C APIs Into Swift](0005-objective-c-name-translation.md)
  ([리뷰](https://forums.swift.org/t/review-se-0005-better-translation-of-objective-c-apis-into-swift/1164))

이 리뷰들이 동시에 진행되는 이유는 이들이 서로 강하게 상호작용하기 때문이다 (예: 표준 라이브러리의 API 변경은 특정 가이드라인에 해당하거나, 임포터 규칙이 특정 가이드라인을 구현하는 등). 이러한 상호작용과 논의를 관리하기 위해 다음 사항을 요청한다:

* **리뷰 코멘트를 작성하기 전에 세 문서 모두를 이해해야 한다.**
* **각 문서에 대한 리뷰는 해당 리뷰 공지에 대한 응답으로 작성한다.** 리뷰에서 문서 간의 상호 참조를 하는 것은 괜찮으며, 오히려 권장된다. 이는 의견을 명확히 전달하는 데 도움이 될 것이다.


## 소개

Swift 3의 일환으로 개발 중인 [Swift API 설계 가이드라인](https://swift.org/documentation/api-design-guidelines/)이 있다. 표준 라이브러리는 Swift API 설계 가이드라인의 모범이 되어야 한다. 표준 라이브러리의 API는 어떤 애플리케이션 도메인에서도 가장 빈번히 사용되는 Swift API일 뿐만 아니라, 다른 라이브러리에도 선례를 제공하기 때문이다.

이 프로젝트에서는 표준 라이브러리 전체를 검토하고 가이드라인을 따르도록 업데이트한다.


## 제안하는 솔루션

실제 작업은 [Swift 저장소][swift-repo]의 [swift-3-api-guidelines 브랜치][swift-3-api-guidelines-branch]에서 진행 중이다. 주요 변경 사항은 다음과 같다.

* 프로토콜 이름에서 `Type` 접미사를 제거한다. 특수한 경우에는 타입 이름과 충돌을 피하기 위해 `Protocol` 접미사를 추가한다. (대부분의 경우 Swift 3 언어 기능으로 대체될 것으로 예상한다.)

* `generator` 개념을 모든 API에서 `iterator`로 변경한다.

* `CollectionOfOne`의 인덱스로만 사용되던 `Bit` 타입을 제거한다. 대신 `Int`를 사용할 것을 권장한다.

* unsafe 포인터 타입의 제네릭 매개변수 이름을 `Memory`에서 `Pointee`로 변경한다.

* unsafe 포인터 타입에서 인수가 없는 이니셜라이저를 제거한다. 대신 `nil` 리터럴을 사용할 것을 권장한다.

* `PermutationGenerator`를 제거한다.

* `MutableSliceable`을 제거한다. 대신 `Collection where SubSequence : MutableCollection`을 사용한다.

* `sort()` => `sorted()`, `sortInPlace()` => `sort()`로 변경한다.

* `reverse()` => `reversed()`로 변경한다.

* `enumerate()` => `enumerated()`로 변경한다.

* `partition()` API를 단순화한다. 이제 컬렉션 슬라이싱과 더 잘 조합된다.

* `SequenceType.minElement()` => `.min()`, `.maxElement()` => `.max()`로 변경한다.

* 시퀀스 및 컬렉션 어댑터의 일부 이니셜라이저를 제거한다. 대신 해당 알고리즘 함수나 메서드를 호출할 것을 권장한다.

* 일부 함수를 프로퍼티로 변경하거나 그 반대로 변경한다.

* nul-terminated UTF-8 데이터(일명 C-strings)를 다루는 `String` 팩토리 메서드를 이니셜라이저로 변경한다.


## API 차이점

Swift 2.2 표준 라이브러리 API와 제안하는 API 간의 차이점은 [swift-3-api-guidelines 브랜치][swift-3-api-guidelines-branch]에서 구현되면서 이 섹션에 추가된다. 많은 타입에 영향을 미치는 반복적인 변경사항의 경우, 대표적인 예시 하나만 보여준다. 예를 들어, `generate()`는 `makeIterator()`로 이름이 변경되었다. 이 메서드의 프로토콜 요구사항만 보여주고, 이 메서드의 다른 모든 이름 변경은 생략한다. 타입 이름이 변경된 경우, 타입 선언에 대한 차이점만 보여주고, 해당 이름이 사용된 API에 미치는 다른 모든 효과는 생략한다.

* 프로토콜 이름에서 `Type` 접미사 제거.

```diff
-public protocol BooleanType { ... }
+public protocol Boolean { ... }

-public protocol SequenceType { ... }
+public protocol Sequence { ... }

-public protocol CollectionType : ... { ... }
+public protocol Collection : ... { ... }

-public protocol MutableCollectionType : ... { ... }
+public protocol MutableCollection : ... { ... }

-public protocol RangeReplaceableCollectionType : ... { ... }
+public protocol RangeReplaceableCollection : ... { ... }

-public protocol AnyCollectionType : ... { ... }
+public protocol AnyCollectionProtocol : ... { ... }

-public protocol IntegerType : ... { ... }
+public protocol Integer : ... { ... }

-public protocol SignedIntegerType : ... { ... }
+public protocol SignedInteger : ... { ... }

-public protocol UnsignedIntegerType : ... { ... }
+public protocol UnsignedInteger : ... { ... }

-public protocol FloatingPointType : ... { ... }
+public protocol FloatingPoint : ... { ... }

-public protocol ForwardIndexType { ... }
+public protocol ForwardIndex { ... }

-public protocol BidirectionalIndexType : ... { ... }
+public protocol BidirectionalIndex : ... { ... }

-public protocol RandomAccessIndexType : ... { ... }
+public protocol RandomAccessIndex : ... { ... }

-public protocol IntegerArithmeticType : ... { ... }
+public protocol IntegerArithmetic : ... { ... }

-public protocol SignedNumberType : ... { ... }
+public protocol SignedNumber : ... { ... }

-public protocol IntervalType : ... { ... }
+public protocol Interval { ... }

-public protocol LazyCollectionType : ... { ... }
+public protocol LazyCollectionProtocol : ... { ... }

-public protocol LazySequenceType : ... { ... }
+public protocol LazySequenceProtocol : ... { ... }

-public protocol OptionSetType : ... { ... }
+public protocol OptionSet : ... { ... }

-public protocol OutputStreamType : ... { ... }
+public protocol OutputStream : ... { ... }

-public protocol BitwiseOperationsType { ... }
+public protocol BitwiseOperations { ... }

-public protocol ReverseIndexType : ... { ... }
+public protocol ReverseIndexProtocol : ... { ... }

-public protocol SetAlgebraType : ... { ... }
+public protocol SetAlgebra : ... { ... }

-public protocol UnicodeCodecType { ... }
+public protocol UnicodeCodec { ... }

-public protocol CVarArgType { ... }
+public protocol CVarArg { ... }

-public protocol MirrorPathType { ... }
+public protocol MirrorPath { ... }

-public protocol ErrorType { ... }
+public protocol ErrorProtocol { ... }
```

* 모든 API에서 "generator" 개념을 "iterator"로 이름 변경.

```diff
-public protocol GeneratorType { ... }
+public protocol IteratorProtocol { ... }

 public protocol Collection : ... {
-  associatedtype Generator : GeneratorType = IndexingGenerator<Self>
+  associatedtype Iterator : IteratorProtocol = IndexingIterator<Self>

-  func generate() -> Generator
+  func makeIterator() -> Iterator
 }

-public struct IndexingGenerator<Elements : Indexable> : ... { ... }
+public struct IndexingIterator<Elements : Indexable> : ... { ... }

-public struct GeneratorOfOne<Element> : ... { ... }
+public struct IteratorOverOne<Element> : ... { ... }

-public struct EmptyGenerator<Element> : ... { ... }
+public struct EmptyIterator<Element> : ... { ... }

-public struct AnyGenerator<Element> : ... { ... }
+public struct AnyIterator<Element> : ... { ... }

-public struct LazyFilterGenerator<Base : GeneratorType> : ... { ... }
+public struct LazyFilterIterator<Base : IteratorProtocol> : ... { ... }

-public struct FlattenGenerator<Base : ...> : ... { ... }
+public struct FlattenIterator<Base : ...> : ... { ... }

-public struct JoinGenerator<Base : ...> : ... { ... }
+public struct JoinedIterator<Base : ...> : ... { ... }

-public struct LazyMapGenerator<Base : ...> ... { ... }
+public struct LazyMapIterator<Base : ...> ... { ... }

-public struct RangeGenerator<Element : ForwardIndexType> : ... { ... }
+public struct RangeIterator<Element : ForwardIndex> : ... { ... }

-public struct GeneratorSequence<Base : GeneratorType> : ... { ... }
+public struct IteratorSequence<Base : IteratorProtocol> : ... { ... }

-public struct StrideToGenerator<Element : Strideable> : ... { ... }
+public struct StrideToIterator<Element : Strideable> : ... { ... }

-public struct StrideThroughGenerator<Element : Strideable> : ... { ... }
+public struct StrideThroughIterator<Element : Strideable> : ... { ... }

-public struct UnsafeBufferPointerGenerator<Element> : ... { ... }
+public struct UnsafeBufferPointerIterator<Element> : ... { ... }
```

* `CollectionOfOne`의 인덱스로만 사용되던 `Bit` 타입이 제거되었다. 대신 `Int`를 사용할 것을 권장한다.

```diff
-public enum Bit : ... { ... }
```

* `PermutationGenerator`가 제거되었다.

```diff
-public struct PermutationGenerator<
-  C : CollectionType, Indices : SequenceType
-  where C.Index == Indices.Generator.Element
-> : ... { ... }
```

* `MutableSliceable`이 제거되었다. 대신 `Collection where SubSequence : MutableCollection`을 사용한다.

```diff
-public protocol MutableSliceable : CollectionType, MutableCollectionType {
-  subscript(_: Range<Index>) -> SubSequence { get set }
-}
```

* 불안전한 포인터 타입의 제네릭 매개변수 이름이 `Memory`에서 `Pointee`로 변경되었다.

* 불안전한 포인터 타입에서 인수가 없는 초기화 메서드가 제거되었다. 대신 `nil` 리터럴을 사용할 것을 권장한다.

```diff
 // `UnsafePointer`, `UnsafeMutablePointer`, `AutoreleasingUnsafeMutablePointer`에도 동일한 변경사항 적용.
 public struct UnsafePointer<
-  Memory
+  Pointee
 > ... : {
-  public var memory: Memory { get set }
+  public var pointee: Pointee { get set }

   // `nil`을 사용한다.
-  public init()
 }

public struct OpaquePointer : ... {
   // `nil`을 사용한다.
-  public init()
}
```

* `sort()` => `sorted()`, `sortInPlace()` => `sort()`. 또한 클로저에 인수 레이블이 추가되었다.

```diff
 extension Sequence where Self.Iterator.Element : Comparable {
   @warn_unused_result
-  public func sort() -> [Generator.Element]
+  public func sorted() -> [Iterator.Element]
 }

 extension Sequence {
   @warn_unused_result
-  public func sort(
+  public func sorted(
-    @noescape                 isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
+    @noescape isOrderedBefore isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
   ) -> [Iterator.Element]
 }

 extension MutableCollection where Self.Iterator.Element : Comparable {
   @warn_unused_result(mutable_variant="sort")
-  public func sort() -> [Generator.Element]
+  public func sorted() -> [Iterator.Element]
 }

 extension MutableCollection {
   @warn_unused_result(mutable_variant="sort")
-  public func sort(
+  public func sorted(
-   @noescape                 isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
+   @noescape isOrderedBefore isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
   ) -> [Iterator.Element]
 }

 extension MutableCollection
   where
   Self.Index : RandomAccessIndex,
   Self.Iterator.Element : Comparable {

-  public mutating func sortInPlace()
+  public mutating func sort()

 }

 extension MutableCollection where Self.Index : RandomAccessIndex {
-  public mutating func sortInPlace(
+  public mutating func sort(
-    @noescape                 isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
+    @noescape isOrderedBefore isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
   )
 }
```

* `reverse()` => `reversed()`.

```diff
 extension SequenceType {
-  public func reverse() -> [Generator.Element]
+  public func reversed() -> [Iterator.Element]
 }

 extension CollectionType where Index : BidirectionalIndexType {
-  public func reverse() -> ReverseCollection<Self>
+  public func reversed() -> ReverseCollection<Self>
 }

 extension CollectionType where Index : RandomAccessIndexType {
-  public func reverse() -> ReverseRandomAccessCollection<Self>
+  public func reversed() -> ReverseRandomAccessCollection<Self>
 }

 extension LazyCollectionProtocol
   where Index : BidirectionalIndexType, Elements.Index : BidirectionalIndexType {

-  public func reverse()
+  public func reversed()
     -> LazyCollection<ReverseCollection<Elements>>
 }

 extension LazyCollectionProtocol
   where Index : RandomAccessIndexType, Elements.Index : RandomAccessIndexType {

-  public func reverse()
+  public func reversed()
     -> LazyCollection<ReverseRandomAccessCollection<Elements>>
 }
```

* `enumerate()` => `enumerated()`.

```diff
 extension Sequence {
-  public func enumerate() -> EnumerateSequence<Self>
+  public func enumerated() -> EnumeratedSequence<Self>
 }

-public struct EnumerateSequence<Base : SequenceType> : ... { ... }
+public struct EnumeratedSequence<Base : Sequence> : ... { ... }

-public struct EnumerateGenerator<Base : GeneratorType> : ... { ... }
+public struct EnumeratedIterator<Base : IteratorProtocol> : ... { ... }
```

* `partition()` API가 단순화되었다: 범위 인수가 제거되었다. 이제 컬렉션 슬라이싱과 더 잘 조합되며, 다른 컬렉션 알고리즘과 더 일관성이 있다. 또한 클로저에 `@noescape`와 인수 레이블이 추가되었다.

```diff
 extension MutableCollection where Index : RandomAccessIndex {
   public mutating func partition(
-    range: Range<Index>,
-                              isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
+    @noescape isOrderedBefore isOrderedBefore: (Iterator.Element, Iterator.Element) -> Bool
   ) -> Index
 }

 extension MutableCollection
   where Index : RandomAccessIndex, Iterator.Element : Comparable {

-  public mutating func partition(range: Range<Index>) -> Index
+  public mutating func partition() -> Index
}
```

* `SequenceType.minElement()` => `.min()`, `.maxElement()` => `.max()`. 또한 클로저에 인수 레이블이 추가되었다.

```diff
 extension Sequence {
-  public func minElement(
+  public func min(
-    @noescape                 isOrderedBefore: (Iterator.Element, Iterator.Element) throws -> Bool
+    @noescape isOrderedBefore isOrderedBefore: (Iterator.Element, Iterator.Element) throws -> Bool
   ) rethrows -> Iterator.Element?

-  public func maxElement(
+  public func max(
-    @noescape                 isOrderedBefore: (Iterator.Element, Iterator.Element) throws -> Bool
+    @noescape isOrderedBefore isOrderedBefore: (Iterator.Element, Iterator.Element) throws -> Bool
   ) rethrows -> Iterator.Element?
 }

 extension Sequence where Iterator.Element : Comparable {
-  public func minElement() -> Iterator.Element?
+  public func min() -> Iterator.Element?

-  public func maxElement() -> Iterator.Element?
+  public func max() -> Iterator.Element?
 }
```

* 시퀀스, 컬렉션, 이터레이터 어댑터의 일부 초기화 메서드가 제거되었다. 대신 해당 알고리즘 함수나 메서드를 호출할 것을 권장한다.

```diff
 extension Repeated {
-  public init(count: Int, repeatedValue: Element)
 }
+/// `elementInstance`를 `n`번 반복하는 컬렉션을 반환한다.
+public func repeatElement<T>(element: T, count n: Int) -> Repeated<T>

 public struct LazyMapSequence<Base : Sequence, Element> : ... {
   // 시퀀스에서 `.lazy.map`을 호출한다.
-  public init(_ base: Base, transform: (Base.Generator.Element) -> Element)
 }

 public struct LazyMapCollection<Base : Collection, Element> : ... {
   // 컬렉션에서 `.lazy.map`을 호출한다.
-  public init(_ base: Base, transform: (Base.Generator.Element) -> Element)
 }

 public struct LazyFilterIterator<Base : IteratorProtocol> : ... {
   // 시퀀스에서 `.lazy.filter`을 호출한다.
-  public init(
-    _ base: Base,
-    whereElementsSatisfy predicate: (Base.Element) -> Bool
-  )
 }

 public struct RangeIterator<Element : ForwardIndex> : ... {
   // 컬렉션에서 'generate()' 메서드를 사용한다.
-  public init(_ bounds: Range<Element>)

   // '..<' 연산자를 사용한다.
-  public init(start: Element, end: Element)
 }

 public struct ReverseCollection<Base : ...> : ... {
   // 컬렉션에서 'reverse()' 메서드를 사용한다.
-  public init(_ base: Base)
 }

 public struct ReverseRandomAccessCollection<Base : ...> : ... {
   // 컬렉션에서 'reverse()' 메서드를 사용한다.
-  public init(_ base: Base)
 }

 public struct Slice<Base : Indexable> : ... {
   // 슬라이싱 문법을 사용한다.
-  public init(base: Base, bounds: Range<Index>)
 }

 public struct MutableSlice<Base : MutableIndexable> : ... {
   // 슬라이싱 문법을 사용한다.
-  public init(base: Base, bounds: Range<Index>)
 }

 public struct EnumeratedIterator<Base : IteratorProtocol> : ... {
   // 'enumerated()' 메서드를 사용한다.
-  public init(_ base: Base)
 }

 public struct EnumeratedSequence<Base : IteratorProtocol> : ... {
   // 'enumerated()' 메서드를 사용한다.
-  public init(_ base: Base)
 }

 public struct IndexingIterator<Elements : Indexable> : ... {
   // 컬렉션에서 'iterator()'를 호출한다.
-  public init(_elements: Elements)
 }

 public struct HalfOpenInterval<Bound : Comparable> : ... {
   // '..<' 연산자를 사용한다.
-  public init(_ start: Bound, _ end: Bound)
 }

 public struct ClosedInterval<Bound : Comparable> : ... {
   // '...' 연산자를 사용한다.
-  public init(_ start: Bound, _ end: Bound)
 }
```

* 일부 함수가 프로퍼티로 변경되었고, 그 반대도 있다.

```diff
-public func unsafeUnwrap<T>(nonEmpty: T?) -> T
 extension Optional {
+  public var unsafelyUnwrapped: Wrapped { get }
 }

 public struct Mirror {
-  public func superclassMirror() -> Mirror?
+  public var superclassMirror: Mirror? { get }
 }

 public protocol CustomReflectable {
-  func customMirror() -> Mirror
+  var customMirror: Mirror { get }
 }

 public protocol Collection : ... {
-  public func underestimateCount() -> Int
+  public var underestimatedCount: Int { get }
 }

 public protocol CustomPlaygroundQuickLookable {
-  func customPlaygroundQuickLook() -> PlaygroundQuickLook
+  var customPlaygroundQuickLook: PlaygroundQuickLook { get }
 }

 extension String {
-  public var lowercaseString: String { get }
+  public func lowercased()

-  public var uppercaseString: String { get }
+  public func uppercased()
 }

 public enum UnicodeDecodingResult {
-  public func isEmptyInput() -> Bool {
+  public var isEmptyInput: Bool
 }

```

* 기본 이름과 인수 레이블이 첫 번째 인수 레이블에 대한 가이드라인을 따르도록 변경되었다. (이 카테고리의 일부 변경사항은 이미 차이점에 명시되어 있으며 반복되지 않는다.)

```diff
 public protocol ForwardIndex {
-  func advancedBy(n: Distance) -> Self
+  func advanced(by n: Distance) -> Self

-  func advancedBy(n: Distance, limit: Self) -> Self
+  func advanced(by n: Distance, limit: Self) -> Self

-  func distanceTo(end: Self) -> Distance
+  func distance(to end: Self) -> Distance
 }

 public struct Set<Element : Hashable> : ... {
-  public mutating func removeAtIndex(index: Index) -> Element
+  public mutating func remove(at index: Index) -> Element
 }

 public struct Dictionary<Key : Hashable, Value> : ... {
-  public mutating func removeAtIndex(index: Index) -> Element
+  public mutating func remove(at index: Index) -> Element

-  public func indexForKey(key: Key) -> Index?
+  public func index(forKey key: Key) -> Index?

-  public mutating func removeValueForKey(key: Key) -> Value?
+  public mutating func removeValue(forKey key: Key) -> Value?
 }

 extension Sequence where Iterator.Element : Sequence {
   // joinWithSeparator(_:) => join(separator:)
-  public func joinWithSeparator<
+  public func joined<
     Separator : Sequence
     where
     Separator.Iterator.Element == Iterator.Element.Iterator.Element
-  >(separator: Separator) -> JoinSequence<Self>
+  >(separator separator: Separator) -> JoinedSequence<Self>
 }

 extension Sequence where Iterator.Element == String {
-  public func joinWithSeparator(separator: String) -> String
+  public func joined(separator separator: String) -> String
 }

 public class ManagedBuffer<Value, Element> : ... {
   public final class func create(
-    minimumCapacity: Int,
+    minimumCapacity minimumCapacity: Int,
     initialValue: (ManagedProtoBuffer<Value, Element>) -> Value
   ) -> ManagedBuffer<Value, Element>
 }

 public protocol Streamable {
-  func writeTo<Target : OutputStream>(inout target: Target)
+  func write<Target : OutputStream>(inout to target: Target)
 }

 public func dump<T, TargetStream : OutputStream>(
   value: T,
-  inout _ target: TargetStream,
+  inout to target: TargetStream,
   name: String? = nil,
   indent: Int = 0,
   maxDepth: Int = .max,
   maxItems: Int = .max
 ) -> T

 extension Sequence {
-  public func startsWith<
+  public func starts<
     PossiblePrefix : Sequence where PossiblePrefix.Iterator.Element == Iterator.Element
   >(
-    possiblePrefix: PossiblePrefix,
+    with possiblePrefix: PossiblePrefix,
     @noescape isEquivalent: (Iterator.Element, Iterator.Element) throws -> Bool
   ) rethrows -> Bool

-  public func startsWith<
+  public func starts<
     PossiblePrefix : Sequence where PossiblePrefix.Iterator.Element == Iterator.Element
   >(
-    possiblePrefix: PossiblePrefix
+    with possiblePrefix: PossiblePrefix
   ) -> Bool
 }

 extension CollectionType where Iterator.Element : Equatable {
-  public func indexOf(element: Iterator.Element) -> Index?
+  public func index(of element: Iterator.Element) -> Index?
 }

 extension CollectionType {
-  public func indexOf(predicate: (Iterator.Element) throws -> Bool) rethrows -> Index?
+  public func index(where predicate: (Iterator.Element) throws -> Bool) rethrows -> Index?
 }

 extension String.Index {
-  public func samePositionIn(utf8: String.UTF8View) -> String.UTF8View.Index
+  public func samePosition(in utf8: String.UTF8View) -> String.UTF8View.Index

-  public func samePositionIn(utf16: String.UTF16View) -> String.UTF16View.Index
+  public func samePosition(in utf16: String.UTF16View) -> String.UTF16View.Index

-  public func samePositionIn(unicodeScalars: String.UnicodeScalarView) -> String.UnicodeScalarView.Index
+  public func samePosition(in unicodeScalars: String.UnicodeScalarView) -> String.UnicodeScalarView.Index
 }

 extension String.UTF16View.Index {
-  public func samePositionIn(characters: String) -> String.Index
+  public func samePosition(in characters: String) -> String.Index

-  public func samePositionIn(utf8: String.UTF8View) -> String.UTF8View.Index
+  public func samePosition(in utf8: String.UTF8View) -> String.UTF8View.Index

-  public func samePositionIn(unicodeScalars: String.UnicodeScalarView) -> String.UnicodeScalarView.Index
+  public func samePosition(in unicodeScalars: String.UnicodeScalarView) -> String.UnicodeScalarView.Index
 }

 extension String.UTF8View.Index {
-  public func samePositionIn(characters: String) -> String.Index
+  public func samePosition(in characters: String) -> String.Index

-  public func samePositionIn(utf16: String.UTF16View) -> String.UTF16View.Index
+  public func samePosition(in utf16: String.UTF16View) -> String.UTF16View.Index

-  public func samePositionIn(unicodeScalars: String.UnicodeScalarView) -> String.UnicodeScalarView.Index
+  public func samePosition(in unicodeScalars: String.UnicodeScalarView) -> String.UnicodeScalarView.Index
 }

 extension String.UnicodeScalarView.Index {
-  public func samePositionIn(characters: String) -> String.Index
+  public func samePosition(in characters: String) -> String.Index

-  public func samePositionIn(utf16: String.UTF16View) -> String.UTF16View.Index
+  public func samePosition(in utf16: String.UTF16View) -> String.UTF16View.Index

-  public func samePositionIn(utf16: String.UTF16View) -> String.UTF16View.Index
+  public func samePosition(in utf16: String.UTF16View) -> String.UTF16View.Index
 }
```

* 열거형 케이스와 정적 프로퍼티를 소문자로 변경.

```diff
 public struct Float {
-  public static var NaN: Float
+  public static var nan: Float
 }

 public struct Double {
-  public static var NaN: Double
+  public static var nan: Double

 public struct CGFloat {
-  public static var NaN: CGFloat
+  public static var nan: CGFloat
 }

 public protocol FloatingPoint : ... {
-  static var NaN: Self { get }
+  static var nan: Self { get }
 }

 public enum FloatingPointClassification {
-  case SignalingNaN
+  case signalingNaN

-  case QuietNaN
+  case quietNaN

-  case NegativeInfinity
+  case negativeInfinity

-  case NegativeNormal
+  case negativeNormal

-  case NegativeSubnormal
+  case negativeSubnormal

-  case NegativeZero
+  case negativeZero

-  case PositiveZero
+  case positiveZero

-  case PositiveSubnormal
+  case positiveSubnormal

-  case PositiveNormal
+  case positiveNormal

-  case PositiveInfinity
+  case positiveInfinity
 }

 public enum ImplicitlyUnwrappedOptional<Wrapped> : ... {
-  case None
+  case none

-  case Some(Wrapped)
+  case some(Wrapped)
 }

 public enum Optional<Wrapped> : ... {
-  case None
+  case none

-  case Some(Wrapped)
+  case some(Wrapped)
 }

 public struct Mirror {
   public enum AncestorRepresentation {
-    case Generated
+    case generated

-    case Customized(() -> Mirror)
+    case customized(() -> Mirror)

-    case Suppressed
+    case suppressed
   }

   public enum DisplayStyle {
-    case struct, class, enum, tuple, optional, collection
+    case `struct`, `class`, `enum`, tuple, optional, collection

-    case dictionary, `set`
+    case dictionary, `set`
   }
 }

 public enum PlaygroundQuickLook {
-  case Text(String)
+  case text(String)

-  case Int(Int64)
+  case int(Int64)

-  case UInt(UInt64)
+  case uInt(UInt64)

-  case Float(Float32)
+  case float(Float32)

-  case Double(Float64)
+  case double(Float64)

-  case Image(Any)
+  case image(Any)

-  case Sound(Any)
+  case sound(Any)

-  case Color(Any)
+  case color(Any)

-  case BezierPath(Any)
+  case bezierPath(Any)

-  case AttributedString(Any)
+  case attributedString(Any)

-  case Rectangle(Float64,Float64,Float64,Float64)
+  case rectangle(Float64,Float64,Float64,Float64)

-  case Point(Float64,Float64)
+  case point(Float64,Float64)

-  case Size(Float64,Float64)
+  case size(Float64,Float64)

-  case Logical(Bool)
+  case bool(Bool)

-  case Range(Int64, Int64)
+  case range(Int64, Int64)

-  case View(Any)
+  case view(Any)

-  case Sprite(Any)
+  case sprite(Any)

-  case URL(String)
+  case url(String)

-  case _Raw([UInt8], String)
+  case _raw([UInt8], String)
 }
```

* 널 종료 UTF-8 데이터(즉, C 문자열)를 다루는 `String` 팩토리 메서드가 초기화 메서드로 변경.

```diff
 extension String {
-  public static func fromCString(cs: UnsafePointer<CChar>) -> String?
+  public init?(validatingUTF8 cString: UnsafePointer<CChar>)

-  public static func fromCStringRepairingIllFormedUTF8(cs: UnsafePointer<CChar>) -> (String?, hadError: Bool)
+  public init(cString: UnsafePointer<CChar>)
+  public static func decodeCString<Encoding : UnicodeCodec>(
+    cString: UnsafePointer<Encoding.CodeUnit>,
+    as encoding: Encoding.Type,
+    repairingInvalidCodeUnits isReparing: Bool = true)
+      -> (result: String, repairsMade: Bool)?
 }
```

* `NSString`에서 임포트된 메서드를 반영하는 `String` 메서드 이름 변경.

```diff
 extension String {
-  public static func localizedNameOfStringEncoding(
-    encoding: NSStringEncoding
-  ) -> String
+  public static func localizedName(
+    ofStringEncoding encoding: NSStringEncoding
+  ) -> String
 
-  public static func pathWithComponents(components: [String]) -> String
+  public static func path(withComponents components: [String]) -> String
 
-  public init?(UTF8String bytes: UnsafePointer<CChar>)
+  public init?(utf8String bytes: UnsafePointer<CChar>)

-  public func canBeConvertedToEncoding(encoding: NSStringEncoding) -> Bool
+  public func canBeConverted(toEncoding encoding: NSStringEncoding) -> Bool
 
-  public var capitalizedString: String
+  public var capitalized: String
 
-  public var localizedCapitalizedString: String
+  public var localizedCapitalized: String
 
-  public func capitalizedStringWithLocale(locale: NSLocale?) -> String
+  public func capitalized(with locale: NSLocale?) -> String
 
-  public func commonPrefixWithString(
-    aString: String, options: NSStringCompareOptions) -> String
+  public func commonPrefix(
+    with aString: String, options: NSStringCompareOptions = []) -> String
 
-  public func completePathIntoString(
-    outputName: UnsafeMutablePointer<String> = nil,
-    caseSensitive: Bool,
-    matchesIntoArray: UnsafeMutablePointer<[String]> = nil,
-    filterTypes: [String]? = nil
-  ) -> Int
+  public func completePath(
+    into outputName: UnsafeMutablePointer<String> = nil,
+    caseSensitive: Bool,
+    matchesInto matchesIntoArray: UnsafeMutablePointer<[String]> = nil,
+    filterTypes: [String]? = nil
+  ) -> Int

-  public func componentsSeparatedByCharactersInSet(
-    separator: NSCharacterSet
-  ) -> [String]
+  public func componentsSeparatedByCharacters(
+    in separator: NSCharacterSet
+  ) -> [String]
 
-  public func componentsSeparatedByString(separator: String) -> [String]
+  public func componentsSeparated(by separator: String) -> [String]

-  public func cStringUsingEncoding(encoding: NSStringEncoding) -> [CChar]?
+  public func cString(usingEncoding encoding: NSStringEncoding) -> [CChar]?
 
-  public func dataUsingEncoding(
-    encoding: NSStringEncoding,
-    allowLossyConversion: Bool = false
-  ) -> NSData?
+  public func data(
+    usingEncoding encoding: NSStringEncoding,
+    allowLossyConversion: Bool = false
+  ) -> NSData?
 
-  public func enumerateLinguisticTagsInRange(
-    range: Range<Index>,
-    scheme tagScheme: String,
-    options opts: NSLinguisticTaggerOptions,
-    orthography: NSOrthography?,
-    _ body:
-      (String, Range<Index>, Range<Index>, inout Bool) -> ()
-  )
+  public func enumerateLinguisticTags(
+    in range: Range<Index>,
+    scheme tagScheme: String,
+    options opts: NSLinguisticTaggerOptions = [],
+    orthography: NSOrthography? = nil,
+    _ body:
+      (String, Range<Index>, Range<Index>, inout Bool) -> ()
+  )

-  public func enumerateSubstringsInRange(
-    range: Range<Index>,
-    options opts:NSStringEnumerationOptions,
-    _ body: (
-      substring: String?, substringRange: Range<Index>,
-      enclosingRange: Range<Index>, inout Bool
-    ) -> ()
-  )
+  public func enumerateSubstrings(
+    in range: Range<Index>,
+    options opts:NSStringEnumerationOptions = [],
+    _ body: (
+      substring: String?, substringRange: Range<Index>,
+      enclosingRange: Range<Index>, inout Bool
+    ) -> ()
+  )

-  public func fileSystemRepresentation() -> [CChar]
+  public var fileSystemRepresentation: [CChar]
 
-  public func getBytes(
-    inout buffer: [UInt8],
-    maxLength maxBufferCount: Int,
-    usedLength usedBufferCount: UnsafeMutablePointer<Int>,
-    encoding: NSStringEncoding,
-    options: NSStringEncodingConversionOptions,
-    range: Range<Index>,
-    remainingRange leftover: UnsafeMutablePointer<Range<Index>>
-  ) -> Bool
+  public func getBytes(
+    inout buffer: [UInt8],
+    maxLength maxBufferCount: Int,
+    usedLength usedBufferCount: UnsafeMutablePointer<Int>,
+    encoding: NSStringEncoding,
+    options: NSStringEncodingConversionOptions = [],
+    range: Range<Index>,
+    remaining leftover: UnsafeMutablePointer<Range<Index>>
+  ) -> Bool
 
-  public func getLineStart(
-    start: UnsafeMutablePointer<Index>,
-    end: UnsafeMutablePointer<Index>,
-    contentsEnd: UnsafeMutablePointer<Index>,
-    forRange: Range<Index>
-  )
+  public func getLineStart(
+    start: UnsafeMutablePointer<Index>,
+    end: UnsafeMutablePointer<Index>,
+    contentsEnd: UnsafeMutablePointer<Index>,
+    for range: Range<Index>
+  )

-  public func getParagraphStart(
-    start: UnsafeMutablePointer<Index>,
-    end: UnsafeMutablePointer<Index>,
-    contentsEnd: UnsafeMutablePointer<Index>,
-    forRange: Range<Index>
-  )
+  public func getParagraphStart(
+    start: UnsafeMutablePointer<Index>,
+    end: UnsafeMutablePointer<Index>,
+    contentsEnd: UnsafeMutablePointer<Index>,
+    for range: Range<Index>
+  )

   public init(
-    contentsOfURL url: NSURL,
+    contentsOf url: NSURL,
     encoding enc: NSStringEncoding
   ) throws
 
   public init(
-    contentsOfURL url: NSURL,
+    contentsOf url: NSURL,
     usedEncoding enc: UnsafeMutablePointer<NSStringEncoding> = nil
   ) throws
 
   public init?(
-    CString: UnsafePointer<CChar>,
+    cString: UnsafePointer<CChar>,
     encoding enc: NSStringEncoding
   )

-  public init(format: String, _ arguments: CVarArgType...)
+  public init(format: String, _ arguments: CVarArg...)

-  public init(format: String, arguments: [CVarArgType])
+  public init(format: String, arguments: [CVarArg])
 
-  public init(format: String, locale: NSLocale?, _ args: CVarArgType...)
+  public init(format: String, locale: NSLocale?, _ args: CVarArg...)
 
-  public init(format: String, locale: NSLocale?, arguments: [CVarArgType])
+  public init(format: String, locale: NSLocale?, arguments: [CVarArg])

-  public func lengthOfBytesUsingEncoding(encoding: NSStringEncoding) -> Int
+  public func lengthOfBytes(usingEncoding encoding: NSStringEncoding) -> Int
 
-  public func lineRangeForRange(aRange: Range<Index>) -> Range<Index>
+  public func lineRange(for aRange: Range<Index>) -> Range<Index>
 
-  public func linguisticTagsInRange(
-    range: Range<Index>,
-    scheme tagScheme: String,
-    options opts: NSLinguisticTaggerOptions = [],
-    orthography: NSOrthography? = nil,
-    tokenRanges: UnsafeMutablePointer<[Range<Index>]> = nil // FIXME:Can this be nil?
-  ) -> [String]
+  public func linguisticTags(
+    in range: Range<Index>,
+    scheme tagScheme: String,
+    options opts: NSLinguisticTaggerOptions = [],
+    orthography: NSOrthography? = nil,
+    tokenRanges: UnsafeMutablePointer<[Range<Index>]> = nil // FIXME:Can this be nil?
+  ) -> [String]

-  public var localizedLowercaseString: String
+  public var localizedLowercase: String
 
-  public func lowercaseStringWithLocale(locale: NSLocale?) -> String
+  public func lowercaseString(with locale: NSLocale?) -> String
 
-  func maximumLengthOfBytesUsingEncoding(encoding: NSStringEncoding) -> Int
+  func maximumLengthOfBytes(usingEncoding encoding: NSStringEncoding) -> Int
 
-  public func paragraphRangeForRange(aRange: Range<Index>) -> Range<Index>
+  public func paragraphRange(for aRange: Range<Index>) -> Range<Index>
 
-  public func rangeOfCharacterFromSet(
-    aSet: NSCharacterSet,
-    options mask:NSStringCompareOptions = [],
-    range aRange: Range<Index>? = nil
-  ) -> Range<Index>?
+  public func rangeOfCharacter(
+    from aSet: NSCharacterSet,
+    options mask:NSStringCompareOptions = [],
+    range aRange: Range<Index>? = nil
+  ) -> Range<Index>?
 
-  func rangeOfComposedCharacterSequenceAtIndex(anIndex: Index) -> Range<Index>
+  func rangeOfComposedCharacterSequence(at anIndex: Index) -> Range<Index>
 
-  public func rangeOfComposedCharacterSequencesForRange(
-    range: Range<Index>
-  ) -> Range<Index>
+  public func rangeOfComposedCharacterSequences(
+    for range: Range<Index>
+  ) -> Range<Index>
 
-  public func rangeOfString(
-    aString: String,
-    options mask: NSStringCompareOptions = [],
-    range searchRange: Range<Index>? = nil,
-    locale: NSLocale? = nil
-  ) -> Range<Index>?
+  public func range(
+    of aString: String,
+    options mask: NSStringCompareOptions = [],
+    range searchRange: Range<Index>? = nil,
+    locale: NSLocale? = nil
+  ) -> Range<Index>?
 
-  public func localizedStandardContainsString(string: String) -> Bool
+  public func localizedStandardContains(string: String) -> Bool
 
-  public func localizedStandardRangeOfString(string: String) -> Range<Index>?
+  public func localizedStandardRange(of string: String) -> Range<Index>?
 
-  public var stringByAbbreviatingWithTildeInPath: String
+  public var abbreviatingWithTildeInPath: String
 
-  public func stringByAddingPercentEncodingWithAllowedCharacters(
-    allowedCharacters: NSCharacterSet
-  ) -> String?
+  public func addingPercentEncoding(
+    withAllowedCharacters allowedCharacters: NSCharacterSet
+  ) -> String?

-  public func stringByAddingPercentEscapesUsingEncoding(
-    encoding: NSStringEncoding
-  ) -> String?
+  public func addingPercentEscapes(
+    usingEncoding encoding: NSStringEncoding
+  ) -> String?
 
-  public func stringByAppendingFormat(
-    format: String, _ arguments: CVarArgType...
-  ) -> String
+  public func appendingFormat(
+    format: String, _ arguments: CVarArg...
+  ) -> String
 
-  public func stringByAppendingPathComponent(aString: String) -> String
+  public func appendingPathComponent(aString: String) -> String
 
-  public func stringByAppendingPathExtension(ext: String) -> String?
+  public func appendingPathExtension(ext: String) -> String?
 
-  public func stringByAppendingString(aString: String) -> String
+  public func appending(aString: String) -> String
 
-  public var stringByDeletingLastPathComponent: String
+  public var deletingLastPathComponent: String
 
-  public var stringByDeletingPathExtension: String
+  public var deletingPathExtension: String
 
-  public var stringByExpandingTildeInPath: String
+  public var expandingTildeInPath: String
 
-  public func stringByFoldingWithOptions(
-    options: NSStringCompareOptions, locale: NSLocale?
-  ) -> String
+  public func folding(
+    options: NSStringCompareOptions = [], locale: NSLocale?
+  ) -> String
 
-  public func stringByPaddingToLength(
-    newLength: Int, withString padString: String, startingAtIndex padIndex: Int
-  ) -> String
+  public func padding(
+    toLength newLength: Int,
+    with padString: String,
+    startingAt padIndex: Int
+  ) -> String
 
-  public var stringByRemovingPercentEncoding: String?
+  public var removingPercentEncoding: String?

-  public func stringByReplacingCharactersInRange(
-    range: Range<Index>, withString replacement: String
-  ) -> String
+  public func replacingCharacters(
+    in range: Range<Index>, with replacement: String
+  ) -> String
 
-  public func stringByReplacingOccurrencesOfString(
-    target: String,
-    withString replacement: String,
-    options: NSStringCompareOptions = [],
-    range searchRange: Range<Index>? = nil
-  ) -> String
+  public func replacingOccurrences(
+    of target: String,
+    with replacement: String,
+    options: NSStringCompareOptions = [],
+    range searchRange: Range<Index>? = nil
+  ) -> String
 
-  public func stringByReplacingPercentEscapesUsingEncoding(
-    encoding: NSStringEncoding
-  ) -> String?
+  public func replacingPercentEscapes(
+    usingEncoding encoding: NSStringEncoding
+  ) -> String?
 
-  public var stringByResolvingSymlinksInPath: String
+  public var resolvingSymlinksInPath: String
 
-  public var stringByStandardizingPath: String
+  public var standardizingPath: String
 
-  public func stringByTrimmingCharactersInSet(set: NSCharacterSet) -> String
+  public func trimmingCharacters(in set: NSCharacterSet) -> String
 
-  public func stringsByAppendingPaths(paths: [String]) -> [String]
+  public func strings(byAppendingPaths paths: [String]) -> [String]
 
-  public func substringFromIndex(index: Index) -> String
+  public func substring(from index: Index) -> String
 
-  public func substringToIndex(index: Index) -> String
+  public func substring(to index: Index) -> String
 
-  public func substringWithRange(aRange: Range<Index>) -> String
+  public func substring(with aRange: Range<Index>) -> String
 
-  public var localizedUppercaseString: String
+  public var localizedUppercase: String
 
-  public func uppercaseStringWithLocale(locale: NSLocale?) -> String
+  public func uppercaseString(with locale: NSLocale?) -> String
 
-  public func writeToFile(
-    path: String, atomically useAuxiliaryFile:Bool,
-    encoding enc: NSStringEncoding
-  ) throws
+  public func write(
+    toFile path: String, atomically useAuxiliaryFile:Bool,
+    encoding enc: NSStringEncoding
+  ) throws
 
-  public func writeToURL(
-    url: NSURL, atomically useAuxiliaryFile: Bool,
-    encoding enc: NSStringEncoding
-  ) throws
+  public func write(
+    to url: NSURL, atomically useAuxiliaryFile: Bool,
+    encoding enc: NSStringEncoding
+  ) throws
 
-  public func stringByApplyingTransform(
-    transform: String, reverse: Bool
-  ) -> String?
+  public func applyingTransform(
+    transform: String, reverse: Bool
+  ) -> String?
 
-  public func containsString(other: String) -> Bool
+  public func contains(other: String) -> Bool

-  public func localizedCaseInsensitiveContainsString(other: String) -> Bool
+  public func localizedCaseInsensitiveContains(other: String) -> Bool
 }
 ```

* 기타 변경 사항.

```diff
 public struct EnumeratedIterator<Base : IteratorProtocol> : ... {
-  public typealias Element = (index: Int, element: Base.Element)
+  public typealias Element = (offset: Int, element: Base.Element)
 }

 public struct Array<Element> : ... {
   // Same changes were also applied to `ArraySlice` and `ContiguousArray`.

-  public init(count: Int, repeatedValue: Element)
+  public init(repeating: Element, count: Int)
 }

 public protocol Sequence : ... {
   public func split(
-    maxSplit: Int = Int.max,
+    maxSplits maxSplits: Int = Int.max,
-    allowEmptySlices: Bool = false,
+    omittingEmptySubsequences: Bool = true,
     @noescape isSeparator: (Iterator.Element) throws -> Bool
   ) rethrows -> [SubSequence]
 }

 extension Sequence where Iterator.Element : Equatable {
   public func split(
-    separator: Iterator.Element,
+    separator separator: Iterator.Element,
-    maxSplit: Int = Int.max,
+    maxSplits maxSplits: Int = Int.max,
-    allowEmptySlices: Bool = false
+    omittingEmptySubsequences: Bool = true
   ) -> [AnySequence<Iterator.Element>] {
 }


 public protocol Sequence : ... {
-  public func lexicographicalCompare<
+  public func lexicographicallyPrecedes<
     OtherSequence : Sequence where OtherSequence.Iterator.Element == Iterator.Element
   >(
     other: OtherSequence,
     @noescape isOrderedBefore: (Iterator.Element, Iterator.Element) throws -> Bool
   ) rethrows -> Bool {
 }

 extension Sequence where Iterator.Element : Equatable {
-  public func lexicographicalCompare<
+  public func lexicographicallyPrecedes<
     OtherSequence : Sequence where OtherSequence.Iterator.Element == Iterator.Element
   >(
     other: OtherSequence
   ) -> Bool {
 }

 public protocol Collection : ... {
-  func prefixUpTo(end: Index) -> SubSequence
+  func prefix(upTo end: Index) -> SubSequence

-  func suffixFrom(start: Index) -> SubSequence
+  func suffix(from start: Index) -> SubSequence

-  func prefixThrough(position: Index) -> SubSequence
+  func prefix(through position: Index) -> SubSequence
 }

 // Changes to this protocol affect `Array`, `ArraySlice`, `ContiguousArray` and
 // other types.
 public protocol RangeReplaceableCollection : ... {
+  public init(repeating repeatedValue: Iterator.Element, count: Int)

-  mutating func replaceRange<
+  mutating func replaceSubrange<
     C : CollectionType where C.Iterator.Element == Iterator.Element
   >(
     subRange: Range<Int>, with newElements: C
   )

-  mutating func insert(newElement: Iterator.Element, atIndex i: Int)
+  mutating func insert(newElement: Iterator.Element, at i: Int)

-  mutating func insertContentsOf<
+  mutating func insert<
     S : Collection where S.Iterator.Element == Iterator.Element
-  >(newElements: S, at i: Index)
+  >(contentsOf newElements: S, at i: Index)

-  mutating func removeAtIndex(index: Int) -> Element
+  mutating func remove(at index: Int) -> Element

-  mutating func removeAll(keepCapacity keepCapacity: Bool = false)
+  mutating func removeAll(keepingCapacity keepingCapacity: Bool = false)

-  mutating func removeRange(subRange: Range<Index>)
+  mutating func removeSubrange(subRange: Range<Index>)

-  mutating func appendContentsOf<S : SequenceType>(newElements: S)
+  mutating func append<S : SequenceType>(contentsOf newElements: S)
 }

+extension Set : SetAlgebra {}

 public struct Dictionary<Key : Hashable, Value> : ... {
-  public typealias Element = (Key, Value)
+  public typealias Element = (key: Key, value: Value)
 }

 public struct DictionaryLiteral<Key, Value> : ... {
-  public typealias Element = (Key, Value)
+  public typealias Element = (key: Key, value: Value)
 }

 extension String {
-  public mutating func appendContentsOf(other: String) {
+  public mutating func append(other: String) {

-  public mutating appendContentsOf<S : SequenceType>(newElements: S)
+  public mutating append<S : SequenceType>(contentsOf newElements: S)

-  public mutating func replaceRange<
+  public mutating func replaceSubrange<
     C: CollectionType where C.Iterator.Element == Character
   >(
     subRange: Range<Index>, with newElements: C
   )

-  public mutating func replaceRange(
+  public mutating func replaceSubrange(
     subRange: Range<Index>, with newElements: String
   )

-  public mutating func insert(newElement: Character, atIndex i: Index)
+  public mutating func insert(newElement: Character, at i: Index)

-  public mutating func insertContentsOf<
+  public mutating func insert<
     S : Collection where S.Iterator.Element == Character
-  >(newElements: S, at i: Index)
+  >(contentsOf newElements: S, at i: Index)

-  public mutating func removeAtIndex(i: Index) -> Character
+  public mutating func remove(at i: Index) -> Character

-  public mutating func removeRange(subRange: Range<Index>)
+  public mutating func removeSubrange(subRange: Range<Index>)

-  mutating func removeAll(keepCapacity keepCapacity: Bool = false)
+  mutating func removeAll(keepingCapacity keepingCapacity: Bool = false)

-  public init(count: Int, repeatedValue c: Character)
+  public init(repeating repeatedValue: Character, count: Int)

-  public init(count: Int, repeatedValue c: UnicodeScalar)
+  public init(repeating repeatedValue: UnicodeScalar, count: Int)

-  public var utf8: UTF8View { get }
+  public var utf8: UTF8View { get set }

-  public var utf16: UTF16View { get }
+  public var utf16: UTF16View { get set }

-  public var characters: CharacterView { get }
+  public var characters: CharacterView { get set }
 }

 public enum UnicodeDecodingResult {
-  case Result(UnicodeScalar)
-  case EmptyInput
-  case Error
+  case scalarValue(UnicodeScalar)
+  case emptyInput
+  case error
 }

 public struct ManagedBufferPointer<Value, Element> : ... {
-  public var allocatedElementCount: Int { get }
+  public var capacity: Int { get }
 }

 public struct RangeIterator<Element : ForwardIndex> : ... {
-  public var startIndex: Element { get set }
-  public var endIndex: Element { get set }
 }

 public struct ObjectIdentifier : ... {
-  public var uintValue: UInt { get }
 }
 extension UInt {
+  /// Create a `UInt` that captures the full value of `objectID`.
+  public init(_ objectID: ObjectIdentifier)
 }
 extension Int {
+  /// Create an `Int` that captures the full value of `objectID`.
+  public init(_ objectID: ObjectIdentifier)
 }

-public struct Repeat<Element> : ... { ... }
+public struct Repeated<Element> : ... { ... }

 public struct StaticString : ... {
-  public var byteSize: Int { get }
+  public var utf8CodeUnitCount: Int { get }

   // Use the 'String(_:)' initializer.
-  public var stringValue: String { get }
 }

 extension Strideable {
-  public func stride(to end: Self, by stride: Stride) -> StrideTo<Self>
 }
+public func stride<T : Strideable>(from start: T, to end: T, by stride: T.Stride) -> StrideTo<T>

 extension Strideable {
-  public func stride(through end: Self, by stride: Stride) -> StrideThrough<Self>
 }
+public func stride<T : Strideable>(from start: T, through end: T, by stride: T.Stride) -> StrideThrough<T>

 public func transcode<
   Input : IteratorProtocol,
   InputEncoding : UnicodeCodec,
   OutputEncoding : UnicodeCodec
   where InputEncoding.CodeUnit == Input.Element>(
   inputEncoding: InputEncoding.Type, _ outputEncoding: OutputEncoding.Type,
   _ input: Input, _ output: (OutputEncoding.CodeUnit) -> Void,
-  stopOnError: Bool
+  stoppingOnError: Bool
 ) -> Bool

 extension UnsafeMutablePointer {
-  public static func alloc(num: Int) -> UnsafeMutablePointer<Pointee>
+  public init(allocatingCapacity count: Int)

-  public func dealloc(num: Int)
+  public func deallocateCapacity(count: Int)

-  public func initialize(newvalue: Memory)
+  public func initializePointee(newValue: Pointee, count: Int = 1)

-  public func move() -> Memory
+  public func take() -> Pointee

-  public func destroy()
-  public func destroy(count: Int)
+  public func deinitializePointee(count count: Int = 1)
 }

-public struct COpaquePointer : ... { ... }
+public struct OpaquePointer : ... { ... }

-public func unsafeAddressOf(object: AnyObject) -> UnsafePointer<Void>
+public func unsafeAddress(of object: AnyObject) -> UnsafePointer<Void>

-public func unsafeBitCast<T, U>(x: T, _: U.Type) -> U
+public func unsafeBitCast<T, U>(x: T, to: U.Type) -> U

-public func unsafeDowncast<T : AnyObject>(x: AnyObject) -> T
+public func unsafeDowncast<T : AnyObject>(x: AnyObject, to: T.Type) -> T

-public func print<Target: OutputStream>(
+public func print<Target : OutputStream>(
   items: Any...,
   separator: String = " ",
   terminator: String = "\n",
-  inout toStream output: Target
+  inout to output: Target
 )

-public func debugPrint<Target: OutputStream>(
+public func debugPrint<Target : OutputStream>(
   items: Any...,
   separator: String = " ",
   terminator: String = "\n",
-  inout toStream output: Target
+  inout to output: Target
 )

 public struct Unmanaged<Instance : AnyObject> {
-  public func toOpaque() -> COpaquePointer
 }
 extension OpaquePointer {
+  public init<T>(bitPattern bits: Unmanaged<T>)
 }

 public enum UnicodeDecodingResult
+  : Equatable {
-  public var isEmptyInput: Bool
}

-public func readLine(stripNewline stripNewline: Bool = true) -> String?
+public func readLine(strippingNewline strippingNewline: Bool = true) -> String?

 struct UnicodeScalar {
   // Use 'UnicodeScalar("\0")' instead.
-  init()

-  public func escape(asASCII forceASCII: Bool) -> String
+  public func escaped(asASCII forceASCII: Bool) -> String
 }

 public func transcode<
   Input : IteratorProtocol,
   InputEncoding : UnicodeCodec,
   OutputEncoding : UnicodeCodec
   where InputEncoding.CodeUnit == Input.Element
 >(
-  inputEncoding: InputEncoding.Type, _ outputEncoding: OutputEncoding.Type,
-  _ input: Input, _ output: (OutputEncoding.CodeUnit) -> Void,
-  stoppingOnError stopOnError: Bool
+  input: Input,
+  from inputEncoding: InputEncoding.Type,
+  to outputEncoding: OutputEncoding.Type,
+  stoppingOnError stopOnError: Bool,
+  sendingOutputTo processCodeUnit: (OutputEncoding.CodeUnit) -> Void
 ) -> Bool

 extension UTF16 {
-  public static func measure<
+  public static func transcodedLength<
     Encoding : UnicodeCodec, Input : IteratorProtocol
     where Encoding.CodeUnit == Input.Element
   >(
-    _: Encoding.Type, input: Input, repairIllFormedSequences: Bool
+    of input: Input,
+    decodedAs sourceEncoding: Encoding.Type,
+    repairingIllFormedSequences: Bool
-  ) -> (Int, Bool)?
+  ) -> (count: Int, isASCII: Bool)? {
 }

-public struct RawByte {}

-final public class VaListBuilder {}

-public func withVaList<R>(
-  builder: VaListBuilder,
-  @noescape _ f: CVaListPointer -> R)
--> R
```


## 기존 코드에 미치는 영향

제안된 변경 사항은 Swift 코드에 큰 영향을 미치며, Swift 2 코드를 Swift 3 코드로 변환하기 위해 마이그레이션 도구가 필요하다. 이 제안에서 나온 API 차이가 필요한 변환에 대한 주요 정보 소스가 될 것이다. 또한, 언어가 허용하는 범위 내에서 라이브러리는 기존 이름을 `renamed` 주석과 함께 사용 불가능한 심볼로 유지할 것이다. 이를 통해 컴파일러가 명확한 오류 메시지를 생성하고 Fix-Its를 제공할 수 있다.

[api-design-guidelines]: https://swift.org/documentation/api-design-guidelines  "API Design Guidelines"
[swift-repo]: https://github.com/apple/swift  "Swift repository"
[swift-3-api-guidelines-branch]: https://github.com/apple/swift/tree/swift-3-api-guidelines  "Swift 3 API Design Guidelines preview"

