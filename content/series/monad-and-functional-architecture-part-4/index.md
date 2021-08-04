---
title: "모나드와 함수형 아키텍처 4장. Monad 실전 예제"
date: 2019-11-01T22:53:14+09:00
draft: false
categories: ["함수형 프로그래밍"]
tags: ["함수형 프로그래밍", "모나드", "함수형 아키텍처"]
---

## 4장. Monad 실전 예제

4부에서는 1부, 2부, 3부에서 배운 이론을 바탕으로 여러 모나드를 만들어 보겠습니다. 구현언어로 Kotlin을 사용하겠습니다.

### 4-1. Optional

Optional 모나드를 구현해 보겠습니다. Optional은 Swift, Kotlin에서 null이 가능한 타입입니다. 주로 Int? 처럼 구체타입에 ? 를 붙여서 표현합니다. 

우선 타입과 map, flatMap을 정의합니다.

```kotlin
sealed class Optional<T> {
    class None<T>: Optional<T>()
    data class Some<T>(val value: T): Optional<T>()
}

infix fun <T, R> Optional<T>.map(functor: (value: T) -> R): Optional<R> {
    return this.flatMap { value ->
        Optional.Some(functor(value))
    }
}

infix fun <T, R> Optional<T>.flatMap(functor: (value: T) -> Optional<R>): Optional<R> {
    return when (this) {
        is Optional.Some -> {
            functor(this.value)
        }
        is Optional.None -> {
            Optional.None()
        }
    }
}
```

Result 타입과 크게 다르지 않습니다. 정의한 Optional 모나드를 사용해 파일을 열고 콘텐츠인 텍스트 읽기를 구현해 보겠습니다.

```kotlin
fun testOptional() {

    fun openFile(fileName: String): Optional<File> {
        return try {
            Optional.Some(File(fileName))
        } catch (e: Throwable) {
            Optional.None()
        }
    }

    fun readFile(file: File): Optional<String> {
        return try {
            Optional.Some(file.inputStream().bufferedReader().use { it.readText() })
        } catch (e: Throwable) {
            Optional.None()
        }
    }

    val content = openFile("greeting.txt") flatMap ::readFile

    when (content) {
        is Optional.Some -> println(content.value)
        is Optional.None -> println("There is no file.")
    }
}
```

위에서 우리는 파일을 열고 파일 내용을 반환하는 작업을 두 개의 함수로 나누었습니다. 함수가 단 하나의 역할만 하도록 잘게 쪼갠 것입니다. 이렇게 함수를 최소 단위로 나누면 단위 테스트가 쉬워져서 좋습니다. 그리고 유지보수도 쉬워집니다. 

프로그램이 실행되는 곳에 `greeting.txt` 파일이 없다면 `There is no file.`이 출력될 것이고 파일이 있다면 파일의 내용이 출력될 것입니다. 아래와 같이 파일 내용을 만들어 `greeting.txt` 이름으로 파일을 저장합니다.

```
Hello! It's me!
```

다시 실행해 보면 이제는 파일의 내용이 출력되는 것을 확인할 수 있습니다. 만약 요구사항이 추가되어 모든 문자를 대문자로 출력하고 싶을 때는 어떻게 해야 할까요? map을 적용하면 그만입니다. 아래처럼 함수를 추가합니다.

```kotlin
fun testOptional() {

    fun openFile(fileName: String): Optional<File> {
        return try {
            Optional.Some(File(fileName))
        } catch (e: Throwable) {
            Optional.None()
        }
    }

    fun readFile(file: File): Optional<String> {
        return try {
            Optional.Some(file.inputStream().bufferedReader().use { it.readText() })
        } catch (e: Throwable) {
            Optional.None()
        }
    }

    fun uppercase(value: String): String {
        return value.toUpperCase()
    }

    val content = openFile("greeting.txt") flatMap ::readFile map ::uppercase

    when (content) {
        is Optional.Some -> println(content.value)
        is Optional.None -> println("There is no file.")
    }
}
```

아래처럼 출력됩니다.

```
HELLO! IT'S ME!
```

중요한 것은 기존 함수는 수정하지 않고 새로운 함수를 추가하여 문제를 해결한 점입니다. 함수를 잘게 나누어 프로그램을 개발하면 기존 로직의 수정과 삭제는 최대한 줄이고 새로운 함수만 추가하여 개발할 수 있습니다. 유지보수 문제가 무척 쉬워지는 것입니다.


### 4-2. Result

