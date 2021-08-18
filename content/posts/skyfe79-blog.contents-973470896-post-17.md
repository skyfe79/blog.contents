---
title: "Swift 알고리즘 클럽: Array2D"
date: 2021-08-18T09:36:20Z
draft: false
tags: ["swift","알고리즘 클럽"]
---

앞으로 시간이 생길 때마다 [Swift 알고리즘 클럽](https://github.com/raywenderlich/swift-algorithm-club) 내용을 공부해 보자😋 우선 쉬운 자료구조부터 살펴보자. [Array2D](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Array2D)!

우선 실습을 위해서 테스트 환경을 구성하자~

```
$ mkdir array2d
$ cd array2d
$ swift package init
```

C와 Objective-C 에서 2차원 배열 선언은 아래와 같다.

```c
int cookies[9][7];
```

원소 접근은 인덱스를 사용한다.

```c
int myCookie = cookies[3][6]
```

Swift는 위처럼 선언할 수 없다. Swift의 2차원 배열 선언 및 초기화는 좀 장황하다.

```swift
var cookies = [[Int]]()
    for _ in 1...9 {
        var row = [Int]()
        for _ in 1...7 {
            row.append(0)
        }
        cookies.append(row)
    }
```

위처럼 초기화한 배열에 인덱스를 사용해 접근할 수 있다.

```swift
let myCookie = cookies[3][6]
print(myCookie)
```

물론 한 줄로 2차원 배열을 선언할 수 있다. 

```swift
let cookies = [[Int]](repeating: [Int](repeating: 0, count: 7), count: 9)
let myCookie = cookies[3][6]
print(myCookie)
```

그래도 C언어만큼 쉬워 보이진 않는다.

축약할 수 있는 함수를 정의해 2차원 배열 선언을 좀 더 쉽게 할 수 있다.

```swift
func dim<T>(_ count: Int, _ value: T) -> [T] {
    return [T](repeating: value, count: count)
}

let cookies = dim(9, dim(7, 0))
let myCookie = cookies[3][6]
print(myCookie)
```

dim 함수를 사용하면 2차원 보다 높은 고차원 배열을 쉽게 만들 수 있다.

```swift
let threeDimensions = dim(2, dim(3, dim(4, 0)))
let element = threeDimensions[1][1][1]
print(element)
```

dim 함수 대신에 좀 더 명확한 Array2D 데이터 타입을 만들어 보자.

```swift
public struct Array2D<T> {
    public let columns: Int
    public let rows: Int
    fileprivate var array: [T]

    public init(columns: Int, rows: Int, initialValue: T) {
        self.columns = columns
        self.rows = rows
        array = .init(repeating: initialValue, count: rows * columns)
    }

    public subscript(column: Int, row: Int) -> T {
        get {
            precondition(0 <= column && column < columns, "Column \(column) Index is out of range. Array<T>(columns: \(columns), rows:\(rows))")
            precondition(0 <= row && row < rows, "Row \(row) Index is out of range. Array<T>(columns: \(columns), rows:\(rows))")
            return array[row * columns + column]
        }
        set {
            precondition(0 <= column && column < columns, "Column \(column) Index is out of range. Array<T>(columns: \(columns), rows:\(rows))")
            precondition(0 <= row && row < rows, "Row \(row) Index is out of range. Array<T>(columns: \(columns), rows:\(rows))")
            array[row * columns + column] = newValue
        }
    }
}
```

1차원 배열을 사용하여 2차원 배열을 간단하게 정의해 보았다. 그리고 `subscript` 메서드를 구현했기 때문에 인덱스로 원소에 접근 가능하다.

```swift
var cookies = Array2D(columns: 9, rows: 7, initialValue: 0)
var myCookie = cookies[3, 6]
print(myCookie)
cookies[3, 6] = 10
myCookie = cookies[3, 6]
print(myCookie)
```

위 코드에서 [precondition](https://developer.apple.com/documentation/swift/1540960-precondition)은 `assert`와 비슷한 구문으로 조건이 false가 되면 프로그램은 실행을 멈추고 뒤의 메시지를 출력한다.