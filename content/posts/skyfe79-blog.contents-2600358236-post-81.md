---
title: "DOM 과 CSSOM 객체 모델 구축"
date: 2024-10-20T11:29:58Z
author: "skyfe79"
draft: false
tags: ["web","아마추어 번역"]
---

브라우저가 페이지를 렌더링하기 전에 DOM과 CSSOM 트리를 구축해야 한다. 따라서 HTML과 CSS를 최대한 빠르게 브라우저에 전달해야 한다.

## 요약

- 바이트 → 문자 → 토큰 → 노드 → 객체 모델의 순서로 변환된다.
- HTML 마크업은 문서 객체 모델(DOM)로 변환되고, CSS 마크업은 CSS 객체 모델(CSSOM)로 변환된다.
- DOM과 CSSOM은 독립적인 데이터 구조다.
- Chrome DevTools의 Performance 패널을 사용하면 DOM과 CSSOM의 구축 및 처리 비용을 캡처하고 검사할 수 있다.

## 문서 객체 모델(DOM)

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <link href="style.css" rel="stylesheet" />
    <title>Critical Path</title>
  </head>
  <body>
    <p>Hello <span>web performance</span> students!</p>
    <div><img src="awesome-photo.jpg" /></div>
  </body>
</html>
```

[직접 해보기](https://googlesamples.github.io/web-fundamentals/fundamentals/performance/critical-rendering-path/basic_dom.html)

가장 간단한 경우부터 시작해보자. 일부 텍스트와 하나의 이미지가 있는 기본 HTML 페이지다. 브라우저는 이 페이지를 어떻게 처리할까?

![DOM 구축 과정](https://github.com/user-attachments/assets/9a81f6d8-1b43-4876-8245-1dd1b083d01f)

1. **변환:** 브라우저는 디스크나 네트워크에서 HTML의 원시 바이트를 읽어 파일의 지정된 인코딩(예: UTF-8)에 따라 개별 문자로 변환한다.

2. **토큰화:** 브라우저는 문자열을 [W3C HTML5 표준](http://www.w3.org/TR/html5/)에 지정된 대로 고유한 토큰으로 변환한다. 예를 들어 `<html>`, `<body>` 등이 있다. 각 토큰은 특별한 의미와 고유한 규칙 세트를 갖는다.

3. **렉싱:** 발행된 토큰은 속성과 규칙을 정의하는 "객체"로 변환된다.

4. **DOM 구축:** HTML 마크업은 서로 다른 태그 간의 관계를 정의한다(일부 태그는 다른 태그 내에 포함됨). 생성된 객체는 원래 마크업에 정의된 부모-자식 관계도 포착하는 트리 데이터 구조로 연결된다. *HTML* 객체는 *body* 객체의 부모이고, *body* 는 *paragraph* 객체의 부모이다. 이런 식으로 문서의 전체 표현이 구축된다.

![DOM 트리](https://github.com/user-attachments/assets/79310c15-bb2b-46ab-a67b-f2994cd7cd7a)

**이 전체 과정의 최종 출력은 우리의 간단한 페이지에 대한 문서 객체 모델(DOM)이다. 브라우저는 이를 페이지의 모든 추가 처리에 사용한다.**

브라우저가 HTML 마크업을 처리할 때마다 위에서 정의한 모든 단계를 거친다. 바이트를 문자로 변환하고, 토큰을 식별하고, 토큰을 노드로 변환하고, DOM 트리를 구축한다. 이 전체 과정은 시간이 걸릴 수 있으며, 특히 처리할 HTML이 많은 경우에 그렇다.

![DevTools에서 DOM 구축 추적](https://github.com/user-attachments/assets/00393863-3968-4143-8a75-7792985a0e77)

> **참고:** Chrome DevTools에 대한 기본적인 지식이 있다고 가정한다. 즉, 네트워크 워터폴을 캡처하거나 타임라인을 기록하는 방법을 알고 있다는 의미다. 빠른 복습이 필요하다면 [Chrome DevTools 문서](https://developer.chrome.com/docs/devtools/overview)를 확인하자. DevTools를 처음 사용한다면 DevTools 문서에서 [초보자를 위한 DevTools](https://developer.chrome.com/docs/devtools/open)를 확인하자.

Chrome DevTools를 열고 페이지가 로드되는 동안 타임라인을 기록하면 이 단계를 수행하는 데 걸리는 실제 시간을 볼 수 있다. 이전 예제에서는 HTML 청크를 DOM 트리로 변환하는 데 약 5ms가 걸렸다. 더 큰 페이지에서는 이 과정이 훨씬 더 오래 걸릴 수 있다. 부드러운 애니메이션을 만들 때 브라우저가 대량의 HTML을 처리해야 한다면 이 과정이 병목 현상이 될 수 있다.

DOM 트리는 문서 마크업의 속성과 관계를 캡처하지만 엘리먼트가 렌더링될 때 어떻게 보일지는 알려주지 않는다. 그것은 CSSOM의 책임이다.

## CSS 객체 모델(CSSOM)

브라우저가 기본 페이지의 DOM을 구축하는 동안 문서의 `<head>` 섹션에서 외부 CSS 스타일시트를 참조하는 `<link>` 엘리먼트를 만났다: `style.css`. 페이지를 렌더링하는 데 이 리소스가 필요할 것으로 예상하여 즉시 이 리소스에 대한 요청을 보내고 다음과 같은 내용을 받는다:

```css
body { font-size: 16px; }
p { font-weight: bold; }
span { color: red; }
p span { display: none; }
img { float: right; }
```

HTML 마크업 내에 직접 스타일을 선언할 수도 있지만(인라인), CSS를 HTML과 독립적으로 유지하면 콘텐츠와 디자인을 별개의 관심사로 다룰 수 있다. 디자이너는 CSS를 작업하고, 개발자는 HTML과 다른 관심사에 집중할 수 있다.

HTML과 마찬가지로 받은 CSS 규칙을 브라우저가 이해하고 작업할 수 있는 형태로 변환해야 한다. 따라서 HTML 프로세스를 반복하지만 이번에는 HTML 대신 CSS를 대상으로 한다:

![CSSOM 구축 단계](https://github.com/user-attachments/assets/0de118d0-aba4-4415-80ee-073d9dece823)

CSS 바이트는 문자로 변환되고, 그다음 토큰과 노드로 변환되며, 마지막으로 "CSS 객체 모델"(CSSOM)이라는 트리 구조로 연결된다:

![CSSOM 트리](https://github.com/user-attachments/assets/fb4661a0-25bc-48dd-9e54-b554aac24b60)

CSSOM이 트리 구조를 가지는 이유는 무엇일까? 페이지의 모든 객체에 대한 최종 스타일 세트를 계산할 때 브라우저는 해당 노드에 적용 가능한 가장 일반적인 규칙(예: `body` 엘리먼트의 자식이라면 모든 `body` 스타일이 적용됨)부터 시작하여 더 구체적인 규칙을 재귀적으로 적용한다. 즉, 규칙이 "아래로 계단식으로 내려간다".

더 구체적으로 설명하기 위해 앞서 설명한 CSSOM 트리를 살펴보자. `<span>` 태그 내에 포함되어 본문 엘리먼트 안에 있는 텍스트는 16픽셀의 글꼴 크기와 빨간색 텍스트를 갖는다. `font-size` 지시문이 본문에서 `span`으로 계단식으로 내려간다. 하지만 `span`이 단락(`p`) 태그의 자식이라면 그 내용은 표시되지 않는다.

또한 앞서 설명한 트리는 완전한 CSSOM 트리가 아니며 스타일시트에서 재정의하기로 결정한 스타일만 보여준다는 점에 유의하자. 모든 브라우저는 "사용자 에이전트 스타일"이라고도 알려진 기본 스타일 세트를 제공한다. 이는 우리가 자체 스타일을 제공하지 않을 때 보이는 스타일이다. 우리의 스타일은 이러한 기본값을 재정의한다.

CSS 처리에 걸리는 시간을 알아보려면 DevTools에서 타임라인을 기록하고 "Recalculate Style" 이벤트를 찾으면 된다. DOM 파싱과 달리 타임라인에는 별도의 "Parse CSS" 항목이 표시되지 않는다. 대신 이 하나의 이벤트에서 파싱과 CSSOM 트리 구축, 그리고 계산된 스타일의 재귀 계산을 모두 캡처한다.

![DevTools에서 CSSOM 구축 추적](https://github.com/user-attachments/assets/0f6d6617-66e7-4426-999e-9d754bb82dd7)

우리의 간단한 스타일시트를 처리하는 데 약 0.6ms가 걸리고 페이지의 8개 엘리먼트에 영향을 미친다. 많지 않지만 여전히 무료는 아니다. 그러나 8개의 엘리먼트는 어디서 왔을까? CSSOM과 DOM은 독립적인 데이터 구조다! 알고 보니 브라우저는 중요한 단계를 숨기고 있다. 다음으로 [렌더 트리](https://web.dev/articles/critical-rendering-path/render-tree-construction)가 DOM과 CSSOM을 어떻게 연결하는지 살펴볼 것이다.

## 알림

이 글은 web.dev 사이트에서 Ilya Grigorik 님이 작성하신 [Constructing the Object Model](https://web.dev/articles/critical-rendering-path/constructing-the-object-model) 글을 한국어로 편역한 내용을 담고 있습니다.



