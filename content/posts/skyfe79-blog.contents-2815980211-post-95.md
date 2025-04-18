---
title: "[PostgreSQL] 제2부. SQL 언어"
date: 2025-01-28T15:12:59Z
author: "skyfe79"
draft: false
tags: ["postgresql"]
---

이 파트는 PostgreSQL에서 SQL 언어를 사용하는 방법을 설명한다. SQL의 기본 문법을 시작으로 테이블 생성 방법, 데이터베이스에 데이터를 추가하는 방법, 그리고 데이터를 조회하는 방법을 다룬다. 중간 부분에서는 SQL 명령에서 사용할 수 있는 데이터 타입과 함수를 소개한다. 마지막으로 데이터베이스 성능 최적화에 중요한 여러 측면을 다룬다.

내용은 초보자가 처음부터 끝까지 순차적으로 학습하면서 앞뒤를 자주 참조하지 않고도 주제를 완벽히 이해할 수 있도록 구성했다. 각 장은 독립적으로 구성되어 있어 고급 사용자는 필요한 장만 선택해서 읽을 수 있다. 모든 내용은 주제별로 나누어 설명하는 형식으로 서술했다. 특정 명령에 대한 완전한 설명이 필요한 독자는 [6부](https://www.postgresql.org/docs/17/reference.html "Part VI. 레퍼런스")를 참고하면 된다.

이 문서를 읽기 전에 PostgreSQL 데이터베이스 연결 방법과 SQL 명령 실행 방법을 알고 있어야 한다. 이러한 기본 지식이 없는 독자는 먼저 [1부](https://www.postgresql.org/docs/17/tutorial.html "Part I. 튜토리얼")를 읽을 것을 권장한다. SQL 명령은 주로 PostgreSQL 대화형 터미널인 psql을 통해 입력하지만, 비슷한 기능을 제공하는 다른 프로그램도 사용할 수 있다.