Result 모나드는 3부에서 만든 모나드입니다. Kotlin으로 만든 것은 이미 보았으므로 여기서는 Swift로 구현해 보겠습니다.

```swift
public enum Result<T> {
    case success(T)
    case fail
    
    func map<R>(_ functor: (T) -> R) -> Result<R> {
        return self.flatMap { value in
            Result<R>.success(functor(value))
        }
    }
    
    func flatMap<R>(_ functor: (T) -> Result<R>) -> Result<R> {
        switch self {
        case .success(let value):
            return functor(value)
        case .fail:
            return Result<R>.fail
        }
    }
}
```

그리고 함수를 정의합니다.

```swift
func a(_ value: Int) -> Result<Int> {
    if (value > 0) {
        return .success(value * 10)
    } else {
        return .fail
    }
}

func d(_ value: Int) -> Int {
    return value * 10
}
```

합성을 적용해 봅니다.

```swift
public func testResultSuccess() {
    let result = a(10).map(d).map(d).map(d).map { "\($0 * 10)" }

    switch result {
    case .success(let value):
        print("SUCCESS :: \(value)")
    case .fail:
        print("failed")
    }
}

public func testResultFail() {
    let result = a(10).map(d).map(d).flatMap { a($0 * -1) }.map(d).map { "\($0 * 10)" }

    switch result {
    case .success(let value):
        print("SUCCESS :: \(value)")
    case .fail:
        print("failed")
    }
}
```

실행하면 결과가 출력됩니다.

```swift
testResultSuccess()
testResultFail()

//출력
SUCCESS :: 1000000
failed
```

### 4-3. Either

Either 모나드를 구현해 봅시다. Either는 Optional과 비슷한 모나드로 Left, Right 두 개의 집합을 갖습니다. Optional의  None이 Left에 그리고 Some이 Right에 매칭됩니다. Optional과 다른 점은 예외나 오류가 발생했을 때 Left에 해당 정보를 담을 수 있다는 점입니다. 그리고 Either 모나드가 Left일 때에는 합성이 더 진행되지 않고 멈춘다는 것입니다. 우리가 예외가 발생했을 때 함수의 실행을 멈추는 것과 동일합니다.

```kotlin
sealed class Either<out L, out R> {
    data class Left<out L>(val value: L): Either<L, Nothing>()
    data class Right<out R>(val value: R): Either<Nothing, R>()
}

infix fun <L, R, P> Either<L, R>.map(functor: (value: R) -> P): Either<L, P> {
    return this.flatMap { value ->
        Either.Right(functor(value))
    }
}

infix fun <L, R, Q> Either<L, R>.flatMap(functor: (value: R) -> Either<L, Q>): Either<L, Q> {
    return when (this) {
        is Either.Left -> {
            return Either.Left(this.value)
        }
        is Either.Right -> {
            functor(this.value)
        }
    }
}
```

Either 모나드가 Left일 때 합성이 더 진행되지 않고 멈추는 이유는 flatMap 에서 Either.Right일 때만 functor를 실행하기 때문입니다. 예외가 발생했을 때 발생한 이후로 실행하지 않으려 할 때 Either를 사용하면 편리합니다.

테스트해 봅시다.

```kotlin
fun testEither() {
    fun openFile(fileName: String): Either<Throwable, File> {
        return try {
            val file = File(fileName)
            if (file.exists()) {
                Either.Right(file)
            } else {
                Either.Left(FileNotFoundException())
            }
        } catch (e: Throwable) {
            Either.Left(e)
        }
    }

    fun readFile(file: File): Either<Throwable, String> {
        return try {
            Either.Right(file.inputStream().bufferedReader().use { it.readText() })
        } catch (e: Throwable) {
            Either.Left(e)
        }
    }

    val content = openFile("greeting.txt") flatMap ::readFile

    when (content) {
        is Either.Left -> println(content.value)
        is Either.Right -> println(content.value)
    }
}
```

파일이 있다면 아래처럼 출력됩니다.

```
Hello! It's me!
```

파일이 없다면 아래처럼 출력됩니다.

```
java.io.FileNotFoundException: greeting2.txt (No such file or directory)
```

### 4-4. Future

Future 모나드는 비동기(async) 작업에 사용되는 모나드입니다. 비동기 작업(async task)은 동기 작업(sync task)과 다르게 값을 얻을 수 있는 시점을 알 수가 없습니다. 그래서 자주 사용되는 패턴이 Callback 패턴입니다. 비동기 작업이 완료되면 콜백을 호출해 값을 전해 줍니다. 그러나 콜백 패턴은 여러 비동기 작업을 체이닝하여 실행할 때 콜백지옥이 형성되어 코드 파악과 유지보수를 어렵게 합니다. 

Future 모나드 사용하면 콜백지옥 없이 비동기 작업을 체이닝할 수 있습니다. Javascript의 Promise도 Future와 비슷한 모나드입니다. 

여기서 구현하는 Future 모나드는 비동기 작업을 모나드로 추상화하는 개념을 보여주는 예제일 뿐입니다. 프로덕션 코드에 사용하시면 안 됩니다. :) 비동기 작업이 반환하는 값을 Future 모나드로 추상화하고 map과 flatMap으로 체이닝을 해 봅시다.

구현 아이디어는 아주 간단합니다. 사실 콜백 패턴을 그대로 따르고 있습니다. 내부에 콜백과 구독자(Subscribers) 목록을 가지고 있습니다. 비동기 작업이 끝나면 구독자에게 콜백을 사용해 값을 전달합니다.

우선 콜백타입을 정의합니다. 콜백은 Either 모나드를 사용해 오류 또는 값을 전달합니다.

```kotlin
typealias Callback<Err, T> = (Either<Err, T>) -> Unit
```

이제 비동기 작업을 실행할 스케줄러를 정의합니다. RxJava 스타일을 만들기 위해 정의한 것입니다. 별 의미는 없습니다.

```kotlin
interface Scheduler {
    fun execute(command: () -> Unit)
    fun shutdown()
}

object SchedulerIO: Scheduler {
    private var executorService = Executors.newFixedThreadPool(1)
    override fun execute(command: () -> Unit) {
        if (executorService.isShutdown) {
            executorService = Executors.newFixedThreadPool(1)
        }
        executorService.execute(command)
    }

    override fun shutdown() {
        if (!executorService.isShutdown) {
            executorService.shutdown()
        }
    }
}

object SchedulerMain: Scheduler {
    override fun execute(command: () -> Unit) {
        Thread(command).run()
    }
    override fun shutdown() = Unit
}
```

이제 Future 모나드를 구현해 봅시다. Future 모나드는 내부에 콜백 람다를 가지고 있습니다. 콜백이 실행되면 결과값을 cache에 담아 두었다가 모나드에서 구체타입의 값을 추출하는 subscribe에서 cache의 값을 사용합니다.

```kotlin
class Future<Err, V>(private var scheduler: Scheduler = SchedulerIO) {
    var subscribers: MutableList<Callback<Err, V>> = mutableListOf()
    private var cache: Optional<Either<Err, V>> = Optional.None()
    var semaphore = Semaphore(1)

    private var callback: Callback<Err, V> = { value ->
        semaphore.acquire()
        cache = Optional.Some(value)
        while (subscribers.size > 0) {
            val subscriber = subscribers.last()
            subscribers = subscribers.dropLast(1).toMutableList()
            scheduler.execute {
                subscriber.invoke(value)
            }
        }
        semaphore.release()
    }

    fun create(f: (Callback<Err, V>) -> Unit): Future<Err, V> {
        scheduler.execute {
            f(callback)
        }
        return this
    }
    
    fun subscribe(cb: Callback<Err, V>): Disposable {
        semaphore.acquire()
        when (cache) {
            is Optional.None -> {
                subscribers.add(cb)
                semaphore.release()
            }
            is Optional.Some -> {
                semaphore.release()
                val c = (cache as Optional.Some<Either<Err, V>>)
                cb.invoke(c.value)
            }
        }
        return Disposable()
    }

    fun <P> map(functor: (value: V) -> P): Future<Err, P> {
        return this.flatMap { value ->
            Future<Err, P>().create { callback ->
                callback(Either.Right(functor(value)))
            }
        }
    }

    fun <Q> flatMap(functor: (value: V) -> Future<Err, Q>): Future<Err, Q> {
        return Future<Err, Q>().create { callback ->
            this.subscribe { value ->
                when (value) {
                    is Either.Left -> {
                        callback(Either.Left(value = value.value))
                    }
                    is Either.Right -> {
                        functor(value.value).subscribe(callback)
                    }
                }
            }
        }
    }

    inner class Disposable {
        fun dispose() {
            scheduler.shutdown()
        }
    }
}
```

Future 모나드를 사용해 보기 위해서 비동기를 흉내 내는 count 함수를 정의합니다. 1초를 sleep하고 1을 증가시키는 함수입니다. 결과는 Future를 반환합니다.

```kotlin
fun count(n: Int): Future<Throwable, Int> {
    return Future<Throwable, Int>().create { callback ->
        Thread.sleep(1000)
        callback.invoke(Either.Right(n + 1))
    }
}
```

RxJava의 Single을 사용하는 것과 비슷합니다.

```kotlin
fun count(n: Int): Single<Int> {
    return Single.create { emitter -> 
        Thread.sleep(1000)
        emitter.success(n + 1)
    }
}
```

count 함수를 사용하여 5까지 카운트를 하도록 체이닝을 해 보겠습니다.

```kotlin
fun testFuture(): Future<Throwable, Int> {
    return Future<Throwable, Int>()
        .create { callback ->
            Thread.sleep(1000)
            callback.invoke(Either.Right(1))
        }
        .flatMap(::count)
        .map { it + 1 }
        .flatMap(::count)
        .flatMap(::count)
}
```

flatMap을 사용하여 비동기 작업을 계속해서 체이닝 할 수 있고 비동기 함수가 반환하는 값을 map을 사용하여 동기 스타일로 연산을 수행할 수 있음을 알 수 있습니다. 이 함수를 구독하는 함수는 아래와 같습니다.

```kotlin
fun main(args: Array<String>) {

    val disposable = testFuture()
        .subscribe {
            when (it) {
                is Either.Left -> print(it.value)
                is Either.Right -> print(it.value)
            }
        }
    Thread.sleep(5000L)
    disposable.dispose()

    val disposable2 = testFuture()
        .subscribe {
            when (it) {
                is Either.Left -> print(it.value)
                is Either.Right -> print(it.value)
            }
        }

    Thread.sleep(5000L)
    disposable2.dispose()
}
```

아래처럼 출력되고 프로그램이 종료됩니다.

```
5
5

Process finished with exit code 0
```

### 4-5. 정리

지금까지 여러 개의 모나드를 구현하고 사용해 보았습니다. 객체지향 시대에는 수많은 클래스와 함께했고 이제는 수많은 모나드와 함께하고 있습니다. 여러분도 함수의 인자 또는 결과를 구체 타입이 아닌 자신만의 모나드로 추상화해 보기를 권해 드립니다. 

모나드를 정의하고 map가 flatMap을 만든 것은 흐름을 만들기 위함이었습니다. 실제로는 flatMap이 흐름을 만들어 줍니다. Either[Left, Right] 모나드를 구현할 때, Right일 때만 흐름이 진행되었습니다. 다시 Either 모나드 정의 코드를 보겠습니다. 

```kotlin
sealed class Either<out L, out R> {
    data class Left<out L>(val value: L): Either<L, Nothing>()
    data class Right<out R>(val value: R): Either<Nothing, R>()
}

infix fun <L, R, P> Either<L, R>.map(functor: (value: R) -> P): Either<L, P> {
    return this.flatMap { value ->
        Either.Right(functor(value))
    }
}

infix fun <L, R, Q> Either<L, R>.flatMap(functor: (value: R) -> Either<L, Q>): Either<L, Q> {
    return when (this) {
        is Either.Left -> {
            return Either.Left(this.value)
        }
        is Either.Right -> {
            functor(this.value)
        }
    }
}
```

위 코드에서 중요하게 기억해야 할 점은 flatMap에서 흐름을 진행할 타입에서 functor를 실행해 주는 것입니다. Either 모나드는 다른 Either 모나드가 Either.Right일 때만 아래처럼 흐름이 만들어집니다.

```kotlin
val content = openFile("greeting.txt") flatMap ::readFile map ::uppercase
```

자신만의 모나드를 만들 때 이점을 기억하면서 만들면 됩니다. 4부에서 구현한 모나드의 구현 코드는 [Option/Maybe, Either, and Future Monads in JavaScript, Python, Ruby, Swift, and Scala](https://www.toptal.com/javascript/option-maybe-either-future-monads-js) 글의 내용을 참고한 것입니다. 이 글에는 JavaScript, Python, Ruby, Swift 그리고 Scala로 지금까지 살펴본 모나드 구현 코드를 볼 수 있습니다. 단, Kotlin으로 구현된 코드가 없어서 4부에서는 Kotlin을 중심으로 구현해 보았습니다.

`알림` 이 글은 [데이블 기술블로그](https://teamdable.github.io/techblog/Moand-and-Functional-Architecture)에 올린 글을 제 블로그에 다시 올린 글임을 알려드립니다.