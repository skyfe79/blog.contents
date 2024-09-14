---
title: "React 19의 새로운 기능"
date: 2024-09-14T05:19:16Z
author: "skyfe79"
draft: false
tags: ["react"]
---

React 19가 곧 출시됩니다. React 코어 팀이 지난 4월에 [React 19 릴리즈 후보(RC)를 공개했습니다](https://react.dev/blog/2024/04/25/react-19#new-feature-use). 이번 메이저 버전은 성능, 사용성, 그리고 개발자 경험을 개선하기 위해 여러 업데이트와 새로운 패턴을 도입했습니다.

React 18에서 실험적으로 선보였던 많은 기능들이 이제 React 19에서 안정화 단계에 접어들었습니다. 개발자 여러분이 알아두면 좋을 주요 변경 사항들을 간단히 정리해 보겠습니다.

## 서버 컴포넌트: React의 혁신적인 변화

React가 처음 나온 지 10년 만에 서버 컴포넌트라는 큰 변화가 찾아왔습니다. 이는 React 19의 새로운 기능들의 토대가 되어 여러 면에서 개선을 가져왔습니다:

- **초기 페이지 로딩 속도 향상**: 서버에서 컴포넌트를 렌더링하면 클라이언트로 보내는 JavaScript 양이 줄어들어 초기 로딩이 빨라집니다. 또한 페이지를 클라이언트로 보내기 전에 서버에서 먼저 데이터를 요청할 수 있게 되었습니다.

- **코드 재사용성 증가**: 개발자들이 서버와 클라이언트 양쪽에서 돌아가는 컴포넌트를 만들 수 있게 되었습니다. 이로 인해 코드 중복이 줄고 유지보수가 쉬워졌으며, 전체 코드베이스에서 로직을 더 쉽게 공유할 수 있게 되었습니다.

- **검색 엔진 최적화(SEO) 개선**: 서버에서 컴포넌트를 렌더링하면 검색 엔진과 대규모 언어 모델(LLM)이 콘텐츠를 더 잘 크롤링하고 색인화할 수 있어 SEO가 향상됩니다.

이 글에서는 [서버 컴포넌트](https://vercel.com/blog/understanding-react-server-components-57brjqQf27QFQaFFm27gZ9)나 [렌더링 전략](https://vercel.com/blog/how-to-choose-the-best-rendering-strategy-for-your-app)을 자세히 다루지는 않겠습니다. 대신 서버 컴포넌트가 왜 중요한지 이해하기 위해 React 렌더링이 어떻게 발전해 왔는지 간단히 살펴보겠습니다.

React는 처음에 클라이언트 측 렌더링(CSR)으로 시작했습니다. 이 방식은 사용자에게 아주 간단한 HTML만 제공했죠.

index.html

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="root"></div>
    <script src="/static/js/bundle.js"></script>
  </body>
</html>
```

여기서 연결된 스크립트에는 애플리케이션의 모든 것이 들어있습니다. React, 외부 라이브러리, 그리고 모든 애플리케이션 코드가 여기에 포함되어 있죠. 애플리케이션이 커질수록 이 번들 크기도 함께 커졌습니다. JavaScript를 다운로드하고 해석한 후에야 React가 빈 div에 DOM 요소를 넣습니다. 이 과정이 진행되는 동안 사용자는 빈 화면만 보게 됩니다.

초기 UI가 드디어 나타나도 페이지 내용은 여전히 비어 있습니다. 이 때문에 로딩 스켈레톤이 인기를 얻게 되었죠. 데이터를 가져온 후에야 UI가 다시 한 번 렌더링되면서 로딩 스켈레톤 대신 실제 내용을 보여줍니다.

![ae267af0851f7c598d28234b76f88f55_MD5](https://github.com/user-attachments/assets/3043b3f2-fce9-44cf-af77-f52bc7f95baf)


이후 React는 서버 측 렌더링(SSR)으로 발전했습니다. 첫 번째 렌더링을 서버로 옮긴 것이죠. 이제 사용자에게 내용이 있는 HTML을 제공하게 되어 초기 UI를 더 빨리 볼 수 있게 되었습니다. 하지만 여전히 실제 내용을 보여주려면 데이터를 따로 가져와야 했습니다.

![1741c073e123439638819b9b3cfe3170_MD5](https://github.com/user-attachments/assets/cf2e5053-5e34-4250-b47e-afeb9642390d)


이후 React 프레임워크들이 등장하면서 사용자 경험을 더욱 개선했습니다. 정적 사이트 생성(SSG)과 같은 기능을 도입해 빌드 때 동적 데이터를 캐시하고 렌더링했죠. 또한 점진적 정적 재생성(ISR)을 통해 필요할 때마다 동적 데이터를 다시 캐시하고 렌더링할 수 있게 되었습니다.

그리고 이제 React 서버 컴포넌트(RSC)가 등장했습니다. React 자체적으로 처음으로 UI를 렌더링하고 사용자에게 보여주기 전에 데이터를 가져올 수 있게 된 것입니다.

page.jsx

```jsx
export default async function Page() {
  const res = await fetch("https://api.example.com/products");
  const products = res.json();

  return (
    <>
      <h1>Products</h1>
      {products.map((product) => (
        <div key={product.id}>
          <h2>{product.title}</h2>
          <p>{product.description}</p>
        </div>
      ))}
    </>
  );
}
```

이제 사용자에게 처음 보내는 HTML에 이미 실제 내용이 모두 들어 있습니다. 추가로 데이터를 가져오거나 다시 렌더링할 필요가 없어진 것이죠.

![d09575787d87d1c2fca548dc008b4c01_MD5](https://github.com/user-attachments/assets/52f402c7-dbe6-4fbb-b6be-eb0911f6b185)

서버 컴포넌트는 속도와 성능 면에서 큰 발전을 이뤄냈습니다. 덕분에 개발자와 사용자 모두 더 나은 경험을 할 수 있게 되었죠. [React 서버 컴포넌트](https://19.react.dev/reference/rsc/server-components#noun-labs-1201738-(2))에 대해 더 자세히 알아보세요.

*렌더링 다이어그램 아이디어를 제공해 주신 [Josh W. Comeau](https://www.joshwcomeau.com)님께 감사드립니다.*

## 새로운 지시문

React 19에서 직접 선보인 기능은 아니지만, 지시문(directives)은 React와 밀접한 관계가 있습니다. React 서버 컴포넌트가 등장하면서 번들러가 컴포넌트와 함수의 실행 환경을 구분해야 할 필요성이 생겼죠. 이를 위해 React 컴포넌트를 만들 때 알아두어야 할 두 가지 새로운 지시문이 있습니다.

1. **`'use client'`**: 이 지시문은 클라이언트에서만 실행할 코드를 가리킵니다. 서버 컴포넌트가 기본값이므로, 상호작용과 상태 관리를 위해 훅을 사용하는 클라이언트 컴포넌트에 `'use client'`를 추가해야 합니다.

2. **`'use server'`**: 이 지시문은 클라이언트 측 코드에서 호출할 수 있는 서버 측 함수를 나타냅니다. 서버 컴포넌트에는 `'use server'`를 추가할 필요가 없고, 오직 서버 액션(Server Actions)에만 추가하면 됩니다. 특정 코드를 반드시 서버에서만 실행하고 싶다면, [`server-only` npm 패키지](https://nextjs.org/docs/app/building-your-application/rendering/composition-patterns#keeping-server-only-code-out-of-the-client-environment)를 활용할 수 있습니다.

이런 지시문들을 사용하면 개발자가 코드의 실행 환경을 명확히 지정할 수 있습니다. 이는 성능 최적화와 보안 강화에 큰 도움이 됩니다. 클라이언트와 서버의 역할을 확실히 구분해 각 환경에 맞는 최적의 코드를 작성할 수 있게 되는 거죠.

[지시문에 대해 더 자세히 알아보기](https://19.react.dev/reference/rsc/directives#noun-labs-1201738-(2))

## 액션

React 19에서는 '액션'이라는 새로운 기능을 선보입니다. 이 기능은 기존의 이벤트 핸들러를 대체하며, React의 전환 기능과 동시성 기능을 더욱 효과적으로 활용할 수 있게 해줍니다.

액션의 가장 큰 장점은 클라이언트와 서버 양쪽에서 모두 사용할 수 있다는 점입니다. 예를 들어, 폼 제출 시 사용하던 `onSubmit` 대신 클라이언트 액션을 활용할 수 있습니다.

이전에는 이벤트를 직접 분석해야 했지만, 이제는 액션이 `FormData`를 바로 받아 처리합니다. 이로 인해 코드가 더욱 간결해지고 이해하기 쉬워집니다.

다음은 app.tsx 파일에서 액션을 사용한 예제입니다:

```jsx
import { useState } from "react";

export default function TodoApp() {
  const [items, setItems] = useState([
    { text: "첫 번째 할 일" },
  ]);

  async function formAction(formData) {
    const newItem = formData.get("item");
    // 여기서 새 항목을 서버에 저장하는 POST 요청을 보낼 수 있습니다
    setItems((items) => [...items, { text: newItem }]);
  }

  return (
    <>
      <h1>할 일 목록</h1>
      <form action={formAction}>
        <input type="text" name="item" placeholder="할 일 추가..." />
        <button type="submit">추가</button>
      </form>
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item.text}</li>
        ))}
      </ul>
    </>
  );
}
```

### 서버 액션

서버 액션은 한 단계 더 나아가 클라이언트 컴포넌트에서 서버의 비동기 함수를 직접 호출할 수 있게 해줍니다. 이 기능을 통해 파일 시스템에 접근하거나 데이터베이스를 직접 조작하는 등 서버의 기능을 더욱 효과적으로 활용할 수 있습니다. 결과적으로 UI를 위한 별도의 API 엔드포인트를 만들 필요가 없어져 개발 과정이 간소화됩니다.

액션을 정의할 때는 `'use server'` 지시어를 사용하며, 이를 통해 클라이언트 측 컴포넌트와 쉽게 연동할 수 있습니다.

클라이언트 컴포넌트에서 서버 액션을 사용하려면 다음과 같이 새 파일을 만들고 가져오면 됩니다:

actions.ts 파일:

```typescript
'use server'

