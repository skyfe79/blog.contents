---
title: "[SE-0007] C 스타일 for 루프 제거: 조건과 증감 연산자"
date: 2025-03-09T00:13:49Z
author: "skyfe79"
draft: false
tags: ["swift"]
---

## C 스타일 for 루프 제거: 조건과 증감 연산자

* 제안: [SE-0007](0007-remove-c-style-for-loops.md)
* 작성자: [Erica Sadun](https://github.com/erica)
* 리뷰 관리자: [Doug Gregor](https://github.com/DougGregor)
* 상태: **구현 완료 (Swift 3.0)**
* 결정 노트: [Rationale](https://forums.swift.org/t/accepted-se-0007-remove-c-style-for-loops-with-conditions-and-incrementers/512)
* 버그: [SR-226](https://bugs.swift.org/browse/SR-226), [SR-227](https://bugs.swift.org/browse/SR-227)


## 소개

C 스타일 `for-loop`는 Swift 고유의 구조라기보다는 C 언어에서 기계적으로 가져온 요소로 보인다. 이 문법은 잘 사용되지 않으며, Swift 스타일과도 잘 맞지 않는다.

Swift에서는 이미 `for-in` 문과 `stride`를 통해 더 전형적인 반복 구문을 제공하고 있다. C 스타일 `for-loop`를 제거하면 언어를 단순화할 수 있으며, `--`와 `++` 연산자의 가장 흔한 사용처를 없앨 수 있다. 이 연산자들은 이미 언어에서 제거될 예정이다.

이 구조의 가치는 제한적이며, 제거를 진지하게 고려해야 한다고 생각한다.

이 제안은 Swift Evolution 포럼의 [C 스타일 For 루프](https://forums.swift.org/t/c-style-for-loops/31) 스레드에서 논의되었으며, [\[리뷰\] 조건과 증감 연산자를 사용한 C 스타일 for 루프 제거](https://forums.swift.org/t/review-remove-c-style-for-loops-with-conditions-and-incrementers/255) 스레드에서 검토되었다.


## for 반복문의 장점

Swift는 친숙한 상수와 제어 구조를 사용해 학습 곡선을 낮추는 설계를 채택했다. `for` 반복문은 C 언어와 유사하게 동작하며, 이 제어 흐름을 익히는 데 필요한 노력을 최소화한다.


## for 루프의 단점

1. `for-in`과 `stride`는 레거시 용어에 얽매이지 않으면서도 Swift 스타일의 일관된 접근 방식을 제공한다.  
1. `for-in`에 비해 `for-loop`는 간결성 측면에서 명확한 표현적 단점이 있다.  
1. `for-loop` 구현은 컬렉션이나 Swift의 핵심 타입과 함께 사용하기에 적합하지 않다.  
1. `for-loop`는 단항 증가/감소 연산자를 사용하도록 유도하며, 이는 곧 언어에서 제거될 예정이다.  
1. 세미콜론으로 구분된 선언 방식은 C 계열 언어가 아닌 다른 언어에서 온 사용자들에게 높은 학습 곡선을 요구한다.  
1. `for-loop`가 존재하지 않았다면, Swift 3에 포함될 가능성은 낮았을 것이다.


## 제안하는 접근 방식

Swift 2.x에서 for-loop를 더 이상 사용하지 않도록 권고하고, Swift 3에서는 완전히 제거할 것을 제안한다. 또한 Swift 프로그래밍 언어에서 관련 내용을 삭제하여 현재 2.2 업데이트의 수정 사항과 일치시키는 방안을 제안한다.


## 고려했던 대안들

Swift에서 `for-loop`를 제거하지 않아 언어를 간소화하고 불필요한 제어 흐름 요소를 제거할 기회를 놓쳤다.


## 기존 코드에 미치는 영향

Apple Swift 코드베이스를 검색한 결과, 이 기능은 거의 사용되지 않는다. Swift-Evolution 메일링 리스트의 커뮤니티 멤버들도 이 기능이 많은 프로급 앱에서 사용되지 않는다고 확인했다. 또한 `for-loop`가 필요한 경우에도 대체 방법을 사용할 수 있다. 예를 들어:

```swift
char *blk_xor(char *dst, const char *src, size_t len)
{
 const char *sp = src;
 for (char *dp = dst; sp - src < len; sp++, dp++)
   *dp ^= *sp;
 return dst;
}
```

위 코드와 아래 코드를 비교해 보자.

```swift
func blk_xor(dst: UnsafeMutablePointer<CChar>, src:
UnsafePointer<CChar>, len: Int) -> UnsafeMutablePointer<CChar> {
   for i in 0..<len {
       dst[i] ^= src[i]
   }
   return dst
}
```

Github의 Swift gist를 검색한 결과, 이 접근 방식은 주로 언어에 익숙하지 않은 초보자들이 사용하며, 언어를 숙달하면 더 이상 사용하지 않는다. 예를 들어:

```swift
for var i = 0 ; i < 10 ; i++ {
    print(i)
}
```

그리고

```swift
var array = [10,20,30,40,50]
for(var i=0 ; i < array.count ;i++){
    println("array[i] \(array[i])")
}
```


## 커뮤니티 반응

* "C 스타일 for 루프를 제거하는 것을 고려해도 좋을 것 같다. 내 생각엔 이 기능은 Swift에서 잘 사용되지 않으며, 그만한 가치가 없다고 본다. 제거해야 할 이유는 --와 ++ 연산자를 제거한 이유와 상당 부분 일치한다." -- Chris Lattner, clattner@apple.com  
* "직관적으로 Swift에서 C 스타일 for 루프가 더 이상 필요하지 않다고 완전히 동의한다. 우리에겐 더 풍부하고 구조화된 루프와 함수형 알고리즘이 있다. 다만, Swift 소스에서 C 스타일 for 루프가 실제로 얼마나 자주 사용되는지 데이터를 확인해보면 좋을 것 같다. GitHub의 Swift 소스를 빠르게 크롤링하면 알 수 있을 것이다. 이 기능이 시대에 뒤떨어진 느낌을 주고 거의 사용되지 않는다면, 제거 대상으로 적합하다." -- Douglas Gregnor, dgregor@apple.com  
* "Swift에서 C 스타일 for 루프를 사용할 때마다 .indices가 존재한다는 사실을 잊어버렸기 때문이었다. 이를 제거한다면, 해당 방향을 알려주는 fixme가 유용할 수 있다." -- David Smith, david_smith@apple.com  
* "참고로 Lyft 코드베이스에는 C 스타일 for 루프가 단 하나도 없다." -- Keith Smiley, keithbsmiley@gmail.com  
* "확인해봤는데, Khan Academy도 마찬가지다." -- Andy Matsuchak, andy@andymatuschak.org  
* "지난해 다양한 클라이언트를 위해 여러 Swift 앱을 개발했지만, C 스타일 for 루프가 필요하지 않았다." -- Eric Chamberlain, eric.chamberlain@arctouch.com  
* "C 스타일 for 루프를 사용하려고 할 때마다, 결국 while 루프로 전환하게 된다. 이는 반복 변수가 잘못된 타입(예: 본문이 실행되려면 값이 non-optional이어야 하는데 optional 타입을 가짐)을 가지기 때문이다. Postmates 코드베이스에는 Swift에서 C 스타일 for 루프가 전혀 없다." -- Lily Ballard, lily@sb.org  
* "내 코드베이스에서 몇 가지 사례를 발견했지만, 모두 '적절한' Swift 스타일 for 루프로 쉽게 변환할 수 있었고, 어쨌든 더 나아 보였다. 투표라면, C 스타일을 제거하는 쪽에 투표할 것이다." -- Sean Heber, sean@fifthace.com




