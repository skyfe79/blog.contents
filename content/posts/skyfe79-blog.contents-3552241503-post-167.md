---
title: "PyTorch Monarch 소개"
date: 2025-10-25T09:47:45Z
author: "skyfe79"
draft: false
tags: ["아마추어 번역","ai"]
---

> https://pytorch.org/blog/introducing-pytorch-monarch/

현재 머신러닝 워크플로우(사전 학습, 사후 학습 등)는 이질적이며, 하드웨어 장애를 처리해야 하고, 점점 더 비동기적이며 고도로 동적인 특성을 지닌다. 전통적으로 PyTorch는 HPC 스타일의 다중 컨트롤러 모델에 의존해 왔다. 이 모델에서는 동일한 스크립트의 여러 복사본이 서로 다른 머신에서 실행되며, 각각이 자체 애플리케이션 인스턴스를 실행한다(일반적으로 SPMD라고 부른다). 

하지만 머신러닝 워크플로우는 점점 더 복잡해지고 있다. 사전 학습은 고급 병렬 처리와 비동기성, 부분적 실패를 결합할 수 있으며, 사후 학습에 사용되는 강화 학습 모델은 복잡한 피드백 루프와 함께 높은 수준의 동적 특성을 요구한다. 이런 워크플로우의 논리는 비교적 간단할 수 있지만, 각 노드가 워크플로우 상태의 로컬 뷰만을 기반으로 동작을 결정해야 하는 다중 컨트롤러 시스템에서 구현하기는 매우 어렵다.

<img width="1600" height="680" alt="Image" src="https://github.com/user-attachments/assets/441852d0-1eaa-4dc7-be51-427fa4d4f1ff" />

이 문제를 해결하기 위한 장기적이고 지속 가능한 방법은 *단일 컨트롤러* 프로그래밍 모델이라고 믿는다. 단일 스크립트가 모든 분산 리소스를 조정하여 거의 로컬처럼 느껴지게 하는 방식이다. 이 아키텍처 변화는 분산 프로그래밍을 단순화한다. 코드는 단일 머신 Python 프로그램처럼 보이고 느껴지지만, 수천 개의 GPU로 확장할 수 있다. 클래스, 함수, 루프, 태스크, 퓨처 같은 Pythonic 구문을 직접 사용해 복잡한 분산 알고리즘을 표현할 수 있다.

단일 머신 PyTorch의 단순함을 전체 클러스터로 확장하는 분산 프로그래밍 프레임워크 **Monarch**를 소개한다.

Monarch는 분산 시스템을 단일 머신처럼 프로그래밍할 수 있게 하여 분산 컴퓨팅의 복잡성을 숨긴다:

1. **클러스터를 배열처럼 프로그래밍.** Monarch는 호스트, 프로세스, 액터를 확장 가능한 *메시*로 구성하여 직접 조작할 수 있다. 간단한 API로 전체 메시(또는 일부)를 조작할 수 있다. Monarch가 자동으로 분산과 벡터화를 처리하므로 코드가 실행되는 위치가 아니라 계산하려는 내용에 집중할 수 있다.

2. **점진적 장애 처리.** Monarch에서는 아무것도 실패하지 않는 것처럼 코드를 작성한다. 문제가 발생하면 Monarch는 기본적으로 빠르게 실패한다(단순 로컬 스크립트에서 처리되지 않은 예외처럼 전체 프로그램을 중지). 나중에 정확히 필요한 위치에서 세분화된 장애 처리를 점진적으로 추가할 수 있으며, 예외를 잡아 복구하는 것처럼 장애를 처리할 수 있다.

3. **제어와 데이터 분리.** Monarch는 제어 플레인(메시징)과 데이터 플레인(RDMA 전송)을 분리하여 클러스터 전체에서 GPU-GPU 메모리 직접 전송을 가능하게 한다. Monarch는 명령을 한 경로로 보내고 데이터는 다른 최적화된 경로로 이동시킬 수 있다.