export async function create() {
  // 여기서 데이터베이스에 정보를 추가합니다
}
```

todo-list.tsx 파일:

```jsx
"use client";

import { create } from "./actions";

export default function TodoList() {
  return (
    <>
      <h1>할 일 목록</h1>
      <form action={create}>
        <input type="text" name="item" placeholder="할 일 추가..." />
        <button type="submit">추가</button>
      </form>
    </>
  );
}
```

서버 액션에 대해 더 자세히 알고 싶다면 [공식 문서](https://19.react.dev/reference/rsc/server-actions)를 참조하세요. 이 문서에서 서버 액션의 고급 기능과 활용 방법에 대해 자세히 설명하고 있습니다.

## 새로운 훅 소개

React 19에서는 상태 관리, 상황 파악, 시각적 피드백을 더욱 쉽게 다룰 수 있는 세 가지 새로운 훅을 선보입니다. 이 훅들은 폼 작업에 특히 유용하지만, 버튼과 같은 다른 요소에도 활용할 수 있습니다.

### `useActionState` 훅으로 폼 관리 간소화하기

이 훅은 폼 상태와 제출 과정을 한 번에 관리할 수 있게 해줍니다. 액션을 활용해 폼 입력 데이터를 모으고, 유효성 검사와 오류 상태를 처리하기 때문에 복잡한 상태 관리 로직을 직접 짜지 않아도 돼요. 또한 `useActionState` 훅은 `pending` 상태를 제공해서 액션 실행 중에 로딩 표시를 할 수 있습니다.

```jsx
"use client";
import { useActionState } from "react";
import { createUser } from "./actions";

