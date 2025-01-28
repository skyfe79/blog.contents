---
title: "PostgreSQL이란?"
date: 2025-01-28T23:00:00+09:00
draft: false
tags: ["postgresql", "database"]
---

PostgreSQL은 객체-관계형 데이터베이스 관리 시스템(ORDBMS)이다. 버클리 대학교 컴퓨터 과학과에서 개발한 [POSTGRES 버전 4.2](https://dsf.berkeley.edu/postgres.html)를 기반으로 한다. POSTGRES는 여러 혁신적인 개념을 선보였는데, 이러한 기능 중 일부는 훨씬 후에야 상용 데이터베이스 시스템에 도입되었다.

PostgreSQL은 이 버클리 코드를 기반으로 한 오픈소스 프로젝트다. SQL 표준을 광범위하게 지원하며 다음과 같은 현대적인 기능을 제공한다:

* [복잡한 쿼리 처리](https://www.postgresql.org/docs/17/sql.html "Part II. The SQL Language")
* [외래 키](https://www.postgresql.org/docs/17/ddl-constraints.html#DDL-CONSTRAINTS-FK "5.5.5. Foreign Keys")
* [트리거](https://www.postgresql.org/docs/17/triggers.html "Chapter 37. Triggers")
* [갱신 가능한 뷰](https://www.postgresql.org/docs/17/sql-createview.html#SQL-CREATEVIEW-UPDATABLE-VIEWS "Updatable Views")
* [트랜잭션 무결성](https://www.postgresql.org/docs/17/transaction-iso.html "13.2. Transaction Isolation")
* [다중 버전 동시성 제어](https://www.postgresql.org/docs/17/mvcc.html "Chapter 13. Concurrency Control")

또한 PostgreSQL은 사용자가 다양한 방식으로 확장할 수 있다. 예를 들어 다음과 같은 요소를 추가할 수 있다:

* [데이터 타입](https://www.postgresql.org/docs/17/datatype.html "Chapter 8. Data Types")
* [함수](https://www.postgresql.org/docs/17/functions.html "Chapter 9. Functions and Operators")
* [연산자](https://www.postgresql.org/docs/17/functions.html "Chapter 9. Functions and Operators")
* [집계 함수](https://www.postgresql.org/docs/17/functions-aggregate.html "9.21. Aggregate Functions")
* [인덱스 메서드](https://www.postgresql.org/docs/17/indexes.html "Chapter 11. Indexes")
* [프로시저 언어](https://www.postgresql.org/docs/17/server-programming.html "Part V. Server Programming")

자유로운 라이선스 정책 덕분에 PostgreSQL은 개인, 상업, 학술 목적 구분 없이 누구나 무료로 사용, 수정, 배포할 수 있다.