4. **로컬처럼 느껴지는 분산 텐서.** Monarch는 PyTorch와 완벽하게 통합되어 GPU 클러스터 전체에 샤딩된 텐서를 제공한다. Monarch 텐서 연산은 로컬처럼 보이지만 수천 개의 GPU로 구성된 대규모 분산 클러스터에서 실행되며, Monarch가 복잡한 조정 작업을 처리한다.


### 프로그래밍 모델

#### 핵심 API: 프로세스 메시와 액터 메시

Monarch는 리소스를 다차원 배열, 즉 **메시**로 구성한다. **프로세스 메시**는 여러 호스트에 분산된 프로세스 배열이며, **액터 메시**는 각각 별도의 프로세스에서 실행되는 액터 배열이다. NumPy나 PyTorch의 배열 프로그래밍과 유사하게, 메시는 대규모 시스템에 걸쳐 작업을 효율적으로 디스패치하는 간편한 방식을 제공한다.

출시 시점에서 Monarch는 GPU 클러스터 기반의 프로세스 메시를 지원한다(일반적으로 GPU당 하나의 프로세스). 이 위에 액터 메시를 생성할 수 있다. 로컬 개발 환경에서는 동일한 메시를 로컬 개발 서버에서도 실행할 수 있다.


#### 고급 API: 텐서 엔진과 RDMA 버퍼

