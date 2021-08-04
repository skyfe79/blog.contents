---
title: "Github Actions로 Hugo 배포하기"
date: 2021-08-04T21:15:20+09:00
draft: false
tags: ["hugo", "github", "actions", "deploy"]
---

`travis-ci.org`를 사용해 `Hugo`를 배포해 오다가 `travis-ci.org`가 유료로 변경되어 `Github actions`로 배포 방법을 변경했다. `utterances` 로 코멘트를 관리하기 때문에 코멘트 이슈를 관리하기 위해 레포를 2개로 분리하여 사용하고 있다. 

- 이슈 관리를 위한 hugo 레포
- publish 대상인 블로그 레포

[peaceiris/actions-hugo@v2](https://github.com/peaceiris/actions-hugo)를 사용해 쉽게 배포 워크플로우를 작성할 수 있었다.

다른 레포에 접근하기 위해서 레포에 접근하기 위한 토큰을 만들어 디플로이를 하면 된다.

```yml
...
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

워크플로우 전체 내용은 [여기에서](https://github.com/skyfe79/blog.contents/blob/master/.github/workflows/deploy-hugo.yml) 확인할 수 있다.