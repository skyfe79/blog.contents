---
title: "코틀린 코루틴 가이드 - 기초"
date: 2019-11-04T22:52:44+09:00
draft: true
---

이 섹션에서는 코루틴의 기초 개념을 다룹니다. 

## 첫번째 코루틴

다음 코드를 실행합니다.

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello,") // main thread continues while coroutine is delayed
    Thread.sleep(2000L) // block main thread for 2 seconds to keep JVM alive
}
```

다음과 같은 결과를 볼 수 있습니다.

```
Hello,
World!
```

코루틴은 경량 스레드입니다. [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 코루틴 빌로 코루틴을 생성하며 생성된 코루틴은 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)에서 실행됩니다. 위 예제에서는 [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 새로운 코루틴을 실행했습니다. 실행된 코루틴의 수명은 전체 앱의 수명을 따릅니다.

`GlobalScope.launch { ... }`를 `thread { ... }`으로 바꾸고 `delay(...)`를 `Thread.sleep(...)`으로 바꾸면 동일한 결과를 얻을 수 있습니다.

`GlobalScope.launch`를 `thread`로 변경한 후 컴파일하면, 다음 오류가 발생합니다.

```
Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function
```

`delay`함수는 스레드는 차단하지 않고 코루틴만 일시 중단 시키는 일시 중단 함수입니다. 일시 중단 함수는 코루틴에서만 실행할 수 있기 때문에 위 오류가 발생한 것입니다.

## 블록킹과 넌블록킹 연결하기

첫번째 예제는 논블록킹 delay(...)와 블록킹 Thread.sleep(...)을 같은 코드에서 사용했습니다. 어느 함수가 블록킹이고 논블록킹인지 구분하기 어렵습니다. `runBlocking` 코루틴 빌더를 사용하여 `차단`을 명확하게 표현할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() { 
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    runBlocking {     // but this expression blocks the main thread
        delay(2000L)  // ... while we delay for 2 seconds to keep JVM alive
    } 
}
```

실행 결과

```
Hello,
World!
```

결과는 동일하지만 이 코드는 논블로킹 delay만 사용합니다. runBlocking을 실행하는 메인스레드는 내부 코루틴이 완료 될 때까지 차단됩니다.

runBlocking으로 메인 함수를 감싸서 좀 더 관용적으로 이 예제를 다시 작성할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // start main coroutine
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main coroutine continues here immediately
    delay(2000L)      // delaying for 2 seconds to keep JVM alive
}
```

실행 결과

```
Hello,
World!
```

여기서 `runBlocking<Unit> { ... }`은 최상위 메인 코루틴을 시작하는데 사용되는 어댑터 역할을 합니다. 코틀린에서 메인 함수는 Unit을 반환하기 때문에 Unit을 반환 타입으로 명확하게 작성했습니다.

이 방식은 일심 중단 함수를 테스트하는 단위테스트를 작성할 때 사용하는 방법입니다.

```kotlin
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // here we can use suspending functions using any assertion style that we like
    }
}
```

## Job 기다리기

다른 코루틴이 실행되는 동안 현재 코루틴을 지연시키는 것은 좋은 방법이 아닙니다. 우리가 실행한 백그라운드 Job이 완료 될 때까지(논블로킹 방식으로) 기다리겠습니다.

```kotlin
val job = GlobalScope.launch { // launch a new coroutine and keep a reference to its Job
    delay(1000L)
    println("World!")
}
println("Hello,")
job.join() // wait until child coroutine completes
```

실행 결과

```
Hello,
World!
```

결과는 여전히 동일하지만 메인 코루틴 코드는 백그라운드 작업 기간에 전혀 영향을 주지 않습니다. 훨씬 좋아 보입니다.

## 구조를 갖춘 동시성

코루틴을 실전에서 사용하려면 몇 가지가 더 필요합니다. GlobalScope.launch를 사용하면 최상위 코루틴이 생성됩니다. 코루틴이 가볍더라도 실행 중에는 메모리 리소스를 사용합니다. 새롭게 실행한 코루틴의 레퍼런스를 유지하지 못하면 그 코루틴은 계속 실행됩니다. 너무 오랫동안 지연해서 코루틴 코드가 정지하거나 너무 많은 코루틴을 실행해서 메모리가 부족하면 어떻게 해야 할까요? 실행 된 모든 코루틴 레퍼런스를 개발자가 직접 유지하고 join 하면 오류가 발생하기 쉽습니다.

더 나은 해결책이 있습니다. 구조를 갖춘 동시성을 사용하는 것입니다. 일반적으로 스레드를 실행하는 것(스레드는 항상 전역적임)과 달리 GlobalScope에서 코루틴을 실행할 필요가 없습니다. 그 대신 작업을 수행할 특정 스코프에서 코루틴을 시작할 수 있습니다.

In our example, we have main function that is turned into a coroutine using runBlocking coroutine builder. Every coroutine builder, including runBlocking, adds an instance of CoroutineScope to the scope of its code block. We can launch coroutines in this scope without having to join them explicitly, because an outer coroutine (runBlocking in our example) does not complete until all the coroutines launched in its scope complete. Thus, we can make our example simpler:
