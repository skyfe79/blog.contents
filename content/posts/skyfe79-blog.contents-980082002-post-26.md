---
title: "Github Actions 로컬 개발 환경 구성하기"
date: 2021-08-26T10:24:00Z
draft: false
tags: ["github","actions"]
---

Github Actions를 개발하기 위해서 로컬 개발 환경을 구성하는 일은 중요하다. 로컬 개발 환경 구성 없이 개발한다면 Github Actions를 테스트하기 위해서 매번 커밋과 푸시를 반복해야 한다. 이 글에서 Github Actions를 생성하기 위해 로컬 개발환경을 구성하는 과정을 알아 본다.

## 필요한 도구

### Github Actions 시뮬레이터

[act](https://github.com/nektos/act)는 Github Actions 를 로컬 환경에서 테스트할 수 있게 도와주는 툴이다. 로컬 개발 환경 구성에 있어 필수 도구이다. 맥에서는 brew를 이용해 쉽게 설치할 수 있다.

```
$ brew install act
```

다른 환경에서 `act`를 설치하는 방법은 [act](https://github.com/nektos/act) 를 참고한다.

### ncc

[ncc](https://github.com/vercel/ncc)는 `go build` 또는 `rust`의 `cargo build`처럼 node.js 프로젝트를 단 하나의 파일로 만들어주는 도구이다. Github Actions를 배포할 때, node_modules 폴더를 같이 배포하지 않아도 되어 매우 편리하다. 

npm으로 쉽게 설치할 수 있다.

```
$ npm i -g @vercel/ncc
```

## Github에 Repo 만들기

대부분의 Github Actions는 작업을 수행하기 전에 repo를 먼저 클론한다. 따라서 이 과정을 시뮬레이션하기 위해서 실제 repo가 필요하다. 이 글에서는 예제로 [github-actions-starter-kit](https://github.com/skyfe79/github-actions-starter-kit) 레포지토리를 생성했다.

### 클론하기

[github-actions-starter-kit](https://github.com/skyfe79/github-actions-starter-kit)를 로컬 컴퓨터의 원하는 위치에 클론한다. 

```
$ git clone https://github.com/skyfe79/github-actions-starter-kit
$ cd github-actions-starter-kit
```

## 자바스크립트 개발 환경 구성

Github Actions를 Javascript로 만들 것이기 때문에 npm으로 개발 환경을 구성한다.

```
$ npm init -y
```

### 필수 패키지 설치

Github Actions를 개발할 때, Github REST API를 사용하는 경우가 많기 때문에 아래의 패키지를 설치한다.

```
$ npm install @actions/core --save
$ npm install @actions/github --save
```

## 액션 정의 파일 생성

Github Actions를 정의하는 파일은 `action.yml` 이다. 프로젝트 루트 폴더에 `action.yml`를 생성한다.

```
$ touch action.yml
```

### action.yml 작성

`action.yml` 파일을 편집기로 열어 아래와 같이 내용을 입력한다.

```yml
name: 'hello-my-action'
description: 'Hello My Action'
inputs:
  github-token:
    description: 'github token'
    required: true
runs:
  using: 'node12'
  main: 'dist/index.js'
```

위 파일의 내용은 Github Actions를 `hello-my-action`으로 명칭하고 `github-token`을 입력해야 함을 의미한다. 그리고 액션의 실행환경은 `node12`이며 실행파일은 `dist/index.js` 파일임을 나타낸다. 

더 자세한 내용은 [Github 문서](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)를 참고한다. 


## 액션 실행 파일 생성

Github Actions에서 액션을 실행하는 것을 `workflow`라고 한다. 액션을 실행하는 일련의 과정을 `.github/workflows` 폴더에 담아 놓는다. 따라서 아래와 같이 폴더를 생성하고 간단한 워크플로우를 생성해 보자.

```
$ mkdir -p .github/workflows
$ touch .github/workflows/main.yml
```

### 워크플로우 작성

`main.yml` 파일을 열고 `dist/index.js` 파일을 실행하게 도록 작성해 보자.

```yml
on:
  workflow_dispatch:
    
jobs:
  hello-my-actions:
    runs-on: ubuntu-latest
    name: Hello My Actions
    steps:
      - name: checkout
        uses: actions/checkout@v1
      - name: Run My Actions 
        uses: ./ # 루트 디렉토리에 있는 액션을 사용하라는 의미이다.
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

로컬 개발환경에서 중요한 점은 `uses`에 `action.yml` 파일이 있는 곳을 설정하는 것이다. `./`로 설정하면 루트 폴더에 있는 `action.yml` 파일을 사용하게 된다. `그리고 action.yml`파일의 `runs`에 `main: 'dist/index.js'` 로 되어 있어 `dist/index.js`가 실행된다.

## 액션 구현

이제 `index.js`를 만들고 구현해 보자. `action.yml`파일이 있는 곳에 `index.js`파일을 생성한다.

```
$ touch index.js
```

파일을 열고 아래와 같이 구현한다.

```js
const core = require('@actions/core');
const github = require('@actions/github');

(async () => {
  try {
    const myToken = core.getInput('github-token');
    const octokit = github.getOctokit(myToken);

    // implement awesome github actions!
    
  } catch (error) {
    core.setFailed(error.message);
  }
})();
```

## 액션 빌드 및 실행

액션을 실행하기 위해서 `ncc`로 `index.js`와 `node_modules`를 합하여 `dist/index.js` 파일 하나로 빌드할 것 이다. 아래와 같이 `package.json`에 `build` 스크립트를 추가한다.

```json
{
  "name": "github-actions-starter-kit",
  "version": "1.0.0",
  "description": "Github Actions starter kit to create actions in javascript",
  "main": "index.js",
  "scripts": {
    "build": "ncc build index.js --license licenses.txt",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/skyfe79/github-actions-starter-kit.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/skyfe79/github-actions-starter-kit/issues"
  },
  "homepage": "https://github.com/skyfe79/github-actions-starter-kit#readme",
  "dependencies": {
    "@actions/core": "^1.5.0",
    "@actions/github": "^5.0.0"
  }
}
```

### 액션 빌드

액션을 빌드해 보자.

```
$ npm run build
> github-actions-starter-kit@1.0.0 build /Users/burt/github/skyfe79/github-actions-starter-kit
> ncc build index.js --license licenses.txt

ncc: Version 0.29.0
ncc: Compiling file index.js into CJS
 30kB  dist/licenses.txt
214kB  dist/index.js
244kB  [682ms] - ncc 0.29.0
```

`dist/index.js` 파일이 생성된 것을 확인할 수 있다.

### 액션 실행

현재 정의한 워크플로우는 액션을 실행할 때, `github-token`을 입력해야 한다. 그리고 `workflow_dispatch` 이벤트가 발생할 때만 실행된다. `acc`로 이 상황을 쉽게 재현할 수 있다. `github-token`은 개인 github 설정에서 발급한다. 

```
$ act workflow_dispatch -s GITHUB_TOKEN=abc..GJXJC
```

빌드와 실행을 하나의 파일에 담아 놓으면 편리하다. `.run.sh`파일을 생성해서 아래와 같이 작성한다. GITHUB_TOKEN은 직접 발급받은 토큰을 넣어주어야 한다.

```sh
ncc build index.js --license licenses.txt 
act workflow_dispatch -s GITHUB_TOKEN=abc..GJXJC
```

```
$ chmod +x .run.sh
```

`주의:` .run.sh에는 중요한 정보인 토큰이 저장되어 있으므로 git 커밋에 포함되면 안된다. .gitignore 에 아래와 같이 입력하여 `.run.sh`파일이 커밋에서 무시되도록 한다.

```
.run.sh
```

`node_modules`폴더를 제외한 현재까지 폴더 구조는 아래와 같다.

```
$ tree -I node_modules
.
├── LICENSE
├── README.md
├── action.yml
├── dist
│   ├── index.js
│   └── licenses.txt
├── index.js
├── package-lock.json
└── package.json
```

이제 액션을 실행해 보자.

```
$ ./.run.sh
ncc: Version 0.29.0
ncc: Compiling file index.js into CJS
 30kB  dist/licenses.txt
214kB  dist/index.js
244kB  [670ms] - ncc 0.29.0
[main.yml/Hello My Actions] 🚀  Start image=nektos/act-environments-ubuntu:18.04
[main.yml/Hello My Actions]   🐳  docker run image=nektos/act-environments-ubuntu:18.04 platform= entrypoint=["/usr/bin/tail" "-f" "/dev/null"] cmd=[]
[main.yml/Hello My Actions]   🐳  docker exec cmd=[mkdir -m 0777 -p /var/run/act] user=root
[main.yml/Hello My Actions]   🐳  docker cp src=/Users/burt/github/skyfe79/github-actions-starter-kit/. dst=/Users/burt/github/skyfe79/github-actions-starter-kit
[main.yml/Hello My Actions]   🐳  docker exec cmd=[mkdir -p /Users/burt/github/skyfe79/github-actions-starter-kit] user=
[main.yml/Hello My Actions] ⭐  Run checkout
[main.yml/Hello My Actions]   ✅  Success - checkout
[main.yml/Hello My Actions] ⭐  Run Run My Actions
[main.yml/Hello My Actions]   🐳  docker exec cmd=[node /Users/burt/github/skyfe79/github-actions-starter-kit/dist/index.js] user=
| internal/modules/cjs/loader.js:985
|   throw err;
|   ^
| 
| Error: Cannot find module '/Users/burt/github/skyfe79/github-actions-starter-kit/dist/index.js'
|     at Function.Module._resolveFilename (internal/modules/cjs/loader.js:982:15)
|     at Function.Module._load (internal/modules/cjs/loader.js:864:27)
|     at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:74:12)
|     at internal/main/run_main_module.js:18:47 {
|   code: 'MODULE_NOT_FOUND',
|   requireStack: []
| }
[main.yml/Hello My Actions]   ❌  Failure - Greeting
Error: exit with `FAILURE`: 1
```

실행하면 오류가 발생한다. `dist/index.js`를 찾을 수 없다는 오류이다. 왜 그럴까? 
원인은 github에서 레포지토리를 생성할 때, 자동으로 만들어진 .gitignore 파일에 있다. `act`는 도커를 이용해 레포지토리 클론을 시뮬레이션하기 때문에 .gitignore에 포함되는 파일과 폴더는 도커로 복사하지 않아 `dist/index.js` 파일을 찾을 수 없는 것이다. 

`.gitignore` 파일을 열고 `dist` 규칙을 삭제하거나 주석으로 변경한다.

```
...
# dist
...
```

이제 다시 실행해 보자.

```
$ ./.run.sh
ncc: Version 0.29.0
ncc: Compiling file index.js into CJS
 30kB  dist/licenses.txt
214kB  dist/index.js
244kB  [672ms] - ncc 0.29.0
[main.yml/Hello My Actions] 🚀  Start image=nektos/act-environments-ubuntu:18.04
[main.yml/Hello My Actions]   🐳  docker run image=nektos/act-environments-ubuntu:18.04 platform= entrypoint=["/usr/bin/tail" "-f" "/dev/null"] cmd=[]
[main.yml/Hello My Actions]   🐳  docker exec cmd=[mkdir -m 0777 -p /var/run/act] user=root
[main.yml/Hello My Actions]   🐳  docker cp src=/Users/burt/github/skyfe79/github-actions-starter-kit/. dst=/Users/burt/github/skyfe79/github-actions-starter-kit
[main.yml/Hello My Actions]   🐳  docker exec cmd=[mkdir -p /Users/burt/github/skyfe79/github-actions-starter-kit] user=
[main.yml/Hello My Actions] ⭐  Run checkout
[main.yml/Hello My Actions]   ✅  Success - checkout
[main.yml/Hello My Actions] ⭐  Run Run My Actions
[main.yml/Hello My Actions]   🐳  docker exec cmd=[node /Users/burt/github/skyfe79/github-actions-starter-kit/dist/index.js] user=
[main.yml/Hello My Actions]   ✅  Success - Run My Actions
```

잘 실행되는 것을 확인할 수 있다. 

`index.js` 파일에 `Github API`로 원하는 액션을 만들면 된다. 

## 배포시 주의할 점

- `.github/workflows/main.yml` 에 `uses: ./`는 수정하지 않아도 된다. 로컬에서 테스트하기 위해서 만든 액션 워크플로우이기 때문이다. 실제 적용할 레포지토리의 액션 워크플로우에서 아래처럼 `uses: ${repo_owner}/${action_name}@${action_version}`를 작성하면 된다.

```
on:
  workflow_dispatch:
    
jobs:
  hello-my-actions:
    runs-on: ubuntu-latest
    name: Hello My Actions
    steps:
      - name: checkout
        uses: actions/checkout@v1
      - name: Run My Actions 
        uses: skyfe79/hugo-with-github-issues@v1.6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## 예제 파일

- [github-actions-starter-kit](https://github.com/skyfe79/github-actions-starter-kit)