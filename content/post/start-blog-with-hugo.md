---
title: "Hugo로 블로그 시작하기"
date: 2019-10-26T13:57:42+09:00
draft: false
categories: ["Blogging"]
tags: ["Hugo", "Custom Domain", "fqdn"]
---

오랫동안 묵혀두었던 Jekyll 기반 블로그를 지우고 Hugo로 블로그를 열어 보았습니다. Hugo로 블로그를 개설할 때 다음 글에서 많은 도움을 받았습니다.
 
 * [테마](https://themes.gohugo.io/theme/hugo-theme-learn/en)
 * [Hugo 로 github 블로그 시작하기](https://wotjd.github.io/categories/blogging/)
 * [TRAVIS CI 를 이용해서 HUGO 기반 웹사이트 배포 하기](https://ironpark.github.io/2017/12/17/hugo-site-deploy-use-travis-ci/)
 * [Hosting a Hugo blog on GitHub Pages with Travis CI](https://medium.com/swlh/hosting-a-hugo-blog-on-github-pages-with-travis-ci-e74a1d686f10)
 * [블로그 구축기 2](https://ialy1595.github.io/post/blog-construct-2/)
 * [Using Travis CI to publish to GitHub pages with custom domain](https://echorand.me/posts/github-pages-custom-domain-travis-ci/)

저처럼 Gtihub Page에 blog.burt.pe.kr 이라는 커스텀 도메인을 연결해 사용할 때, Travis-ci 로 배포를 하면 매번 커스텀 도메인 설정이 Reset되는 문제점이 있습니다. 이 때에는 `.travis.yaml`에 `fqdn`을 설정해 주면 됩니다.

```yaml
install:
    - wget https://github.com/gohugoio/hugo/releases/download/v0.59.0/hugo_0.59.0_Linux-64bit.deb
    - sudo dpkg -i hugo*.deb
    - hugo version
script:
  - hugo
deploy:
  local_dir: public
  repo: skyfe79/skyfe79.github.io
  target_branch: master
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  email: skyfe79@gmail.com
  name: "skyfe79"
  fqdn: "blog.burt.pe.kr"
  on:
    branch: master
```

