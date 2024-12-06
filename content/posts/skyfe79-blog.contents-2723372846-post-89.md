---
title: "React v19 안정화 버전 출시"
date: 2024-12-06T15:53:21Z
author: "skyfe79"
draft: false
tags: ["react","아마추어 번역"]
---

# React v19

2024년 12월 5일, React 팀

___

### 주목할 점

### React 19 안정화 버전 출시!

2024년 4월에 공개된 React 19 RC 이후 추가된 기능은 다음과 같다:

- **중단된 트리에 대한 사전 준비 기능**: [서스펜스 개선사항](https://react.dev/blog/2024/04/25/react-19-upgrade-guide#improvements-to-suspense)에서 자세히 확인할 수 있다.
- **React DOM 정적 API**: [새로운 React DOM 정적 API](https://react.dev/blog/2024/12/05/react-19#new-react-dom-static-apis)에서 자세히 확인할 수 있다.

*이 글의 날짜는 안정화 버전 출시일을 반영하여 업데이트되었다.*

React v19를 이제 npm에서 사용할 수 있다!

[React 19 업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)에서 애플리케이션을 React 19로 업그레이드하는 단계별 지침을 이미 공유했다. 이 글에서는 React 19의 새로운 기능들을 개괄적으로 살펴보고, 이를 어떻게 도입할 수 있는지 설명한다.

- [React 19의 새로운 기능](https://react.dev/blog/2024/12/05/react-19#whats-new-in-react-19)
- [React 19의 개선사항](https://react.dev/blog/2024/12/05/react-19#improvements-in-react-19)
- [업그레이드 방법](https://react.dev/blog/2024/12/05/react-19#how-to-upgrade)

주요 변경사항 목록은 [업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)에서 확인할 수 있다.

___

# React 19의 새로운 기능

### 액션(Actions)

React 앱에서 자주 발생하는 패턴은 데이터를 변경하고 그에 따라 상태를 업데이트하는 것이다. 예를 들어, 사용자가 이름을 변경하기 위해 폼을 제출할 때 API 요청을 보내고 응답을 처리한다. 이전에는 대기 상태, 오류, 낙관적 업데이트, 연속적인 요청 등을 수동으로 처리해야 했다.

예를 들어, `useState`를 사용하여 대기 상태와 오류를 처리하는 방식은 다음과 같다:

```javascript
// 액션 이전의 방식
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    }
    
    redirect("/path");
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

React 19에서는 트랜지션에서 비동기 함수를 사용하여 대기 상태, 오류, 폼, 낙관적 업데이트를 자동으로 처리할 수 있다.

예를 들어, `useTransition`을 사용하여 대기 상태를 자동으로 처리할 수 있다:

```javascript
// 액션을 사용한 대기 상태 처리
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

비동기 트랜지션은 즉시 `isPending` 상태를 true로 설정하고, 비동기 요청을 실행한 후, 모든 트랜지션이 완료되면 `isPending`을 false로 전환한다. 이를 통해 데이터가 변경되는 동안에도 현재 UI를 반응적이고 상호작용 가능한 상태로 유지할 수 있다.

### 참고

#### 관례상 비동기 트랜지션을 사용하는 함수를 "액션"이라고 부른다.

액션은 데이터 제출을 자동으로 관리한다:

- **대기 상태**: 액션은 요청 시작 시점부터 대기 상태를 제공하며, 최종 상태 업데이트가 완료되면 자동으로 초기화한다.
- **낙관적 업데이트**: 액션은 새로운 [`useOptimistic`](https://react.dev/reference/react/useOptimistic) 훅을 지원하여 요청이 처리되는 동안 사용자에게 즉각적인 피드백을 제공할 수 있다.
- **오류 처리**: 액션은 오류 처리 기능을 제공하여 요청 실패 시 에러 바운더리를 표시하고, 낙관적 업데이트를 자동으로 원래 값으로 되돌린다.
- **폼**: `<form>` 엘리먼트는 이제 `action`과 `formAction` props에 함수를 전달할 수 있다. `action` props에 함수를 전달하면 기본적으로 액션을 사용하며, 제출 후 폼을 자동으로 초기화한다.

React 19는 액션을 기반으로 낙관적 업데이트를 관리하는 [`useOptimistic`](https://react.dev/reference/react/useOptimistic)와 액션의 일반적인 사용 사례를 처리하는 새로운 훅 [`React.useActionState`](https://react.dev/reference/react/useActionState)를 도입한다. `react-dom`에서는 폼을 자동으로 관리하는 [`<form>` 액션](https://react.dev/reference/react-dom/components/form)과 폼에서 액션의 일반적인 사용 사례를 지원하는 [`useFormStatus`](https://react.dev/reference/react-dom/hooks/useFormStatus)를 추가한다.

React 19에서는 위의 예제를 다음과 같이 간단하게 작성할 수 있다:

```javascript
// <form> 액션과 useActionState 사용
function ChangeName({ name, setName }) {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const error = await updateName(formData.get("name"));
      if (error) {
        return error;
      }
      redirect("/path");
      return null;
    },
    null,
  );

  return (
    <form action={submitAction}>
      <input type="text" name="name" />
      <button type="submit" disabled={isPending}>Update</button>
      {error && <p>{error}</p>}
    </form>
  );
}
```

다음 섹션에서는 React 19의 새로운 액션 기능을 하나씩 자세히 살펴본다.

### 새로운 훅: `useActionState`

액션의 일반적인 사용 사례를 더 쉽게 만들기 위해 새로운 훅 `useActionState`를 추가했다:

```javascript
const [error, submitAction, isPending] = useActionState(
  async (previousState, newName) => {
    const error = await updateName(newName);
    if (error) {
      // 액션의 결과를 반환할 수 있다.
      // 여기서는 오류만 반환한다.
      return error;
    }
    // 성공 처리
    return null;
  },
  null,
);
```

`useActionState`는 함수(액션)를 받고 호출할 수 있는 래핑된 액션을 반환한다. 액션은 합성이 가능하기 때문에 이러한 방식이 작동한다. 래핑된 액션이 호출되면, `useActionState`는 액션의 마지막 결과를 `data`로, 액션의 대기 상태를 `pending`으로 반환한다.

### 참고

`React.useActionState`는 이전 Canary 릴리스에서 `ReactDOM.useFormState`로 불렸지만, 이름이 변경되었고 `useFormState`는 더 이상 사용되지 않는다.

자세한 내용은 [#28491](https://github.com/facebook/react/pull/28491)을 참조한다.

더 자세한 정보는 [`useActionState`](https://react.dev/reference/react/useActionState) 문서를 참조한다.

### React DOM: `<form>` 액션

액션은 또한 React 19의 새로운 `<form>` 기능과 통합되어 있다. `react-dom`에서는 `<form>`, `<input>`, `<button>` 엘리먼트의 `action`과 `formAction` props에 함수를 전달하여 액션으로 폼을 자동으로 제출하는 기능을 추가했다:

```javascript
<form action={actionFunction}>
```

`<form>` 액션이 성공하면 React는 비제어 컴포넌트의 폼을 자동으로 초기화한다. 수동으로 `<form>`을 초기화해야 하는 경우 새로운 React DOM API인 `requestFormReset`을 호출할 수 있다.

자세한 내용은 `react-dom`의 [`<form>`](https://react.dev/reference/react-dom/components/form), [`<input>`](https://react.dev/reference/react-dom/components/input), `<button>` 문서를 참조한다.

### React DOM: 새로운 훅: `useFormStatus`

디자인 시스템에서는 props를 컴포넌트까지 전달하지 않고도 포함된 `<form>`에 대한 정보에 접근해야 하는 디자인 컴포넌트를 작성하는 것이 일반적이다. 이는 Context를 통해 수행할 수 있지만, 일반적인 사용 사례를 더 쉽게 만들기 위해 새로운 훅 `useFormStatus`를 추가했다:

```javascript
import {useFormStatus} from 'react-dom';

function DesignButton() {
  const {pending} = useFormStatus();
  return <button type="submit" disabled={pending} />
}
```

`useFormStatus`는 마치 폼이 Context 프로바이더인 것처럼 상위 `<form>`의 상태를 읽는다.

자세한 내용은 `react-dom`의 [`useFormStatus`](https://react.dev/reference/react-dom/hooks/useFormStatus) 문서를 참조한다.

### 새로운 훅: `useOptimistic`

데이터 변경을 수행할 때 또 다른 일반적인 UI 패턴은 비동기 요청이 진행되는 동안 최종 상태를 낙관적으로 보여주는 것이다. React 19에서는 이를 더 쉽게 만들기 위해 `useOptimistic`이라는 새로운 훅을 추가했다:

```javascript
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>이름: {optimisticName}</p>
      <p>
        <label>이름 변경:</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

`useOptimistic` 훅은 `updateName` 요청이 진행되는 동안 즉시 `optimisticName`을 렌더링한다. 업데이트가 완료되거나 오류가 발생하면 React는 자동으로 `currentName` 값으로 되돌아간다.

자세한 내용은 [`useOptimistic`](https://react.dev/reference/react/useOptimistic) 문서를 참조한다.

### 새로운 API: `use`

React 19에서는 렌더링 중에 리소스를 읽는 새로운 API인 `use`를 도입한다.

예를 들어, `use`로 Promise를 읽을 수 있으며, React는 Promise가 해결될 때까지 일시 중단된다:

```javascript
import {use} from 'react';

function Comments({commentsPromise}) {
  // `use`는 Promise가 해결될 때까지 일시 중단된다.
  const comments = use(commentsPromise);
  return comments.map(comment => <p key={comment.id}>{comment}</p>);
}

function Page({commentsPromise}) {
  // Comments에서 `use`가 일시 중단될 때
  // 이 Suspense 경계가 표시된다.
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <Comments commentsPromise={commentsPromise} />
    </Suspense>
  )
}
```

### 참고

#### `use`는 렌더링 중에 생성된 Promise를 지원하지 않는다.

렌더링 중에 생성된 Promise를 `use`에 전달하려고 하면 React는 경고를 표시한다:

콘솔
캐시되지 않은 Promise에 의해 컴포넌트가 일시 중단되었다. 클라이언트 컴포넌트나 훅 내에서 Promise를 생성하는 것은 아직 지원되지 않으며, Suspense 호환 라이브러리나 프레임워크를 통해서만 가능하다.

이를 해결하려면 Promise를 캐싱하는 Suspense 지원 라이브러리나 프레임워크의 Promise를 전달해야 한다. 향후에는 렌더링 중에 Promise를 캐싱하기 쉽게 만드는 기능을 제공할 계획이다.

또한 `use`로 Context를 읽을 수 있어서 조건부로 Context를 읽는 것이 가능하다. 예를 들어 조기 반환 이후에도 Context를 읽을 수 있다:

```javascript
import {use} from 'react';
import ThemeContext from './ThemeContext'

function Heading({children}) {
  if (children == null) {
    return null;
  }
  
  // useContext와 달리 조기 반환 이후에도
  // Context를 읽을 수 있다
  const theme = use(ThemeContext);
  return (
    <h1 style={{color: theme.color}}>
      {children}
    </h1>
  );
}
```

`use` API는 훅과 마찬가지로 렌더링 중에만 호출할 수 있다. 하지만 훅과 달리 조건부로 호출할 수 있다는 점이 다르다. 앞으로는 `use`를 통해 렌더링 중에 리소스를 소비하는 더 많은 방법을 지원할 계획이다.

자세한 내용은 [`use`](https://react.dev/reference/react/use) 문서를 참조한다.

# React DOM의 새로운 정적 API

`react-dom/static`에 정적 사이트 생성을 위한 두 가지 새로운 API를 추가했다:

- [`prerender`](https://react.dev/reference/react-dom/static/prerender)
- [`prerenderToNodeStream`](https://react.dev/reference/react-dom/static/prerenderToNodeStream)

이 새로운 API들은 정적 HTML 생성 시 데이터 로딩이 완료될 때까지 대기하여 `renderToString`의 기능을 개선했다. Node.js 스트림과 웹 스트림과 같은 스트리밍 환경에서 작동하도록 설계되었다. 예를 들어, 웹 스트림 환경에서는 `prerender`를 사용하여 React 트리를 정적 HTML로 미리 렌더링할 수 있다:

```javascript
import { prerender } from 'react-dom/static';

async function handler(request) {
  const {prelude} = await prerender(<App />, {
    bootstrapScripts: ['/main.js']
  });
  
  return new Response(prelude, {
    headers: { 'content-type': 'text/html' },
  });
}
```

프리렌더 API는 정적 HTML 스트림을 반환하기 전에 모든 데이터가 로드될 때까지 대기한다. 스트림은 문자열로 변환하거나 스트리밍 응답으로 전송할 수 있다. 하지만 기존 [React DOM 서버 렌더링 API](https://react.dev/reference/react-dom/server)에서 지원하는 것처럼 콘텐츠를 로드되는 대로 스트리밍하는 기능은 지원하지 않는다.

자세한 내용은 [React DOM 정적 API](https://react.dev/reference/react-dom/static) 문서를 참고하면 된다.

# React 서버 컴포넌트

## 서버 컴포넌트

서버 컴포넌트는 번들링 이전에 클라이언트 애플리케이션이나 SSR 서버와 분리된 환경에서 미리 컴포넌트를 렌더링할 수 있는 새로운 기능이다. React 서버 컴포넌트에서 말하는 "서버"가 바로 이 분리된 환경을 의미한다. 서버 컴포넌트는 CI 서버에서 빌드 시점에 한 번 실행하거나, 웹 서버를 통해 각 요청마다 실행할 수 있다.

React 19는 Canary 채널에서 제공하던 모든 서버 컴포넌트 기능을 포함한다. 이제 서버 컴포넌트를 제공하는 라이브러리는 React 19를 피어 디펜던시로 지정하고, [전체 스택 React 아키텍처](https://react.dev/learn/start-a-new-react-project#which-features-make-up-the-react-teams-full-stack-architecture-vision)를 지원하는 프레임워크에서 사용할 수 있도록 `react-server` [내보내기 조건](https://github.com/reactjs/rfcs/blob/main/text/0227-server-module-conventions.md#react-server-conditional-exports)을 활용할 수 있다.

### 참고사항

#### 서버 컴포넌트 지원 기능을 어떻게 구현할까?

React 19의 서버 컴포넌트는 안정적이며 메이저 버전 간에 호환성이 보장된다. 하지만 번들러나 프레임워크에서 React 서버 컴포넌트를 구현하는 데 사용되는 기본 API는 semver를 따르지 않으며, React 19.x의 마이너 버전 간에 변경될 수 있다.

번들러나 프레임워크에서 React 서버 컴포넌트를 지원하려면 특정 React 버전을 고정하거나 Canary 릴리스를 사용하는 것이 좋다. 앞으로도 번들러와 프레임워크 개발자들과 협력하여 React 서버 컴포넌트 구현에 사용되는 API를 안정화할 계획이다.

자세한 내용은 [React 서버 컴포넌트 문서](https://react.dev/reference/rsc/server-components)를 참고하면 된다.

## 서버 액션

서버 액션을 사용하면 클라이언트 컴포넌트에서 서버에서 실행되는 비동기 함수를 호출할 수 있다.

서버 액션을 `"use server"` 지시어로 정의하면, 프레임워크가 자동으로 서버 함수에 대한 참조를 생성하여 클라이언트 컴포넌트에 전달한다. 클라이언트에서 이 함수를 호출하면 React는 서버에 요청을 보내 함수를 실행하고 그 결과를 반환한다.

### 참고사항

#### 서버 컴포넌트용 지시어는 없다

흔히 하는 오해 중 하나는 서버 컴포넌트를 `"use server"`로 표시한다고 생각하는 것이다. 하지만 서버 컴포넌트를 위한 별도의 지시어는 없다. `"use server"` 지시어는 오직 서버 액션을 위해 사용된다.

자세한 내용은 [지시어 문서](https://react.dev/reference/rsc/directives)를 참고하면 된다.

서버 액션은 서버 컴포넌트에서 생성하여 클라이언트 컴포넌트에 props로 전달하거나, 클라이언트 컴포넌트에서 직접 임포트하여 사용할 수 있다.

더 자세한 내용은 [React 서버 액션 문서](https://react.dev/reference/rsc/server-actions)를 참고하면 된다.

# React 19의 개선사항

## 프롭으로서의 `ref`

React 19부터 함수형 컴포넌트에서 `ref`를 프롭으로 직접 사용할 수 있다:

```javascript
function MyInput({placeholder, ref}) {
  return <input placeholder={placeholder} ref={ref} />
}

// ...
<MyInput ref={ref} />
```

새로운 함수형 컴포넌트는 더 이상 `forwardRef`가 필요하지 않다. 기존 컴포넌트를 새로운 `ref` 프롭 방식으로 자동 변환하는 코드모드를 제공할 예정이다. 향후 버전에서는 `forwardRef`를 단계적으로 제거할 계획이다.

### 참고

클래스형 컴포넌트에 전달되는 `ref`는 컴포넌트 인스턴스를 참조하므로 프롭으로 전달되지 않는다.

## 하이드레이션 오류의 차이점 표시

`react-dom`의 하이드레이션 오류 보고 기능도 개선했다. 예를 들어, 개발 환경에서 불일치에 대한 정보 없이 여러 오류를 표시하는 대신:

콘솔
```
Warning: Text content did not match. Server: "Server" Client: "Client" at span at App

Warning: An error occurred during hydration. The server HTML was replaced with client content in <div>.

Warning: Text content did not match. Server: "Server" Client: "Client" at span at App

Warning: An error occurred during hydration. The server HTML was replaced with client content in <div>.

Uncaught Error: Text content does not match server-rendered HTML. at checkForUnmatchedText …
```

이제는 불일치 내용을 한 번에 보여주는 단일 메시지로 표시한다:

콘솔
```
Uncaught Error: 서버에서 렌더링한 HTML이 클라이언트와 일치하지 않아 하이드레이션에 실패했다. 
결과적으로 이 트리는 클라이언트에서 다시 생성된다. 
이는 SSR된 클라이언트 컴포넌트가 다음과 같은 요소를 사용했을 때 발생할 수 있다:
- 서버/클라이언트 분기 `if (typeof window !== 'undefined')`
- `Date.now()` 또는 `Math.random()`과 같이 호출할 때마다 변하는 변수 입력
- 서버와 일치하지 않는 사용자 로케일의 날짜 형식
- HTML과 함께 스냅샷을 전송하지 않은 외부 변경 데이터
- 잘못된 HTML 태그 중첩

React가 로드되기 전에 HTML을 조작하는 브라우저 확장 프로그램이 설치된 경우에도 발생할 수 있다.
https://react.dev/link/hydration-mismatch

<App>
  <span>
    + Client
    - Server
    at throwOnHydrationMismatch …
```

## 프로바이더로서의 `<Context>`

React 19에서는 `<Context.Provider>` 대신 `<Context>`를 프로바이더로 렌더링할 수 있다:

```javascript
const ThemeContext = createContext('');

function App({children}) {
  return (
    <ThemeContext value="dark">
      {children}
    </ThemeContext>
  );
}
```

새로운 Context 프로바이더는 `<Context>`를 사용할 수 있으며, 기존 프로바이더를 변환하는 코드모드도 제공할 예정이다. 향후 버전에서는 `<Context.Provider>`를 단계적으로 제거할 계획이다.

## ref의 정리 함수

이제 `ref` 콜백에서 정리 함수를 반환할 수 있다:

```javascript
<input
  ref={(ref) => {
    // ref 생성
    // 새로운 기능: DOM에서 엘리먼트가 제거될 때
    // ref를 초기화하는 정리 함수 반환
    return () => {
      // ref 정리
    };
  }}
/>
```

컴포넌트가 언마운트될 때 React는 `ref` 콜백에서 반환된 정리 함수를 호출한다. 이는 DOM ref, 클래스 컴포넌트에 대한 ref, `useImperativeHandle`에서 모두 작동한다.

### 참고

이전에는 컴포넌트 언마운트 시 React가 `ref` 함수를 `null`로 호출했다. `ref`가 정리 함수를 반환하면 이제 React는 이 단계를 건너뛴다.

향후 버전에서는 컴포넌트 언마운트 시 ref를 `null`로 호출하는 기능을 단계적으로 제거할 예정이다.

ref 정리 함수가 도입됨에 따라 이제 TypeScript는 `ref` 콜백에서 다른 값을 반환하는 것을 거부한다. 일반적으로 암시적 반환을 사용하지 않도록 수정하면 된다. 예를 들면:

```javascript
- <div ref={current => (instance = current)} />
+ <div ref={current => {instance = current}} />
```

원래 코드는 `HTMLDivElement`의 인스턴스를 반환했고, TypeScript는 이것이 정리 함수여야 하는지 아니면 정리 함수를 반환하지 않으려는 것인지 알 수 없었다.

[`no-implicit-ref-callback-return`](https://github.com/eps1lon/types-react-codemod/#no-implicit-ref-callback-return)을 사용하여 이 패턴을 코드모드할 수 있다.

## `useDeferredValue`의 초기값

`useDeferredValue`에 `initialValue` 옵션을 추가했다:

```javascript
function Search({deferredValue}) {
  // 초기 렌더링에서 값은 ''이다.
  // 그런 다음 deferredValue로 리렌더링이 예약된다.
  const value = useDeferredValue(deferredValue, '');
  
  return (
    <Results query={value} />
  );
}
```

initialValue를 제공하면 `useDeferredValue`는 컴포넌트의 초기 렌더링에서 이를 `value`로 반환하고, 백그라운드에서 deferredValue를 반환하는 리렌더링을 예약한다.

자세한 내용은 [`useDeferredValue`](https://react.dev/reference/react/useDeferredValue)를 참조한다.

## 문서 메타데이터 지원

HTML에서 `<title>`, `<link>`, `<meta>`와 같은 문서 메타데이터 태그는 문서의 `<head>` 섹션에 배치해야 한다. React에서는 메타데이터를 결정하는 컴포넌트가 `<head>`를 렌더링하는 위치와 멀리 떨어져 있거나 React가 `<head>`를 전혀 렌더링하지 않을 수 있다. 이전에는 이러한 엘리먼트를 effect에서 수동으로 삽입하거나 [`react-helmet`](https://github.com/nfl/react-helmet)과 같은 라이브러리를 사용해야 했고, React 애플리케이션을 서버 렌더링할 때 신중한 처리가 필요했다.

React 19에서는 컴포넌트에서 문서 메타데이터 태그를 네이티브로 렌더링할 수 있도록 지원한다:

```javascript
function BlogPost({post}) {
  return (
    <article>
      <h1>{post.title}</h1>
      <title>{post.title}</title>
      <meta name="author" content="Josh" />
      <link rel="author" href="https://twitter.com/joshcstory/" />
      <meta name="keywords" content={post.keywords} />
      <p>
        Eee equals em-see-squared...
      </p>
    </article>
  );
}
```

React가 이 컴포넌트를 렌더링할 때 `<title>`, `<link>`, `<meta>` 태그를 감지하고 자동으로 문서의 `<head>` 섹션으로 올린다. 이러한 메타데이터 태그를 네이티브로 지원함으로써 클라이언트 전용 앱, 스트리밍 SSR, 서버 컴포넌트에서도 원활하게 작동하도록 보장할 수 있다.

### 참고

#### 메타데이터 라이브러리가 여전히 필요할 수 있다

간단한 사용 사례의 경우 태그로 문서 메타데이터를 렌더링하는 것이 적절할 수 있지만, 라이브러리는 현재 라우트를 기반으로 일반 메타데이터를 특정 메타데이터로 재정의하는 것과 같은 더 강력한 기능을 제공할 수 있다. 이러한 기능은 [`react-helmet`](https://github.com/nfl/react-helmet)과 같은 프레임워크와 라이브러리가 메타데이터 태그를 대체하는 것이 아니라 지원하기 쉽게 만든다.

자세한 내용은 [`<title>`](https://react.dev/reference/react-dom/components/title), [`<link>`](https://react.dev/reference/react-dom/components/link), [`<meta>`](https://react.dev/reference/react-dom/components/meta) 문서를 참조한다.

# 스타일시트 지원

스타일시트는 외부 링크(`<link rel="stylesheet" href="...">`)와 인라인(`<style>...</style>`) 방식 모두 스타일 우선순위 규칙으로 인해 DOM에서 정확한 위치 지정이 필요하다. 컴포넌트 내에서 합성 가능한 스타일시트 기능을 구현하는 것은 어렵다. 이로 인해 개발자는 주로 컴포넌트와 멀리 떨어진 곳에 모든 스타일을 로드하거나, 이러한 복잡성을 캡슐화하는 스타일 라이브러리를 사용한다.

React 19는 이러한 복잡성을 해결하고 클라이언트의 동시 렌더링과 서버의 스트리밍 렌더링에 더 깊이 통합된 스타일시트 지원 기능을 제공한다. 스타일시트의 `precedence`를 지정하면 React는 DOM에서 스타일시트의 삽입 순서를 관리하고, 외부 스타일시트의 경우 해당 스타일 규칙에 의존하는 콘텐츠를 표시하기 전에 스타일시트가 로드되도록 보장한다.

```jsx
function ComponentOne() {
  return (
    <Suspense fallback="loading...">
      <link rel="stylesheet" href="foo" precedence="default" />
      <link rel="stylesheet" href="bar" precedence="high" />
      <article class="foo-class bar-class">
        {...}
      </article>
    </Suspense>
  )
}

function ComponentTwo() {
  return (
    <div>
      <p>{...}</p>
      <link rel="stylesheet" href="baz" precedence="default" /> {/* foo와 bar 사이에 삽입된다 */}
    </div>
  )
}
```

서버 사이드 렌더링 중에 React는 스타일시트를 `<head>`에 포함시켜 브라우저가 스타일시트를 로드할 때까지 페인팅하지 않도록 한다. 스트리밍을 시작한 후 늦게 발견된 스타일시트의 경우, React는 해당 스타일시트에 의존하는 Suspense 경계의 콘텐츠를 표시하기 전에 클라이언트에서 스타일시트를 `<head>`에 삽입한다.

클라이언트 사이드 렌더링 중에는 React가 새로 렌더링된 스타일시트가 로드될 때까지 기다린 후 렌더링을 완료한다. 애플리케이션의 여러 위치에서 이 컴포넌트를 렌더링하더라도 React는 문서에 스타일시트를 한 번만 포함한다:

```jsx
function App() {
  return <>
    <ComponentOne />
    ...
    <ComponentOne /> // DOM에 중복된 스타일시트 링크를 생성하지 않는다
  </>
}
```

수동으로 스타일시트를 로드하는 데 익숙한 개발자는 이제 스타일시트를 의존하는 컴포넌트 옆에 배치할 수 있다. 이를 통해 더 나은 지역적 추론이 가능하며 실제로 필요한 스타일시트만 로드하도록 보장하기가 더 쉬워진다.

스타일 라이브러리와 번들러의 스타일 통합 기능도 이 새로운 기능을 채택할 수 있다. 따라서 직접 스타일시트를 렌더링하지 않더라도 도구가 이 기능을 사용하도록 업그레이드되면서 혜택을 받을 수 있다.

자세한 내용은 [`<link>`](https://react.dev/reference/react-dom/components/link)와 [`<style>`](https://react.dev/reference/react-dom/components/style)의 문서를 참조한다.

# 비동기 스크립트 지원

HTML에서 일반 스크립트(`<script src="...">`)와 지연 스크립트(`<script defer="" src="...">`)는 문서 순서대로 로드되므로 컴포넌트 트리 깊은 곳에서 이러한 스크립트를 렌더링하기가 어렵다. 반면 비동기 스크립트(`<script async="" src="...">`)는 임의의 순서로 로드된다.

React 19는 비동기 스크립트에 대한 더 나은 지원을 제공한다. 스크립트 인스턴스의 위치 이동과 중복 제거를 관리할 필요 없이 컴포넌트 트리의 어느 위치에서나 렌더링할 수 있다.

```jsx
function MyComponent() {
  return (
    <div>
      <script async={true} src="..." />
      Hello World
    </div>
  )
}

function App() {
  <html>
    <body>
      <MyComponent>
      ...
      <MyComponent> // DOM에 중복된 스크립트를 생성하지 않는다
    </body>
  </html>
}
```

모든 렌더링 환경에서 비동기 스크립트는 중복이 제거되어 여러 컴포넌트에서 렌더링하더라도 스크립트를 한 번만 로드하고 실행한다.

서버 사이드 렌더링에서는 비동기 스크립트가 `<head>`에 포함되며, 스타일시트, 폰트, 이미지 프리로드와 같은 페인팅을 차단하는 더 중요한 리소스보다 낮은 우선순위를 갖는다.

자세한 내용은 [`<script>`](https://react.dev/reference/react-dom/components/script)의 문서를 참조한다.

# 리소스 프리로딩 지원

초기 문서 로드와 클라이언트 사이드 업데이트 중에 브라우저에게 곧 필요할 리소스를 최대한 빨리 알려주면 페이지 성능에 큰 영향을 미칠 수 있다.

React 19는 브라우저 리소스를 로드하고 프리로드하기 위한 여러 새로운 API를 제공한다. 이를 통해 비효율적인 리소스 로딩으로 인해 지연되지 않는 훌륭한 사용자 경험을 구축하기가 더욱 쉬워졌다.

```jsx
import { prefetchDNS, preconnect, preload, preinit } from 'react-dom'

function MyComponent() {
  preinit('https://.../path/to/some/script.js', {as: 'script' }) // 이 스크립트를 미리 로드하고 실행한다
  preload('https://.../path/to/font.woff', { as: 'font' }) // 이 폰트를 프리로드한다
  preload('https://.../path/to/stylesheet.css', { as: 'style' }) // 이 스타일시트를 프리로드한다
  prefetchDNS('https://...') // 이 호스트에서 실제로 요청하지 않을 수 있을 때
  preconnect('https://...') // 무언가를 요청할 것은 확실하지만 무엇인지 모를 때
}
```

```html
<!-- 위 코드는 다음과 같은 DOM/HTML을 생성한다 -->
<html>
  <head>
    <!-- 링크/스크립트는 호출 순서가 아닌 초기 로딩에 대한 유용성에 따라 우선순위가 정해진다 -->
    <link rel="prefetch-dns" href="https://...">
    <link rel="preconnect" href="https://...">
    <link rel="preload" as="font" href="https://.../path/to/font.woff">
    <link rel="preload" as="style" href="https://.../path/to/stylesheet.css">
    <script async="" src="https://.../path/to/some/script.js"></script>
  </head>
  <body>
    ...
  </body>
</html>
```

이러한 API는 스타일시트 로딩에서 폰트 발견을 분리하여 초기 페이지 로드를 최적화하는 데 사용할 수 있다. 또한 예상되는 네비게이션에 사용되는 리소스 목록을 미리 가져오고 클릭하거나 심지어 호버할 때 해당 리소스를 미리 로드하여 클라이언트 업데이트를 더 빠르게 만들 수 있다.

자세한 내용은 [리소스 프리로딩 API](https://react.dev/reference/react-dom#resource-preloading-apis)를 참조한다.

# 서드파티 스크립트와 확장 프로그램 호환성

서드파티 스크립트와 브라우저 확장 프로그램을 고려하여 하이드레이션을 개선했다.

하이드레이션 중에 클라이언트에서 렌더링되는 엘리먼트가 서버의 HTML에서 발견된 엘리먼트와 일치하지 않으면 React는 콘텐츠를 수정하기 위해 클라이언트 리렌더링을 강제한다. 이전에는 서드파티 스크립트나 브라우저 확장 프로그램이 엘리먼트를 삽입하면 불일치 오류가 발생하고 클라이언트 렌더링이 트리거되었다.

React 19에서는 `<head>`와 `<body>`에서 예상치 못한 태그는 건너뛰어 불일치 오류를 방지한다. 관련 없는 하이드레이션 불일치로 인해 React가 전체 문서를 다시 렌더링해야 하는 경우에도 서드파티 스크립트와 브라우저 확장 프로그램이 삽입한 스타일시트는 그대로 유지한다.

# 더 나은 오류 보고

React 19에서는 오류 처리를 개선하여 중복을 제거하고 포착된 오류와 포착되지 않은 오류를 처리하는 옵션을 제공한다. 예를 들어, 오류 경계가 포착한 렌더링 중 오류의 경우, 이전에는 React가 오류를 두 번 발생시켰다(원래 오류에 대해 한 번, 자동 복구 실패 후 다시 한 번). 그리고 오류가 발생한 위치에 대한 정보와 함께 `console.error`를 호출했다.

이로 인해 포착된 각 오류마다 세 개의 오류가 발생했다:

```
콘솔

Uncaught Error: hit at Throws at renderWithHooks …

Uncaught Error: hit <-- 중복 at Throws at renderWithHooks …

위 오류는 Throws 컴포넌트에서 발생했다: at Throws at ErrorBoundary at App React는 제공한 오류 경계인 ErrorBoundary를 사용하여 이 컴포넌트 트리를 처음부터 다시 생성하려고 시도할 것이다.
```

React 19에서는 모든 오류 정보가 포함된 단일 오류를 기록한다:

```
콘솔

Error: hit at Throws at renderWithHooks … 위 오류는 Throws 컴포넌트에서 발생했다: at Throws at ErrorBoundary at App React는 제공한 오류 경계인 ErrorBoundary를 사용하여 이 컴포넌트 트리를 처음부터 다시 생성하려고 시도할 것이다. at ErrorBoundary at App
```

추가로 `onRecoverableError`를 보완하는 두 가지 새로운 루트 옵션을 추가했다:

- `onCaughtError`: React가 오류 경계에서 오류를 포착할 때 호출된다.
- `onUncaughtError`: 오류가 발생하고 오류 경계에 의해 포착되지 않을 때 호출된다.
- `onRecoverableError`: 오류가 발생하고 자동으로 복구될 때 호출된다.

자세한 정보와 예제는 [`createRoot`](https://react.dev/reference/react-dom/client/createRoot)와 [`hydrateRoot`](https://react.dev/reference/react-dom/client/hydrateRoot)의 문서를 참조한다.

# 커스텀 엘리먼트 지원

React 19는 커스텀 엘리먼트에 대한 완전한 지원을 추가하여 [Custom Elements Everywhere](https://custom-elements-everywhere.com/)의 모든 테스트를 통과한다.

이전 버전에서는 인식되지 않은 프로퍼티를 속성으로 처리했기 때문에 React에서 커스텀 엘리먼트를 사용하기가 어려웠다. React 19는 클라이언트와 SSR 환경에서 모두 작동하는 프로퍼티 지원을 다음과 같은 전략으로 추가했다:

- **서버 사이드 렌더링**: 커스텀 엘리먼트에 전달된 프로퍼티는 `string`, `number`와 같은 원시 값이거나 값이 `true`인 경우 속성으로 렌더링한다. `object`, `symbol`, `function` 같은 비원시 타입이나 값이 `false`인 프로퍼티는 생략한다.
- **클라이언트 사이드 렌더링**: 커스텀 엘리먼트 인스턴스의 프로퍼티와 일치하는 프로퍼티는 프로퍼티로 할당하고, 그렇지 않은 경우 속성으로 할당한다.

커스텀 엘리먼트 지원을 설계하고 구현한 [Joey Arhar](https://github.com/josepharhar)에게 감사드린다.

# 업그레이드 방법

단계별 지침과 주요 변경 사항의 전체 목록은 [React 19 업그레이드 가이드](https://react.dev/blog/2024/04/25/react-19-upgrade-guide)를 참조한다.

*참고: 이 게시물은 2024년 4월 25일에 처음 게시되었으며 2024년 12월 5일 정식 출시와 함께 업데이트되었다.*

## 알림

이 글은 2024년 12월 5일 발행된 https://react.dev/blog/2024/12/05/react-19 글을 한국어로 편역한 내용을 담고 있습니다.

