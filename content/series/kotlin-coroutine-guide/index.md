---
title: "코틀린 코루틴 가이드 - 시작"
date: 2019-11-04T22:52:35+09:00
draft: true
---

코틀린은 표준 라이브러리에서 저수준 API를 최소한으로 제공합니다. 그래서 다양한 다른 라이브러리가 코루틴을 활용할 수 있도록합니다.

비슷한 기능을 가진 다른 언어와 달리 async 및 await는 코틀린 키워드가 아닙니다. 또한 표준 라이브러리의 일부도 아닙니다. 

코틀린의 일시 중단 함수는 비동기 작업에 대해 Future와 Promise보다 안전하고 오류가 적은 추상화를 제공합니다. 

kotlinx.coroutines는 JetBrains가 개발한 풍부한 코루틴 라이브러리입니다. 이 글은 launch, async 등을 포함하여 코루틴을 위한 고급 프리미티브를 다루고 있습니다.

이 가이드는 kotlinx.coroutines의 핵심 기능에 대한 글로 다양한 주제로 나뉘어져 있습니다. 가이드의 예제를 따라해 보면서 코루틴을 만들어 보기 위해서는 [README의 설명](https://github.com/kotlin/kotlinx.coroutines/blob/master/README.md#using-in-your-projects)대로 kotlinx-coroutines-core 모듈을 디펜던시로 추가해야 합니다.