Monarch의 *텐서 엔진*은 프로세스 메시에 분산 텐서를 제공한다. 이 기능을 사용하면 GPU 클러스터 전체가 스크립트를 실행하는 머신에 연결된 것처럼 PyTorch 프로그램을 작성할 수 있다. 대량 데이터 이동을 위해 Monarch는 RDMA 버퍼 API도 제공한다. 이 API는 지원되는 NIC에서 프로세스 간 직접 고성능 전송을 가능하게 한다. [상세 설명](https://meta-pytorch.org/monarch/generated/examples/getting_started.html)과 [추가 예제](https://meta-pytorch.org/monarch/generated/examples/index.html)는 [GitHub 페이지](https://meta-pytorch.org/monarch/index.html)에서 확인할 수 있다.


#### 간단한 예제

Monarch 코드는 간결한 Python API를 사용해 프로세스와 액터를 생성하는 방법을 명령형으로 설명한다:

```python
from monarch.actor import Actor, endpoint, this_host

procs = this_host().spawn_procs({"gpus": 8})

# 메서드 하나를 가진 액터 정의

class Example(Actor):
   @endpoint
   def say_hello(self, txt):
       return f"hello {txt}"

# 액터 생성

actors = procs.spawn("actors", Example)

# 액터에 인사 요청

hello_future = actors.say_hello.call("world")

# 결과 출력

print(hello_future.get())
```

위 예제에서는 로컬 호스트의 8개 GPU에 배포되는 "Example"이라는 액터를 정의한다. 컨트롤러는 이 액터를 호스트 전체에 걸쳐 호출하고 응답을 기다린다. 액터는 다양한 인터페이스를 노출할 수 있다.


#### 메시 분할하기

브로드캐스트 통신을 구현하기 위해 액터들을 Mesh로 조직한다. Mesh는 이름이 붙은 차원을 가진 다차원 컨테이너다. 예를 들어 클러스터는 `{"hosts": 32, "gpus": 8}` 같은 차원 구성을 가질 수 있다. 차원 이름은 일반적으로 "hosts"(클러스터 내 호스트들 간 인덱싱)나 "gpus"(머신 내 요소들 간 인덱싱) 같은 식으로 지정한다.

```python
from monarch.actor import Actor, endpoint, this_host

procs = this_host().spawn_procs({"gpus": 8})

# 두 개의 메서드를 가진 액터 정의

class Example(Actor):
   @endpoint
   def say_hello(self, txt):
       return f"hello {txt}"

   @endpoint
   def say_bye(self, txt):
       return f"goodbye {txt}"

# 액터 생성

actors = procs.spawn("actors", Example)

# 절반은 hello 메시지 전송

hello_fut = actors.slice(gpus=slice(0,4)).say_hello.call("world")

# 나머지 절반은 bye 메시지 전송

bye_fut = actors.slice(gpus=slice(4,8)).say_bye.call("world")

print(hello_fut.get())
print(bye_fut.get())
```


#### 장애 복구


사용자는 파이썬의 try, except 블록을 통해 오류가 발생할 수 있는 분산 프로그램을 작성할 수 있다. 이러한 기본 구성 요소 위에 복잡한 오류 감지 및 복구 체계를 구축할 수 있다. 다음은 원격 액터에서 발생하는 간단한 런타임 예외를 처리하는 예제를 보여준다.

```python
from monarch.actor import Actor, endpoint, this_host

procs = this_host().spawn_procs({"gpus": 8})

class Example(Actor):
   @endpoint
   def say_hello(self, txt):
       return f"hello {txt}"

   @endpoint
   def say_bye(self, txt):
       raise Exception("saying bye is hard")

actors = procs.spawn("actors", Example)
hello_fut = actors.slice(gpus=slice(0,4)).say_hello.call("world")
bye_fut = actors.slice(gpus=slice(4,8)).say_bye.call("world")

try:
   print(hello_fut.get())
except:
   print("couldn't say hello")

try:
   print(bye_fut.get())
except Exception:
   print("got an exception saying bye")
```

보다 현실적인 사용 사례는 "사례 연구 2: 대규모 사전 학습에서의 장애 허용"을 참조한다.


## Monarch 백엔드 구조

Monarch는 Python 기반의 *프론트엔드*와 Rust로 구현된 백엔드로 구성된다. Python은 머신러닝 분야의 공용어 역할을 하며, Python 프론트엔드 API를 통해 사용자는 기존 코드와 라이브러리(예: PyTorch!)와 원활하게 통합할 수 있고 Jupyter 노트북 같은 대화형 컴퓨팅 도구에서도 Monarch를 활용할 수 있다. Rust 기반 백엔드는 성능, 확장성, 안정성을 담당한다. Monarch 구현 과정에서 Rust의 *공포 없는 동시성(fearless concurrency)* 기능을 적극 활용한다.


### Hyperactor와 hyperactor_mesh

스택의 최하단에는 [*hyperactor*](https://github.com/meta-pytorch/monarch/tree/main/hyperactor)라는 Rust 기반 액터 프레임워크가 위치한다. Hyperactor는 고성능 메시지 전달과 견고한 감독에 중점을 둔 저수준 분산 액터 시스템이다. *hyperactor_mesh*는 hyperactor 위에 구축되며, 다양한 컴포넌트를 효율적인 "벡터화된" 액터 구현으로 결합한다. Hyperactor_mesh는 대규모 액터 *메시* 간에 비용 효율적인 액터 연산을 제공하는 데 특화되어 있다.

Monarch의 핵심 Python API는 hyperactor_mesh를 둘러싼 비교적 얇은 래퍼 레이어로 구성된다.


### 확장 가능한 메시징 시스템

Monarch의 모든 기능은 *확장 가능한 메시징*을 기반으로 작동한다. 이는 액터 메시 시스템에 메시지를 *캐스팅*하는 핵심 API를 지원한다. Hyperactor는 멀티캐스트 트리와 멀티파트 메시징이라는 두 가지 메커니즘을 통해 이를 구현한다.

첫째, 멀티캐스팅을 지원하기 위해 Hyperactor는 메시지를 분산시키는 멀티캐스트 트리를 구성한다. 메시지가 캐스팅되면 먼저 초기 노드로 전송되고, 이 노드는 메시지 복사본을 자식 노드 집합에 전달한다. 이 과정이 반복되면서 메시지는 전체 메시 시스템에 완전히 분산된다. 이 방식은 단일 호스트 병목 현상을 피하고, 메시 시스템 전체를 분산 클러스터처럼 활용해 메시지 전달에 사용한다.

둘째, 제어 플레인이 데이터 전달의 핵심 경로에 절대 개입하지 않도록 보장한다. 예를 들어 멀티파트 메시징을 사용해 복사를 최소화하고, 멀티캐스트 트리에서 발생하는 높은 팬아웃 전송 시 데이터 공유를 가능하게 한다. 또한 운영체제가 관리하는 효율적인 벡터화된 쓰기 작업으로 변환한다.


## 실제 적용 사례

이 범용 API와 PyTorch의 네이티브 통합 기능이 대규모 AI 애플리케이션 개발의 새로운 지평을 열 것이라 확신한다. 더 나아가 복잡한 오케스트레이션 요구사항을 해결하는 데도 크게 기여할 것이다.


### 사례 연구 1: 강화 학습

강화 학습(Reinforcement Learning)은 최신 첨단 모델 개발에 핵심적인 역할을 한다. 이 기술은 모델이 심층 연구를 수행하고, 특정 환경에서 작업을 처리하며, 수학이나 코드 같은 복잡한 문제를 해결할 수 있게 한다. 보다 깊이 있는 이해를 원한다면 해당 [포스트](https://pytorch.org/blog/a-primer-on-llm-post-training/?ajs_aid=57910bf1-d592-4619-a7d5-295ab7d39433)를 참고하기를 권한다.

아래 그림과 같은 추론 모델을 학습시키기 위해, 특정 도메인(예: 프로그래밍 코드 생성)에 특화된 추론 모델에서 생성 프로세스가 프롬프트를 만든다. 생성기는 이러한 프롬프트(불완전한 코딩 문제 설명)를 사용해 도구(컴파일러)와 환경을 통해 상호작용하면서 솔루션 또는 실행 경로(이 예에서는 실행 가능한 코드)를 도출한다. 보상 파이프라인은 이러한 솔루션을 평가해 점수를 매긴다. 이 점수와 보상은 모델 학습에 활용되며, 가중치는 다시 프롬프트 응답을 생성한 시스템으로 전달된다.

이것이 하나의 학습 루프를 구성한다! 아래 그림에서 볼 수 있듯, 이는 ***학습 루프 내부***에서 여러 이기종 계산을 조율하고 개별적으로 확장해야 하는 실시간 파이프라인과 같다.

<img width="1600" height="985" alt="Image" src="https://github.com/user-attachments/assets/57d49e1e-867e-45fe-b43c-90ee7a7ab684" />

위의 RL 예제를 Monarch로 구현할 때 각 구성 요소(생성기, 트레이너, 추론 엔진, 보상 파이프라인)는 메시로 표현될 수 있다. 생성기 메시, 트레이너 메시, 추론 노드 메시, 보상 파이프라인 메시 등이 그것이다(위 그림은 생성기와 트레이너 두 가지 메시만 있는 단순화된 예제를 보여준다).

학습 스크립트는 이러한 메시를 사용해 전체 작업 흐름을 조율한다. 새로운 프롬프트 배치로 생성기 메시가 작업을 시작하도록 지시하고, 완료되면 데이터를 트레이닝 메시로 전달하며, 새로운 모델 스냅샷이 준비되면 추론 메시를 업데이트한다. 오케스트레이터는 일반 파이썬 프로그램으로 작성되며, 메시의 메서드를 호출하고 데이터를 전달한다. Monarch는 원격 메모리 전송(RDMA)을 기본적으로 지원하므로, 실제 데이터는 메시 구성원 간에 직접 전송된다(한 GPU에서 다른 GPU로 텐서를 복사하는 것과 유사). 이를 통해 효율적이고 확장 가능한 워크플로우가 가능해진다.


#### VERL(Volcano Engine Reinforcement Learning)은 현재 업계에서 널리 사용되는 강화 학습 프레임워크다.

우리는 Monarch와 VERL을 통합해 개념 검증을 수행했다. [Qwen-2.5-7B 수학 모델](https://huggingface.co/open-r1/Qwen2.5-Math-7B-RoPE-300k)을 선별된 수학 데이터셋에 대해 GRPO로 사후 학습하고, AIME 2024 벤치마크에서 평가했다. H200 GPU에서 [Megatron-LM](https://github.com/NVIDIA/Megatron-LM)을 사용해 16, 64, 1024, 2048 GPU로 점진적으로 확장하며 500+ 학습 스텝을 진행했다. 학습 과정은 안정적이었으며 기존 옵션과 유사한 수치 결과를 보여, Monarch가 기존 RL 프레임워크를 잘 조율할 수 있음을 입증했다.

이 통합 기능을 오픈소스로 공개해 향후 사용자들이 VERL 작업에서 Monarch를 활용할 수 있도록 하는 작업을 진행 중이다.

<img width="984" height="584" alt="Image" src="https://github.com/user-attachments/assets/8eb25e73-a970-403d-bb2f-cd9405ca935e" />


#### TorchForge

TorchForge는 기존과 다른 접근 방식을 제시한다. Monarch 프리미티브를 기반으로 처음부터 설계된 PyTorch 네이티브 RL 프레임워크다.

TorchForge의 목표는 연구자가 RL 알고리즘을 의사 코드처럼 자연스럽게 표현할 수 있도록 하는 동시에, Monarch가 분산 처리의 복잡성을 처리하도록 하는 것이다. 그 결과는 다음과 같은 코드 형태로 나타난다:

```python
async def continuous_rollouts():
    while True:
        prompt, target = await dataloader.sample.route()
        response = await policy.generate.route(prompt)
        reward = await reward.evaluate_response.route(prompt, response.text, target)
        await replay_buffer.add.route(Episode(...))
```

분산 조정 코드나 재시도 로직 없이 순수 Python으로 작성된 RL 코드만 존재한다.


#### Monarch 기반 구축: 서비스와 TorchStore

이 깔끔한 API는 torchforge가 Monarch의 기본 요소 위에 구축한 두 가지 핵심 추상화 덕분에 가능하다:

**서비스**는 Monarch ActorMeshes에 RL(강화 학습) 전용 패턴을 적용한 래퍼다. Monarch의 장애 허용, 리소스 할당, 메일박스 시스템을 활용하면서도 로드 밸런싱 라우팅(`.route()`), 병렬 브로드캐스트(`.fanout()`), 상태 저장 연산을 위한 스티키 세션 같은 패턴을 추가한다.

```python
# 서비스는 라우팅 기본 요소를 갖춘 ActorMeshes의 관리형 그룹이다

policy = PolicyActor.options(
    procs=8, with_gpus=True, num_replicas=16 # 8개 GPU를 가진 16개 복제본 생성
).as_service()

# 서비스는 Monarch 액터 기반의 RL 친화적 동작을 제공한다

response = await policy.generate.route(prompt)     # 로드 밸런싱 라우팅
await policy.update_weights.fanout(version)        # 병렬 브로드캐스트
```

**TorchStore**는 훈련과 추론 간 가중치 동기화를 처리하는 분산형 PyTorch 텐서 키-값 저장소다. Monarch의 RDMA 기본 요소와 단일 컨트롤러 설계를 기반으로 하며, 간단한 DTensor API를 제공하면서도 실시간으로 가중치를 효율적으로 재분할한다. 이는 훈련과 추론이 서로 다른 레이아웃을 사용하는 오프-폴리시 RL에서 매우 중요하다.

이러한 추상화는 Monarch의 구성 가능성을 보여준다. torchforge는 Monarch의 기본 요소(액터, RDMA, 장애 허용)를 빌딩 블록으로 사용해 RL 전용 인프라를 생성한다. 결과적으로 생성된 프레임워크는 인프라 계층에서 조정의 복잡성을 처리하므로 연구자들은 알고리즘 개발에 집중할 수 있다. Forge의 API, 컴포넌트 통합, 설계 철학에 대한 자세한 예제는 [torchforge 블로그 포스트](https://pytorch.org/blog/introducing-torchforge/)를 참조한다.


### 사례 연구 2: 대규모 사전 학습에서의 장애 허용 기술

대규모 시스템에서는 하드웨어와 소프트웨어 장애가 빈번하게 발생한다. 예를 들어, [Llama3 학습 과정](https://arxiv.org/pdf/2407.21783)에서 16,000개 GPU를 사용하는 54일간의 학습 동안 총 419회의 중단이 발생했다. 이는 평균 3시간마다 한 번꼴로 장애가 발생한 것이다. 만약 GPU 수를 수만 개로 확장하면 장애 발생 빈도는 시간당 한 번 이상으로 증가할 것이다. 매번 전체 작업을 재시작하면 효과적인 학습 시간이 크게 줄어든다.

해결책 중 하나는 분산 학습 방식을 더욱 효과적으로 활용해 모델의 수치 계산이 다양한 그룹 간 비동기적 실행을 더 잘 허용하도록 만드는 것이다. PyTorch에서 공개한 [TorchFT](https://pytorch.org/blog/fault-tolerant-llama-training-with-2000-synthetic-failures-every-15-seconds-and-no-checkpoints-on-crusoe-l40s/)는 GPU 장애를 견디며 학습을 계속할 수 있는 방법을 제공한다. 한 가지 전략은 장애 허용 DDP와 FSDP v2, PP를 결합한 Hybrid Sharded Data Parallelism을 사용하는 것이다. 장애 발생 시 [torchcomms](https://pytorch.org/blog/torchcomms/)를 활용해 우아하게 오류를 처리하고, 다운타임 없이 다음 배치 학습을 재개한다. 이렇게 하면 장애가 "복제 그룹" 하나로만 격리되며, 원래 작업의 일부만으로도 학습을 계속할 수 있다.

Monarch는 [TorchFT](https://github.com/meta-pytorch/torchft)와 [통합](https://github.com/meta-pytorch/torchft/tree/main/examples/monarch)되어 작동한다. Monarch는 제어 평면을 단일 컨트롤러 모델로 중앙 집중화한다. Monarch는 자체 장애 감지 기능으로 문제를 탐지하고, 감지 즉시 새로운 논리적 복제 그룹(Monarch Mesh)을 생성해 학습에 참여시킨다. TorchFT의 Lighthouse 서버는 Monarch 액터 역할을 한다. Monarch는 장애 유형에 따라 복구 전략을 유연하게 구성할 수 있다. 장애 발생 시 컨트롤러는 기존 할당 내에서 빠른 프로세스 수준 재시작을 먼저 시도하며, 필요한 경우에만 작업 재할당으로 확장한다. 이 동안 TorchFT는 정상적인 복제본이 계속 학습하도록 유지한다.

Coreweave의 30노드(240개 H100) 클러스터에서 SLURM 스케줄러를 사용해 Qwen3-32B를 torchtitan과 TorchFT로 학습하며 이 코드를 실행했다. 세그폴트, 프로세스 강제 종료, NCCL 중단, 호스트 제거, GIL 데드락 등 다양한 장애 모드를 시뮬레이션하기 위해 3분마다 100건의 장애를 주입했다. Monarch의 구성 가능한 복구 전략은 불필요한 작업 재스케줄링을 피해 전체 SLURM 작업 재시작 대비 60% 더 빠른 복구 속도를 보였다. 프로세스 장애는 평균 90초, 머신 장애는 평균 2.5분 내에 복구됐다. 자세한 내용은 [README](https://github.com/meta-pytorch/torchft/tree/main/examples/monarch)를 참조한다.

<img width="990" height="590" alt="Image" src="https://github.com/user-attachments/assets/2244c5b5-7e6e-4f04-9025-476e29d6e793" />


### 사례 연구 3: 대규모 GPU 클러스터에서의 대화형 디버깅

액터 프레임워크는 복잡한 작업의 대규모 조정에만 국한되지 않는다. 이 프레임워크는 다중 GPU 연산을 대화형으로 원활하게 디버깅할 수 있는 기능을 제공한다. 이 기능은 기존의 배치 방식 디버깅에서 현대적 AI 시스템의 규모와 복잡성에 맞는 실시간 탐구적 문제 해결 방식으로의 근본적 전환을 의미한다.

기존 디버깅 워크플로는 현대적 ML 시스템의 현실에 부딪히면 한계에 직면한다. 단일 GPU에서는 완벽하게 학습되던 모델이 수십 개의 가속기로 확장되면 미묘한 경쟁 조건, 교착 상태, 메모리 단편화 또는 통신 병목 현상을 보일 수 있다.

Monarch는 대화형 개발자 경험을 제공한다. 로컬 주피터 노트북을 사용해 사용자는 Monarch 메시로 클러스터를 제어할 수 있다.

1. 지속적 분산 컴퓨팅으로 새 작업을 제출하지 않고도 매우 빠르게 반복할 수 있다.
2. `sync_workspace` API를 통해 로컬 콘다 환경 코드를 메시 노드로 신속하게 동기화한다.
3. Monarch는 메시 네이티브 [분산 디버거](https://meta-pytorch.org/monarch/generated/examples/debugging.html)를 제공한다.

주피터 튜토리얼은 [pytorch.org](https://docs.pytorch.org/tutorials/intermediate/monarch_distributed_tutorial.html)에서 확인할 수 있다.


### Monarch + Lightning AI 노트북

<img width="1600" height="990" alt="Image" src="https://github.com/user-attachments/assets/6d28c691-a241-41a7-88ea-c78a168f71b9" />

TorchTitan으로 구동되는 단일 스튜디오 노트북에서 256-GPU 학습 작업을 시작하는 Monarch의 작동 방식을 확인할 수 있다. 원활한 확장성, 지속적인 리소스 관리, 대화형 디버깅을 모두 하나의 노트북에서 경험할 수 있다. 위 그림은 이 구조를 보여준다. 자세한 내용은 [Monarch-Lightning 블로그 포스트](https://pytorch.org/blog/integration-idea-monarch/)를 참고한다. 이 예시에서는 기존 SPMD TorchTitan 워크로드를 Monarch 내부의 Actor로 캡슐화해 사용자가 스튜디오 노트북에서 Llama-3 및 Llama-4와 같은 대규모 언어 모델을 대화형으로 사전 학습할 수 있게 한다.

Monarch를 사용하면 로컬 스튜디오 노트북에서 직접 컴퓨팅 리소스를 예약하고 유지할 수 있다. 노트북 세션이 중단되거나 코드 연결이 끊어져도 Multi-Machine Training(MMT)를 통해 클러스터 할당은 계속 유지된다. 이 프로세스 할당자의 지속성 덕분에 수동 개입 없이 작업을 반복하고 실험하며 재개할 수 있어, 노트북이 분산 학습 작업을 위한 안정적인 제어 센터 역할을 한다.

Monarch의 Actor 모델을 사용하면 프로세스 메시에서 Titan Trainer를 Actor로 정의하고 실행할 수 있다. 스튜디오 노트북 안에서 수백 개의 GPU로 학습 작업을 확장할 수 있다. Monarch는 오케스트레이션, 코드 및 파일 공유, 로그 수집을 처리하므로 작업을 빠르게 재구성하고 재시작할 수 있다. 로그와 메트릭은 노트북 내에서 직접 확인할 수 있을 뿐만 아니라 Litlogger 및 WandB와 같은 외부 도구를 통해 모니터링하고 관리할 수 있어 대규모 학습을 쉽게 제어할 수 있다.

Monarch는 분산 학습에 대화형 디버깅 기능을 제공한다. Actor 코드에 Python 중단점을 설정하고 실행 중인 프로세스를 검사하며, 특정 Actor에 연결해 실시간 문제 해결을 할 수 있다. 모두 노트북 인터페이스에서 가능하다. 학습 후에는 새 할당을 기다릴 필요 없이 동일한 리소스에서 구성을 수정하거나 새로운 Actor를 정의해 작업을 재시작할 수 있다. 이 동적 워크플로는 실험 속도를 높이고 분산 학습 실행에 대한 깊은 통찰력을 제공한다.

[Monarch-Lightning 블로그 포스트](https://pytorch.org/blog/integration-idea-monarch/)의 코드 스니펫은 256개의 GPU에서 TorchTitan을 사용해 Llama-3.1 – 8B 모델을 사전 학습하는 Monarch용 Lightning 스튜디오 노트북 예제를 보여준다.


### 지금 바로 Monarch를 사용해 보세요: 손쉽게 분산 AI 워크플로를 구축, 확장, 디버깅하세요

Monarch는 지금 GitHub에서 바로 사용할 수 있습니다. 여러분이 직접 탐색하고, 구축에 참여하며, 기여할 수 있도록 준비되어 있습니다. 시작하려면 [**Monarch 저장소**](https://github.com/meta-pytorch/monarch)를 살펴보고, 더 깊이 있는 기술적 내용은 [**공식 문서**](https://meta-pytorch.org/monarch/)를 확인하세요. 실제 동작을 확인하고 싶다면 [**인터랙티브 Jupyter 노트북**](https://github.com/meta-pytorch/monarch/blob/main/examples/slurm_titan.ipynb)을 실행해 보세요. 노트북에서 직접 대규모 학습을 실행하는 종단간 예제를 경험하려면 [**Lightning.ai 연동 가이드**](https://pytorch.org/blog/integration-idea-monarch/)를 참고하세요. 대규모 학습 작업을 조정하든, 강화 학습을 실험하든, 분산 시스템을 대화형으로 디버깅하든, Monarch는 모든 작업을 간단하고 확장 가능한 방식으로 수행할 수 있는 도구를 제공합니다.


### 감사의 말

이 연구를 가능하게 해준 Monarch 팀 전원에게 감사드립니다. 또한 GitHub에서 [Top Contributors](https://github.com/meta-pytorch/monarch/graphs/contributors)로 활약해 주신 분들께도 특별한 감사를 표합니다!

Ahmad Sharif, Allen Wang, Alireza Shamsoshoara, Amir Afzali, Amr Mahdi, Andrew Gallagher, Benji Pelletier, Carole-Jean Wu, Chris Gottbrath, Colin Taylor, Davide Italiano, Dennis van der Staay, Eliot Hedeman, Gayathri Aiyer, Gregory Chanan, Hamid Shojanazeri, James Perng, James Sun, Jana van Greunen, Jayasi Mehar, Joe Spisak, John William Humphreys, Jun Li, Kai Li, Keyan Pishdadian, Kiuk Chung, Lucas Pasqualin, Marius Eriksen, Marko Radmilac, Mathew Oldham, Matthew Zhang, Michael Suo, Matthias Reso, Osama Abuelsorour, Pablo Ruiz Fischer Bennetts, Peng Zhang, Rajesh Nishtala, Riley Dulin, Rithesh Baradi, Robert Rusch, Sam Lurye, Samuel Hsia, Shayne Fletcher, Tao Lin, Thomas Wang, Victoria Dudin, Vidhya Venkat, Vladimir Ivanov, Zachary DeVito


