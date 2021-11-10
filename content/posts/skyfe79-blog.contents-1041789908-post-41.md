---
title: "자바스크립트 개발 환경 구성 팁 모음."
date: 2021-11-02T01:11:59Z
author: "skyfe79"
draft: false
tags: ["javascript"]
---

자바스크립트 개발 환경 구성 중 만난 이슈와 해결 과정을 정리한다. 🤪

## singleQuote

eslint 와 prettier 간의 singleQuote와 doubleQuote 설정이 충돌날 때가 있다. 

- eslint의 prettier 설정은  doubleQuote.
- .prettierrc 는 singleQuote.

`.eslintrc.js` 의 rules에서 prettier 설정을 재설정한다.

```javascript
rules: {
  ...
  'prettier/prettier': [
    'error',
    {
      singleQuote: true,
    },
  ],
},
```

## Unknown at rule @tailwindcss(unknownAtRules)

vscode에서 tailwindcss 를 사용할 때, `@tailwind` 같은 지시자를 오류로 인식한다면? 

- `.vscode` 폴더를 생성한다.
- `settings.json` 을 만든다.


### 1. 간단한 방법

간단히 무시하는 방법이 있다.

```
{
  "css.lint.unknownAtRules": "ignore"
}
```

### 2. 해당 지시자를 vscode가 인식하게 하는 방법

- [Unknown at rule @tailwind 경고 회피하기](https://imkh.dev/vue-tailwind-rule/)을 참고한다.



## 추천 도구: script-list

`npm run`을 실행하면 package.json 에서 정의한 `scripts` 섹션의 내용을 보여준다.

```
$ npm run

Lifecycle scripts included in nestjs-board-app:
  start
    nest start
  test
    jest

available via `npm run-script`:
  prebuild
    rimraf dist
  build
    nest build
  format
    prettier --write "src/**/*.ts" "test/**/*.ts"
  start:dev
    nest start --watch
  start:debug
    nest start --debug --watch
  start:prod
    node dist/main
  lint
    eslint "{src,apps,libs,test}/**/*.ts" --fix
  test:watch
    jest --watch
  test:cov
    jest --coverage
  test:debug
    node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand
  test:e2e
    jest --config ./test/jest-e2e.json
```

보는 것이 좀 불편해서 주로 [script-list](https://www.npmjs.com/package/script-list) 도구를 사용한다. 결과가 깔금하게 나와서 좋다.

```
$ sl

   my-nestjs-proj
    - prebuild    : rimraf dist
    - build       : nest build
    - format      : prettier --write "src/**/*.ts" "test/**/*.ts"
    - start       : nest start
    - start:dev   : nest start --watch
    - start:debug : nest start --debug --watch
    - start:prod  : node dist/main
    - lint        : eslint "{src,apps,libs,test}/**/*.ts" --fix
    - test        : jest
    - test:watch  : jest --watch
    - test:cov    : jest --coverage
    - test:debug  : node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand
    - test:e2e    : jest --config ./test/jest-e2e.json
```

### script-list 설치

글로벌로 설치하여 사용하면 편하다.

```
$ npm install -g script-list
$ sl
```

자세한 내용은 [script-list](https://www.npmjs.com/package/script-list)에서 확인하자.