---
title: "[SE-0012] 공개 라이브러리 API에 @noescape 추가(Rejected)"
date: 2025-03-09T00:22:36Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## 공개 라이브러리 API에 `@noescape` 추가

* 제안: [SE-0012](0012-add-noescape-to-public-library-api.md)
* 작성자: [Jacob Bandes-Storch](https://github.com/jtbandes)
* 리뷰 매니저: [Philippe Hausler](https://github.com/phausler)
* 상태: **거부됨**
* 결정 근거: [Rationale](https://forums.swift.org/t/rejected-se-0097-normalizing-naming-for-negative-attributes/2854/9)


##### 변경 이력

* **v1** 초기 버전
* **v1.2** 컴포넌트 소유자 리뷰 및 논의 후 업데이트


## 요약

* Swift는 클로저의 실행이 함수 호출 범위를 벗어나지 않음을 보장하는 `@noescape` 선언 속성을 제공한다.
* clang은 "noescape" 속성을 통해 이를 지원하며, 이는 자동으로 Swift의 `@noescape`로 임포트된다.
* CF와 Foundation에서 이 속성을 `CF_NOESCAPE`와 `NS_NOESCAPE`로 노출할 것을 제안한다.
* 또한 CF와 Foundation의 여러 클로저를 사용하는 API에 이 선언을 적용할 것을 제안한다.

[Swift Evolution 토론 스레드: \[Pitch\] make @noescape the default](https://forums.swift.org/t/pitch-make-noescape-the-default/664)


## 소개

### `@noescape`

Swift는 클로저 파라미터에 적용할 수 있는 `@noescape` 선언 [속성](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Closures.html#//apple_ref/doc/uid/TP40014097-CH11-ID546)을 제공한다. 이 속성은 클로저의 실행이 함수 호출 범위를 벗어나지 않음을 보장한다.

```swift
func withLock(@noescape perform closure: () -> Void) {
    myLock.lock()
    closure()
    myLock.unlock()
}
```

따라서 클로저 인자는 함수가 반환되기 *전에* 실행된다(실행된다면). 이는 컴파일러가 `self`의 불필요한 캡처/유지/해제를 생략하는 등의 다양한 최적화를 수행할 수 있게 한다.

예를 들어, 메서드 내부에서 "`self.`"를 생략할 수 있는 것처럼, `@noescape` 클로저는 `self`를 캡처하지 않음이 명확하므로 `self.` 접두어 없이 프로퍼티와 메서드에 접근할 수 있다:

```swift
class MyClass {
    var counter = 0

    func incrementCounter() {
        counter += 1  // 인스턴스 메서드 내에서 "self." 생략

        withLock {
            // @noescape가 없다면, 아래 라인은 다음 오류를 발생시킨다
            //   "클로저 내에서 프로퍼티 'counter'에 대한 참조는
            //    캡처 의미를 명확히 하기 위해 'self.'를 명시해야 함".
            counter += 1
        }
    }
}
```


### C와 Objective-C에서

Clang은 `noescape` 속성을 이해하며, 이는 `__attribute__((noescape))` 또는 `__attribute__((__noescape__))`로 표기한다. 블록이나 함수 포인터 매개변수에 이 속성이 있는 함수 정의를 Swift로 임포트하면, Swift의 `@noescape` 속성으로 표시된다.

```c
void performWithLock(__attribute__((noescape)) void (^block)()) {  // Swift에서 @noescape로 노출됨
    lock(myLock);
    block();
    unlock(myLock);
}
```


```objective-c
- (void)performWithLock:(__attribute__((noescape)) void (^)())block {  // Swift에서 @noescape로 노출됨
    [myLock lock];
    block();
    [myLock unlock];
}
```


## 동기

Foundation과 libdispatch 같은 표준 라이브러리의 많은 메서드와 함수는 **비탈출 클로저(non-escaping closure) 의미론**을 따르지만, **`__attribute__((noescape))` 속성이 누락된 경우가 많다**. 이로 인해 `@noescape`가 제공하는 컴파일러 최적화와 문법 단축 기능을 활용할 수 없게 된다. 이러한 기능은 원래 적용되어야 하는 상황임에도 불구하고 말이다.

순수 Swift 환경에서는 이 문제를 해결할 방법이 없다. 하지만 커스텀 C/Objective-C 래퍼 함수를 작성하면 이러한 제약을 우회할 수 있다:

```objective-c
// MyProject-Bridging-Header.h

NS_INLINE void MyDispatchSyncWrapper(dispatch_queue_t queue, __attribute__((noescape)) dispatch_block_t block)
{
    dispatch_sync(queue, block);
}
```

하지만 비탈출 의미론을 가진 라이브러리 함수는 소스 코드에서 `noescape` 속성으로 명시적으로 표시되어야 한다. 이렇게 하면 사용자가 매번 함수를 래핑할 필요 없이 바로 활용할 수 있다.


## 제안하는 해결 방안

1. 클로저 파라미터가 호출 수명을 벗어나지 않음이 보장되는 함수와 메서드를 위해 시스템 C/Objective-C 라이브러리(libdispatch, Foundation 등)를 감사한다.
   - *후보 함수/메서드 목록은 이 문서 끝부분을 참고한다.*
2. 적절한 경우, 블록/함수 포인터 파라미터에 `__attribute__((noescape))`를 매크로를 통해 추가한다.
3. CoreFoundation/Foundation과 같은 공통 영역에 새로운 매크로를 추가해 이 속성을 컴파일러가 지원하도록 한다. 이 매크로는 상위 프레임워크와 애플리케이션이 적절한 경우 이 주석을 채택할 수 있게 한다.
4. Swift 전용 포크가 있는 라이브러리(예: [swift-corelibs-libdispatch](https://github.com/apple/swift-corelibs-libdispatch))의 경우, Apple 내부 상위 버전에서도 동일한 변경을 적용한다.


###### 예제 패치

libdispatch에 대한 예제 패치는 https://github.com/apple/swift-corelibs-libdispatch/pull/6/files에서 확인할 수 있다.


## 기존 코드에 미치는 영향

이전에 새로 `@noescape`로 지정된 함수를 사용한 사용자들은 코드에 불필요한 `self.` 인스턴스가 있을 수 있다. 그러나 문법적으로 문제가 생기거나 기능상 차이가 발생하지는 않는다.


## 고려한 대안들

Swift 컴파일러가 지원하는 추가적인 "API 노트"(`.apinotes` 파일)를 확장하여 클로저 파라미터를 비탈출(non-escaping)으로 표시하는 데 사용할 수 있다.

하지만 다음과 같은 이유로 헤더에 주석을 추가하는 것이 더 나은 방법이라고 생각한다:
- 라이브러리 헤더에 `__attribute__((noescape))`가 있으면 API 계약이 명확해지고, 사용자들이 이 속성을 적절한 경우 자신의 코드에 사용하도록 장려한다.
- API 노트를 사용할 경우, 이점이 특정 라이브러리와 함수에만 한정되며, 주석 추가는 Swift 컴파일러 프로젝트의 손에 달려 있다. 반면, 주석이 추가된 헤더가 있는 라이브러리 버전이 있다면, 추가 컴파일러 설정 없이도 주석의 이점을 활용할 수 있다.
- Clang 자체가 개선됨에 따라 `__attribute__((noescape))`의 이점이 Swift뿐만 아니라 Objective-C 호출자에게도 제공될 수 있다(예: `-Wimplicit-retain-self` 경고 억제 &lt;rdar://19914650>).


## CoreFoundation

CoreFoundation은 이제 noescape 메서드를 주석 처리하기 위한 매크로를 제공한다. 다음 공개 함수들은 이에 따라 주석 처리될 것이다:

```c
#if __has_attribute(noescape)
#define CF_NOESCAPE __attribute__((noescape))
#else
#define CF_NOESCAPE
#endif
```


### CFArray

```c
void CFArrayApplyFunction(CFArrayRef theArray, CFRange range, CFArrayApplierFunction CF_NOESCAPE applier, void *context);
```


### CFBag

```c
void CFBagApplyFunction(CFBagRef theBag, CFBagApplierFunction CF_NOESCAPE applier, void *context);
```


### CFDictionary

```c
void CFDictionaryApplyFunction(CFDictionaryRef theDict, CFDictionaryApplierFunction CF_NOESCAPE applier, void *context);
```


### CFSet

```c
void CFSetApplyFunction(CFSetRef theSet, CFSetApplierFunction CF_NOESCAPE applier, void *context);
```


### CFTree

```c
void CFTreeApplyFunctionToChildren(CFTreeRef tree, CFTreeApplierFunction CF_NOESCAPE applier, void *context);
```


## 기초

Foundation은 다음과 같이 주석이 달린 매크로와 메서드를 제공한다:

```objc
#define NS_NOESCAPE CF_NOESCAPE
```


### NSArray

```objc
- (NSArray<ObjectType> *)sortedArrayUsingFunction:(NSInteger (NS_NOESCAPE *)(ObjectType, ObjectType, void * _Nullable))comparator context:(nullable void *)context;

- (NSArray<ObjectType> *)sortedArrayUsingFunction:(NSInteger (NS_NOESCAPE *)(ObjectType, ObjectType, void * _Nullable))comparator context:(nullable void *)context hint:(nullable NSData *)hint;

- (void)enumerateObjectsUsingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateObjectsAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexOfObjectPassingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexOfObjectWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexOfObjectAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSIndexSet *)indexesOfObjectsPassingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSIndexSet *)indexesOfObjectsWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSIndexSet *)indexesOfObjectsAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr NS_AVAILABLE(10_6, 4_0);

- (NSArray<ObjectType> *)sortedArrayWithOptions:(NSSortOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmptr NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexOfObject:(ObjectType)obj inSortedRange:(NSRange)r options:(NSBinarySearchingOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmp NS_AVAILABLE(10_6, 4_0); // 바이너리 검색
```


### NSMutableArray

```objc
- (void)sortUsingFunction:(NSInteger (NS_NOESCAPE *)(ObjectType,  ObjectType, void * _Nullable))compare context:(nullable void *)context;

- (void)sortUsingComparator:(NSComparator NS_NOESCAPE)cmptr NS_AVAILABLE(10_6, 4_0);

- (void)sortWithOptions:(NSSortOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmptr NS_AVAILABLE(10_6, 4_0);
```


### NSAttributedString

```objc
- (void)enumerateAttributesInRange:(NSRange)enumerationRange options:(NSAttributedStringEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE  ^)(NSDictionary<NSString *, id> *attrs, NSRange range, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateAttribute:(NSString *)attrName inRange:(NSRange)enumerationRange options:(NSAttributedStringEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(id _Nullable value, NSRange range, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);
```


### NSCalendar

```objc
- (void)enumerateDatesStartingAfterDate:(NSDate *)start matchingComponents:(NSDateComponents *)comps options:(NSCalendarOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSDate * _Nullable date, BOOL exactMatch, BOOL *stop))block NS_AVAILABLE(10_9, 8_0);
```


### NSData

```objc
- (void) enumerateByteRangesUsingBlock:(void (NS_NOESCAPE ^)(const void *bytes, NSRange byteRange, BOOL *stop))block NS_AVAILABLE(10_9, 7_0);
```


### NSDictionary

```objc
- (void)enumerateKeysAndObjectsUsingBlock:(void (NS_NOESCAPE ^)(KeyType key, ObjectType obj, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateKeysAndObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(KeyType key, ObjectType obj, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (NSArray<KeyType> *)keysSortedByValueUsingComparator:(NSComparator NS_NOESCAPE)cmptr NS_AVAILABLE(10_6, 4_0);

- (NSArray<KeyType> *)keysSortedByValueWithOptions:(NSSortOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmptr NS_AVAILABLE(10_6, 4_0);

- (NSSet<KeyType> *)keysOfEntriesPassingTest:(BOOL (NS_NOESCAPE ^)(KeyType key, ObjectType obj, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSSet<KeyType> *)keysOfEntriesWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(KeyType key, ObjectType obj, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
```


```objc
- (void)enumerateIndexesUsingBlock:(void (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateIndexesWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateIndexesInRange:(NSRange)range options:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexPassingTest:(BOOL (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSUInteger)indexInRange:(NSRange)range options:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSIndexSet *)indexesPassingTest:(BOOL (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSIndexSet *)indexesWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSIndexSet *)indexesInRange:(NSRange)range options:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(NSUInteger idx, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (void)enumerateRangesUsingBlock:(void (NS_NOESCAPE ^)(NSRange range, BOOL *stop))block NS_AVAILABLE(10_7, 5_0);

- (void)enumerateRangesWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSRange range, BOOL *stop))block NS_AVAILABLE(10_7, 5_0);

- (void)enumerateRangesInRange:(NSRange)range options:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSRange range, BOOL *stop))block NS_AVAILABLE(10_7, 5_0);
```


### NSLinguisticTagger

```objc
- (void)enumerateTagsInRange:(NSRange)range scheme:(NSString *)tagScheme options:(NSLinguisticTaggerOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSString *tag, NSRange tokenRange, NSRange sentenceRange, BOOL *stop))block NS_AVAILABLE(10_7, 5_0);

- (void)enumerateLinguisticTagsInRange:(NSRange)range scheme:(NSString *)tagScheme options:(NSLinguisticTaggerOptions)opts orthography:(nullable NSOrthography *)orthography usingBlock:(void (NS_NOESCAPE ^)(NSString *tag, NSRange tokenRange, NSRange sentenceRange, BOOL *stop))block NS_AVAILABLE(10_7, 5_0);
```


### NSMetadataQuery

```objc
- (void)enumerateResultsUsingBlock:(void (NS_NOESCAPE ^)(id result, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_9, 7_0);

- (void)enumerateResultsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(id result, NSUInteger idx, BOOL *stop))block NS_AVAILABLE(10_9, 7_0);
```


### NSOrderedSet

```objc
- (void)enumerateObjectsUsingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))block;

- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))block;

- (void)enumerateObjectsAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))block;

- (NSUInteger)indexOfObjectPassingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate;

- (NSUInteger)indexOfObjectWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate;

- (NSUInteger)indexOfObjectAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate;

- (NSIndexSet *)indexesOfObjectsPassingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate;

- (NSIndexSet *)indexesOfObjectsWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate;

- (NSIndexSet *)indexesOfObjectsAtIndexes:(NSIndexSet *)s options:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, NSUInteger idx, BOOL *stop))predicate;

- (NSUInteger)indexOfObject:(ObjectType)object inSortedRange:(NSRange)range options:(NSBinarySearchingOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmp; // 바이너리 검색

- (NSArray<ObjectType> *)sortedArrayUsingComparator:(NSComparator NS_NOESCAPE)cmptr;

- (NSArray<ObjectType> *)sortedArrayWithOptions:(NSSortOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmptr;
```


### NSMutableOrderedSet

```objc
- (void)sortUsingComparator:(NSComparator NS_NOESCAPE)cmptr;

- (void)sortWithOptions:(NSSortOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmptr;

- (void)sortRange:(NSRange)range options:(NSSortOptions)opts usingComparator:(NSComparator NS_NOESCAPE)cmptr;
```


### NSRegularExpression

```objc
- (void)enumerateMatchesInString:(NSString *)string options:(NSMatchingOptions)options range:(NSRange)range usingBlock:(void (NS_NOESCAPE ^)(NSTextCheckingResult * _Nullable result, NSMatchingFlags flags, BOOL *stop))block;
```


### NSSet

```objc
- (void)enumerateObjectsUsingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (void)enumerateObjectsWithOptions:(NSEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(ObjectType obj, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);

- (NSSet<ObjectType> *)objectsPassingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);

- (NSSet<ObjectType> *)objectsWithOptions:(NSEnumerationOptions)opts passingTest:(BOOL (NS_NOESCAPE ^)(ObjectType obj, BOOL *stop))predicate NS_AVAILABLE(10_6, 4_0);
```


### NSString

```objc
- (void)enumerateSubstringsInRange:(NSRange)range options:(NSStringEnumerationOptions)opts usingBlock:(void (NS_NOESCAPE ^)(NSString * _Nullable substring, NSRange substringRange, NSRange enclosingRange, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);
- (void)enumerateLinesUsingBlock:(void (NS_NOESCAPE ^)(NSString *line, BOOL *stop))block NS_AVAILABLE(10_6, 4_0);
```




