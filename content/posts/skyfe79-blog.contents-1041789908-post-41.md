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

