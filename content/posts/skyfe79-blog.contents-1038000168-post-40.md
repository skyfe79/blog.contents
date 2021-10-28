---
title: "Electron, Vue3 개발 환경 구성"
date: 2021-10-28T01:46:23Z
author: "skyfe79"
draft: false
tags: ["vuejs","electron"]
---

데스크탑 앱을 만들기 위해서 Electron에 관심을 가지던 중, Electron과 Vue3를 사용해 앱을 만들어 보고 싶어졌다. 앱을 만들기 전에 Electron과 Vue3 개발 환경을 구성해 보자.

참고로 Node 버전은 14 이상에서 진행한다.



## Vue CLI 

Vue 프로젝트를 vue cli 로 구성하고 electron 구성을 추가하는 순서로 진행한다. 이 글에서 사용하는 vue cli 버전은 `4.5.14`이다.

```
$ vue -V
$ @vue/cli 4.5.14
```

## Vue 프로젝트 생성

```
$ vue create vue-electron
```

<img width="1368" alt="Manually select feeature" src="https://user-images.githubusercontent.com/309935/139172231-19a11850-5983-4f34-a90b-818e631036e2.png">

`Manually select feeatures`를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 50 25" src="https://user-images.githubusercontent.com/309935/139172389-b3f92833-f26a-46d9-a472-bc5d1d603d6c.png">

아래 항목을 선택하고 Enter를 누른다.
 
 - ◉ Choose Vue version
 - ◉ Babel
 - ◉ TypeScript
 - ◉ Router
 - ◉ Vuex
 - ◉ Linter / Formatter


<img width="1368" alt="스크린샷 2021-10-28 오전 10 51 47" src="https://user-images.githubusercontent.com/309935/139172508-89450388-099b-40a0-96c0-a3db8399051d.png">

vue는 3.x 버전을 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 52 27" src="https://user-images.githubusercontent.com/309935/139172562-d661a4ed-2593-43fb-8eee-2a15b70d5f83.png">

`Use class-style component syntax?` 는 No를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 53 20" src="https://user-images.githubusercontent.com/309935/139172625-f32f1fe9-42df-4630-89eb-86a592ca931c.png">

`Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)?` 는 Yes를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 54 02" src="https://user-images.githubusercontent.com/309935/139172680-6267ad3f-b64e-4b2b-a6aa-1e219617940d.png">

`Use history mode for router?`는 Yes를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 54 38" src="https://user-images.githubusercontent.com/309935/139172729-c974b53c-dfce-4900-bb6a-a20a3a11931b.png">

`Pick a linter / formatter config`는 ESLint와 Prettier를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 55 33" src="https://user-images.githubusercontent.com/309935/139172800-6e76d1fb-618c-4ae0-b8a5-44e1f6a792d3.png">

`Pick additional lint features`에서 파일 저장시 linting을 하도록 Lint on save를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 56 33" src="https://user-images.githubusercontent.com/309935/139172900-6671daa9-6d82-41f4-b95a-d45bfd5cebe4.png">

각 도구의 설정 파일을 package.json 한 곳에 몰아 넣지 말고 각각 따로 만들 수 있게 `In dedicated config files`를 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 10 57 51" src="https://user-images.githubusercontent.com/309935/139173055-78111f13-dcf4-4f0d-a5a2-67a2d477411b.png">
<img width="1368" alt="스크린샷 2021-10-28 오전 10 58 13" src="https://user-images.githubusercontent.com/309935/139173074-81cbdff8-450d-4aef-af30-0b74b5617968.png">

나중에 같은 설정을 반복해서 사용할 수 있도록 설정을 저장해 놓는다. 

<img width="1368" alt="스크린샷 2021-10-28 오전 10 59 39" src="https://user-images.githubusercontent.com/309935/139173179-11f0b207-e7e3-4e93-af74-045c5e0b5b4a.png">

vue 프로젝트가 생성된다.

<img width="1368" alt="스크린샷 2021-10-28 오전 11 00 47" src="https://user-images.githubusercontent.com/309935/139173273-3fc13f7c-244b-459d-9a5f-aa74a857548b.png">

vue 프로젝트가 생성되면 프로젝트 폴더로 이동한다.

```
$ cd vue-electron
```


## Electron 추가

[electron-builder](https://www.electron.build/)를 사용해서 electron 개발 환경을 구성한다. Vue는 CLI용 [electron-builder 플러그인](https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/)을 제공한다. 프로젝트 폴더에서 아래와 같이 vue cli를 사용해 electron-builder를 추가한다.

```
$ vue add electron-builder
```

<img width="1368" alt="스크린샷 2021-10-28 오전 11 04 38" src="https://user-images.githubusercontent.com/309935/139173633-36ae84b9-c772-428f-ac8f-122e555fe5f7.png">


### Electron 버전 선택

프로젝트에 필요한 Electron 버전을 선택한다.

<img width="1368" alt="스크린샷 2021-10-28 오전 11 05 15" src="https://user-images.githubusercontent.com/309935/139173683-2d7cb31e-dff2-4866-baaa-aaa46dea0424.png">



## tailwindcss 추가

요즘 대세는 [tailwindcss]()! tailwindcss를 추가하자. [tailwindcss도 vue cli용  플러그인](https://github.com/forsartis/vue-cli-plugin-tailwind)을 제공한다.

```
$ vue add tailwind
```

<img width="1368" alt="스크린샷 2021-10-28 오전 11 09 49" src="https://user-images.githubusercontent.com/309935/139174032-20939197-8fbd-4ecc-bc45-e16acd4b7ccd.png">

tailwind css 구성 파일을 minimal 로 선택한다.

## 제공하는 명령어 

유명하진 않지만 [sl](https://www.npmjs.com/package/sl)을 사용하면 package.json이 제공하는 스크립트 명령어를 편하게 볼 수 있다.

<img width="1368" alt="스크린샷 2021-10-28 오전 11 16 22" src="https://user-images.githubusercontent.com/309935/139174663-30323877-40a7-4ad1-98c9-6c77103f4b47.png">



## 앱 실행

아래 명령어를 실행하면 electron + vue3 앱이 실행된다.

```
$ npm run electron:serve
```

<img width="912" alt="스크린샷 2021-10-28 오전 11 17 38" src="https://user-images.githubusercontent.com/309935/139174818-9d884250-c3a0-46cc-88b2-d67511a2564f.png">

앱 빌드는 `npm run electron:build` 로 할 수 있다. 인증서 등이 필요하므로 자세한 내용은 [Electron Builder](https://www.electron.build/) 문서를 참고한다.