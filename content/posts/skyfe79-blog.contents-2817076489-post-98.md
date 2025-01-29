---
title: "Vercel AI SDK 4.1 소개"
date: 2025-01-29T02:06:39Z
author: "skyfe79"
draft: false
tags: ["ai"]
---

# AI SDK 4.1 - Vercel

> 알림: [AI SDK 4.1](https://vercel.com/blog/ai-sdk-4-1) 글을 한국어 번역한 내용을 담고 있습니다.

이미지 생성, 논블로킹 데이터 스트리밍, 향상된 도구 호출 등 새로운 기능이 추가됐다.

[AI SDK](https://sdk.vercel.ai/)는 JavaScript와 TypeScript로 AI 애플리케이션을 구축하기 위한 오픈소스 도구 모음이다. 통합 프로바이더 API를 통해 모든 언어 모델을 사용할 수 있으며, [Next.js](https://nextjs.org/)와 [Svelte](https://svelte.dev/)와 같은 주요 웹 프레임워크에 강력한 UI 통합을 지원한다.

4.0 버전 출시 이후, AI SDK로 구축된 놀라운 제품들이 등장했다:

* [Languine](https://languine.ai/en)은 애플리케이션 현지화를 자동화하는 오픈소스 CLI 도구로, 번역 변경사항을 감지하고 모든 주요 i18n 라이브러리에서 일관된 톤을 유지한다.
* [Scira](https://scira.app/)는 미니멀한 AI 기반 검색 엔진으로, AI SDK를 활용해 검색 기반 LLM 응답과 강력한 생성형 UI 상호작용을 구현했다.
* [Fullmoon](https://fullmoon.app/)은 로컬 LLM을 이용한 크로스 플랫폼 채팅을 지원하여, 누구나 안전한 AI 대화를 나눌 수 있게 한다.

![Languine](https://github.com/user-attachments/assets/012706de-4a72-4300-8793-089efad522cd)
개발자를 위한 AI 기반 CLI와 파이프라인을 제공하는 [Languine](https://languine.ai/)을 확인해보자.

이 모든 프로젝트는 오픈소스([Languine](https://github.com/midday-ai/languine), [Scira](https://github.com/zaidmukaddam/scira), [Fullmoon](https://github.com/mainframecomputer/fullmoon-web))로 공개되어 있어, AI 기반 애플리케이션의 구축 방법을 직접 살펴볼 수 있다.

오늘 우리는 [이미지 생성 기능](https://vercel.com/#image-generation)을 포함한 AI SDK 4.1 버전을 발표한다. 이번 업데이트는 [Replicate](https://replicate.com/), [OpenAI](https://openai.com/), [Google Vertex](https://cloud.google.com/vertex-ai), [Fireworks](https://fireworks.ai/) 등의 프로바이더에서 통합 API를 통해 원활하게 이미지를 생성할 수 있게 한다.

이미지 생성 외에도 이번 릴리스에는 다음과 같은 기능이 포함된다:

* [스트림 변환 및 스무딩](https://vercel.com/#stream-transformation-&-smoothing)
* [useChat을 이용한 단순화된 영속성](https://vercel.com/#simplified-persistence-with-usechat)
* [논블로킹 데이터 스트리밍](https://vercel.com/#non-blocking-data-streaming)
* [도구 호출 개선](https://vercel.com/#tool-calling-improvements)
* [구조화된 출력 개선](https://vercel.com/#structured-output-improvements)
* [신규 및 업데이트된 프로바이더](https://vercel.com/#new-and-updated-providers)

이제 이러한 새로운 기능과 개선 사항을 자세히 살펴보자.

## 이미지 생성 기능

텍스트 프롬프트로 이미지를 생성하는 기능은 생성형 AI의 새로운 역량으로, 다양한 응용 프로그램과 작업 방식을 가능하게 한다. Replicate와 같은 프로바이더는 수백 가지의 이미지 생성 모델을 지원하며, 매일 새로운 모델을 추가하면서 생태계가 빠르게 성장하고 있다.

AI SDK 4.1은 새로운 실험적 [`generateImage`](https://sdk.vercel.ai/docs/reference/ai-sdk-core/generate-image#generateimage) 함수를 도입하여 멀티모달 출력을 지원하는 첫걸음을 내딛었다.

```javascript
import { experimental_generateImage as generateImage } from 'ai';
import { replicate } from '@ai-sdk/replicate';

const { image } = await generateImage({
  model: replicate.image('black-forest-labs/flux-1.1-pro-ultra'),
  prompt: 'A futuristic cityscape at sunset',
});
```

![Image](https://github.com/user-attachments/assets/51062b9e-eaf3-4a25-9ed3-f3e6103b50b9)
Replicate의 [black-forest-labs/flux-1.1-pro-ultra](https://replicate.com/black-forest-labs/flux-1.1-pro-ultra) 모델로 생성한 이미지

AI 프로바이더 간 전환은 단 2줄의 코드 변경만으로 가능하다. 프롬프트와 설정은 그대로 유지된다:

```javascript
import { experimental_generateImage as generateImage } from 'ai';
import { fireworks } from '@ai-sdk/fireworks';

const { image } = await generateImage({
  model: fireworks.image('accounts/fireworks/models/SSD-1B'),
  prompt: 'A futuristic cityscape at sunset',
});
```

![Image](https://github.com/user-attachments/assets/c700d110-da68-45ad-9a86-11c5e15e1089)
Fireworks의 [SSD-1B](https://fireworks.ai/models/fireworks/SSD-1B) 모델로 생성한 이미지

`generateImage` 함수를 사용하면 다음과 같은 매개변수를 완벽하게 제어할 수 있다:

* `size` 또는 `aspectRatio`로 이미지 크기 조절
* `n` 매개변수로 여러 이미지를 동시에 생성
* base64와 uint8Array 형식으로 이미지 접근
* `seed` 값으로 무작위성 제어

프로바이더별 옵션은 `providerOptions` 매개변수를 통해 지원된다:

```javascript
const { image } = await generateImage({
  model: replicate.image('black-forest-labs/flux-1.1-pro-ultra'),
  prompt: 'A futuristic cityscape at sunset',
  size: "16:9",
  n: 3,
  seed: 0,
  providerOptions: {
    replicate: { style: 'realistic_image' },
  },
});
```

AI SDK는 [Replicate](https://sdk.vercel.ai/providers/ai-sdk-providers/replicate#image-models), [OpenAI](https://sdk.vercel.ai/providers/ai-sdk-providers/openai#image-models), [Google Vertex AI](https://sdk.vercel.ai/providers/ai-sdk-providers/google-vertex#image-models), [Fireworks](https://sdk.vercel.ai/providers/ai-sdk-providers/fireworks#image-models) 등 다양한 프로바이더의 이미지 생성을 지원한다.

[이미지 생성 데모](https://ai-sdk-image-generator.vercel.app/)를 통해 각 프로바이더가 동일한 프롬프트를 어떻게 처리하는지 확인하고, 각 모델의 성능을 직접 경험해볼 수 있다.

![Image](https://github.com/user-attachments/assets/54b755b6-f946-404e-ad22-03623ae2097e)

## 스트림 변환과 스무딩

AI SDK 4.1은 서버 측에서 스트림 출력을 변환하는 새로운 기능을 도입했다. 이를 통해 다음과 같은 강력한 기능을 구현할 수 있다:

* 내장된 [`smoothStream`](https://sdk.vercel.ai/docs/reference/ai-sdk-core/smooth-stream) 변환 함수를 사용해 더 자연스러운 스트리밍 경험을 제공한다 (문자, 단어, 줄 단위 등 다양한 청킹 옵션 지원)

* 콘텐츠 필터링과 안전 장치 적용이 가능하다

* 모든 종류의 커스텀 변환이 가능하다 (예: [대문자 변환](https://sdk.vercel.ai/docs/ai-sdk-core/generating-text#custom-transformations))

예를 들어, 내장된 `smoothStream` 함수는 프로바이더의 불규칙하거나 덩어리진 응답을 더 자연스러운 흐름으로 변환하여 텍스트 스트리밍을 개선한다:

```javascript
import { smoothStream, streamText } from 'ai';

const result = streamText({
  model,
  prompt,
  experimental_transform: smoothStream(),
});
```

![Image](https://github.com/user-attachments/assets/a0641c4b-2d2f-475a-bf41-39a3ca022515)

여러 변환 작업을 배열로 전달하여 순차적으로 적용할 수 있다:

```javascript
const result = streamText({
  model,
  prompt,
  experimental_transform: [firstTransform, secondTransform],
});
```

청킹 패턴 설정, 콘텐츠 필터링 구현, 그리고 커스텀 변환 작성 방법에 대해 자세히 알아보려면 [스트림 변환 문서](https://sdk.vercel.ai/docs/ai-sdk-core/generating-text#stream-transformation)를 참고한다.

## useChat을 이용한 간편한 상태 유지 기능

`useChat` 기능의 상태 유지가 너무 복잡하다는 개발자들의 의견을 반영하여, 다음과 같은 세 가지 핵심 개선 사항을 추가했다:

* 채팅 ID를 클라이언트에서 서버로 전달할 수 있다
* 응답 메시지 ID를 서버에서 클라이언트로 전달할 수 있다
* 새로운 `appendResponseMessages` 유틸리티를 통해 메시지 저장을 단순화했다

자세한 내용은 [채팅 상태 유지 가이드](https://sdk.vercel.ai/docs/ai-sdk-ui/chatbot-message-persistence)에서 확인할 수 있다. 바로 코드를 살펴보고 싶다면 [최소 구현 예제](https://github.com/vercel/ai/blob/main/examples/next-openai/app/api/use-chat-persistence/route.ts)를 참고하면 된다.

## 논블로킹 데이터 스트리밍

AI SDK 4.1은 [`createDataStreamResponse`](https://sdk.vercel.ai/docs/reference/ai-sdk-ui/create-data-stream-response#createdatastreamresponse) 함수를 통해 강력한 새로운 스트리밍 기능을 제공한다. 이를 통해 LLM의 응답이 시작되기 전에 검색 기반 생성(RAG) 컨텍스트와 검색 결과를 클라이언트로 스트리밍하는 등의 다양한 활용이 가능하다. 이전 버전에서는 단일 LLM 호출 결과만 스트리밍할 수 있었다(예: `streamText().toDataStreamResponse()`). 이제 다음과 같은 기능을 제공하는 논블로킹 데이터 스트림을 생성할 수 있다:

* 즉시 반환하고 필요할 때마다 데이터를 스트리밍할 수 있다
* 언제, 어떤 데이터를 스트리밍할지 완전히 제어할 수 있다
* 메시지에 주석과 메타데이터를 추가할 수 있다

다음은 `createDataStreamResponse`를 사용하여 LLM 출력과 함께 사용자 정의 데이터를 스트리밍하는 예제이다:

```typescript
import { openai } from "@ai-sdk/openai";
import { createDataStreamResponse, Message, streamText } from "ai";
import { getRelevantContent } from "./get-relevant-content"; // 사용자 정의 함수

export async function POST(req: Request) {
  const { messages }: { messages: Message[] } = await req.json();
  const lastMessage = messages.pop();
  
  return createDataStreamResponse({
    execute: async (dataStream) => {
      // 관련 콘텐츠 가져오기
      const relevantContent = await getRelevantContent(lastMessage.content);
      
      // 각 콘텐츠에 대한 소스 정보 스트리밍
      for (const content of relevantContent) {
        dataStream.writeData({
          type: "source",
          url: content.url,
          title: content.title,
        });
      }
      
      // 마지막 메시지에 관련 정보 추가
      lastMessage.content =
        lastMessage.content +
        "\n\n다음 정보를 사용하여 질문에 답변하세요: " +
        relevantContent.join("\n");
        
      // 텍스트 스트리밍 결과 생성
      const result = streamText({
        model: openai("gpt-4o"),
        messages: [...messages, lastMessage],
        onFinish: async ({}) => {
          dataStream.writeMessageAnnotation({ sources: relevantContent });
        },
      });
      
      // 결과를 데이터 스트림에 병합
      result.mergeIntoDataStream(dataStream);
    },
  });
}
```

스트리밍된 데이터는 클라이언트 측의 [`useChat`](https://sdk.vercel.ai/docs/reference/ai-sdk-ui/use-chat) 훅에서 자동으로 처리되어, 메시지 내용과 추가 스트리밍 데이터에 쉽게 접근할 수 있다:

```typescript
"use client";

import { useChat } from "ai/react";

export default function Chat() {
  const { messages, data } = useChat();
  
  // 스트리밍된 데이터 접근
  console.log(data);
  
  // 메시지 주석 접근
  messages.forEach(m => console.log(m.annotations));
  
  return (/* ... */);
}
```

더 자세한 내용은 [사용자 정의 데이터 스트리밍 문서](https://sdk.vercel.ai/docs/ai-sdk-ui/streaming-data)를 참고하면 된다.

## 도구(tool) 호출 기능 개선

도구는 실제 운영 환경의 AI 애플리케이션을 구축하는 핵심 요소다. 언어 모델이 실제 시스템 및 데이터와 상호작용할 수 있게 해준다. 하지만 도구를 안정적으로 작동시키는 것은 까다로운 과제였다. AI SDK 4.1은 도구 호출 기능을 더욱 견고하게 개선하는 데 중점을 두었다.

### 도구 호출 시 컨텍스트 개선

`execute` 함수가 도구를 실행할 때 이제 두 번째 매개변수를 통해 다음과 같은 유용한 컨텍스트에 접근할 수 있다:

* `toolCallId`: 특정 실행을 추적하고 도구 관련 주석을 추가하는 데 사용
* `messages` 배열: 이전 도구 호출과 결과를 포함한 전체 대화 내역을 포함
* `abortSignal`: 오래 실행되는 작업을 취소하고 fetch 호출에 전달하는 데 사용

다음은 이러한 컨텍스트 옵션을 활용하는 예제이다:

```javascript
const result = await generateText({
  model,
  abortSignal,
  tools: {
    weather: tool({
      parameters: z.object({ location: z.string() }),
      execute: async ({ location }, { toolCallId, messages, abortSignal }) => {
        // toolCallId를 추적에 사용
        data.appendMessageAnnotation({
          type: 'tool-status',
          toolCallId,
          status: 'in-progress',
        });
        
        // abort 시그널 전달
        const response = await fetch(
          `https://api.weatherapi.com/v1/current.json?q=${location}`,
          { signal: abortSignal },
        );
        return response.json();
      },
    }),
  },
});
```

더 자세한 내용은 [도구 호출 문서](https://sdk.vercel.ai/docs/ai-sdk-core/tools-and-tool-calling#tool-execution-options)를 참고한다.

### 도구 호출 복구

도구 호출이 실패할 경우, [`experimental_toToolCallRepair`](https://sdk.vercel.ai/docs/ai-sdk-core/tools-and-tool-calling#tool-call-repair) 함수를 사용하여 다음과 같은 방식으로 복구를 시도할 수 있다:

* 구조화된 출력을 지원하는 모델을 사용하여 인자를 생성한다.
* 메시지, 시스템 프롬프트, 도구 스키마를 더 강력한 모델에 전달하여 인자를 생성한다.
* 호출된 도구에 따라 더 구체적인 복구 지침을 제공한다.

```typescript
import { openai } from '@ai-sdk/openai';
import { generateObject, generateText, NoSuchToolError, tool } from 'ai';

const result = await generateText({
  model,
  tools,
  prompt,
  // 복구를 위해 구조화된 출력을 지원하는 모델을 사용하는 예제
  // (다른 전략도 사용 가능)
  experimental_repairToolCall: async ({
    toolCall,
    tools,
    parameterSchema,
    error,
  }) => {
    if (NoSuchToolError.isInstance(error)) {
      return null; // 유효하지 않은 도구 이름은 수정을 시도하지 않음
    }

    const tool = tools[toolCall.toolName as keyof typeof tools];
    const { object: repairedArgs } = await generateObject({
      model: openai('gpt-4o', { structuredOutputs: true }),
      schema: tool.parameters,
      prompt: [
        `모델이 "${toolCall.toolName}" 도구를 다음 인자로 호출하려 했습니다:`,
        JSON.stringify(toolCall.args),
        `이 도구는 다음 스키마를 받아들입니다:`,
        JSON.stringify(parameterSchema(toolCall)),
        '인자를 수정해 주세요.',
      ].join('\n'),
    });

    return { ...toolCall, args: JSON.stringify(repairedArgs) };
  },
});
```

### 세분화된 오류 처리

더욱 안정적인 도구 호출을 구현하기 위해, AI SDK는 이제 디버깅과 오류 처리를 더욱 정확하게 만드는 세분화된 오류 타입을 제공한다. 각 오류 타입은 무엇이 잘못되었는지에 대한 상세 정보를 노출하며, 문제를 진단하고 해결하는 데 도움이 되는 맥락 데이터를 포함한다:

* `NoSuchToolError`: 모델이 정의되지 않은 도구를 호출하려 할 때 발생하는 오류를 처리한다.
* `InvalidToolArgumentsError`: 도구 인자가 예상된 매개변수와 일치하지 않을 때 발생하는 스키마 유효성 검사 실패를 포착한다.
* `ToolExecutionError`: 도구 실행 중에 발생하는 런타임 문제를 식별한다.
* `ToolCallRepairError`: 자동 도구 호출 복구 시도 중 발생하는 실패를 추적한다.

이러한 구체적인 오류 타입을 통해 목표 지향적인 오류 처리 전략을 구현하고, 도구 실행이 실패했을 때 사용자에게 더 나은 피드백을 제공할 수 있다. 자세한 내용은 [오류 처리 문서](https://sdk.vercel.ai/docs/ai-sdk-core/tools-and-tool-calling#handling-errors)를 참조한다.

## 구조화된 출력 개선사항

구조화된 출력 기능을 확장하여 더욱 유연하고 안정적인 AI 애플리케이션 개발이 가능해졌다.

### 도구를 활용한 구조화된 출력

많은 개발자가 요청한 기능을 마침내 선보인다. 구조화된 출력과 도구 사용을 결합할 수 있게 되었다. `generateText`와 `streamText` 함수에 새로 추가된 `experimental_output` 옵션을 사용하면 외부 시스템과 상호작용하고 예측 가능한 구조의 데이터를 반환하는 정교한 대규모 언어 모델(LLM) 호출을 구축할 수 있다.

다음은 구조화된 출력과 도구가 함께 작동하는 예제이다:

```javascript
import { openai } from '@ai-sdk/openai';
import { generateText, tool, Output } from 'ai';
import { z } from 'zod';

const result = await generateText({
  model: openai('gpt-4o', { structuredOutputs: true }),
  prompt: "What's the weather like in London and New York?",
  maxSteps: 5,
  tools: {
    getWeather: tool({
      parameters: z.object({
        city: z.string(),
        units: z.enum(['celsius', 'fahrenheit']),
      }),
      execute: async ({ city, units }) => {
        // 날씨 데이터 가져오기
      },
    }),
  },
  experimental_output: Output.object({
    schema: z.object({
      cities: z.array(
        z.object({
          name: z.string(),
          temperature: z.number(),
          conditions: z.string(),
        }),
      ),
    }),
  }),
});
```

이제 확인할 도시를 결정하고 각 도시의 날씨 도구를 개별적으로 호출할 필요 없이, 모델이 전체 작업 흐름을 단일 함수로 처리한다. 이는 예측할 수 없는 입력이나 여러 도구 경로가 있는 복잡한 상황에서 특히 코드를 더 효율적이고 유지보수하기 쉽게 만든다. 자세한 내용은 [generateText와 streamText를 사용한 구조화된 출력 문서](https://sdk.vercel.ai/docs/ai-sdk-core/generating-structured-data#structured-outputs-with-generatetext-and-streamtext)를 참조한다.

현재 도구를 사용한 구조화된 출력은 OpenAI 모델에서만 사용할 수 있다.

### 향상된 오류 처리

버전 4.1에서 구조화된 출력에 대한 오류 처리 기능이 크게 개선되었다. 이전에는 구조 파싱이나 유효성 검사가 실패할 경우 오류만 받을 수 있었으며, 기본 응답에 접근할 수 없었다. 이로 인해 요청을 다시 시도하는 것 외에는 다른 선택지가 없었다. 새로운 [`NoObjectGeneratedError`](https://sdk.vercel.ai/docs/reference/ai-sdk-errors/ai-no-object-generated-error)를 통해 이제 다음과 같은 정보에 접근할 수 있다:

* 디버깅이나 부분 응답 복구를 위한 원시 모델 출력
* 전체 요청 컨텍스트 (응답 ID, 타임스탬프, 모델)
* 토큰 사용량과 비용 분석 데이터

향상된 오류 처리를 구현하는 방법은 다음과 같다:

```javascript
try {
  const result = await generateObject({
    model,
    schema,
    prompt,
  });
} catch (error) {
  if (error instanceof NoObjectGeneratedError) {
    console.log('생성된 텍스트:', error.text);
    console.log('응답 메타데이터:', error.response);
    console.log('토큰 사용량:', error.usage);
    console.log('오류 원인:', error.cause);
  }
}
```

이러한 세부적인 오류 정보를 통해 파싱, 유효성 검사, 모델 생성 단계에서 발생하는 구조화된 출력 생성 문제를 더 쉽게 진단하고 해결할 수 있다.

이러한 패턴을 구현하는 방법에 대해 자세히 알아보려면 [구조화된 출력 오류 처리 문서](https://sdk.vercel.ai/docs/ai-sdk-core/generating-structured-data#error-handling)를 참고하면 된다.

## 새롭게 추가되고 개선된 프로바이더

AI SDK 프로바이더 생태계가 새롭고 향상된 프로바이더들과 함께 지속적으로 성장하고 있다:

* [Google Vertex](https://sdk.vercel.ai/providers/ai-sdk-providers/google-vertex) AI 2.0: Vertex AI 통합을 전면 개편했다. 성능이 향상되고, 오류 처리가 개선되었으며, 검색 기반 그라운딩 지원이 추가되었다.
* [OpenAI](https://sdk.vercel.ai/providers/ai-sdk-providers/openai): 최신 추론 모델을 위한 전면적인 지원 개편이 이루어졌다.
* [OpenAI Compatible](https://sdk.vercel.ai/providers/openai-compatible-providers): OpenAI 호환 API를 위한 새로운 전용 프로바이더가 추가되었다.
* [Replicate](https://sdk.vercel.ai/providers/ai-sdk-providers/replicate): Replicate(이미지 모델)를 위한 자체 프로바이더가 추가되었다.
* [Fireworks](https://sdk.vercel.ai/providers/openai-compatible-providers/fireworks): Fireworks(언어 및 이미지 모델)를 위한 자체 프로바이더가 추가되었다.
* [Cohere](https://sdk.vercel.ai/providers/ai-sdk-providers/cohere): Cohere(언어 및 임베딩 모델)를 위한 자체 프로바이더가 추가되었다.
* [Together AI](https://sdk.vercel.ai/providers/openai-compatible-providers/togetherai): Together AI(언어 모델)를 위한 자체 프로바이더가 추가되었다.
* [DeepInfra](https://sdk.vercel.ai/providers/ai-sdk-providers/deepinfra): DeepInfra(언어 모델)를 위한 자체 프로바이더가 추가되었다.
* [DeepSeek](https://sdk.vercel.ai/providers/ai-sdk-providers/deepseek): DeepSeek(언어 모델)를 위한 자체 프로바이더가 추가되었다.
* [Cerebras](https://sdk.vercel.ai/providers/ai-sdk-providers/cerebras): Cerebras(언어 모델)를 위한 자체 프로바이더가 추가되었다.

## 시작하기

이미지 생성, 논블로킹 데이터 스트리밍, 향상된 도구 호출 등 강력한 새로운 기능이 추가되어 AI SDK로 AI 애플리케이션을 구축하기에 더없이 좋은 시기가 되었다.

* **새로운 AI 프로젝트 시작하기**: 새로운 프로젝트를 구축할 준비가 되었다면 [**최신 가이드**](https://sdk.vercel.ai/cookbook)를 확인한다.
* **템플릿 살펴보기**: [**템플릿 갤러리**](https://sdk.vercel.ai/docs/introduction#templates)에서 AI SDK의 실제 활용 사례를 확인할 수 있다.
* **커뮤니티 참여하기**: [**GitHub Discussions**](https://github.com/vercel/ai/discussions)에서 여러분이 만들고 있는 프로젝트를 공유하고 토론할 수 있다.

## 기여자 목록

AI SDK 4.1은 Vercel의 핵심 팀([Lars](https://x.com/lgrammel), [Jeremy](https://x.com/jrmyphlmn), [Walter](https://x.com/shaper), [Nico](https://x.com/nicoalbanese10))과 많은 커뮤니티 기여자들의 공동 작업으로 만들어졌다. 병합된 풀 리퀘스트를 제출한 다음 기여자들에게 특별한 감사를 표한다:

[patelvivekdev](https://github.com/patelvivekdev), [zeke](https://github.com/zeke), [daviddkkim](https://github.com/daviddkkim), [klren0312](https://github.com/klren0312), [viktorlarsson](https://github.com/viktorlarsson), [richhuth](https://github.com/richhuth), [dragos-cojocaru](https://github.com/dragos-cojocaru), [olyaiy](https://github.com/olyaiy), [minpeter](https://github.com/minpeter), [nathanwijaya](https://github.com/nathanwijaya), [timconnorz](https://github.com/timconnorz), [palmm](https://github.com/palmm), [Ojansen](https://github.com/Ojansen), [ggallon](https://github.com/ggallon), [williamlmao](https://github.com/williamlmao), [nasjp](https://github.com/nasjp), [ManuLpz4](https://github.com/ManuLpz4), [aaronccasanova](https://github.com/aaronccasanova), [marcklingen](https://github.com/marcklingen), [aaishikasb](https://github.com/aaishikasb), [michael-hhai](https://github.com/michael-hhai), [jeremypress](https://github.com/jeremypress), [yoshinorisano](https://github.com/yoshinorisano).

AI SDK를 계속 발전시켜 나가는 데 있어 여러분의 피드백과 기여는 매우 소중한 가치를 지닌다.