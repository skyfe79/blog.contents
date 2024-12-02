---
title: "렌더 트리 구성, 레이아웃, 페인트"
date: 2024-10-20T11:46:10Z
author: "skyfe79"
draft: false
tags: ["web","아마추어 번역"]
---

CSSOM과 DOM 트리는 렌더 트리로 합쳐진다. 이 렌더 트리는 모든 가시적 엘리먼트의 레이아웃을 계산하고, 화면에 픽셀을 그리는 페인트 과정의 입력으로 사용된다. 최적의 렌더링 성능을 달성하려면 이 모든 단계를 최적화하는 것이 중요하다.

[DOM 과 CSSOM 객체 모델 구축](https://blog.burt.pe.kr/posts/skyfe79-blog.contents-2600358236-post-81/)에서 우리는 HTML과 CSS 입력을 기반으로 DOM과 CSSOM 트리를 구축했다. 하지만 이 두 객체는 문서의 서로 다른 측면을 나타내는 독립적인 객체다. 하나는 내용을 설명하고, 다른 하나는 문서에 적용해야 할 스타일 규칙을 설명한다. 이 둘을 어떻게 합쳐서 브라우저가 화면에 픽셀을 렌더링하도록 할까?

## 요약

- DOM과 CSSOM 트리가 결합하여 렌더 트리를 형성한다.
- 렌더 트리는 페이지를 렌더링하는 데 필요한 노드만 포함한다.
- 레이아웃은 각 객체의 정확한 위치와 크기를 계산한다.
- 마지막 단계인 페인트는 최종 렌더 트리를 받아 화면에 픽셀을 그린다.

먼저, 브라우저는 DOM과 CSSOM을 "렌더 트리"로 결합한다. 이 렌더 트리는 페이지의 보이는 모든 DOM 내용과 각 노드의 모든 CSSOM 스타일 정보를 포함한다.

![DOM과 CSSOM이 결합하여 렌더 트리 생성](https://github.com/user-attachments/assets/23a27d35-fd1f-40a5-8c34-6728b402dd67)

렌더 트리를 구성하기 위해 브라우저는 대략 다음과 같은 과정을 거친다:

1. DOM 트리의 루트에서 시작하여 보이는 모든 노드를 순회한다.
   - 일부 노드는 보이지 않는다(예: script 태그, meta 태그 등). 이들은 렌더링된 출력에 반영되지 않으므로 생략한다.
   - CSS를 사용해 숨긴 일부 노드도 렌더 트리에서 생략한다. 예를 들어, 위 예제의 span 노드는 "display: none" 속성을 명시적으로 설정했기 때문에 렌더 트리에서 제외된다.

2. 각 보이는 노드에 대해 적절한 CSSOM 규칙을 찾아 적용한다.

3. 내용과 계산된 스타일이 있는 보이는 노드를 출력한다.

> **주의** 
>
> 잠깐 언급하자면, `visibility: hidden`은 `display: none`과 다르다. 전자는 엘리먼트를 보이지 않게 만들지만 여전히 레이아웃에서 공간을 차지한다(즉, 빈 상자로 렌더링된다). 반면 후자(`display: none`)는 엘리먼트를 렌더 트리에서 완전히 제거하여 보이지 않게 만들고 레이아웃의 일부가 되지 않게 한다.

최종 출력은 화면의 보이는 모든 콘텐츠의 내용과 스타일 정보를 모두 포함하는 렌더 트리다. **렌더 트리가 준비되면 "레이아웃" 단계로 넘어갈 수 있다.**

지금까지 어떤 노드가 보여야 하는지 결정하고 그들의 스타일을 계산했지만 기기의 뷰포트 내에서 그들의 정확한 위치와 크기는 계산하지 않았다. 이를 계산하는 과정이 바로 "레이아웃" 단계다. "리플로우"라고도 알려져 있다.

페이지의 각 객체의 정확한 크기와 위치를 파악하기 위해 브라우저는 렌더 트리의 루트에서 시작하여 순회한다. 다음 예제를 살펴보자:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>Critial Path: Hello world!</title>
  </head>
  <body>
    <div style="width: 50%">
      <div style="width: 50%">Hello world!</div>
    </div>
  </body>
</html>
```

[직접 해보기](https://googlesamples.github.io/web-fundamentals/fundamentals/performance/critical-rendering-path/nested.html)

이전 예제의 `<body>`는 두 개의 중첩된 `<div>`를 포함한다: 첫 번째(부모) `<div>`는 노드의 표시 크기를 뷰포트 너비의 `50%`로 설정하고, 두 번째 `<div>`(부모에 포함됨)는 그 `너비`를 부모의 50%로 설정한다. 즉, 뷰포트 너비의 25%다.

![레이아웃 정보 계산하기](https://github.com/user-attachments/assets/c247a099-159d-4c3f-b28d-829f9e8a67e1)

레이아웃 과정의 출력은 "박스 모델"이다. 이는 뷰포트 내에서 각 엘리먼트의 정확한 위치와 크기를 정밀하게 포착한다: 모든 상대적 측정값은 화면의 절대적인 픽셀로 변환된다.

마지막으로, 이제 어떤 노드가 보이는지, 그들의 계산된 스타일과 기하학적 구조를 알았으니, 이 정보를 최종 단계로 전달할 수 있다. 이 단계는 렌더 트리의 각 노드를 화면의 실제 픽셀로 변환한다. 이 단계를 종종 "페인팅" 또는 "래스터화"라고 부른다.

이 과정은 브라우저가 상당한 작업을 수행해야 하므로 시간이 걸릴 수 있다. 하지만 Chrome DevTools는 앞서 설명한 세 단계에 대한 통찰을 제공할 수 있다. 원래의 "hello world" 예제에 대한 레이아웃 단계를 살펴보자:

![DevTools에서 레이아웃 측정하기](https://github.com/user-attachments/assets/164c356e-2622-42f3-a13b-043dc087341d)

- "Layout" 이벤트는 타임라인에서 렌더 트리 구성, 위치 및 크기 계산을 포착한다.
- 레이아웃이 완료되면 브라우저는 "Paint Setup"과 "Paint" 이벤트를 발생시킨다. 이는 렌더 트리를 화면의 픽셀로 변환한다.

렌더 트리 구성, 레이아웃 및 페인트에 필요한 시간은 문서의 크기, 적용된 스타일, 그리고 실행 중인 기기에 따라 다르다: 문서가 클수록 브라우저의 작업량이 많아진다; 스타일이 복잡할수록 페인팅에 더 많은 시간이 소요된다(예를 들어, 단색은 페인트하기 "저렴"하지만, 그림자 효과는 계산하고 렌더링하기 "비싸다").

페이지가 마침내 뷰포트에 보이게 된다:

![렌더링된 Hello World 페이지](https://github.com/user-attachments/assets/7a775bee-fa97-4b17-9e24-ab3f7fa3b558)

브라우저의 단계를 간단히 요약하면 다음과 같다:

1. HTML 마크업을 처리하고 DOM 트리를 구축한다.
2. CSS 마크업을 처리하고 CSSOM 트리를 구축한다.
3. DOM과 CSSOM을 렌더 트리로 결합한다.
4. 렌더 트리에서 레이아웃을 실행하여 각 노드의 기하학적 구조를 계산한다.
5. 개별 노드를 화면에 페인트한다.

이 데모 페이지는 간단해 보이지만, 브라우저 입장에서는 상당한 작업이 필요하다. DOM이나 CSSOM이 수정되면, 화면에 다시 렌더링해야 할 픽셀을 파악하기 위해 이 과정을 반복해야 한다.

> **중요** 
>
> **렌더링 경로 최적화는 위 순서의 1단계부터 5단계까지 수행하는 데 걸리는 총 시간을 최소화하는 과정이다.** 이렇게 하면 콘텐츠를 화면에 가능한 한 빨리 렌더링할 수 있고, 초기 렌더링 후 화면 업데이트 사이의 시간도 줄일 수 있다. 즉, 대화형 콘텐츠에 대해 더 높은 리프레시 속도를 달성할 수 있다.

## 알림

이 글은 web.dev 사이트에서 Ilya Grigorik 님이 작성하신 [Render-tree Construction, Layout, and Paint](https://web.dev/articles/critical-rendering-path/render-tree-construction) 글을 한국어로 편역한 내용을 담고 있습니다.