const initialState = {
  message: "",
};

export function Signup() {
  const [state, formAction, pending] = useActionState(createUser, initialState);
 
  return (
    <form action={formAction}>
      <label htmlFor="email">이메일</label>
      <input type="text" id="email" name="email" required />
      {/* ... */}
      {state?.message && <p aria-live="polite">{state.message}</p>}
      <button aria-disabled={pending} type="submit">
        {pending ? "제출 중..." : "가입하기"}
      </button>
    </form>
  );
}
```

[`useActionState`](https://19.react.dev/reference/react/useActionState#noun-labs-1201738-(2))에 대해 더 자세히 알아보고 싶다면 링크를 참고하세요.

### `useFormStatus` 훅으로 폼 상태 추적하기

이 훅은 가장 최근에 제출한 폼의 상태를 추적합니다. 주의할 점은 반드시 폼 안에 있는 컴포넌트에서 호출해야 한다는 거예요.

```jsx
import { useFormStatus } from "react-dom";
import action from "./actions";

function Submit() {
  const status = useFormStatus();
  return <button disabled={status.pending}>제출</button>;
}

export default function App() {
  return (
    <form action={action}>
      <Submit />
    </form>
  );
}
```

`useActionState`가 기본적으로 `pending` 상태를 제공하지만, `useFormStatus`는 다음과 같은 상황에서 특히 유용해요:

- 폼 상태가 필요 없을 때
- 여러 폼에서 공통으로 사용할 컴포넌트를 만들 때
- 한 페이지에 여러 개의 폼이 있을 때 - `useFormStatus`는 해당 폼의 상태 정보만 알려줍니다

[`useFormStatus`](https://19.react.dev/reference/react-dom/hooks/useFormStatus#noun-labs-1201738-(2))에 대해 더 자세히 알고 싶다면 이 링크를 확인해보세요.

### `useOptimistic` 훅으로 UI 즉시 업데이트하기

이 훅을 사용하면 서버 액션이 완료되기를 기다리지 않고도 UI를 바로 갱신할 수 있어요. 비동기 액션이 끝나면 서버에서 받은 최종 상태로 UI를 다시 업데이트합니다.

다음 예제를 보면 새 메시지를 스레드에 즉시 추가하면서 동시에 서버 액션으로 보내 저장하는 과정을 이해할 수 있어요.

```jsx
"use client";
import { useOptimistic } from "react";
import { send } from "./actions";

