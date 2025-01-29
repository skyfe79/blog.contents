---
title: "RAG 챗봇 만들기"
date: 2025-01-29T04:13:09Z
author: "skyfe79"
draft: false
tags: ["ai"]
---

# RAG 챗봇

> 알림: [RAG Chatbot Guide](https://sdk.vercel.ai/docs/guides/rag-chatbot#rag-chatbot-guide)를 한국어로 번역한 글입니다.

## RAG 챗봇 가이드

이 가이드에서는 검색 기반 생성(Retrieval-Augmented Generation, RAG) 챗봇 애플리케이션을 구축하는 방법을 배운다. 

![Image](https://github.com/user-attachments/assets/722c5b9d-23f7-4b79-8b6c-daef42e01840)

본격적인 내용에 들어가기 전에 RAG가 무엇이고 왜 필요한지 알아보자.

### RAG란 무엇인가?

RAG는 검색 기반 생성(Retrieval Augmented Generation)의 약자다. 쉽게 말해서 RAG는 대규모 언어 모델(Large Language Model, LLM)에 프롬프트와 관련된 특정 정보를 제공하는 과정이다.

### RAG가 왜 중요한가?

LLM은 강력한 성능을 보여주지만, 추론할 수 있는 정보는 학습 데이터로 제한된다. 이러한 한계는 모델의 학습 데이터에 포함되지 않은 정보, 예를 들어 독점 데이터나 모델 학습 기간 이후에 발생한 일반적인 지식을 요청할 때 분명히 드러난다. RAG는 프롬프트와 관련된 정보를 가져와 모델에 문맥으로 전달함으로써 이 문제를 해결한다.

간단한 예시를 통해 살펴보자. 모델에게 좋아하는 음식을 물어보면:

```text
입력: 내가 좋아하는 음식이 뭐야?
생성: 개인의 좋아하는 음식을 포함한 개인정보에 접근할 수 없다.
```

당연하게도 모델은 알지 못한다. 하지만 프롬프트와 함께 추가 문맥을 제공하면:

```text
입력: 제공된 문맥만을 사용하여 사용자의 프롬프트에 답하시오.
사용자 프롬프트: '내가 좋아하는 음식이 뭐야?'
문맥: 사용자는 치킨너겟을 좋아한다
생성: 치킨너겟이 당신이 좋아하는 음식입니다!
```

이처럼 관련 정보를 제공하여 모델의 생성 능력을 향상시킬 수 있다. 모델이 적절한 정보를 가지고 있다면, 사용자의 질문에 정확한 응답을 할 가능성이 매우 높아진다. 그렇다면 어떻게 관련 정보를 검색할까? 그 답은 임베딩이라는 개념에 있다.

RAG 애플리케이션에서는 다양한 방식으로 문맥을 가져올 수 있다(예: 구글 검색). 임베딩과 벡터 데이터베이스는 의미론적 검색을 실현하기 위한 특정한 검색 방식일 뿐이다.

### 임베딩

[임베딩](https://sdk.vercel.ai/docs/ai-sdk-core/embeddings)은 단어, 구문 또는 이미지를 고차원 공간의 벡터로 표현하는 방법이다. 이 공간에서 유사한 단어들은 서로 가깝게 위치하며, 단어 간의 거리로 유사성을 측정할 수 있다.

실제로 `cat`과 `dog`라는 단어를 임베딩하면 벡터 공간에서 서로 가깝게 위치할 것이다. 두 벡터 간의 유사성을 계산하는 과정을 '코사인 유사도'라고 부른다. 값이 1이면 높은 유사성을, -1이면 높은 대립성을 나타낸다.

복잡해 보인다고 걱정하지 말자. 시작하는 데는 기본적인 이해만으로도 충분하다! 임베딩에 대한 더 자세한 내용은 [이 가이드](https://jalammar.github.io/illustrated-word2vec/)를 참고하면 된다.

앞서 언급했듯이 임베딩은 **단어와 구문**의 의미론적 의미를 표현하는 방법이다. 이는 임베딩의 입력이 길어질수록 품질이 떨어질 수 있다는 것을 의미한다. 그렇다면 간단한 구문보다 긴 내용을 임베딩할 때는 어떻게 접근해야 할까?

### 청킹

청킹은 특정 소스 자료를 작은 조각으로 나누는 과정을 말한다. 청킹에는 여러 가지 접근 방식이 있으며, 사용 사례에 따라 가장 효과적인 방법이 다를 수 있으므로 실험해볼 가치가 있다. 간단하고 일반적인 접근 방식(이 가이드에서 사용할 방식)은 문장 단위로 글을 나누는 것이다.

소스 자료를 적절히 청킹한 후에는 각각을 임베딩하고, 임베딩과 청크를 함께 데이터베이스에 저장할 수 있다. 임베딩은 벡터를 지원하는 모든 데이터베이스에 저장할 수 있다. 이 튜토리얼에서는 [pgvector](https://github.com/pgvector/pgvector) 플러그인과 함께 [Postgres](https://www.postgresql.org/)를 사용할 것이다.

![Image](https://github.com/user-attachments/assets/9d716fb8-0972-4636-bf91-898381853a43)

### 모든 것을 종합하면

이 모든 것을 종합하면, RAG는 모델이 학습 데이터를 넘어선 정보로 응답할 수 있게 하는 과정이다. 사용자의 질의를 임베딩하고, 의미론적 유사성이 가장 높은 관련 소스 자료(청크)를 검색한 다음, 이를 초기 질의와 함께 문맥으로 전달한다. 앞서 살펴본 좋아하는 음식을 묻는 예시로 돌아가보면, 프롬프트 준비 과정은 다음과 같다.

![Image](https://github.com/user-attachments/assets/633a2e1b-7d54-423a-81ea-181921607348)

적절한 문맥을 제공하고 모델의 목적을 조정함으로써 추론 엔진으로서의 강점을 충분히 활용할 수 있다.

이제 프로젝트를 시작해보자!

## 프로젝트 설정

이 프로젝트에서는 지식 기반 내에서만 응답하는 챗봇을 구축한다. 이 챗봇은 정보를 저장하고 검색하는 기능을 모두 갖추게 된다. 고객 지원부터 개인 지식 관리 시스템 구축까지 다양한 용도로 활용할 수 있다.

프로젝트는 다음 기술 스택을 사용한다:

* [Next.js](https://nextjs.org/) 14 (앱 라우터)
* [AI SDK](https://sdk.vercel.ai/docs)
* [OpenAI](https://openai.com/)
* [Drizzle ORM](https://orm.drizzle.team/)
* [pgvector](https://github.com/pgvector/pgvector)가 포함된 [Postgres](https://www.postgresql.org/)
* 스타일링을 위한 [shadcn-ui](https://ui.shadcn.com/)와 [TailwindCSS](https://tailwindcss.com/)

### 저장소 복제

이 가이드의 범위를 줄이기 위해, 몇 가지 사항이 이미 설정된 [저장소](https://github.com/vercel/ai-sdk-rag-starter)에서 시작한다:

* Drizzle ORM(`lib/db`) - 초기 마이그레이션과 마이그레이션 스크립트(`db:migrate`) 포함
* `resources` 테이블을 위한 기본 스키마 (소스 자료용)
* `resource` 생성을 위한 서버 액션

시작하려면 다음 명령으로 스타터 저장소를 복제한다:

```bash
git clone https://github.com/vercel/ai-sdk-rag-starter
```
```bash
cd ai-sdk-rag-starter
```

우선 다음 명령으로 프로젝트 의존성을 설치한다:

```
pnpm install
```

### 데이터베이스 생성

이 튜토리얼을 완료하려면 Postgres 데이터베이스가 필요하다. 로컬 컴퓨터에 Postgres가 설정되어 있지 않다면 다음 방법 중 하나를 선택할 수 있다:

* [Vercel Postgres](https://vercel.com/docs/storage/vercel-postgres)로 무료 Postgres 데이터베이스 생성
* 로컬에 직접 설정하려면 [이 가이드](https://www.prisma.io/dataguide/postgresql/setting-up-a-local-postgresql-database)를 따른다

### 데이터베이스 마이그레이션

Postgres 데이터베이스를 준비했다면 연결 문자열을 환경 시크릿으로 추가해야 한다.

`.env.example` 파일을 복사하여 `.env`로 이름을 바꾼다.

새로 만든 `.env` 파일을 연다. `DATABASE_URL`이라는 항목을 볼 수 있다. 등호 뒤에 데이터베이스 연결 문자열을 붙여넣는다.

이제 첫 번째 데이터베이스 마이그레이션을 실행할 수 있다. 다음 명령을 실행한다:

이 명령은 먼저 데이터베이스에 `pgvector` 확장을 추가한다. 그런 다음 `lib/db/schema/resources.ts`에 정의된 `resources` 스키마에 대한 새 테이블을 생성한다. 이 스키마는 `id`, `content`, `createdAt`, `updatedAt` 네 개의 컬럼을 가진다.

마이그레이션에서 오류가 발생하면 마이그레이션 파일(`lib/db/migrations/0000_yielding_bloodaxe.sql`)을 열어 첫 번째 줄을 잘라내고(복사 및 제거) Postgres 인스턴스에서 직접 실행한다. 이제 업데이트된 마이그레이션을 실행할 수 있다. [자세한 정보](https://github.com/vercel/ai-sdk-rag-starter/issues/1)

### OpenAI API 키

이 가이드에서는 OpenAI API 키가 필요하다. API 키를 생성하려면 [platform.openai.com](http://platform.openai.com/)으로 이동한다.

API 키를 받았다면 `.env` 파일의 `OPENAI_API_KEY`에 붙여넣는다.

## 만들기

구현해야 할 작업 목록을 정리해보자:

1. 임베딩을 저장할 데이터베이스 테이블 생성
2. 리소스 생성 시 데이터를 청크로 나누고 임베딩을 생성하는 로직 추가
3. 챗봇 생성 
4. 챗봇에 지식 기반을 검색하고 생성할 수 있는 도구 제공

### 임베딩 테이블 생성

현재 애플리케이션에는 콘텐츠를 저장하는 컬럼(`content`)이 있는 하나의 테이블(`resources`)이 있다. 각 `리소스`(소스 자료)는 청크로 나누어 임베딩한 후 저장해야 한다. 이러한 청크를 저장하기 위해 `embeddings`라는 새로운 테이블을 만들자.

새 파일(`lib/db/schema/embeddings.ts`)을 생성하고 다음 코드를 추가한다:

```typescript
import { nanoid } from '@/lib/utils';
import { index, pgTable, text, varchar, vector } from 'drizzle-orm/pg-core';
import { resources } from './resources';

export const embeddings = pgTable(
  'embeddings',
  {
    id: varchar('id', { length: 191 })
      .primaryKey()
      .$defaultFn(() => nanoid()),
    resourceId: varchar('resource_id', { length: 191 }).references(
      () => resources.id,
      { onDelete: 'cascade' },
    ),
    content: text('content').notNull(),
    embedding: vector('embedding', { dimensions: 1536 }).notNull(),
  },
  table => ({
    embeddingIndex: index('embeddingIndex').using(
      'hnsw',
      table.embedding.op('vector_cosine_ops'),
    ),
  }),
);
```

이 테이블은 4개의 컬럼으로 구성된다:

* `id` - 고유 식별자
* `resourceId` - 전체 소스 자료와 연결하는 외래 키
* `content` - 일반 텍스트 청크
* `embedding` - 일반 텍스트 청크의 벡터 표현

유사도 검색의 성능을 높이기 위해 이 컬럼에 인덱스([HNSW](https://github.com/pgvector/pgvector?tab=readme-ov-file#hnsw) 또는 [IVFFlat](https://github.com/pgvector/pgvector?tab=readme-ov-file#ivfflat))도 추가해야 한다.

이 변경사항을 데이터베이스에 반영하려면 다음 명령어를 실행한다:

### 임베딩 로직 추가

이제 임베딩을 저장할 테이블이 준비되었으니, 임베딩을 생성하는 로직을 작성할 차례다.

다음 명령어로 새 파일을 만든다:

```bash
mkdir lib/ai && touch lib/ai/embedding.ts
```

### 청크 생성

임베딩을 생성하기 위해서는 먼저 소스 자료(길이 미정)를 작은 청크로 나누고, 각 청크를 임베딩한 후 데이터베이스에 저장해야 한다. 우선 소스 자료를 작은 청크로 나누는 함수를 만들어보자.

```typescript
const generateChunks = (input: string): string[] => {
  return input
    .trim()
    .split('.')
    .filter(i => i !== '');
};
```

이 함수는 입력 문자열을 마침표를 기준으로 나누고 빈 항목을 제거하여 문자열 배열을 반환한다. 프로젝트에 따라 다양한 청크 생성 기법을 시도해볼 수 있다.

### AI SDK 설치

임베딩을 생성하기 위해 AI SDK를 사용한다. 다음 명령어로 두 개의 의존성 패키지를 추가로 설치한다:

```bash
pnpm add ai @ai-sdk/openai
```

이렇게 하면 [AI SDK](https://sdk.vercel.ai/docs)와 [OpenAI 프로바이더](https://sdk.vercel.ai/providers/ai-sdk-providers/openai)가 설치된다.

### 임베딩 생성

이제 임베딩을 생성하는 함수를 추가한다. 다음 코드를 `lib/ai/embedding.ts` 파일에 복사한다.

```typescript
import { embedMany } from 'ai';
import { openai } from '@ai-sdk/openai';

const embeddingModel = openai.embedding('text-embedding-ada-002');

const generateChunks = (input: string): string[] => {
  return input
    .trim()
    .split('.')
    .filter(i => i !== '');
};

export const generateEmbeddings = async (
  value: string,
): Promise<Array<{ embedding: number[]; content: string }>> => {
  const chunks = generateChunks(value);
  const { embeddings } = await embedMany({
    model: embeddingModel,
    values: chunks,
  });
  return embeddings.map((e, i) => ({ content: chunks[i], embedding: e }));
};
```

이 코드에서는 먼저 임베딩에 사용할 모델을 정의한다. 여기서는 OpenAI의 `text-embedding-ada-002` 임베딩 모델을 사용한다.

다음으로 `generateEmbeddings`라는 비동기 함수를 만든다. 이 함수는 소스 자료(`value`)를 입력으로 받아 임베딩과 콘텐츠를 포함하는 객체 배열을 Promise로 반환한다. 함수 내부에서는 먼저 입력을 청크로 나눈다. 그런 다음 AI SDK에서 가져온 [`embedMany`](https://sdk.vercel.ai/docs/reference/ai-sdk-core/embed-many) 함수에 청크를 전달하여 임베딩을 생성한다. 마지막으로 임베딩을 데이터베이스에 저장할 수 있는 형식으로 매핑하여 반환한다.

### 서버 액션 업데이트

`lib/actions/resources.ts` 파일을 열어보자. 이 파일에는 이름 그대로 리소스를 생성하는 `createResource` 함수가 하나 있다.

```typescript
'use server';

import {
  NewResourceParams,
  insertResourceSchema,
  resources,
} from '@/lib/db/schema/resources';
import { db } from '../db';

export const createResource = async (input: NewResourceParams) => {
  try {
    const { content } = insertResourceSchema.parse(input);
    const [resource] = await db
      .insert(resources)
      .values({ content })
      .returning();
    return 'Resource successfully created.';
  } catch (e) {
    if (e instanceof Error)
      return e.message.length > 0 ? e.message : 'Error, please try again.';
  }
};
```

이 함수는 파일 상단의 `"use server";` 지시문이 나타내듯이 [서버 액션](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#with-client-components)이다. 이는 Next.js 애플리케이션 어디에서나 호출할 수 있다는 의미다. 이 함수는 입력을 받아 [Zod](https://zod.dev/) 스키마로 검증한 후 데이터베이스에 새 리소스를 생성한다. 여기가 바로 새로 생성된 리소스의 임베딩을 생성하고 저장하는 로직을 추가하기에 적절한 위치다.

파일을 다음 코드로 업데이트한다:

```typescript
'use server';

import {
  NewResourceParams,
  insertResourceSchema,
  resources,
} from '@/lib/db/schema/resources';
import { db } from '../db';
import { generateEmbeddings } from '../ai/embedding';
import { embeddings as embeddingsTable } from '../db/schema/embeddings';

export const createResource = async (input: NewResourceParams) => {
  try {
    const { content } = insertResourceSchema.parse(input);
    const [resource] = await db
      .insert(resources)
      .values({ content })
      .returning();

    const embeddings = await generateEmbeddings(content);
    await db.insert(embeddingsTable).values(
      embeddings.map(embedding => ({
        resourceId: resource.id,
        ...embedding,
      })),
    );

    return 'Resource successfully created and embedded.';
  } catch (error) {
    return error instanceof Error && error.message.length > 0
      ? error.message
      : 'Error, please try again.';
  }
};
```

먼저 이전 단계에서 만든 `generateEmbeddings` 함수를 호출하여 소스 자료(`content`)의 임베딩을 생성한다. 임베딩(`e`)이 생성되면 각 임베딩과 함께 `resourceId`를 전달하여 데이터베이스에 저장한다.

### 루트 페이지 생성

좋다! 이제 프론트엔드를 만들어보자. AI SDK의 [`useChat`](https://sdk.vercel.ai/docs/reference/ai-sdk-ui/use-chat) 훅을 사용하면 챗봇 애플리케이션을 위한 대화형 사용자 인터페이스를 쉽게 만들 수 있다.

루트 페이지(`app/page.tsx`)를 다음 코드로 교체한다.

```typescript
'use client';

import { useChat } from 'ai/react';

export default function Chat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat();

  return (
    <div className="flex flex-col w-full max-w-md py-24 mx-auto stretch">
      <div className="space-y-4">
        {messages.map(m => (
          <div key={m.id} className="whitespace-pre-wrap">
            <div>
              <div className="font-bold">{m.role}</div>
              <p>{m.content}</p>
            </div>
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit}>
        <input
          className="fixed bottom-0 w-full max-w-md p-2 mb-8 border border-gray-300 rounded shadow-xl"
          value={input}
          placeholder="Say something..."
          onChange={handleInputChange}
        />
      </form>
    </div>
  );
}
```

`useChat` 훅은 AI 프로바이더(여기서는 OpenAI를 사용)로부터 채팅 메시지를 스트리밍하고, 채팅 입력의 상태를 관리하며, 새 메시지가 수신될 때마다 UI를 자동으로 업데이트한다.

Next.js 개발 서버를 시작하기 위해 다음 명령어를 실행한다:

[http://localhost:3000](http://localhost:3000/)으로 이동하자. 하단에 입력창이 떠 있는 빈 화면이 보일 것이다. 메시지를 보내보자. 메시지가 UI에 잠깐 표시되었다가 사라진다. 이는 모델을 호출할 API 라우트를 아직 설정하지 않았기 때문이다! 기본적으로 `useChat`은 `messages`를 요청 본문에 담아 `/api/chat` 엔드포인트로 POST 요청을 보낸다.

useChat 설정 객체에서 엔드포인트를 커스터마이징할 수 있다.

### API 라우트 생성

Next.js에서는 [라우트 핸들러](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)를 사용하여 특정 라우트에 대한 커스텀 요청 핸들러를 만들 수 있다. 라우트 핸들러는 `route.ts` 파일에 정의되며 `GET`, `POST`, `PUT`, `PATCH` 등의 HTTP 메서드를 내보낼 수 있다.

다음 명령어로 `app/api/chat/route.ts` 파일을 생성한다:

```bash
mkdir -p app/api/chat && touch app/api/chat/route.ts
```

파일을 열고 다음 코드를 추가한다:

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

// 응답 스트리밍 시간을 최대 30초로 제한한다
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({
    model: openai('gpt-4o'),
    messages,
  });
  return result.toDataStreamResponse();
}
```

이 코드에서는 POST라는 비동기 함수를 선언하고 내보낸다. 요청 본문에서 `messages`를 추출한 후, AI SDK에서 가져온 [`streamText`](https://sdk.vercel.ai/docs/reference/ai-sdk-core/stream-text) 함수에 사용할 모델과 함께 전달한다. 마지막으로 모델의 응답을 `AIStreamResponse` 형식으로 반환한다.

브라우저로 돌아가서 다시 메시지를 보내보자. 이제 모델의 응답이 실시간으로 스트리밍되는 것을 확인할 수 있다!

### 프롬프트 개선하기

이제 작동하는 챗봇이 생겼지만, 특별한 기능은 없는 상태다.

시스템 지침을 추가해서 모델의 동작을 개선하고 제한해보자. 여기서는 모델이 검색한 정보만을 사용해서 응답하도록 만들고 싶다. 라우트 핸들러를 다음 코드로 업데이트한다:

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

// 응답 스트리밍 시간을 최대 30초로 제한한다
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({
    model: openai('gpt-4o'),
    system: `당신은 도움이 되는 어시스턴트입니다. 질문에 답하기 전에 지식 기반을 확인하세요.
    도구 호출에서 얻은 정보만을 사용해서 답변하세요.
    도구 호출에서 관련 정보를 찾지 못한 경우 "죄송합니다. 제가 모르는 내용입니다."라고 답변하세요.`,
    messages,
  });
  return result.toDataStreamResponse();
}
```

브라우저로 돌아가서 모델에게 좋아하는 음식이 무엇인지 물어보자. 관련 정보가 없으므로 모델은 지시된 대로 "죄송합니다. 제가 모르는 내용입니다."라고 답변할 것이다.

현재 상태에서 챗봇은 사실상 쓸모가 없다. 모델에게 정보를 추가하고 검색할 수 있는 능력을 어떻게 부여할 수 있을까?

### 도구 사용하기

[도구](https://sdk.vercel.ai/docs/foundations/tools)는 모델이 특정 작업을 수행하기 위해 호출할 수 있는 함수다. 도구는 모델에게 제공하는 프로그램처럼 생각할 수 있으며, 모델은 필요할 때마다 이를 실행할 수 있다.

챗봇의 지식 기반에 리소스를 생성하고, 임베딩하고, 저장할 수 있는 능력을 모델에게 부여하는 도구를 어떻게 만드는지 알아보자.

### 리소스 추가 도구

라우트 핸들러를 다음 코드로 업데이트한다:

```typescript
import { createResource } from '@/lib/actions/resources';
import { openai } from '@ai-sdk/openai';
import { streamText, tool } from 'ai';
import { z } from 'zod';

// 응답 스트리밍 시간을 최대 30초로 제한한다
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({
    model: openai('gpt-4o'),
    system: `당신은 도움이 되는 어시스턴트입니다. 질문에 답하기 전에 지식 기반을 확인하세요.
    도구 호출에서 얻은 정보만을 사용해서 답변하세요.
    도구 호출에서 관련 정보를 찾지 못한 경우 "죄송합니다. 제가 모르는 내용입니다."라고 답변하세요.`,
    messages,
    tools: {
      addResource: tool({
        description: `지식 기반에 리소스를 추가한다.
          사용자가 임의의 지식을 자발적으로 제공하는 경우, 확인을 요청하지 않고 이 도구를 사용한다.`,
        parameters: z.object({
          content: z
            .string()
            .describe('지식 기반에 추가할 콘텐츠나 리소스'),
        }),
        execute: async ({ content }) => createResource({ content }),
      }),
    },
  });
  return result.toDataStreamResponse();
}
```

이 코드에서는 `addResource`라는 도구를 정의한다. 이 도구는 세 가지 요소로 구성된다:

* **description**: 도구가 언제 선택될지에 영향을 미치는 설명
* **parameters**: 도구 실행에 필요한 매개변수를 정의하는 [Zod 스키마](https://sdk.vercel.ai/docs/foundations/tools#schema-specification-and-validation-with-zod)
* **execute**: 도구 호출의 인자로 실행되는 비동기 함수

간단히 말해서, 모델은 매 생성마다 도구를 호출할지 결정한다. 도구를 호출하기로 결정하면, 입력에서 매개변수를 추출하여 `tool-call` 타입의 새로운 `message`를 `messages` 배열에 추가한다. 그러면 AI SDK가 `tool-call` 메시지가 제공한 매개변수로 `execute` 함수를 실행한다.

브라우저로 돌아가서 모델에게 좋아하는 음식을 알려주자. UI에는 빈 응답이 표시된다. 뭔가 일어났을까? 새 터미널 창에서 다음 명령어를 실행해보자.

이 명령은 데이터베이스의 행을 볼 수 있는 Drizzle Studio를 시작한다. `embeddings`와 `resources` 테이블 모두에 좋아하는 음식에 대한 새로운 행이 생성된 것을 확인할 수 있다!

도구가 호출됐을 때 사용자에게 알려주도록 UI를 약간 수정해보자. 루트 페이지(`app/page.tsx`)로 돌아가서 다음 코드를 추가한다:

```typescript
'use client';

import { useChat } from 'ai/react';

export default function Chat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat();

  return (
    <div className="flex flex-col w-full max-w-md py-24 mx-auto stretch">
      <div className="space-y-4">
        {messages.map(m => (
          <div key={m.id} className="whitespace-pre-wrap">
            <div>
              <div className="font-bold">{m.role}</div>
              <p>
                {m.content.length > 0 ? (
                  m.content
                ) : (
                  <span className="italic font-light">
                    {'도구 호출 중: ' + m?.toolInvocations?.[0].toolName}
                  </span>
                )}
              </p>
            </div>
          </div>
        ))}
      </div>

      <form onSubmit={handleSubmit}>
        <input
          className="fixed bottom-0 w-full max-w-md p-2 mb-8 border border-gray-300 rounded shadow-xl"
          value={input}
          placeholder="대화를 시작해보세요..."
          onChange={handleInputChange}
        />
      </form>
    </div>
  );
}
```

이제 모델의 일반적인 텍스트 응답 대신 호출된 도구를 UI에 직접 표시한다. 파일을 저장하고 브라우저로 돌아가자. 모델에게 좋아하는 영화를 알려주면 어떤 도구가 호출되는지 볼 수 있다.

### 다단계 호출로 UX 개선하기

모델이 수행한 작업도 요약해주면 좋을 것 같다. 하지만 기술적으로 모델이 도구를 호출하면 '도구 호출'을 생성했으므로 생성이 완료된다. 어떻게 이 원하는 동작을 구현할 수 있을까?

AI SDK에는 [`maxSteps`](https://sdk.vercel.ai/docs/ai-sdk-core/tools-and-tool-calling#multi-step-calls)라는 기능이 있어서 도구 호출 결과를 자동으로 모델에게 다시 전달한다!

루트 페이지(`app/page.tsx`)를 열고 `useChat` 설정 객체에 다음 속성을 추가한다:

```typescript
// ... 나머지 코드는 그대로
const { messages, input, handleInputChange, handleSubmit } = useChat({
  maxSteps: 3,
});
// ... 나머지 코드는 그대로
```

브라우저로 돌아가서 모델에게 좋아하는 피자 토핑을 알려주자(참고: 파인애플은 선택지에 없다). 모델이 작업을 확인하는 후속 응답을 보여줄 것이다.

### 리소스 검색 도구

이제 모델은 임의의 정보를 지식 기반에 추가하고 임베딩할 수 있다. 하지만 아직 검색은 할 수 없다. 모델이 지식 기반에서 관련 정보를 찾아 질문에 답할 수 있도록 새로운 도구를 만들어보자.

유사한 콘텐츠를 찾으려면 사용자의 질문을 임베딩하고, 데이터베이스에서 의미적 유사성을 검색한 다음, 찾은 항목을 질문과 함께 모델에 컨텍스트로 전달해야 한다. 이를 위해 임베딩 로직 파일(`lib/ai/embedding.ts`)을 업데이트하자:

```typescript
import { embed, embedMany } from 'ai';
import { openai } from '@ai-sdk/openai';
import { db } from '../db';
import { cosineDistance, desc, gt, sql } from 'drizzle-orm';
import { embeddings } from '../db/schema/embeddings';

const embeddingModel = openai.embedding('text-embedding-ada-002');

const generateChunks = (input: string): string[] => {
  return input
    .trim()
    .split('.')
    .filter(i => i !== '');
};

export const generateEmbeddings = async (
  value: string,
): Promise<Array<{ embedding: number[]; content: string }>> => {
  const chunks = generateChunks(value);
  const { embeddings } = await embedMany({
    model: embeddingModel,
    values: chunks,
  });
  return embeddings.map((e, i) => ({ content: chunks[i], embedding: e }));
};

export const generateEmbedding = async (value: string): Promise<number[]> => {
  const input = value.replaceAll('\n', ' ');
  const { embedding } = await embed({
    model: embeddingModel,
    value: input,
  });
  return embedding;
};

export const findRelevantContent = async (userQuery: string) => {
  const userQueryEmbedded = await generateEmbedding(userQuery);
  const similarity = sql<number>`1 - (${cosineDistance(
    embeddings.embedding,
    userQueryEmbedded,
  )})`;

  const similarGuides = await db
    .select({ name: embeddings.content, similarity })
    .from(embeddings)
    .where(gt(similarity, 0.5))
    .orderBy(t => desc(t.similarity))
    .limit(4);

  return similarGuides;
};
```

이 코드에서는 두 가지 함수를 추가한다:

* `generateEmbedding`: 입력 문자열에서 하나의 임베딩을 생성한다
* `findRelevantContent`: 사용자의 질문을 임베딩하고, 데이터베이스에서 유사한 항목을 찾아서 반환한다

마지막 단계다: 도구를 만들자.

라우트 핸들러(`api/chat/route.ts`)로 돌아가서 `getInformation`이라는 새로운 도구를 추가한다:

```typescript
import { createResource } from '@/lib/actions/resources';
import { openai } from '@ai-sdk/openai';
import { streamText, tool } from 'ai';
import { z } from 'zod';
import { findRelevantContent } from '@/lib/ai/embedding';

// 응답 스트리밍 시간을 최대 30초로 제한한다
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({
    model: openai('gpt-4o'),
    messages,
    system: `당신은 도움이 되는 어시스턴트입니다. 질문에 답하기 전에 지식 기반을 확인하세요.
    도구 호출에서 얻은 정보만을 사용해서 답변하세요.
    도구 호출에서 관련 정보를 찾지 못한 경우 "죄송합니다. 제가 모르는 내용입니다."라고 답변하세요.`,
    tools: {
      addResource: tool({
        description: `지식 기반에 리소스를 추가한다.
          사용자가 임의의 지식을 자발적으로 제공하는 경우, 확인을 요청하지 않고 이 도구를 사용한다.`,
        parameters: z.object({
          content: z
            .string()
            .describe('지식 기반에 추가할 콘텐츠나 리소스'),
        }),
        execute: async ({ content }) => createResource({ content }),
      }),
      getInformation: tool({
        description: `지식 기반에서 정보를 가져와 질문에 답변한다.`,
        parameters: z.object({
          question: z.string().describe('사용자의 질문'),
        }),
        execute: async ({ question }) => findRelevantContent(question),
      }),
    },
  });
  return result.toDataStreamResponse();
}
```

브라우저로 돌아가서 페이지를 새로고침한 다음, 좋아하는 음식이 무엇인지 물어보자. 모델이 `getInformation` 도구를 호출하고, 관련 정보를 사용해서 응답을 만드는 것을 볼 수 있다!

## 마무리

축하한다! 지식 기반에서 정보를 동적으로 추가하고 검색할 수 있는 AI 챗봇을 성공적으로 구축했다. 이 가이드를 통해 임베딩을 생성하고 저장하는 방법, 리소스를 관리하는 서버 액션을 설정하는 방법, 그리고 챗봇의 기능을 확장하기 위한 도구 사용법을 배웠다.