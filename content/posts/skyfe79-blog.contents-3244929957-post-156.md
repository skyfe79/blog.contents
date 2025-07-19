---
title: "세마포어의 놀라운 다양성"
date: 2025-07-19T05:10:37Z
author: "skyfe79"
draft: false
tags: ["아마추어 번역"]
---

> 원문: https://preshing.com/20150316/semaphores-are-surprisingly-versatile/

멀티스레드 프로그래밍에서 스레드가 대기하도록 만드는 것은 중요하다. 스레드는 리소스에 대한 배타적 접근을 위해 대기해야 한다. 또한 처리할 작업이 없을 때도 대기해야 한다. 스레드를 대기 상태로 만들고 커널 내부에서 잠재워 CPU 시간을 소모하지 않게 하는 한 가지 방법은 **세마포어**를 사용하는 것이다.

나는 세마포어가 이상하고 구식이라고 생각했었다. 세마포어는 Edsger Dijkstra가 [1960년대 초](http://en.wikipedia.org/wiki/Semaphore_%28programming%29)에 발명했다. 당시에는 멀티스레드 프로그래밍은 물론 일반적인 프로그래밍도 크게 발달하지 않았던 시기였다. 나는 세마포어가 리소스의 사용 가능한 단위를 추적하거나 [뮤텍스](http://en.wikipedia.org/wiki/Semaphore_%28programming%29#Semaphores_vs._mutexes)의 투박한 버전으로 동작할 수 있다는 정도만 알고 있었다.

하지만 내 생각은 바뀌었다. 세마포어와 원자적 연산만 사용해도 다음과 같은 기본 요소들을 모두 구현할 수 있다는 사실을 깨달았기 때문이다:

1.  경량 뮤텍스
2.  경량 자동 리셋 이벤트 객체
3.  경량 읽기-쓰기 잠금
4.  철학자들의 만찬 문제에 대한 또 다른 해결책
5.  부분적 스피닝을 지원하는 경량 세마포어

뿐만 아니라, 이러한 구현들은 몇 가지 바람직한 특성을 공유한다. 이들은 *경량*이다. 즉, 일부 연산이 완전히 사용자 공간에서 이루어지며, 커널에서 잠들기 전에 짧은 시간 동안 (선택적으로) 스피닝할 수 있다. 모든 C++11 소스 코드는 [GitHub](https://github.com/preshing/cpp11-on-multicore)에서 확인할 수 있다. 표준 C++11 라이브러리에는 세마포어가 포함되어 있지 않기 때문에, Windows, MacOS, iOS, Linux 및 기타 POSIX 환경에서 네이티브 세마포어에 직접 매핑되는 이식 가능한 [`Semaphore`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/sema.h) 클래스도 제공한다. 이러한 기본 요소들을 거의 모든 기존 C++11 프로젝트에 바로 적용할 수 있을 것이다.

- https://github.com/preshing/cpp11-on-multicore

## 세마포어는 마치 문지기와 같다

대기 중인 스레드들이 큐에 줄을 서 있는 모습을 상상해 보자. 마치 바쁜 나이트클럽이나 극장 앞에 줄을 서 있는 사람들처럼. 세마포어는 이 줄 맨 앞에 서 있는 문지기와 같다. 문지기는 특정 지시가 있을 때만 스레드들이 진행하도록 허용한다. 

<img width="636" height="208" alt="Image" src="https://github.com/user-attachments/assets/8d09c3cd-48db-4a4f-8d1e-b5f6669e8c47" />

각 스레드는 스스로 큐에 참여할 시점을 결정한다. 다익스트라는 이를 `P` 연산이라고 불렀다. `P`는 원래 네덜란드어로 재미있게 들리는 단어였지만, 현대의 세마포어 구현에서는 이 연산을 `wait`이라고 부르는 경우가 더 많다. 기본적으로, 스레드가 세마포어의 `wait` 연산을 호출하면 큐에 들어간다.

문지기 자신은 단 하나의 명령만 이해하면 된다. 원래 다익스트라는 이를 `V` 연산이라고 불렀다. 오늘날 이 연산은 `post`, `release`, `signal` 등 다양한 이름으로 불린다. 개인적으로는 `signal`이라는 용어를 선호한다. 실행 중인 스레드는 언제든지 `signal`을 호출할 수 있으며, 호출하면 문지기는 큐에서 정확히 하나의 대기 중인 스레드를 풀어준다. (반드시 도착한 순서대로 풀어주는 것은 아니다.)

만약 어떤 스레드가 큐에 대기 중인 스레드가 없을 때 `signal`을 호출하면 어떻게 될까? 문제없다. 다음 스레드가 큐에 도착하는 즉시 문지기는 그 스레드를 바로 통과시킨다. 그리고 `signal`이 빈 큐에 대해 예를 들어 3번 호출되면, 문지기는 다음에 도착하는 3개의 스레드를 바로 통과시킨다.

<img width="636" height="146" alt="Image" src="https://github.com/user-attachments/assets/755a69ef-6051-4253-a7d2-8989cd171689" />

 물론, 문지기는 이 숫자를 추적해야 하기 때문에 모든 세마포어는 [정수 카운터](http://linux.die.net/man/7/sem_overview)를 유지한다. `signal`은 카운터를 증가시키고, `wait`은 카운터를 감소시킨다.

이 전략의 아름다움은 `wait`이 몇 번 호출되고, `signal`이 몇 번 호출되든 결과는 항상 동일하다는 점이다. 문지기는 항상 같은 수의 스레드를 풀어주고, 큐에는 항상 같은 수의 스레드가 대기하게 된다. `wait`과 `signal` 호출의 순서와는 상관없이 말이다.

<img width="551" height="440" alt="Image" src="https://github.com/user-attachments/assets/7c7b0d5d-6311-4a0b-9a7a-48a61f44650d" />

## 1. 경량 뮤텍스 구현

이전 [포스트](http://preshing.com/2022/roll-your-own-lightweight-mutex)에서 경량 뮤텍스를 구현하는 방법을 이미 소개했다. 당시에는 알지 못했지만, 그 포스트는 재사용 가능한 패턴의 한 예시였다. 핵심은 세마포어 앞에 또 다른 메커니즘을 구축하는 것이다. 이를 나는 **박스 오피스**라고 부른다.

<img width="650" height="196" alt="Image" src="https://github.com/user-attachments/assets/0362048e-3231-4b19-afc3-ed9c6268a531" />

박스 오피스는 실제 결정이 이루어지는 곳이다. 현재 스레드가 대기열에 들어가야 할까? 아니면 대기열을 완전히 우회해야 할까? 대기열에 있는 다른 스레드를 해제해야 할까? 박스 오피스는 세마포어를 기다리는 스레드의 수를 직접 확인할 수 없고, 세마포어의 현재 시그널 카운트도 확인할 수 없다. 대신, 박스 오피스는 이전 결정을 스스로 추적해야 한다. 경량 뮤텍스의 경우, 원자적 카운터만 있으면 충분하다. 이 카운터를 `m_contention`이라고 부르는데, 이는 뮤텍스를 동시에 경쟁하는 스레드의 수를 추적하기 때문이다.

```cpp
class LightweightMutex
{
private:
    std::atomic<int> m_contention;         // The "box office"
    Semaphore m_semaphore;                 // The "bouncer"
```

스레드가 뮤텍스를 잠그려고 결정하면, 먼저 박스 오피스를 방문해 `m_contention`을 증가시킨다.

```cpp
public:
    void lock()
    {
        if (m_contention.fetch_add(1, std::memory_order_acquire) > 0)  // Visit the box office
        {
            m_semaphore.wait();     // Enter the wait queue
        }
    }
```

이전 값이 0이면, 아직 다른 스레드가 뮤텍스를 경쟁하지 않았다는 의미다. 따라서 현재 스레드는 즉시 새로운 소유자로 간주하고, 세마포어를 우회한 후 `lock`에서 반환되어 뮤텍스가 보호하려는 코드로 진행한다.

반면, 이전 값이 0보다 크면, 이미 다른 스레드가 뮤텍스를 소유하고 있다는 뜻이다. 이 경우 현재 스레드는 자신의 차례가 올 때까지 대기열에서 기다려야 한다. 

<img width="650" height="217" alt="Image" src="https://github.com/user-attachments/assets/b658a62a-b4b3-459d-8637-cfad461f6a5c" />

이전 스레드가 뮤텍스를 해제하면, 박스 오피스를 방문해 카운터를 감소시킨다:

```cpp
    void unlock()
    {
        if (m_contention.fetch_sub(1, std::memory_order_release) > 1)  // Visit the box office
        {
            m_semaphore.signal();   // Release a waiting thread from the queue
        }
    }
```

이전 카운터 값이 1이면, 그동안 다른 스레드가 도착하지 않았다는 의미이므로 추가 작업이 필요 없다. `m_contention`은 단순히 0으로 남는다.

반면, 이전 카운터 값이 1보다 크면, 다른 스레드가 뮤텍스를 잠그려고 시도했고, 따라서 대기열에서 기다리고 있다는 뜻이다. 이 경우, 다음 스레드를 해제해도 안전하다고 경고한다. 그 스레드가 새로운 소유자로 간주될 것이다.

<img width="650" height="218" alt="Image" src="https://github.com/user-attachments/assets/4ba77214-21dd-4605-8e96-ca523dd780d7" />

박스 오피스를 방문하는 모든 작업은 원자적이며 분할할 수 없다. 따라서 여러 스레드가 동시에 `lock`과 `unlock`을 호출하더라도, 항상 한 번에 하나씩 박스 오피스를 방문한다. 더 나아가, 뮤텍스의 동작은 박스 오피스에서 내린 결정에 의해 완전히 결정된다. 박스 오피스를 방문한 후, 스레드는 예측할 수 없는 순서로 세마포어를 작동할 수 있지만, 이는 문제가 되지 않는다. 이미 설명했듯이, 세마포어 작업이 어떤 순서로 발생하든 결과는 유효하다. (최악의 경우, 일부 스레드가 대기열에서 자리를 바꿀 수 있다.) 

이 클래스는 경쟁이 없을 때 세마포어를 우회함으로써 시스템 호출을 피하기 때문에 "경량"으로 간주된다. 이를 [`NonRecursiveBenaphore`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/benaphore.h)로 GitHub에 공개했으며, [재귀 버전](http://preshing.com/20120305/implementing-a-recursive-mutex)도 있다. 그러나 실제로 이 클래스를 사용할 필요는 없다. 대부분의 뮤텍스 구현은 [이미 경량](http://preshing.com/20111124/always-use-a-lightweight-mutex)이다. 그럼에도 불구하고, 여기서 설명할 다른 프리미티브에 대한 영감을 제공한다는 점에서 주목할 만하다.

## 2. 경량 자동 리셋 이벤트 객체

자동 리셋 이벤트 객체는 자주 언급되지는 않지만, [CppCon 2014 발표](http://preshing.com/20141024/my-multicore-talk-at-cppcon-2014)에서 언급했듯이 게임 엔진에서 널리 사용된다. 주로 다른 스레드(잠자고 있을 수도 있는)에게 작업이 준비되었음을 알리는 용도로 활용된다.

<img width="544" height="312" alt="Image" src="https://github.com/user-attachments/assets/9a89fa20-726e-49dd-b18b-098f05b3237e" />

자동 리셋 이벤트 객체는 기본적으로 중복 시그널을 무시하는 세마포어이다. 즉, `signal`을 여러 번 호출해도 이벤트 객체의 시그널 카운트는 절대 1을 초과하지 않는다. 이는 작업 단위를 어딘가에 게시할 때마다 `signal`을 맹목적으로 호출할 수 있음을 의미한다. 이 기법은 큐가 아닌 다른 데이터 구조에 작업 단위를 게시할 때도 유연하게 작동한다. Windows는 이벤트 객체를 기본적으로 지원하지만, `signal`과 동등한 [`SetEvent`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686211.aspx) 함수는 비용이 많이 든다. 한 기계에서 측정한 결과, 이벤트가 이미 시그널 상태일 때도 호출당 **700 ns**가 소요되었다. 스레드 간에 수천 개의 작업 단위를 게시하는 경우, 각 `SetEvent`의 오버헤드가 빠르게 누적될 수 있다.

다행히 박스 오피스/경비원 패턴은 이 오버헤드를 크게 줄인다. 모든 자동 리셋 이벤트 로직은 박스 오피스에서 원자 연산을 사용해 구현할 수 있으며, 박스 오피스는 스레드가 대기해야 할 때만 세마포어를 호출한다.

<img width="650" height="198" alt="Image" src="https://github.com/user-attachments/assets/36826c6f-db02-4bb8-a2bc-685290775714" />

이 구현을 [`AutoResetEvent`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/autoresetevent.h)로 공개했다. 이번에는 박스 오피스가 큐에서 대기 중인 스레드 수를 추적하는 방식이 다르다. `m_status`가 음수일 때, 그 크기는 대기 중인 스레드 수를 나타낸다:

```cpp
class AutoResetEvent
{
private:
    // m_status == 1: Event object is signaled.
    // m_status == 0: Event object is reset and no threads are waiting.
    // m_status == -N: Event object is reset and N threads are waiting.
    std::atomic<int> m_status;
    Semaphore m_sema;
```

이벤트 객체의 `signal` 연산에서 `m_status`를 원자적으로 증가시키되, 최대 1로 제한한다:

```cpp
public:
    void signal()
    {
        int oldStatus = m_status.load(std::memory_order_relaxed);
        for (;;)    // Increment m_status atomically via CAS loop.
        {
            assert(oldStatus <= 1);
            int newStatus = oldStatus < 1 ? oldStatus + 1 : 1;
            if (m_status.compare_exchange_weak(oldStatus, newStatus, std::memory_order_release, std::memory_order_relaxed))
                break;
            // The compare-exchange failed, likely because another thread changed m_status.
            // oldStatus has been updated. Retry the CAS loop.
        }
        if (oldStatus < 0)
            m_sema.signal();    // Release one waiting thread.
    }
```

`m_status`의 초기 로드가 relaxed이기 때문에, `m_status`가 이미 1인 경우에도 `compare_exchange_weak`를 호출하는 것이 중요하다. 이 점을 지적한 댓글 작성자 Tobias Brüll에게 감사드린다. 더 많은 정보는 [이 README 파일](https://github.com/preshing/cpp11-on-multicore/tree/master/tests/lostwakeup)을 참고한다.

## 3. 경량 읽기-쓰기 잠금

동일한 박스 오피스/경비원 패턴을 사용하면 훌륭한 [읽기-쓰기 잠금](http://en.wikipedia.org/wiki/Readers%E2%80%93writer_lock)을 구현할 수 있다. 이 읽기-쓰기 잠금은 쓰기 작업이 없는 경우 완전히 잠금 없이 동작하며, 읽기와 쓰기 모두에서 기아 현상이 발생하지 않는다. 또한 다른 기본 요소와 마찬가지로 스레드를 재우기 전에 스핀할 수 있다. 이를 위해 두 개의 세마포어가 필요하다: 하나는 대기 중인 읽기 작업을 위한 것이고, 다른 하나는 대기 중인 쓰기 작업을 위한 것이다. 이 코드는 [`NonRecursiveRWLock`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/rwlock.h)에서 확인할 수 있다.

<img width="650" height="200" alt="Image" src="https://github.com/user-attachments/assets/8a5b110c-faf8-43d4-9da9-ef42e1a4e530" />

## 4. 식사하는 철학자 문제에 대한 또 다른 해결책

박스 오피스/문지기 패턴은 데이크스트라의 [식사하는 철학자 문제](http://en.wikipedia.org/wiki/Dining_philosophers_problem)를 해결하는 데도 사용할 수 있다. 이 해결책은 다른 곳에서 설명된 적이 없는 방식이다. 이 문제에 익숙하지 않다면, 철학자들이 서로 식사용 포크를 공유하는 상황을 다룬다. 각 철학자는 두 개의 특정 포크를 얻어야만 식사를 할 수 있다. 이 해결책이 누군가에게 유용할 것 같지는 않아 자세히 설명하지는 않겠다. 단지 세마포어의 다용도성을 보여주기 위해 포함시켰다.

이 해결책에서는 각 철학자(스레드)에게 전용 세마포어를 할당한다. 박스 오피스는 어떤 철학자가 식사 중인지, 누가 식사를 요청했는지, 그리고 요청이 도착한 순서를 추적한다. 이 정보를 바탕으로 박스 오피스는 모든 철학자를 최적의 방식으로 문지기를 통해 안내할 수 있다.

<img width="650" height="263" alt="Image" src="https://github.com/user-attachments/assets/777ce38b-67e2-4e2d-9d40-b844133369f1" />

두 가지 구현을 공유한다. 하나는 [`DiningPhilosophers`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/diningphilosophers.h)로, 뮤텍스를 사용해 박스 오피스를 구현했다. 다른 하나는 [`LockReducedDiningPhilosophers`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/diningphilosophers.h)로, 박스 오피스에 대한 모든 접근이 락-프리 방식으로 이루어진다.

## 5. 부분 스피닝을 지원하는 경량 세마포어

놀랍게도, 세마포어와 박스 오피스를 결합해 또 다른 세마포어를 구현할 수 있다. 

왜 이런 방식을 사용할까? 그 이유는 [`LightweightSemaphore`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/sema.h)를 만들기 위해서다. 이 세마포어는 대기열이 비어 있고 시그널 카운트가 0보다 큰 상황에서 매우 저렴한 비용으로 동작한다. 이 때 박스 오피스는 원자적 연산에만 의존하며, 기본 세마포어를 건드리지 않는다.

<img width="650" height="195" alt="Image" src="https://github.com/user-attachments/assets/f589eda0-43ba-46c6-a0ac-d76f2a96d047" />
  
뿐만 아니라, 기본 세마포어를 호출하기 전에 짧은 시간 동안 스레드를 [스핀 루프](http://en.wikipedia.org/wiki/Spinlock)에서 대기시킬 수 있다. 이 트릭은 대기 시간이 짧을 때 비싼 시스템 호출을 피하는 데 도움이 된다.

[GitHub 저장소](https://github.com/preshing/cpp11-on-multicore/tree/master/common)에서 다른 모든 프리미티브는 `Semaphore`를 직접 사용하지 않고 `LightweightSemaphore` 위에 구현된다. 이렇게 해서 모든 프리미티브가 부분 스피닝 기능을 상속받는다. `LightweightSemaphore`는 `Semaphore` 위에 위치하며, 이 `Semaphore`는 플랫폼별 세마포어를 캡슐화한다.

<img width="557" height="384" alt="Image" src="https://github.com/user-attachments/assets/a80fdeca-6cba-4e29-8da1-de24093e432e" />

저장소에는 간단한 테스트 스위트가 포함되어 있으며, 각 테스트 케이스는 다른 프리미티브를 검증한다. `LightweightSemaphore`를 제거하고 모든 프리미티브가 `Semaphore`를 직접 사용하도록 강제할 수도 있다. 다음은 내 Windows PC에서 측정한 결과다:

|  | LightweightSemaphore | Semaphore |
| --- | --- | --- |
| testBenaphore | 375 ms | 5503 ms |
| testRecursiveBenaphore | 393 ms | 404 ms |
| testAutoResetEvent | 593 ms | 4665 ms |
| testRWLock | 598 ms | 7126 ms |
| testDiningPhilosophers | 309 ms | 580 ms |

보듯이, 이 환경에서는 `LightweightSemaphore`가 테스트 스위트에 큰 이점을 제공한다. 하지만 현재 스피닝 전략이 모든 환경에 최적화되어 있지는 않다. 단순히 10000번 고정된 횟수로 스핀한 후 `Semaphore`로 돌아간다. 적응형 스피닝을 잠깐 살펴봤지만, 최적의 접근 방식은 명확하지 않았다. 어떤 제안이 있는가?


## 조건 변수와의 비교

여러 가지 응용 사례를 통해 세마포어는 생각보다 더 일반적인 목적으로 사용할 수 있다는 것을 알 수 있다. 심지어 이 목록도 완전하지 않다. 그렇다면 왜 C++11 표준 라이브러리에서 세마포어가 빠져 있을까? Boost에서도 세마포어가 없는 이유와 동일하다. 바로 **뮤텍스와 조건 변수**를 선호하기 때문이다. 라이브러리 관리자들의 관점에서, 전통적인 세마포어 기법은 [너무 오류가 발생하기 쉽다](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2043.html#SemaphoreTypes).

하지만 여기서 보여준 박스 오피스/경비원 패턴은 특정 경우에 조건 변수를 최적화한 것에 불과하다. 이 경우는 모든 조건 변수 연산이 임계 영역의 끝에서 수행되는 경우이다.

앞서 설명한 `AutoResetEvent` 클래스를 생각해 보자. 동일한 저장소에 조건 변수를 기반으로 한 동등한 클래스인 [`AutoResetEventCondVar`](https://github.com/preshing/cpp11-on-multicore/blob/master/common/autoreseteventcondvar.h)를 구현했다. 이 클래스의 조건 변수는 항상 임계 영역의 끝에서 조작된다.

```cpp
void AutoResetEventCondVar::signal()
{
    // Increment m_status atomically via critical section.
    std::unique_lock<std::mutex> lock(m_mutex);
    int oldStatus = m_status;
    if (oldStatus == 1)
        return;     // Event object is already signaled.
    m_status++;
    if (oldStatus < 0)
        m_condition.notify_one();   // Release one waiting thread.
}
```

`AutoResetEventCondVar`를 두 단계로 최적화할 수 있다:

1. 각 조건 변수를 임계 영역 밖으로 빼내고 세마포어로 변환한다. 세마포어 연산의 순서 독립성 덕분에 이 작업은 안전하다. 이 단계를 거치면 박스 오피스/경비원 패턴을 이미 구현한 것이다. (일반적으로 이 단계는 여러 스레드가 동시에 시그널을 받을 때 [thundering herd](http://javaagile.blogspot.ca/2012/12/the-thundering-herd.html) 현상을 피할 수 있게 해준다.)
    
2. 박스 오피스를 [모든 연산을 CAS 루프로 변환](http://preshing.com/20150402/you-can-do-any-kind-of-atomic-read-modify-write-operation)하여 락 프리로 만들어 확장성을 크게 향상시킨다. 이 단계를 거치면 `AutoResetEvent`가 된다.

<img width="617" height="630" alt="Image" src="https://github.com/user-attachments/assets/a34b1482-6c2c-48fe-bfa1-30a124cfbac0" />

내 Windows PC에서 `AutoResetEventCondVar` 대신 `AutoResetEvent`를 사용하면 관련 테스트 케이스가 **10배** 더 빨리 실행된다.