export function Thread({ messages }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, { message: newMessage }],
  );
 
  const formAction = async (formData) => {
    const message = formData.get("message");
    addOptimisticMessage(message);
    await send(message);
  };
 
  return (
    <div>
      {optimisticMessages.map((m, i) => (
        <div key={i}>{m.message}</div>
      ))}
      <form action={formAction}>
        <input type="text" name="message" />
        <button type="submit">보내기</button>
      </form>
    </div>
  );
}
```

[`useOptimistic`](https://19.react.dev/reference/react/useOptimistic#noun-labs-1201738-(2))에 대해 더 자세히 알아보고 싶다면 이 링크를 참고하세요.

## 새로운 API: use

React가 선보인 `use` 함수는 렌더링 중 Promise와 컨텍스트를 더욱 효과적으로 다룰 수 있게 해줍니다. 기존 React 훅들과 달리 `use` 함수는 반복문, 조건문, 심지어 함수 중간에서 반환할 때도 사용할 수 있어 매우 유연합니다. 오류가 발생하거나 데이터 로딩이 필요할 때는 가장 가까운 Suspense 경계가 이를 알아서 처리합니다.

장바구니 항목을 가져오는 동안 로딩 메시지를 보여주는 예제를 살펴보겠습니다.

```jsx
import { use } from "react";

function Cart({ cartPromise }) {
  // `use` 함수가 Promise 해결을 기다립니다
  const cart = use(cartPromise);
  return cart.map((item) => <p key={item.id}>{item.title}</p>);
}

