---
title: "Claude Desktop Extensions: 클로드 데스크톱용 원클릭 MCP 서버 설치"
date: 2025-06-28T01:10:04Z
author: "skyfe79"
draft: false
tags: ["ai"]
---

> 원문: https://www.anthropic.com/engineering/desktop-extensions

작년에 Model Context Protocol(MCP)을 출시했을 때, 개발자들이 파일 시스템부터 데이터베이스에 이르기까지 클로드에게 접근 권한을 부여하는 놀라운 로컬 서버를 구축하는 모습을 보았다. 하지만 우리는 같은 피드백을 계속해서 들었다. 설치 과정이 너무 복잡하다는 것이었다. 사용자들은 개발자 도구가 필요했고, 설정 파일을 수동으로 편집해야 했으며, 종종 의존성 문제에 막혔다.

오늘, 우리는 Desktop Extensions를 소개한다. MCP 서버 설치를 단순히 버튼 클릭만으로 가능하게 해주는 새로운 패키징 형식이다.


### MCP 설치 문제 해결하기

로컬 MCP 서버는 Claude Desktop 사용자에게 강력한 기능을 제공한다. 로컬 애플리케이션과 상호작용하고, 개인 데이터에 접근하며, 개발 도구와 통합할 수 있다. 이 모든 작업을 사용자의 기기 내에서 데이터를 유지하며 수행할 수 있다. 하지만 현재 설치 프로세스는 다음과 같은 문제점을 가지고 있다:

- **개발자 도구 필요**: 사용자는 Node.js, Python 또는 기타 런타임을 설치해야 한다.
- **수동 설정**: 각 서버마다 JSON 설정 파일을 편집해야 한다.
- **의존성 관리**: 사용자는 패키지 충돌과 버전 불일치 문제를 직접 해결해야 한다.
- **발견 메커니즘 부재**: 유용한 MCP 서버를 찾기 위해 GitHub를 검색해야 한다.
- **업데이트 복잡성**: 서버를 최신 상태로 유지하려면 수동으로 재설치해야 한다.

이러한 문제점들로 인해 MCP 서버는 강력한 기능을 가지고 있음에도 불구하고, 비기술 사용자들에게는 접근하기 어려운 상태로 남아 있다.


### 데스크톱 확장 기능 소개

데스크톱 확장 기능(.dxt 파일)은 모든 종속성을 포함한 MCP 서버 전체를 단일 설치 패키지로 묶어 이러한 문제를 해결한다. 사용자에게는 다음과 같은 변화가 생긴다:

**이전 방식:**

```
# 먼저 Node.js 설치

npm install -g @example/mcp-server
# ~/.claude/claude_desktop_config.json 파일을 수동으로 편집

# Claude Desktop 재시작

# 제대로 작동하기를 바람

```

**새로운 방식:**

1. .dxt 파일 다운로드
2. Claude Desktop으로 파일 열기
3. "설치" 클릭

이게 전부다. 더 이상 터미널이나 설정 파일, 종속성 충돌을 걱정할 필요가 없다.


## 아키텍처 개요

데스크톱 익스텐션은 로컬 MCP 서버와 `manifest.json` 파일을 포함한 ZIP 아카이브이다. `manifest.json` 파일은 Claude 데스크톱과 다른 앱들이 데스크톱 익스텐션을 지원하기 위해 필요한 모든 정보를 담고 있다.

```
extension.dxt (ZIP 아카이브)
├── manifest.json         # 익스텐션 메타데이터와 설정
├── server/               # MCP 서버 구현
│   └── [서버 파일들]    
├── dependencies/         # 필요한 패키지/라이브러리
└── icon.png             # 선택사항: 익스텐션 아이콘

# 예제: Node.js 익스텐션

extension.dxt
├── manifest.json         # 필수: 익스텐션 메타데이터와 설정
├── server/               # 서버 파일
│   └── index.js          # 메인 진입점
├── node_modules/         # 번들된 의존성
├── package.json          # 선택사항: NPM 패키지 정의
└── icon.png              # 선택사항: 익스텐션 아이콘

# 예제: Python 익스텐션

extension.dxt (ZIP 파일)
├── manifest.json         # 필수: 익스텐션 메타데이터와 설정
├── server/               # 서버 파일
│   ├── main.py           # 메인 진입점
│   └── utils.py          # 추가 모듈
├── lib/                  # 번들된 Python 패키지
├── requirements.txt      # 선택사항: Python 의존성 목록
└── icon.png              # 선택사항: 익스텐션 아이콘
```

데스크톱 익스텐션에서 필수 파일은 `manifest.json`뿐이다. Claude 데스크톱은 모든 복잡성을 처리한다:

*   **내장 런타임**: Claude 데스크톱에 Node.js를 포함해 외부 의존성을 제거한다.
*   **자동 업데이트**: 새로운 버전이 나오면 익스텐션을 자동으로 업데이트한다.
*   **보안 비밀**: API 키와 같은 민감한 설정은 OS 키체인에 안전하게 저장한다.

`manifest.json`에는 사람이 읽을 수 있는 정보(이름, 설명, 저자 등), 기능 선언(도구, 프롬프트), 사용자 설정, 런타임 요구 사항이 포함된다. 대부분의 필드는 선택사항이므로 최소 버전은 매우 짧다. 하지만 실제로는 지원되는 세 가지 익스텐션 타입(Node.js, Python, 클래식 바이너리/실행 파일) 모두 파일을 포함할 것으로 예상한다:

```
{
  "dxt_version": "0.1",                     // 이 manifest가 따르는 DXT 스펙 버전
  "name": "my-extension",                   // 머신 리더블 이름 (CLI, API에서 사용)
  "version": "1.0.0",                       // 익스텐션의 시맨틱 버전
  "description": "A simple MCP extension",  // 익스텐션 기능에 대한 간단한 설명
  "author": {                               // 저자 정보 (필수)
    "name": "Extension Author"              // 저자 이름 (필수 필드)
  },
  "server": {                               // 서버 설정 (필수)
    "type": "node",                         // 서버 타입: "node", "python", 또는 "binary"
    "entry_point": "server/index.js",       // 메인 서버 파일 경로
    "mcp_config": {                         // MCP 서버 설정
      "command": "node",                    // 서버를 실행할 커맨드
      "args": [                             // 커맨드에 전달할 인자
        "${__dirname}/server/index.js"      // ${__dirname}은 익스텐션 디렉토리로 대체됨
      ]                              
    }
  }
}
```

`manifest.json` 스펙에는 로컬 MCP 서버의 설치와 설정을 쉽게 하기 위한 다양한 편의 옵션이 있다. 서버 설정 객체는 템플릿 리터럴 형태의 사용자 정의 설정과 플랫폼별 오버라이드를 모두 수용할 수 있게 정의할 수 있다. 익스텐션 개발자는 사용자로부터 어떤 종류의 설정을 수집할지 상세히 정의할 수 있다.

`manifest.json`이 설정을 어떻게 도와주는지 구체적인 예를 살펴보자. 아래 manifest에서 개발자는 사용자가 `api_key`를 제공해야 한다고 선언한다. Claude는 사용자가 해당 값을 제공할 때까지 익스텐션을 활성화하지 않으며, 운영 체제의 비밀 저장소에 자동으로 저장하고, 서버를 실행할 때 `${user_config.api_key}`를 사용자가 제공한 값으로 대체한다. 마찬가지로 `${__dirname}`은 익스텐션의 압축 해제된 디렉토리의 전체 경로로 대체된다.

```
{
  "dxt_version": "0.1",
  "name": "my-extension",
  "version": "1.0.0",
  "description": "A simple MCP extension",
  "author": {
    "name": "Extension Author"
  },
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"],
      "env": {
        "API_KEY": "${user_config.api_key}"
      }
    }
  },
  "user_config": {
    "api_key": {
      "type": "string",
      "title": "API Key",
      "description": "Your API key for authentication",
      "sensitive": true,
      "required": true
    }
  }
}
```

대부분의 선택 필드를 포함한 전체 `manifest.json`은 다음과 같다:

```
{
  "dxt_version": "0.1",
  "name": "My MCP Extension",
  "display_name": "My Awesome MCP Extension",
  "version": "1.0.0",
  "description": "A brief description of what this extension does",
  "long_description": "A detailed description that can include multiple paragraphs explaining the extension's functionality, use cases, and features. It supports basic markdown.",
  "author": {
    "name": "Your Name",
    "email": "yourname@example.com",
    "url": "https://your-website.com"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/your-username/my-mcp-extension"
  },
  "homepage": "https://example.com/my-extension",
  "documentation": "https://docs.example.com/my-extension",
  "support": "https://github.com/your-username/my-mcp-extension/issues",
  "icon": "icon.png",
  "screenshots": [
    "assets/screenshots/screenshot1.png",
    "assets/screenshots/screenshot2.png"
  ],
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"],
      "env": {
        "ALLOWED_DIRECTORIES": "${user_config.allowed_directories}"
      }
    }
  },
  "tools": [
    {
      "name": "search_files",
      "description": "Search for files in a directory"
    }
  ],
  "prompts": [
    {
      "name": "poetry",
      "description": "Have the LLM write poetry",
      "arguments": ["topic"],
      "text": "Write a creative poem about the following topic: ${arguments.topic}"
    }
  ],
  "tools_generated": true,
  "keywords": ["api", "automation", "productivity"],
  "license": "MIT",
  "compatibility": {
    "claude_desktop": ">=1.0.0",
    "platforms": ["darwin", "win32", "linux"],
    "runtimes": {
      "node": ">=16.0.0"
    }
  },
  "user_config": {
    "allowed_directories": {
      "type": "directory",
      "title": "Allowed Directories",
      "description": "Directories the server can access",
      "multiple": true,
      "required": true,
      "default": ["${HOME}/Desktop"]
    },
    "api_key": {
      "type": "string",
      "title": "API Key",
      "description": "Your API key for authentication",
      "sensitive": true,
      "required": false
    },
    "max_file_size": {
      "type": "number",
      "title": "Maximum File Size (MB)",
      "description": "Maximum file size to process",
      "default": 10,
      "min": 1,
      "max": 100
    }
  }
}
```

