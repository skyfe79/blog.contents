---
title: "AI SDK를 Node.js HTTP 서버에서 사용하는 방법"
date: 2025-01-29T00:40:35Z
author: "skyfe79"
draft: false
tags: ["ai"]
---

Node.js HTTP 서버에서 AI SDK를 활용하여 텍스트를 생성하고 클라이언트에 스트리밍하는 방법을 알아본다.

## 예제

아래 예제는 8080 포트에서 수신 대기하는 간단한 HTTP 서버를 시작한다. `curl` 명령어를 사용하여 테스트할 수 있다:

```bash
curl -X POST http://localhost:8080
```

##### 참고
  이 예제는 OpenAI의 `gpt-4o` 모델을 사용한다. OpenAI API 키가 `OPENAI_API_KEY` 환경 변수에 설정되어 있는지 확인한다.

**전체 예제**: [github.com/vercel/ai/examples/node-http-server](https://github.com/vercel/ai/tree/main/examples/node-http-server)

### 데이터 스트림

`pipeDataStreamToResponse` 메서드를 사용하여 스트림 데이터를 서버 응답으로 전달할 수 있다.

```ts filename='index.ts'
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import { createServer } from 'http';

createServer(async (req, res) => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  result.pipeDataStreamToResponse(res);
}).listen(8080);
```

### 커스텀 데이터 전송

`pipeDataStreamToResponse`를 사용하여 클라이언트에 커스텀 데이터를 전송할 수 있다.

```ts filename='index.ts' highlight="6-9,16"
import { openai } from '@ai-sdk/openai';
import { pipeDataStreamToResponse, streamText } from 'ai';
import { createServer } from 'http';

createServer(async (req, res) => {
  // 즉시 응답 스트리밍 시작
  pipeDataStreamToResponse(res, {
    execute: async dataStreamWriter => {
      dataStreamWriter.writeData('initialized call');

      const result = streamText({
        model: openai('gpt-4o'),
        prompt: 'Invent a new holiday and describe its traditions.',
      });

      result.mergeIntoDataStream(dataStreamWriter);
    },
    onError: error => {
      // 보안상의 이유로 오류 메시지는 기본적으로 마스킹 처리된다.
      // 클라이언트에 오류 메시지를 노출하려면 다음과 같이 처리한다:
      return error instanceof Error ? error.message : String(error);
    },
  });
}).listen(8080);
```

### 텍스트 스트림

`pipeTextStreamToResponse`를 사용하여 클라이언트에 텍스트 스트림을 전송할 수 있다.

```ts filename='index.ts'
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import { createServer } from 'http';

createServer(async (req, res) => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  result.pipeTextStreamToResponse(res);
}).listen(8080);
```

# Express 서버

Express 서버에서 AI SDK를 활용하여 텍스트와 객체를 생성하고 클라이언트에 스트리밍할 수 있다. 

## 예제

아래 예제는 8080 포트에서 수신 대기하는 간단한 HTTP 서버를 시작한다. `curl` 명령어를 사용하여 테스트할 수 있다:

```bash
curl -X POST http://localhost:8080
```

##### 참고
  이 예제에서는 OpenAI의 `gpt-4o` 모델을 사용한다. 실행하기 전에 `OPENAI_API_KEY` 환경 변수에 OpenAI API 키를 설정해야 한다.


**전체 예제 코드**: [github.com/vercel/ai/examples/express](https://github.com/vercel/ai/tree/main/examples/express)

### 데이터 스트림

`pipeDataStreamToResponse` 메서드를 사용하여 스트림 데이터를 서버 응답으로 전송할 수 있다.

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import express, { Request, Response } from 'express';

const app = express();

app.post('/', async (req: Request, res: Response) => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  result.pipeDataStreamToResponse(res);
});

app.listen(8080, () => {
  console.log(`예제 앱이 ${8080} 포트에서 실행 중입니다`);
});
```

### 커스텀 데이터 전송

`pipeDataStreamToResponse`를 사용하여 클라이언트에 사용자 정의 데이터를 전송할 수 있다.

```typescript
import { openai } from '@ai-sdk/openai';
import { pipeDataStreamToResponse, streamText } from 'ai';
import express, { Request, Response } from 'express';

const app = express();

app.post('/stream-data', async (req: Request, res: Response) => {
  // 즉시 응답 스트리밍 시작
  pipeDataStreamToResponse(res, {
    execute: async dataStreamWriter => {
      dataStreamWriter.writeData('호출 초기화됨');

      const result = streamText({
        model: openai('gpt-4o'),
        prompt: 'Invent a new holiday and describe its traditions.',
      });

      result.mergeIntoDataStream(dataStreamWriter);
    },
    onError: error => {
      // 보안상의 이유로 오류 메시지는 기본적으로 숨겨진다.
      // 클라이언트에 오류 메시지를 노출하려면 다음과 같이 처리할 수 있다:
      return error instanceof Error ? error.message : String(error);
    },
  });
});

app.listen(8080, () => {
  console.log(`예제 앱이 ${8080} 포트에서 실행 중입니다`);
});
```

### 텍스트 스트림

`pipeTextStreamToResponse`를 사용하여 클라이언트에 텍스트 스트림을 전송할 수 있다.

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import express, { Request, Response } from 'express';

const app = express();

app.post('/', async (req: Request, res: Response) => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  result.pipeTextStreamToResponse(res);
});

app.listen(8080, () => {
  console.log(`예제 앱이 ${8080} 포트에서 실행 중입니다`);
});
```

# Hono

Hono 서버에서 AI SDK를 사용하여 텍스트와 객체를 생성하고 클라이언트로 스트리밍할 수 있다.

## 예제

이 예제들은 8080 포트에서 수신 대기하는 간단한 HTTP 서버를 시작한다. `curl` 명령어를 사용하여 테스트할 수 있다:

```bash
curl -X POST http://localhost:8080
```

##### 참고
  이 예제들은 OpenAI의 `gpt-4o` 모델을 사용한다. OpenAI API 키가 `OPENAI_API_KEY` 환경 변수에 설정되어 있는지 확인한다.


**전체 예제**: [github.com/vercel/ai/examples/hono](https://github.com/vercel/ai/tree/main/examples/hono)

### 데이터 스트림

`toDataStream` 메서드를 사용하여 결과에서 데이터 스트림을 가져온 다음 응답으로 전달할 수 있다.

```typescript filename='index.ts'
import { openai } from '@ai-sdk/openai';
import { serve } from '@hono/node-server';
import { streamText } from 'ai';
import { Hono } from 'hono';
import { stream } from 'hono/streaming';

const app = new Hono();

app.post('/', async c => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  // 응답을 v1 데이터 스트림으로 표시한다:
  c.header('X-Vercel-AI-Data-Stream', 'v1');
  c.header('Content-Type', 'text/plain; charset=utf-8');

  return stream(c, stream => stream.pipe(result.toDataStream()));
});

serve({ fetch: app.fetch, port: 8080 });
```

### 커스텀 데이터 전송

`createDataStream`을 사용하여 클라이언트에 커스텀 데이터를 전송할 수 있다.

```typescript filename='index.ts'
import { openai } from '@ai-sdk/openai';
import { serve } from '@hono/node-server';
import { createDataStream, streamText } from 'ai';
import { Hono } from 'hono';
import { stream } from 'hono/streaming';

const app = new Hono();

app.post('/stream-data', async c => {
  // 즉시 응답 스트리밍을 시작한다
  const dataStream = createDataStream({
    execute: async dataStreamWriter => {
      dataStreamWriter.writeData('initialized call');

      const result = streamText({
        model: openai('gpt-4o'),
        prompt: 'Invent a new holiday and describe its traditions.',
      });

      result.mergeIntoDataStream(dataStreamWriter);
    },
    onError: error => {
      // 보안상의 이유로 오류 메시지는 기본적으로 숨겨진다.
      // 클라이언트에 오류 메시지를 노출하려면 여기서 처리할 수 있다:
      return error instanceof Error ? error.message : String(error);
    },
  });

  // 응답을 v1 데이터 스트림으로 표시한다:
  c.header('X-Vercel-AI-Data-Stream', 'v1');
  c.header('Content-Type', 'text/plain; charset=utf-8');

  return stream(c, stream =>
    stream.pipe(dataStream.pipeThrough(new TextEncoderStream())),
  );
});

serve({ fetch: app.fetch, port: 8080 });
```

### 텍스트 스트림

`textStream` 속성을 사용하여 결과에서 텍스트 스트림을 가져온 다음 응답으로 전달할 수 있다.

```typescript filename='index.ts'
import { openai } from '@ai-sdk/openai';
import { serve } from '@hono/node-server';
import { streamText } from 'ai';
import { Hono } from 'hono';
import { stream } from 'hono/streaming';

const app = new Hono();

app.post('/', async c => {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  c.header('Content-Type', 'text/plain; charset=utf-8');

  return stream(c, stream => stream.pipe(result.textStream));
});

serve({ fetch: app.fetch, port: 8080 });
```

# Fastify

Fastify 서버에서 AI SDK를 활용하여 텍스트와 객체를 생성하고 클라이언트로 스트리밍할 수 있다.

## 예제

아래 예제는 8080 포트에서 수신 대기하는 간단한 HTTP 서버를 시작한다. `curl` 명령어를 사용하여 테스트할 수 있다:

```bash
curl -X POST http://localhost:8080
```

##### 참고
  이 예제는 OpenAI의 `gpt-4o` 모델을 사용한다. OpenAI API 키를 `OPENAI_API_KEY` 환경 변수에 설정해야 한다.


**전체 예제**: [github.com/vercel/ai/examples/fastify](https://github.com/vercel/ai/tree/main/examples/fastify)

### 데이터 스트림

`toDataStream` 메서드를 사용하여 결과에서 데이터 스트림을 가져온 다음 응답으로 전송할 수 있다.

```typescript filename='index.ts'
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import Fastify from 'fastify';

const fastify = Fastify({ logger: true });

fastify.post('/', async function (request, reply) {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  // 응답을 v1 데이터 스트림으로 표시:
  reply.header('X-Vercel-AI-Data-Stream', 'v1');
  reply.header('Content-Type', 'text/plain; charset=utf-8');

  return reply.send(result.toDataStream({ data }));
});

fastify.listen({ port: 8080 });
```

### 커스텀 데이터 전송

`createDataStream`을 사용하여 클라이언트에 커스텀 데이터를 전송할 수 있다.

```typescript filename='index.ts' highlight="8-11,18"
import { openai } from '@ai-sdk/openai';
import { createDataStream, streamText } from 'ai';
import Fastify from 'fastify';

const fastify = Fastify({ logger: true });

fastify.post('/stream-data', async function (request, reply) {
  // 즉시 응답 스트리밍 시작
  const dataStream = createDataStream({
    execute: async dataStreamWriter => {
      dataStreamWriter.writeData('초기화 완료');

      const result = streamText({
        model: openai('gpt-4o'),
        prompt: 'Invent a new holiday and describe its traditions.',
      });

      result.mergeIntoDataStream(dataStreamWriter);
    },
    onError: error => {
      // 보안상의 이유로 오류 메시지는 기본적으로 감춰진다.
      // 클라이언트에 오류 메시지를 노출하려면 다음과 같이 작성한다:
      return error instanceof Error ? error.message : String(error);
    },
  });

  // 응답을 v1 데이터 스트림으로 표시:
  reply.header('X-Vercel-AI-Data-Stream', 'v1');
  reply.header('Content-Type', 'text/plain; charset=utf-8');

  return reply.send(dataStream);
});

fastify.listen({ port: 8080 });
```

### 텍스트 스트림

`textStream` 속성을 사용하여 결과에서 텍스트 스트림을 가져온 다음 응답으로 전송할 수 있다.

```typescript filename='index.ts' highlight="15"
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import Fastify from 'fastify';

const fastify = Fastify({ logger: true });

fastify.post('/', async function (request, reply) {
  const result = streamText({
    model: openai('gpt-4o'),
    prompt: 'Invent a new holiday and describe its traditions.',
  });

  reply.header('Content-Type', 'text/plain; charset=utf-8');

  return reply.send(result.textStream);
});

fastify.listen({ port: 8080 });
```

# Nest.js

Nest.js 서버에서 AI SDK를 활용하면 텍스트와 객체를 생성하고 클라이언트에 스트리밍할 수 있다. 

## 예제

아래 예제는 AI SDK를 사용하여 텍스트와 객체를 클라이언트에 스트리밍하는 Nest.js 컨트롤러 구현 방법을 보여준다.

**전체 예제 코드**: [github.com/vercel/ai/examples/nest](https://github.com/vercel/ai/tree/main/examples/nest)

##### 참고
이 예제는 OpenAI의 gpt-4o 모델을 사용한다. OpenAI API 키를 OPENAI_API_KEY 환경 변수에 설정해야 한다.

### 데이터 스트림

`pipeDataStreamToResponse` 메서드를 사용하면 결과에서 데이터 스트림을 얻어 응답으로 전송할 수 있다.

```typescript filename='app.controller.ts'
import { Controller, Post, Res } from '@nestjs/common';
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import { Response } from 'express';

@Controller()
export class AppController {
  @Post()
  async example(@Res() res: Response) {
    const result = streamText({
      model: openai('gpt-4o'),
      prompt: 'Invent a new holiday and describe its traditions.',
    });

    result.pipeDataStreamToResponse(res);
  }
}
```

### 커스텀 데이터 전송

`createDataStream`을 사용하면 클라이언트에 커스텀 데이터를 전송할 수 있다.

```typescript filename='app.controller.ts' highlight="10-12,19"
import { Controller, Post, Res } from '@nestjs/common';
import { openai } from '@ai-sdk/openai';
import { createDataStream, streamText } from 'ai';
import { Response } from 'express';

@Controller()
export class AppController {
  @Post('/stream-data')
  async streamData(@Res() res: Response) {
    pipeDataStreamToResponse(res, {
      execute: async dataStreamWriter => {
        dataStreamWriter.writeData('initialized call');

        const result = streamText({
          model: openai('gpt-4o'),
          prompt: 'Invent a new holiday and describe its traditions.',
        });

        result.mergeIntoDataStream(dataStreamWriter);
      },
      onError: error => {
        // 보안상의 이유로 기본적으로 오류 메시지는 숨겨진다.
        // 클라이언트에 오류 메시지를 노출하려면 다음과 같이 작성한다:
        return error instanceof Error ? error.message : String(error);
      },
    });
  }
}
```

### 텍스트 스트림

`pipeTextStreamToResponse` 메서드를 사용하면 결과에서 텍스트 스트림을 얻어 응답으로 전송할 수 있다.

```typescript filename='app.controller.ts' highlight="15"
import { Controller, Post, Res } from '@nestjs/common';
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import { Response } from 'express';

@Controller()
export class AppController {
  @Post()
  async example(@Res() res: Response) {
    const result = streamText({
      model: openai('gpt-4o'),
      prompt: 'Invent a new holiday and describe its traditions.',
    });

    result.pipeTextStreamToResponse(res);
  }
}
```