function Page({ cartPromise }) {
  return (
    /*{ ... }*/
    // Cart에서 `use` 함수가 대기 중일 때 이 Suspense 경계가 작동합니다
    <Suspense fallback={<div>불러오는 중...</div>}>
      <Cart cartPromise={cartPromise} />
    </Suspense>
  );
}
```

이 방식을 활용하면 여러 컴포넌트의 데이터가 모두 준비된 후에야 한꺼번에 화면에 나타나도록 할 수 있습니다.

`use` 함수에 대해 더 자세히 알고 싶다면 [이 링크](https://19.react.dev/reference/react/use)를 확인해보세요.

## 리소스 미리 불러오기

React 19는 페이지 로딩 속도와 사용자 경험을 개선하기 위해 새로운 API들을 선보였습니다. 이 API들은 스크립트, 스타일시트, 폰트 등의 리소스를 미리 불러올 수 있게 해줍니다. 개발자들은 이를 통해 웹 애플리케이션의 성능을 더욱 세밀하게 조절할 수 있게 되었습니다.

### 새로운 API 살펴보기

React 19에서 제공하는 새로운 API들을 자세히 알아봅시다:

1. [`prefetchDNS`](https://19.react.dev/reference/react-dom/prefetchDNS): 앞으로 연결할 DNS 도메인의 IP 주소를 미리 가져옵니다. 이로써 실제 연결 시 DNS 조회 시간을 크게 줄일 수 있습니다.

2. [`preconnect`](https://19.react.dev/reference/react-dom/preconnect): 리소스를 요청할 서버와 미리 연결을 맺습니다. 정확한 리소스를 모르더라도 서버와의 연결을 미리 준비할 수 있어 유용합니다.

3. [`preload`](https://19.react.dev/reference/react-dom/preload): 곧 사용할 스타일시트, 폰트, 이미지, 외부 스크립트를 미리 불러옵니다. 이를 통해 필요한 순간에 리소스를 즉시 사용할 수 있습니다.

4. [`preloadModule`](https://19.react.dev/reference/react-dom/preloadModule): 사용할 ESM(ECMAScript Module)을 미리 가져옵니다. 모듈 시스템을 사용하는 애플리케이션의 로딩 속도를 높일 수 있습니다.

5. [`preinit`](https://19.react.dev/reference/react-dom/preinit): 외부 스크립트를 가져와 실행하거나, 스타일시트를 가져와 적용합니다. 중요한 리소스를 더 빠르게 로드하고 적용할 수 있습니다.

6. [`preinitModule`](https://19.react.dev/reference/react-dom/preinitModule): ESM 모듈을 가져와 실행합니다. 모듈 기반 애플리케이션의 시작 시간을 단축할 수 있습니다.

### 사용 예제

이런 API들을 실제로 어떻게 사용하는지 예제를 통해 알아봅시다:

```javascript
import { prefetchDNS, preconnect, preload, preinit } from "react-dom";

function MyComponent() {
  preinit("https://.../path/to/some/script.js", { as: "script" });
  preload("https://.../path/to/some/font.woff", { as: "font" });
  preload("https://.../path/to/some/stylesheet.css", { as: "style" });
  prefetchDNS("https://...");
  preconnect("https://...");
}
```

이 코드는 다음과 같은 HTML을 생성합니다:

```html
<html>
  <head>
    <link rel="prefetch-dns" href="https://..." />
    <link rel="preconnect" href="https://..." />
    <link rel="preload" as="font" href="https://.../path/to/some/font.woff" />
    <link
      rel="preload"
      as="style"
      href="https://.../path/to/some/stylesheet.css"
    />
    <script async="" src="https://.../path/to/some/script.js"></script>
  </head>
  <body>
    <!-- ... -->
  </body>
</html>
```

여기서 주목할 점은 링크와 스크립트의 순서입니다. React에서 사용한 순서가 아니라, 얼마나 빨리 로드해야 하는지에 따라 우선순위가 정해지고 정렬됩니다. 이렇게 하면 중요한 리소스를 더 빨리 불러올 수 있습니다.

### 프레임워크와의 관계

React 프레임워크들은 보통 이런 리소스 로딩을 알아서 처리합니다. 그래서 개발자가 직접 이 API들을 호출할 필요가 없을 수도 있습니다. 프레임워크가 제공하는 최적화 기능을 활용하면 더 쉽게 성능을 높일 수 있습니다.

리소스 미리 불러오기 API에 대해 더 자세히 알고 싶다면 [Resource Preloading APIs](https://react.dev/reference/react-dom#resource-preloading-apis) 문서를 참고하세요. 여기서는 각 API의 자세한 사용법과 언제 쓰면 좋은지 설명합니다.

## 기타 개선 사항

React 19에서는 다양한 흥미로운 기능이 새로 추가되었습니다. 이제 이 새로운 기능과 변경 사항들을 자세히 알아보겠습니다.

### props로 `ref` 사용하기

이제 `forwardRef`를 쓰지 않아도 됩니다. React 팀에서 이 변화에 쉽게 적응할 수 있도록 코드 변환 도구를 제공할 예정입니다.

```jsx
function CustomInput({ placeholder, ref }) {
  return <input placeholder={placeholder} ref={ref} />;
}

