---
title: "vscode에서 Swift 자동 완성 사용하기"
date: 2021-08-18T09:49:42Z
draft: false
tags: ["swift"]
---

간단한 코드를 작성할 때는 프로젝트를 만들어야 하는 Xcode 보다는 vscode가 편할 때가 많다. Swift 자동 완성을 vscode에서 하기 위해선 `SourceKit-LSP` 를 vscode에 설치해야 한다.

이 글은 [Swift Development with Visual Studio Code](https://nshipster.com/vscode/) 을 참고했다.

## Xcode 설치

```
$ xcode-select --install
```

### Xcode 가 잘 설치되어 있는지 확인하기

```
$ xcrun sourcekit-lsp
```

## node 설치

```
$ brew install node
```

## sourcekit-lsp 설치

[sourcekit-lsp 레포](https://github.com/apple/sourcekit-lsp) 를 클론한다.

```
$ git clone https://github.com/apple/sourcekit-lsp.git
$ cd sourcekit-lsp/Editors/vscode/
```

## npm 모듈 설치

필요한 모듈을 설치한다.

```
$ npm i
```

## vscode 확장 빌드 및 설치

```
$ npm run dev-package    
$ code --install-extension sourcekit-lsp-development.vsix
```

## 자동완성

다음 그림과 같이 코드 작성시 경고 및 자동 완성 등을 확인할 수 있다.

<img width="1136" alt="경고" src="https://user-images.githubusercontent.com/309935/129879581-2765650c-a00a-47f0-939e-e65d6e81afef.png">

<img width="1136" alt="자동완성" src="https://user-images.githubusercontent.com/309935/129880607-de33121b-7ef7-45b4-bc4c-754f790cfb1d.png">



## 실행 결과

<img width="1136" alt="실행 결과" src="https://user-images.githubusercontent.com/309935/129879719-793479f2-4f5c-488c-af37-8b3bc2272cdb.png">
