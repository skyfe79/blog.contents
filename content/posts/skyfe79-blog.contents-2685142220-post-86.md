---
title: "동적 임포트(dynamic import)"
date: 2024-11-23T02:49:44Z
author: "skyfe79"
draft: false
tags: ["javascript","web","아마추어 번역","V8","\bECMAScript"]
---


[동적(dynamic) `import()`](https://github.com/tc39/proposal-dynamic-import)는 기존의 정적(static) `import`를 보완하는 새로운 기능이다. 함수처럼 사용할 수 있는 이 새로운 `import` 형식은 정적 `import`가 할 수 없었던 작업을 가능하게 한다. 이 글에서는 두 가지 `import` 방식의 차이점을 살펴보고, 동적 `import()`가 제공하는 새로운 기능들을 소개한다.

## 정적 `import` 복습 [#](https://v8.dev/features/dynamic-import#static)

Chrome 61 버전부터 [모듈](https://v8.dev/features/modules) 내에서 ES2015의 `import` 문을 지원하기 시작했다. 다음 예제를 통해 정적 `import`의 사용법을 살펴보자.

`./utils.mjs` 파일에 있는 모듈의 내용:

```javascript
// 기본 내보내기
export default () => {
  console.log('기본 내보내기에서 안녕하세요!');
};

// 이름 붙은 내보내기 `doStuff`
export const doStuff = () => {
  console.log('작업 중…');
};
```

이 모듈을 정적으로 가져와 사용하는 방법:

```html
<script type="module">
  import * as module from './utils.mjs';
  module.default();  // → '기본 내보내기에서 안녕하세요!' 출력
  module.doStuff();  // → '작업 중…' 출력
</script>
```

**참고:** 여기서는 `.mjs` 확장자를 사용해 이 파일이 일반 스크립트가 아닌 모듈임을 나타낸다. 웹에서는 파일 확장자보다 MIME 타입이 중요하다. JavaScript 파일은 `Content-Type` HTTP 헤더에 `text/javascript`로 명시되어야 한다.

`.mjs` 확장자는 [Node.js](https://nodejs.org/api/esm.html#esm_enabling)나 [`d8`](https://v8.dev/docs/d8) 같은 플랫폼에서 특히 유용하다. 이런 환경에서는 MIME 타입이나 `type="module"` 같은 요소가 없어, 파일 확장자로 모듈과 일반 스크립트를 구분한다.

정적 `import`는 다음과 같은 특징이 있다:
- 모듈 지정자로 문자열 리터럴만 사용 가능
- 파일의 최상위 레벨에서만 사용 가능
- 런타임 이전에 모듈 간 연결을 설정

이러한 특성 덕분에 정적 분석, 번들링, 트리 쉐이킹 등이 가능해진다.

하지만 정적 `import`로는 다음과 같은 상황을 해결하기 어렵다:
- 조건에 따라 모듈 가져오기
- 런타임에 모듈 경로 결정하기
- 일반 스크립트에서 모듈 가져오기

## 동적 `import()` 소개 🔥 [#](https://v8.dev/features/dynamic-import#dynamic)

[동적 `import()`](https://github.com/tc39/proposal-dynamic-import)는 이러한 제한을 해결하는 새로운 방식이다. `import(moduleSpecifier)`는 프로미스를 반환하며, 이 프로미스는 요청한 모듈의 네임스페이스 객체로 해결된다. 이 객체는 모듈과 그 의존성을 가져오고, 인스턴스화하고, 평가한 후에 생성된다.

동적으로 `./utils.mjs` 모듈을 가져와 사용하는 예:

```html
<script type="module">
  const moduleSpecifier = './utils.mjs';
  import(moduleSpecifier)
    .then((module) => {
      module.default();  // → '기본 내보내기에서 안녕하세요!' 출력
      module.doStuff();  // → '작업 중…' 출력
    });
</script>
```

`import()`가 프로미스를 반환하므로 `async`/`await`를 사용할 수도 있다:

```html
<script type="module">
  (async () => {
    const moduleSpecifier = './utils.mjs';
    const module = await import(moduleSpecifier);
    module.default();  // → '기본 내보내기에서 안녕하세요!' 출력
    module.doStuff();  // → '작업 중…' 출력
  })();
</script>
```

**주의:** `import()`는 함수처럼 보이지만 실제로는 특별한 문법이다. 따라서 `Function.prototype`을 상속받지 않으며, `call`이나 `apply` 메서드를 사용할 수 없다. 또한 `const importAlias = import`와 같은 할당도 불가능하다.

## 동적 `import()` 활용 예제

다음은 단일 페이지 애플리케이션에서 동적 `import()`를 사용해 페이지 전환 시 필요한 모듈을 지연 로딩하는 예제다:

```html
<!DOCTYPE html>
<meta charset="utf-8">
<title>내 라이브러리</title>
<nav>
  <a href="books.html" data-entry-module="books">도서</a>
  <a href="movies.html" data-entry-module="movies">영화</a>
  <a href="video-games.html" data-entry-module="video-games">비디오 게임</a>
</nav>
<main>여기에 필요에 따라 콘텐츠가 로드됩니다.</main>
<script>
  const main = document.querySelector('main');
  const links = document.querySelectorAll('nav > a');
  for (const link of links) {
    link.addEventListener('click', async (event) => {
      event.preventDefault();
      try {
        const module = await import(`/${link.dataset.entryModule}.mjs`);
        // 모듈은 `loadPageInto`라는 함수를 내보낸다고 가정
        module.loadPageInto(main);
      } catch (error) {
        main.textContent = error.message;
      }
    });
  }
</script>
```

이 예제에서는 사용자가 링크를 클릭할 때만 해당 페이지의 모듈을 로드한다. 이를 통해 초기 로딩 시간을 줄이고 필요한 리소스만 효율적으로 사용할 수 있다.

[Addy Osmani](https://twitter.com/addyosmani)는 이 기법을 활용해 [Hacker News PWA 예제](https://hnpwa-vanilla.firebaseapp.com/)를 개선했다. 원래 버전은 모든 콘텐츠를 한 번에 로드했지만, [수정된 버전](https://dynamic-import.firebaseapp.com/)에서는 댓글을 동적으로 가져와 성능을 향상시켰다.

**참고:** 다른 도메인의 스크립트를 가져올 때는 해당 스크립트가 적절한 CORS 헤더(예: `Access-Control-Allow-Origin: *`)와 함께 제공되어야 한다. 모듈 스크립트는 일반 스크립트와 달리 CORS 규칙을 따르기 때문이다.

## 권장 사항 [#](https://v8.dev/features/dynamic-import#recommendations)

정적 `import`와 동적 `import()`는 각각의 용도가 있다:
- 초기 렌더링에 필요한 핵심 의존성은 정적 `import` 사용
- 조건부로 필요하거나 나중에 로드해도 되는 모듈은 동적 `import()` 사용

상황에 맞는 적절한 방식을 선택하여 애플리케이션의 성능과 사용자 경험을 최적화할 수 있다.

## 동적 `import()` 지원 현황 [#](https://v8.dev/features/dynamic-import#support)

- Chrome: 63 버전부터 지원
- Firefox: 67 버전부터 지원
- Safari: 11.1 버전부터 지원
- [Node.js: 13.2 버전부터 지원](https://nodejs.medium.com/announcing-core-node-js-support-for-ecmascript-modules-c5d6dc29b663)
- [Babel: 지원됨](https://babeljs.io/docs/en/babel-plugin-syntax-dynamic-import)

## 알림

이 글은 [v8.dev](https://v8.dev/)에 2017년 11월 21일 발행된 [Dynamic import()](https://v8.dev/features/dynamic-import) 글을 한국어로 편역한 내용을 담고 있습니다.