// ...
<CustomInput ref={ref} />;
```

### `ref` 콜백 함수

props로 `ref`를 전달하는 것 외에도, refs에서 정리(cleanup) 작업을 위한 콜백 함수를 반환할 수 있게 되었습니다. 컴포넌트가 화면에서 사라질 때 React가 이 정리 함수를 실행합니다.

```jsx
<input
  ref={(ref) => {
    // ref 생성
    // DOM에서 엘리먼트가 제거될 때
    // ref를 초기화하는 정리 함수를 반환합니다.
    return () => {
      // ref 정리
    };
  }}
/>;
```

### `Context`를 프로바이더로 사용하기

이제 `<Context.Provider>`를 쓰지 않아도 됩니다. 대신 `<Context>`를 직접 사용할 수 있습니다. React 팀에서 기존 프로바이더를 새로운 방식으로 바꿔주는 코드 변환 도구를 제공할 예정입니다.

```jsx
const ThemeContext = createContext("");

function App({ children }) {
  return <ThemeContext value="dark">{children}</ThemeContext>;
}
```

### `useDeferredValue`에 초기값 설정하기

`useDeferredValue`에 `initialValue` 옵션이 추가되었습니다. 이 옵션을 사용하면 `useDeferredValue`가 첫 번째 렌더링에서 해당 값을 사용하고, 백그라운드에서 리렌더링을 예약해 `deferredValue`를 반환합니다.

```jsx
function Search({ deferredValue }) {
  // 첫 번째 렌더링에서 값은 ''입니다.
  // 그 후 deferredValue로 리렌더링이 예약됩니다.
  const value = useDeferredValue(deferredValue, "");
 
  return <Results value={value} />;
}
```

### 문서 메타데이터 지원

React 19는 중첩된 컴포넌트에서도 title, link, meta 태그를 자동으로 끌어올려 렌더링합니다. 이제 이런 태그를 관리하기 위해 별도의 라이브러리를 쓰지 않아도 됩니다.

```jsx
function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta name="author" content="Jane Doe" />
      <link rel="author" href="https://x.com/janedoe" />
      <meta name="keywords" content={post.keywords} />
      <p>...</p>
    </article>
  );
}
```

### 스타일시트 지원

React 19에서는 `precedence` 속성으로 스타일시트 로딩 순서를 조절할 수 있습니다. 이를 통해 컴포넌트 가까이에 스타일시트를 쉽게 배치할 수 있고, React는 필요할 때만 이를 불러옵니다.

주의할 점은 다음과 같습니다:

- 앱의 여러 곳에서 같은 컴포넌트를 렌더링해도 React는 스타일시트를 중복 제거해 문서에 한 번만 포함시킵니다.
- 서버 사이드 렌더링 시, React는 스타일시트를 head에 넣습니다. 이렇게 하면 브라우저가 스타일시트를 불러올 때까지 화면을 그리지 않습니다.
- 스트리밍이 시작된 후 스타일시트를 발견하면, React는 해당 스타일시트에 의존하는 내용을 Suspense 경계를 통해 보여주기 전에 클라이언트에서 스타일시트를 `<head>`에 넣습니다.
- 클라이언트 사이드 렌더링 중에는 React가 새로 렌더링된 스타일시트를 다 불러올 때까지 기다린 후 렌더링을 마칩니다.

```jsx
function ComponentOne() {
  return (
    <Suspense fallback="loading...">
      <link rel="stylesheet" href="one" precedence="default" />
      <link rel="stylesheet" href="two" precedence="high" />
      <article>...</article>
    </Suspense>
  );
}

