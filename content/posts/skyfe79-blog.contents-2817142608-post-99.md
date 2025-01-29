---
title: "React로 채팅 UI 만들기: useStickToBottom 라이브러리 소개"
date: 2025-01-29T03:16:14Z
author: "skyfe79"
draft: false
tags: ["web"]
---

<img width="1027" alt="Image" src="https://github.com/user-attachments/assets/c4211343-e70e-4000-9c9c-b5aedefd37a8" />

 
- [use-stick-to-bottom](https://www.npmjs.com/package/use-stick-to-bottom)
- [Demo](https://stackblitz.com/~/github.com/samdenty/use-stick-to-bottom?file=demo/Demo.tsx)

지난 주말, 채팅 기능이 있는 React 앱을 만들다가 흥미로운 라이브러리를 발견했다. 채팅창의 스크롤 처리로 고생하던 중에 찾은 `useStickToBottom`인데, 정말 깔끔하게 문제를 해결했다.

## 왜 이 라이브러리인가?

채팅 UI 개발할 때 가장 골치 아픈 게 스크롤 처리다. 새 메시지가 오면 자동으로 스크롤을 내려야 하는데, 사용자가 이전 메시지를 보고 있을 때는 그러면 안 된다. 게다가 Safari 브라우저는 `overflow-anchor` 속성도 지원하지 않아서 더 복잡해진다.

특히 이런 상황에서 애먹었다:
1. 사용자가 옛날 메시지를 보고 있는데 새 메시지가 왔을 때
2. 긴 메시지가 천천히 로딩되면서 화면이 계속 바뀔 때
3. 모바일에서 스크롤이 이상하게 움직일 때

`useStickToBottom`은 이런 문제들을 모두 해결한다. 실제로 [bolt.new](https://bolt.new)라는 StackBlitz의 프로젝트에서도 이 라이브러리로 AI 챗봇 인터페이스를 구현했다.

## 어떻게 사용하는가?

두 가지 방식으로 사용할 수 있다. 먼저 가장 간단한 컴포넌트 방식이다:

```jsx
import { StickToBottom, useStickToBottomContext } from 'use-stick-to-bottom';

function Chat() {
  return (
    <StickToBottom className="h-[50vh] relative" resize="smooth" initial="smooth">
      <StickToBottom.Content className="flex flex-col gap-4">
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </StickToBottom.Content>
      <ScrollToBottom />
      <ChatBox />
    </StickToBottom>
  );
}
```

더 세밀한 제어가 필요하다면 Hook 방식도 있다:

```jsx
import { useStickToBottom } from 'use-stick-to-bottom';

function Component() {
  const { scrollRef, contentRef } = useStickToBottom();
  
  return (
    <div style={{ overflow: 'auto' }} ref={scrollRef}>
      <div ref={contentRef}>
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </div>
    </div>
  );
}
```

## 내부적으로는 어떻게 동작하는가?

이 라이브러리의 진짜 똑똑한 점은 스크롤 처리 방식이다. `ResizeObserver` API를 사용해서 컨텐츠 크기 변화를 감지하고, 스프링 애니메이션으로 부드러운 스크롤을 구현했다. 다른 라이브러리들은 대부분 정해진 시간에 맞춰 스크롤하는데, 이 라이브러리는 속도 기반으로 움직여서 훨씬 자연스럽다.

특히 인상적인 건 사용자의 스크롤과 자동 스크롤을 구분하는 로직이다. 디바운싱 같은 걸 사용하지 않고도 정확하게 구분해서, 사용자가 이전 메시지를 읽고 있을 때는 방해하지 않는다.

## 마무리

채팅 UI 만들 때 스크롤 처리는 생각보다 복잡하다. 처음에는 직접 구현하려고 했다가 며칠을 날린 적도 있다. 이제는 새 프로젝트 시작할 때마다 `useStickToBottom`을 제일 먼저 설치할 것 같다. 특히 AI 챗봇처럼 메시지가 실시간으로 길어지는 상황에서는 더더욱 유용하지 않을까?

