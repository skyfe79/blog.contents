---
title: "이중 키 캐싱: 브라우저 캐시 분할이 웹에 미친 변화"
date: 2025-04-12T04:57:52Z
author: "skyfe79"
draft: false
tags: ["web"]
---

> 원문: https://addyosmani.com/blog/double-keyed-caching/

웹의 캐싱 모델은 지난 20년 이상 우리에게 잘 작동해 왔다. 최근 들어, 개인정보 보호를 이유로 캐싱 방식에 근본적인 변화가 일어나면서 기존의 성능 최적화 가정들이 도전받고 있다. 이를 이중 키 캐싱(Double-keyed Caching) 또는 더 일반적으로 [캐시 분할](https://developer.chrome.com/blog/http-cache-partitioning)이라고 부른다. 이 변화가 무엇인지, 왜 중요한지, 그리고 어떻게 적응해야 하는지 알아보자.

## 캐싱의 과거 동작 방식 (2020년 이전)

기존 모델에서 브라우저는 캐시된 리소스를 위한 간단한 키-값 저장소를 유지했다:

```
cache = {
  "https://cdn.example.com/jquery-3.6.0.min.js": resourceData,
  "https://fonts.googleapis.com/css?family=Roboto": resourceData
}
```

이 방식은 사용자가 공용 CDN에서 jQuery를 로드하는 사이트를 방문하면, 동일한 CDN에서 호스팅된 jQuery를 사용하는 다른 사이트를 방문할 때 즉시 캐시 히트가 발생한다는 의미다. 이 모델은 2010년대 웹 개발을 지배한 "CDN 우선" 접근 방식을 뒷받침했다.

이 방식의 장점은 다음과 같다:

* 사이트 간 리소스 공유를 통해 대역폭 사용량 감소
* 공통 리소스를 사용하는 여러 사이트를 방문하는 사용자의 성능 향상
* 공용 CDN 활용을 통한 호스팅 비용 절감
* 캐시 히트를 통한 빠른 페이지 로드 속도

## 프라이버시 문제

이 모델은 효율적이지만 정보 유출 문제가 있었다. 주요 공격 벡터는 다음과 같다:

1. 캐시 탐색: 사이트 A가 사이트 B의 리소스가 캐시에 있는지 확인해 사용자의 방문 기록을 알아낼 수 있었다.
2. 타이밍 공격: 리소스 로딩 시간을 측정해 캐시 상태를 노출시킬 수 있었다.
3. 크로스 사이트 추적: 캐시된 리소스를 지속적인 식별자로 사용할 수 있었다.

## 새로운 모델: 이중 키 캐싱

이중 키 캐싱은 브라우저가 리소스를 저장하고 검색하는 방식을 근본적으로 바꾼다. 기존에는 리소스 URL만 캐시 키로 사용했지만, 이제는 요청을 보내는 최상위 사이트와 리소스 URL 두 가지 정보를 함께 사용한다. 예제를 통해 자세히 살펴보자.

site-a.com이 CDN에서 jQuery를 요청하면, 브라우저는 site-a.com(요청자)과 리소스 URL을 조합한 HTTP 캐시 항목을 생성한다. 이후 site-b.com이 동일한 jQuery 파일을 요청할 때, 캐시된 복사본을 재사용하는 대신 site-b.com을 키의 일부로 하는 완전히 새로운 캐시 항목을 만든다. 이는 현대적인 캐시 항목의 모습을 보여준다:

```
cache = {
  {
    topLevelSite: "site-a.com",
    resource: "https://cdn.example.com/jquery-3.6.0.min.js"
  }: resourceData,
  {
    topLevelSite: "site-b.com",
    resource: "https://cdn.example.com/jquery-3.6.0.min.js"
  }: resourceData
}
```

이 방식은 동일한 리소스라도 요청하는 사이트마다 별도로 캐시된다는 것을 의미한다. 즉, 캐시가 요청 사이트의 오리진에 따라 분할된다. 이는 사이트 간 추적과 같은 프라이버시 문제를 방지하지만, 동일한 리소스의 중복 복사본을 저장해야 한다는 단점도 있다. 이는 보안과 효율성 사이의 균형을 맞추는 트레이드오프다.

## 캐시 적중률에 미치는 성능 영향

Chrome의 [구현 데이터](https://developer.chrome.com/blog/http-cache-partitioning/?utm_source=chatgpt.com#what_is_the_impact_of_this_behavioral_change)에 따르면, 이중 키 캐싱은 다음과 같은 영향을 미친다:

*   전체 캐시 미스율이 약 3.6% 증가
*   네트워크에서 로드되는 데이터 양이 약 4% 증가
*   첫 번째 콘텐츠 렌더링 시간(FCP)에 약 0.3%의 영향

이 수치들은 작아 보일 수 있지만, 리소스 타입과 사용 패턴에 따라 그 영향은 크게 달라진다.

이중 키 캐싱은 성능 오버헤드를 유발하지만, 이는 보안과 개인정보 보호를 강화하기 위한 필수적인 절충안으로 받아들여진다. 이 메커니즘은 다음과 같은 다양한 보안 취약점을 방지하는 데 도움을 준다:

*   사용자의 브라우징 기록을 노출할 수 있는 타이밍 공격 방지
*   캐시 기반 지문 추적을 통한 크로스 사이트 추적 방지
*   공유 캐시 상태를 악용하는 사이드 채널 공격 완화
*   크로스 사이트 검색 공격에 대한 저항력 강화

이러한 보안상의 이점은 성능 비용을 정당화한다. 하지만 이러한 영향을 이해하고 최적화하는 것은 여전히 중요하다.

### 네트워크 대역폭에 미치는 영향

캐시 미스율이 증가하면 추가적인 네트워크 요청이 발생한다. 일반적인 웹 리소스의 경우, 이로 인한 영향은 다음과 같다:

#### 공유 라이브러리

*   이전에는 크로스 사이트 캐싱의 혜택을 받던 인기 있는 자바스크립트 라이브러리가 이제 별도로 다운로드해야 한다.
*   jQuery나 React 같은 CDN 호스팅 프레임워크는 캐싱의 이점이 줄어들었다.
*   각 최상위 사이트가 자체 복사본을 유지하므로 전체 대역폭 사용량이 증가한다.

#### 웹 폰트

*   Google Fonts와 같은 서비스에서 제공하는 일반적인 폰트는 여러 번 다운로드해야 한다.
*   여러 도메인에서 공유 폰트 리소스를 사용하는 조직은 대역폭 비용이 증가한다.
*   폰트 로딩 성능이 네트워크 상태에 더 많이 의존하게 된다.

#### 대용량 리소스

* 머신러닝 모델과 같은 대용량 리소스가 가장 큰 영향을 받는다.
* 메가바이트 단위의 리소스는 이제 여러 번 다운로드해야 할 가능성이 있다.
* 여러 도메인에 걸쳐 대용량 리소스를 제공하는 조직의 경우 CDN 비용이 크게 증가할 수 있다.

이 평균값은 중요한 예외 사례를 숨기고 있다. 실제 사례를 살펴보면:

### 엔터프라이즈 SaaS 제품군

```
// 이전: 하나의 캐시 항목
cdn.company.com/shared-lib.js → 2.5MB

// 이후: 여러 항목
{crm.company.com, cdn.company.com/shared-lib.js} → 2.5MB
{mail.company.com, cdn.company.com/shared-lib.js} → 2.5MB
{docs.company.com, cdn.company.com/shared-lib.js} → 2.5MB

// 결과: 총 7.5MB 캐시 사용량
```

### 인기 프레임워크 CDN

```
// 이전
unpkg.com/react@18.2.0 한 번 캐시 → 118KB

// 이후
{site1.com, unpkg.com/react@18.2.0} → 118KB
{site2.com, unpkg.com/react@18.2.0} → 118KB
{site3.com, unpkg.com/react@18.2.0} → 118KB
```

### 자주 사용하는 React 의존성

인기 있는 React 라이브러리도 비슷한 영향을 받는다:

```
// Material-UI
{
    app_domain: "myapp.com",
    resource: "unpkg.com/@mui/material@5.14.0/umd/material-ui.production.min.js"
}

// React Router
{
    app_domain: "myapp.com",
    resource: "unpkg.com/react-router@6.14.0/umd/react-router.production.min.js"
}

// Redux
{
    app_domain: "myapp.com",
    resource: "unpkg.com/redux@4.2.1/dist/redux.min.js"
}
```

### 마이크로 프론트엔드의 영향

React 마이크로 프론트엔드에서는 다음과 같은 영향이 두드러진다:

```
// 메인 셸 애플리케이션 (shell.company.com)
{
    app_domain: "shell.company.com",
    resources: [
        "unpkg.com/react@18.2.0/umd/react.production.min.js",
        "unpkg.com/react-dom@18.2.0/umd/react-dom.production.min.js"
    ]
}

// 마이크로 프론트엔드 1 (app1.company.com)
{
    app_domain: "app1.company.com",
    resources: [
        "unpkg.com/react@18.2.0/umd/react.production.min.js",
        "unpkg.com/react-dom@18.2.0/umd/react-dom.production.min.js"
    ]
}

// 결과: 각 마이크로 프론트엔드마다 React가 중복 다운로드됨
```

## 성능 영향 예시

### 일반적인 React 애플리케이션 번들

일반적인 React 애플리케이션 설정을 살펴보자:

```
// 번들 크기
const commonDependencies = {
    react: "118 KB",
    reactDom: "1.1 MB",
    materialUI: "469 KB",
    reactRouter: "27 KB",
    redux: "22 KB"
};

// 도메인당 총합: ~1.7 MB
```

여러 하위 도메인이나 마이크로 프론트엔드를 사용할 경우, 각 도메인은 캐시에 별도의 복사본을 저장해야 한다.

### 공유 컴포넌트 라이브러리

공유 React 컴포넌트 라이브러리를 사용하는 조직의 경우:

```
// 공유 UI 라이브러리
{
    size: "2.5 MB",
    domains: [
        "main-app.company.com",
        "dashboard.company.com",
        "admin.company.com"
    ],
    totalCacheSize: "7.5 MB" // 2.5 MB × 3 domains
}
```

## 실질적인 영향

### CDN 경제성의 변화

* "퍼블릭 CDN 사용" 권장 사항을 재검토해야 한다
* 자체 호스팅이 대역폭 효율성 측면에서 더 나은 선택이 될 수 있다
* 캐시 적중률 감소로 인해 CDN 비용이 증가할 가능성이 있다

### 도메인 전략이 더 중요해졌다

*   각 서브도메인이 이제 별도의 캐시를 유지한다
*   도메인 통합은 성능상 이점을 제공한다
*   오리진과 연계된 CDN 도메인을 고려해야 한다

### 번들 전략 업데이트 필요

*   공유 청크보다는 작고 집중된 번들이 더 나을 수 있다
*   코드 분할 경계는 도메인 경계와 일치해야 한다
*   리소스 우선순위 지정이 더욱 중요해진다

## 적응 전략

### 도메인 통합

다음과 같이 사용하는 대신:

```
assets1.company.com/lib.js
assets2.company.com/lib.js
```

다음과 같이 고려해 본다:

```
static.company.com/app1/lib.js
static.company.com/app2/lib.js
```

```
// 다음과 같이 사용하는 대신:
const domains = [
    'app1.company.com/static/js/react.js',
    'app2.company.com/static/js/react.js'
];

// 다음과 같이 고려해 본다:
const consolidatedDomain = 'static.company.com/js/react.js';
```

모듈 연합:

```
// webpack.config.js
module.exports = {
    plugins: [
        new ModuleFederationPlugin({
            name: 'host',
            filename: 'remoteEntry.js',
            remotes: {
                app1: 'app1@http://static.company.com/app1/remoteEntry.js',
                app2: 'app2@http://static.company.com/app2/remoteEntry.js'
            },
            shared: {
                react: { singleton: true },
                'react-dom': { singleton: true }
            }
        })
    ]
};
```

### 스마트 리소스 로딩

```
// 이전: 캐시에 의존
<script src="https://unpkg.com/react@18.2.0"></script>

// 이후: 핵심 리소스를 직접 호스팅
<script src="/vendor/react-18.2.0.min.js"></script>
```

## 브라우저별 캐시 분할 방식

각 브라우저는 캐시 분할을 다르게 구현한다:

*   **Chrome**: 최상위 사이트 + 프레임 사이트를 기준으로 분할
*   **Safari**: 최상위 eTLD+1을 기준으로 분할
*   **Firefox**: scheme://eTLD+1을 기준으로 분할할 계획

이러한 차이로 인해 브라우저별 성능 영향이 달라진다. 2013년 도입된 Safari의 방식이 가장 강력한 분할을 적용한다.

## 권장 사항

### 즉시 실행할 작업

* 도메인 전략 점검
* 실제 캐시 적중률 측정
* 핵심 리소스 자체 호스팅 고려

### 아키텍처 업데이트

*   번들 경계를 도메인 경계와 일치시킨다
*   강력한 성능 모니터링을 구현한다
*   리소스 힌트를 전략적으로 활용한다

### 장기적인 계획 수립

* 아키텍처 설계 시 캐시 분할(cache partitioning)을 고려한다
* 증가할 대역폭 비용에 대비해 계획을 세운다

## 잘 알려진 리소스에 대한 특별한 고려가 필요할까?

웹 플랫폼은 개인정보 보호를 유지하면서도 잘 알려진 대용량 리소스에 대해 더 나은 크로스 오리진 캐싱을 가능하게 하는 새로운 솔루션을 고려할 필요가 있다. 특히 클라이언트 측 AI 모델과 같이 크기가 중간에서 매우 큰 리소스가 이에 해당한다. 클라이언트 측 머신러닝 애플리케이션에 미치는 영향은 특별히 주목할 만한 부분이다.

### 모델 로딩

*   다른 오리진에서 접근하는 모델의 로딩 시간 증가
*   모델 배포를 위한 대역폭 비용 상승
*   애플리케이션 초기화 시간에 미칠 수 있는 잠재적 영향

### 최적화 접근 방식

* 모델 분할과 점진적 로딩
* 공유 기본 모델 활용과 차등 업데이트
* 애플리케이션 수준에서의 모델 캐싱 전략 구현

여기서는 과거의 솔루션 탐구(예: Web Bundles, Cache Transparency)에 대해 자세히 다루지 않지만, 앞서 언급한 패턴 외에도 이 문제에 대해 더 깊이 고민할 가치가 있다고 생각한다.

## 앞으로 나아갈 길

캐시 분할은 웹 개인정보 보호를 위한 필수적인 진화이지만, 실제 성능 저하를 동반한다. 이러한 변화에 저항하기보다는 아키텍처와 최적화 전략을 적절히 조정해야 한다.

웹 플랫폼은 계속 발전하고 있으며, 새로운 API와 패턴이 등장해 개인정보 보호와 성능 간의 균형을 맞추는 데 도움을 줄 것이다. 그때까지는 신중한 도메인 전략과 리소스 관리가 새로운 환경에서 성능을 최적화하는 최선의 도구로 남아 있다.

공용 CDN을 공유하던 시대는 끝나가고 있지만, 웹의 적응력과 진화 능력은 여전히 유효하다. 언제나 그렇듯이, 특정 사용 사례에 맞춰 측정하고 최적화하며 적응해야 한다.