function ComponentTwo() {
  return (
    <div>
      <p>...</p>
      {/* "three" 스타일시트는 "one"과 "two" 사이에 들어갑니다 */}
      <link rel="stylesheet" href="three" precedence="default" />
    </div>
  );
}
```

### 비동기 스크립트 지원

이제 어떤 컴포넌트에서든 비동기 스크립트를 렌더링할 수 있습니다. 이를 통해 컴포넌트 가까이에 스크립트를 쉽게 배치할 수 있고, React는 필요할 때만 이를 불러옵니다.

주의할 점은 다음과 같습니다:

- 앱의 여러 곳에서 같은 컴포넌트를 렌더링해도 React는 스크립트를 중복 제거해 문서에 한 번만 포함시킵니다.
- 서버 사이드 렌더링 시, 비동기 스크립트는 head에 들어가지만, 스타일시트, 폰트, 이미지 프리로드와 같은 더 중요한 리소스보다는 낮은 우선순위로 처리됩니다.

```jsx
function Component() {
  return (
    <div>
      <script async={true} src="..." />
      // ...
    </div>
  );
}

function App() {
  return (
    <html>
      <body>
        <Component>
          // ...
        </Component> // DOM에서 스크립트를 중복해서 넣지 않습니다
      </body>
    </html>
  );
}
```

### 커스텀 엘리먼트 지원

커스텀 엘리먼트를 사용하면 개발자가 [웹 컴포넌트](https://developer.mozilla.org/en-US/docs/Web/API/Web_components) 명세의 일부로 자신만의 HTML 엘리먼트를 만들 수 있습니다. 이전 버전의 React에서는 커스텀 엘리먼트 사용이 어려웠습니다. React가 인식하지 못하는 props를 속성이 아닌 프로퍼티로 처리했기 때문입니다.

React 19는 커스텀 엘리먼트를 완전히 지원하며 [Custom Elements Everywhere](https://custom-elements-everywhere.com/)의 모든 테스트를 통과합니다.

### 더 나은 오류 보고

오류 처리가 개선되어 중복된 오류 메시지가 사라졌습니다.

![이전에는 React가 오류를 두 번 던졌습니다. 원래 오류에 대해 한 번, 그리고 자동 복구에 실패한 후 두 번째로 오류에 대한 정보와 함께 던졌습니다.](https://github.com/user-attachments/assets/6f23af68-89e6-4edb-a460-fe12c1484a20)

![React 19에서는 오류가 한 번만 표시됩니다.](https://github.com/user-attachments/assets/f73327bb-a74f-450d-a016-0abc76462c40)

하이드레이션 오류도 개선되어 여러 개의 오류 대신 단일 불일치 오류만 로그에 남깁니다. 오류 메시지에는 문제를 해결할 수 있는 방법에 대한 정보도 포함됩니다.

![React 18의 하이드레이션 오류 메시지 예시](https://github.com/user-attachments/assets/62f7acb0-c830-46e8-b5a5-079a81ea40ef)

![React 19에서 개선된 하이드레이션 오류 메시지 예시](https://github.com/user-attachments/assets/38489b7d-557c-4197-81c8-96a1f4cb24c6)

서드파티 스크립트와 브라우저 확장 프로그램을 사용할 때 발생하는 하이드레이션 오류도 개선되었습니다. 이전에는 서드파티 스크립트나 브라우저 확장 프로그램이 넣은 엘리먼트 때문에 불일치 오류가 발생했습니다. React 19에서는 head와 body에 예상치 못한 태그가 있어도 이를 무시하고 오류를 발생시키지 않습니다.

마지막으로, React 19는 기존의 `onRecoverableError` 외에 두 가지 새로운 루트 옵션을 추가해 오류가 발생하는 이유를 더 정확히 파악할 수 있게 했습니다.

- `onCaughtError`: React가 오류 경계에서 오류를 잡았을 때 실행됩니다.
- `onUncaughtError`: 오류가 발생했지만 오류 경계에서 잡지 못했을 때 실행됩니다.
- `onRecoverableError`: 오류가 발생했지만 자동으로 복구되었을 때 실행됩니다.



## 알림

이 글은 [What’s new in React 19](https://vercel.com/blog/whats-new-in-react-19) 글을 한국어로 편역한 내용을 담고 있습니다.