익스텐션과 manifest 예제는 [dxt 저장소의 예제](https://github.com/anthropics/dxt/tree/main/examples)를 참고하면 된다.

`manifest.json`의 모든 필수 및 선택 필드에 대한 전체 스펙은 [오픈소스 툴체인](https://github.com/anthropics/dxt/blob/main/MANIFEST.md)에서 확인할 수 있다.


### 첫 번째 확장 프로그램 만들기

기존 MCP 서버를 데스크톱 확장 프로그램으로 패키징하는 과정을 살펴보자. 간단한 파일 시스템 서버를 예제로 사용한다.


#### 1단계: 매니페스트 생성

먼저, 서버를 위한 매니페스트를 초기화한다:

```
npx @anthropic-ai/dxt init
```

이 대화형 도구는 서버에 대한 정보를 묻고 완전한 manifest.json 파일을 생성한다. 가장 기본적인 manifest.json 파일을 빠르게 생성하려면 --yes 매개변수와 함께 명령어를 실행하면 된다.


#### Step 2: 사용자 설정 처리

서버가 API 키나 허용된 디렉토리와 같은 사용자 입력을 필요로 한다면, 매니페스트 파일에 다음과 같이 선언한다:

```
"user_config": {
  "allowed_directories": {
    "type": "directory",
    "title": "Allowed Directories",
    "description": "Directories the server can access",
    "multiple": true,
    "required": true,
    "default": ["${HOME}/Documents"]
  }
}
```

Claude Desktop은 다음과 같은 작업을 수행한다:

* 사용자 친화적인 설정 UI를 표시한다
* 확장 기능을 활성화하기 전에 입력값을 검증한다
* 민감한 값을 안전하게 저장한다
* 개발자 설정에 따라 사용자 설정을 인자나 환경 변수로 서버에 전달한다

아래 예제에서는 사용자 설정을 환경 변수로 전달하지만, 인자로 전달할 수도 있다.

```
"server": {
   "type": "node",
   "entry_point": "server/index.js",
   "mcp_config": {
   "command": "node",
   "args": ["${__dirname}/server/index.js"],
   "env": {
      "ALLOWED_DIRECTORIES": "${user_config.allowed_directories}"
   }
   }
}
```


#### 3단계: 확장 기능 패키징

모든 파일을 `.dxt` 파일로 묶는다:

```
npx @anthropic-ai/dxt pack
```

이 커맨드는 다음 작업을 수행한다:

1. 매니페스트 파일을 검증한다
2. `.dxt` 아카이브를 생성한다


#### 4단계: 로컬에서 테스트하기

`.dxt` 파일을 Claude Desktop의 설정 윈도우로 드래그 앤 드롭한다. 그러면 다음과 같은 정보를 확인할 수 있다:

*   확장 기능에 대한 사람이 읽을 수 있는 정보
*   필요한 권한 및 구성 설정
*   간단한 "설치" 버튼

이 과정을 통해 확장 기능을 로컬 환경에서 테스트하고 설치할 수 있다.


### 고급 기능

#### 크로스 플랫폼 지원

확장 기능은 다양한 운영체제에 맞게 조정할 수 있다:

```
"server": {
  "type": "node",
  "entry_point": "server/index.js",
  "mcp_config": {
    "command": "node",
    "args": ["${__dirname}/server/index.js"],
    "platforms": {
      "win32": {
        "command": "node.exe",
        "env": {
          "TEMP_DIR": "${TEMP}"
        }
      },
      "darwin": {
        "env": {
          "TEMP_DIR": "${TMPDIR}"
        }
      }
    }
  }
}
```


#### 동적 설정

런타임 값에 템플릿 리터럴을 사용한다:

* `${__dirname}`: 확장 기능의 설치 디렉토리
* `${user_config.key}`: 사용자가 제공한 설정 값
* `${HOME}, ${TEMP}`: 시스템 환경 변수


#### 기능 정의

사용자가 기능을 미리 이해할 수 있도록 명확히 정의한다:

```
"tools": [
  {
    "name": "read_file",
    "description": "파일 내용을 읽는다"
  }
],
"prompts": [
  {
    "name": "code_review",
    "description": "코드 리뷰를 통해 모범 사례를 확인한다",
    "arguments": ["file_path"]
  }
]
```


### 확장 기능 디렉토리

클로드 데스크톱에 내장된 확장 기능 디렉토리를 공개한다. 사용자는 GitHub에서 코드를 검색하거나 직접 확인할 필요 없이, 원클릭으로 원하는 확장 기능을 탐색하고 설치할 수 있다.

클로드 데스크톱 확장 기능 사양과 macOS 및 Windows용 클로드 구현은 시간이 지남에 따라 발전할 것으로 예상되지만, 확장 기능이 클로드의 능력을 창의적으로 확장하는 다양한 방법을 기대해 본다.

확장 기능을 제출하려면 다음 단계를 따른다:

1. 제출 양식에 나온 가이드라인을 준수한다
2. Windows와 macOS에서 테스트를 완료한다
3. [확장 기능 제출](https://docs.google.com/forms/d/14_Dmcig4z8NeRMB_e7TOyrKzuZ88-BLYdLvS6LPhiZU/edit)
4. 팀에서 품질과 보안을 검토한다


### 오픈 생태계 구축

MCP 서버 주변의 오픈 생태계에 대한 우리의 의지는 확고하다. MCP가 다양한 애플리케이션과 서비스에 의해 보편적으로 채택될 수 있는 능력은 커뮤니티에 큰 혜택을 주었다. 이러한 의지에 발맞춰, 우리는 데스크톱 확장(Desktop Extension) 사양, 툴체인, 그리고 Claude가 macOS와 Windows에서 데스크톱 확장을 지원하기 위해 사용하는 스키마와 핵심 기능을 오픈소스로 공개한다. dxt 형식이 단순히 Claude를 위한 로컬 MCP 서버의 이식성을 높이는 데 그치지 않고, 다른 AI 데스크톱 애플리케이션에서도 유용하게 사용되기를 바란다.

우리가 오픈소스로 공개하는 내용은 다음과 같다:

*   전체 DXT 사양
*   패키징 및 검증 도구
*   참조 구현 코드
*   TypeScript 타입과 스키마

이것이 의미하는 바는 다음과 같다:

*   **MCP 서버 개발자**: 한 번 패키징하면 DXT를 지원하는 모든 곳에서 실행 가능
*   **앱 개발자**: 처음부터 구축하지 않고도 확장 기능 지원 추가 가능
*   **사용자**: 모든 MCP 지원 애플리케이션에서 일관된 경험 제공

사양과 툴체인은 의도적으로 0.1 버전으로 출시했다. 우리는 더 큰 커뮤니티와 협력해 이 형식을 발전시키고 변경해 나가기를 기대한다. 여러분의 의견을 기다린다.


### 보안 및 기업 환경 고려사항

확장 기능은 특히 기업 환경에서 새로운 보안 문제를 야기할 수 있다. 이에 대해 데스크톱 확장 기능의 프리뷰 릴리스에 여러 가지 안전 장치를 마련했다:


#### 사용자를 위한 기능

* 민감한 데이터는 운영체제의 키체인에 안전하게 저장
* 자동 업데이트 기능 제공
* 설치된 확장 프로그램을 감사할 수 있는 기능


#### 기업용 기능

*   윈도우 그룹 정책(Group Policy)과 macOS MDM 지원
*   사전 승인된 확장 기능 미리 설치 가능
*   특정 확장 기능이나 퍼블리셔 차단
*   확장 기능 디렉토리 전체 비활성화
*   개인 확장 기능 디렉토리 배포

조직 내에서 확장 기능을 관리하는 방법에 대한 자세한 내용은 [문서](https://support.anthropic.com/en/articles/10949351-getting-started-with-model-context-protocol-mcp-on-claude-for-desktop)를 참고한다.


### 시작하기

자신만의 확장 기능을 만들 준비가 되었는가? 시작하는 방법은 다음과 같다:

**MCP 서버 개발자**를 위한 가이드: [개발자 문서](https://github.com/anthropics/dxt)를 확인하거나, 로컬 MCP 서버 디렉토리에서 다음 명령어를 실행해 바로 시작할 수 있다:

```
npm install -g @anthropic-ai/dxt
dxt init
dxt pack
```

**Claude Desktop 사용자**를 위한 가이드: 최신 버전으로 업데이트한 후 설정 메뉴에서 확장 기능 섹션을 찾아본다.

**기업 사용자**를 위한 가이드: 배포 옵션에 대한 엔터프라이즈 문서를 검토한다.


### Claude Code로 확장 기능 구축하기

Anthropic 내부에서 Claude를 사용해 최소한의 개입으로 확장 기능을 구축하는 데 뛰어난 성과를 거두었다. 여러분도 Claude Code를 활용하려면, 원하는 확장 기능이 무엇인지 간단히 설명한 후 다음 프롬프트에 컨텍스트를 추가한다.

```
이 확장 기능을 데스크톱 확장(Desktop Extension, 약어 "DXT")으로 구축하고 싶다. 다음 단계를 따라 진행한다:

1. **명세서를 철저히 읽는다:**
   - https://github.com/anthropics/dxt/blob/main/README.md - DXT 아키텍처 개요, 기능, 통합 패턴
   - https://github.com/anthropics/dxt/blob/main/MANIFEST.md - 확장 매니페스트 구조와 필드 정의 전체
   - https://github.com/anthropics/dxt/tree/main/examples - "Hello World" 예제를 포함한 참조 구현

2. **적절한 확장 구조를 만든다:**
   - MANIFEST.md 명세에 따라 유효한 manifest.json 생성
   - @modelcontextprotocol/sdk를 사용해 적절한 도구 정의와 함께 MCP 서버 구현
   - 적절한 오류 처리와 타임아웃 관리 포함

3. **최상의 개발 관행을 따른다:**
   - stdio 전송을 통해 적절한 MCP 프로토콜 통신 구현
   - 명확한 스키마, 검증, 일관된 JSON 응답을 가진 도구 구조화
   - 이 확장이 로컬에서 실행된다는 사실 활용
   - 적절한 로깅과 디버깅 기능 추가
   - 적절한 문서화와 설정 지침 포함

4. **테스트 고려사항:**
   - 모든 도구 호출이 올바르게 구조화된 응답을 반환하는지 검증
   - 매니페스트가 올바르게 로드되고 호스트 통합이 작동하는지 확인

생산 준비가 완료된 코드를 생성해 즉시 테스트할 수 있도록 한다. 방어적 프로그래밍, 명확한 오류 메시지, 정확한 DXT 명세 준수에 초점을 맞춰 생태계와의 호환성을 보장한다.
```


### 결론

데스크톱 익스텐션은 사용자가 로컬 AI 도구와 상호작용하는 방식을 근본적으로 바꿔놓았다. 설치 과정의 번거로움을 없앰으로써, 개발자뿐만 아니라 모든 이가 강력한 MCP 서버를 쉽게 사용할 수 있게 되었다.

내부적으로, 우리는 데스크톱 익스텐션을 통해 다양한 실험적인 MCP 서버를 공유하고 있다. 어떤 것은 재미를 위한 것이고, 어떤 것은 실용적인 목적을 가지고 있다. 한 팀은 [“Claude plays Pokémon” 연구](https://www.anthropic.com/news/visible-extended-thinking)와 유사하게, 모델이 GameBoy에 직접 연결되었을 때 얼마나 멀리 나아갈 수 있는지 실험했다. 데스크톱 익스텐션을 사용해 인기 있는 [PyBoy](https://github.com/Baekalfen/PyBoy) GameBoy 에뮬레이터를 열고 Claude가 제어할 수 있게 하는 단일 익스텐션을 패키징했다. 우리는 모델의 능력을 사용자가 이미 로컬 머신에 가지고 있는 도구, 데이터, 애플리케이션과 연결할 수 있는 무수한 기회가 있다고 믿는다.



![PyBoy MCP와 Super Mario Land 시작 화면을 보여주는 데스크톱](https://github.com/user-attachments/assets/f04bd36b-ed8d-4ea1-805e-d936038da634) 여러분이 어떤 것을 만들어낼지 기대가 크다. 수천 개의 MCP 서버를 만들어낸 그 창의성이 이제 단 한 번의 클릭으로 수백만 명의 사용자에게 도달할 수 있다. 

여러분의 MCP 서버를 공유할 준비가 되셨나요? [익스텐션을 리뷰를 위해 제출하세요](https://forms.gle/tyiAZvch1kDADKoP9).




