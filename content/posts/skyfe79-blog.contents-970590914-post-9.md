---
title: "Hugo에 쉽게 글쓰기"
date: 2021-08-13T17:28:46Z
draft: false
tags: ["tips"]
---

Hugo에 쉽게 글쓰기를 고민하던 중에 Github Issue를 사용해 보면 어떨까? 라는 생각이 들었습니다. Github Issue는 이미지 업로드도 지원하기 때문에 마크다운 문서에 이미지를 포함하는 것도 아주 쉽게 될 것 같았습니다.



지금 이 글도 Github Issue를 사용해 작성하고 있습니다. 이슈 제목과 코멘트를 붙여서 하나의 마크다운 문서로 만들어 Hugo에 포스팅합니다. 

 - [https://github.com/skyfe79/blog.contents/issues/9](https://github.com/skyfe79/blog.contents/issues/9)

## How to

 1. Github Issue를 마크다운으로 변경하는 Github Actions를 만듭니다.
 2. Hugo 레포지토리에 해당 Actions를 설치하고 Workflow를 작성합니다.
 3. 이슈에 글을 작성하고 워크플로우를 실행하면 자동으로 블로그로 배포됩니다.
 4. 이 때, Hugo를 Github Actions로 배포하는 워크플로우가 이미 있어야 합니다. 해당 내용은 [여기](https://blog.burt.pe.kr/posts/deploy-hugo-with-github-actions/)에서 참고할 수 있습니다.
 5. 저는 `utterances-bot`을 쓰기 때문에 `utterances-bot`이 쓴 글은 마크다운으로 변환되지 않도록 했습니다. 😄
 6. ~~그리고 open 이슈는 draft로 생각하고 closed 된 것만 컨버팅하도록 적용했습니다 👍~~
 7. 커맨드 레이블을 도입하여 글이 저장될 폴더와 배포 여부 등을 설정할 수 있게 했습니다.
 8. `::./content/{저장할 폴더 이름}` 레이블을 달면 해당 폴더에 글이 저장됩니다.
 9. `::DRAFT` 레이블이 달린 이슈는 마크다운 문서로 변환되지 않습니다.
 10.  `::DONE` 레이블이 달린 이슈는 마크다운 문서로 변환되지 않습니다.

## Uses it

- Github Actions의 내용은 [https://github.com/skyfe79/hugo-with-github-issues](https://github.com/skyfe79/hugo-with-github-issues)에서 확인 가능합니다.
- 테스팅 예제는 [https://github.com/skyfe79/testing-hugo-with-github-issues](https://github.com/skyfe79/testing-hugo-with-github-issues)에서 확인 가능합니다.
- 또한 이 글도 확인하실 수 있습니다. [https://github.com/skyfe79/blog.contents/issues/9](https://github.com/skyfe79/blog.contents/issues/9)

## Workflows

워크플로우는 이슈를 마크다운으로 컨버팅하는 것과 휴고를 배포하는 것으로 구성되어 있습니다.

### Converting Issues

```yml
name: "Convert issues to markdowns"
on:
  workflow_dispatch:
    
jobs:
  convert_issues_to_markdown_job:
    runs-on: ubuntu-latest
    name: Convert issues to markdowns.
    steps:
      - name: checkout
        uses: actions/checkout@v1
      - name: Fetch issues and generate markdowns
        uses: skyfe79/hugo-with-github-issues@v1.3
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repo: 'blog.contents'
          owner: 'skyfe79'
          skip-author: 'utterances-bot'
          use-issue-seperator: 'false'
          issue-state: 'closed'
          output: 'content/posts'
      - name: Commit files
        run: |
          git config --local user.email "skyfe79@gmail.com"
          git config --local user.name "sungcheol kim"
          git add .
          git commit -m "Add Posts"
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.ref }}
```

### Deploy Hugo

```yml
name: Deploy Hugo

on:
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Disable quotePath
        run: git config core.quotePath false

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.59.0'
          extended: true

      - name: Build
        run: hugo

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/master' }}
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
          external_repository: skyfe79/skyfe79.github.io
          publish_branch: master
          publish_dir: ./public
          cname: blog.burt.pe.kr
```



## 그림도 쉽게 넣을 수 있어요 😄 

![20210619_133757](https://user-images.githubusercontent.com/309935/129399943-24c8213a-e188-415d-81e2-5edef077726b.jpg)
  

## Tags

- 태그는 github issue에 붙이는 Label이 Hugo 의 태그로 변환됩니다. 😋