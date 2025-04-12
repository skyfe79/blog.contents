---
title: "Chrome에서 이미지 요청 우선순위를 어떻게 결정할까?"
date: 2025-04-12T05:11:46Z
author: "skyfe79"
draft: false
tags: ["web"]
---

> 원문: https://www.debugbear.com/blog/chrome-image-request-prioritization

이 글에서는 Chrome이 이미지에 대한 [요청 우선순위](https://www.debugbear.com/blog/request-priorities)를 어떻게 결정하는지 알아본다. 우선순위 결정 방식과 중요한 이미지가 더 빠르게 로드되도록 최적화하는 방법을 설명한다.

## 이미지 요청은 기본적으로 낮은 우선순위를 가진다

기본적으로 이미지 요청은 낮은 우선순위로 처리된다. 브라우저는 렌더링을 블로킹하는 리소스나 중요한 요청 체인에 속한 리소스를 먼저 우선순위로 처리한다.

이 요청 워터폴에서 여러 개의 낮은 우선순위 이미지 요청을 확인할 수 있다.

![낮은 우선순위 이미지를 보여주는 요청 워터폴](https://github.com/user-attachments/assets/92d22cff-f289-4078-87df-3ec96b334dbd)

요청의 회색 막대는 대기 시간을 나타내며, 이 예제에서는 종종 700밀리초를 초과한다. 크롬은 페이지 렌더링이 시작될 때까지 이러한 리소스를 요청하지 않는다.

가장 중요한 페이지 리소스가 로드된 후, 크롬은 낮은 우선순위 이미지를 로드하기 시작한다. 

![728ms의 대기 시간을 보여주는 네트워크 요청 기간 분석](https://github.com/user-attachments/assets/393f0df7-274c-4d97-859e-e3024697880f)

> 팁
> 
> 크롬은 리소스를 두 단계로 로드한다. 초기 단계인 'tight mode'에서는 크롬이 낮은 우선순위 요청에 리소스를 소비하지 않는다.

## Chrome이 처음 5개의 큰 이미지 우선순위를 높인다

[Chrome 117](https://web.dev/articles/fetch-priority#resource-priority)에서 Google은 처음 5개의 큰 이미지의 우선순위를 Low에서 Medium으로 높이는 기능을 도입했다. 이를 통해 주요 이미지가 더 빠르게 로드되도록 보장한다.

이전에는 이러한 이미지가 기본적으로 Low 우선순위를 가졌다. 아래 예제 요청 워터폴을 보면, 이 변경으로 인해 Medium 우선순위가 할당된 5개의 이미지가 있다. 그 뒤에는 Low 우선순위를 유지하는 두 개의 이미지가 있다. 이 접근 방식은 [Largest Contentful Paint](https://www.debugbear.com/docs/metrics/largest-contentful-paint) 점수를 빠르게 높이면서도 리소스 로딩 효율을 유지하는 데 도움이 된다.

![Chrome이 처음 5개의 큰 이미지 우선순위를 Medium으로 높인 요청 워터폴](https://github.com/user-attachments/assets/63ba82bf-74a3-49d1-92ae-146d095b1d3e)


### 큰 이미지의 기준은 무엇인가?

10,000 픽셀² 이상의 이미지는 큰 이미지로 간주되며 Medium 우선순위로 높일 수 있다. 이 데모 페이지에서 큰 이미지를 추가했을 때, 예상대로 초기 요청 우선순위가 Medium으로 설정되었다.

![Medium 우선순위로 식별된 큰 이미지](https://github.com/user-attachments/assets/def25d39-ba76-42ba-8462-742beda3efcd)

이미지 태그에서 width와 height 속성을 10×10 픽셀로 설정하면, Chrome은 이 이미지를 더 이상 큰 이미지로 간주하지 않아 기본 Low 초기 요청 우선순위를 적용한다.

![이미지 크기를 10x10 픽셀로 설정한 후 Low 요청 우선순위가 적용된 동일한 큰 이미지](https://github.com/user-attachments/assets/e9310a89-2e55-4a27-be99-a4fc2a320390)

> 팁
> 
> Chrome은 width와 height가 지정되지 않은 경우 기본적으로 이미지를 큰 이미지로 간주한다.

## 뷰포트에 들어온 이미지는 높은 우선순위를 가진다

기본적으로 모든 이미지는 낮은 우선순위를 가진다. 이미지는 뷰포트에 들어오는 순간 높은 우선순위로 변경된다. 이 이벤트는 [첫 번째 콘텐츠가 그려지는 시점(First Contentful Paint, FCP)](https://www.debugbear.com/docs/metrics/first-contentful-paint) 이후에 발생한다. 이는 LCP(Largest Contentful Paint) 이미지에 문제를 일으킬 수 있다. 대부분의 경우 페이지 로드 중에 우선순위가 높음으로 변경되는 것을 볼 수 있다.

이 변경은 브라우저가 문서를 파싱하기 시작한 후에 발생하며, 이미지를 초기 계획보다 더 빨리 렌더링하도록 돕는다. DebugBear의 요청 워터폴 차트는 이 우선순위 변경을 빨간색 마커로 표시해 잠재적인 최적화를 식별하는 데 도움을 준다.

![첫 번째 콘텐츠가 그려지는 시점 이후의 이미지 우선순위 변경](https://github.com/user-attachments/assets/17802277-e594-4c83-9cd8-93f4bd9c160c)

이 예제에서 LCP 이미지의 우선순위 변경을 확인할 수 있다. FCP 마커는 1.09초에 나타나며, 그 후 2ms 뒤에 이미지가 뷰포트에 들어오면서 LCP 우선순위가 낮음에서 높음으로 변경된다.

아래에 보이는 다른 이미지 요청은 여전히 낮은 우선순위를 유지한다. 이 이미지는 화면 아래쪽에 위치하기 때문이다.

## fetchpriority 속성으로 우선순위 힌트 제공

개발자는 [`fetchpriority`](https://www.debugbear.com/blog/fetchpriority-attribute) 속성을 사용해 이미지 로딩 우선순위를 제어할 수 있다. `fetchpriority="high"` 또는 `fetchpriority="low"`를 설정하면 브라우저의 로직을 재정의한다.

LCP 이미지에 `fetchpriority="high"`를 설정하면 브라우저가 가능한 한 빨리 이미지를 로드하도록 우선순위를 부여한다. 이는 [LCP 렌더링 지연](https://www.debugbear.com/blog/lcp-render-delay)과 같은 근본적인 문제가 없는 경우 LCP 점수를 크게 향상시킬 수 있다.

![fetchpriority="high"가 설정된 LCP 이미지](https://github.com/user-attachments/assets/ff83a5b3-f2d4-4682-b275-d187c718c48c)

반대로, 화면 아래쪽에 위치한 큰 이미지의 우선순위를 중간에서 낮음으로 설정할 수도 있다.

이 예제에서는 5개의 큰 이미지 중 하나에 `fetchpriority="low"` 속성이 설정된 사이트를 테스트했다. 요청 워터폴 차트를 비교해보면 이제 중간 우선순위 이미지가 4개만 남은 것을 확인할 수 있다.

![fetchpriority="low" 속성이 설정된 큰 이미지](https://github.com/user-attachments/assets/fc7361ba-be63-4e4c-abda-ffcad6abb983)

## 이미지 미리 가져오기 요청의 우선순위

페이지 로딩을 관리하는 또 다른 방법은 [미리 가져오기](https://www.debugbear.com/blog/preload-largest-contentful-paint-image)를 사용하는 것이다. 이 방법은 브라우저가 리소스를 평소보다 일찍 가져오도록 지시한다.

미리 가져오기는 페이지에서 중요한 역할을 하지만 HTML에 직접 참조되지 않아 일찍 로드되지 않는 이미지에 특히 유용하다. HTML에 미리 가져오기를 추가하면 이미지가 일찍 로드되기 시작한다.

미리 가져오기된 리소스의 우선순위는 해당 리소스의 기본 우선순위와 동일하다. 따라서 이미지는 미리 가져오기되어도 여전히 낮은 우선순위를 유지한다.

![낮은 우선순위로 미리 가져오기된 이미지](https://github.com/user-attachments/assets/60ff76d5-a237-438d-a90b-65eea11a9a6d)

미리 가져오기 태그에 `fetchpriority="high"`를 추가하면 초기 요청 우선순위를 높게 설정할 수 있다.

```
<link rel="preload" as="image" href="/image.webp" fetchpriority="high" />
```

이 방법은 LCP 이미지에 권장되는 최적화 방법이며, DebugBear을 통해 [실험](https://www.debugbear.com/docs/experiments)으로 테스트할 수 있다.

![fetchpriority="high"로 미리 가져오기된 LCP 이미지](https://github.com/user-attachments/assets/8d2163cb-a686-475f-946f-47d94dc78b19)

## 웹사이트에서 요청 우선순위 테스트하기

이 글에서 언급한 최적화 가능성을 확인하려면 [무료 웹사이트 속도 테스트](https://www.debugbear.com/test/website-speed)를 사용해 보자.

예를 들어, 테스트 결과에서 미리 가져오기할 수 있는 LCP 이미지와 나중에 우선순위가 변경된 미리 가져오기된 이미지를 확인할 수 있다.

![무료 웹사이트 속도 테스트 최적화 가능성](https://github.com/user-attachments/assets/017472f6-082c-4c76-95e8-476c22c2414c)

## 웹사이트 성능 모니터링

[DebugBear](https://www.debugbear.com/)을 사용하면 웹사이트의 페이지 속도와 Core Web Vitals를 모니터링할 수 있다. 성능 저하를 쉽게 식별하고 웹사이트 로딩 속도를 높이는 방법을 찾을 수 있다.

![페이지 속도 대시보드](https://github.com/user-attachments/assets/3d3038c2-c40f-40e9-a854-d5033a81f542) 


