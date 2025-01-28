---
title: "PostgreSQL의 간단한 역사"
date: 2025-01-28T23:00:00+09:00
draft: false
tags: ["postgresql", "database"]
---

현재 객체 관계형 데이터베이스 관리 시스템(ORDBMS)으로 알려진 PostgreSQL은 캘리포니아 버클리 대학교에서 개발한 POSTGRES 패키지에서 시작됐다. 수십 년에 걸친 발전을 거듭한 끝에 PostgreSQL은 이제 세계 어디에서도 찾아볼 수 없는 가장 진보된 오픈소스 데이터베이스가 됐다.

## 2.1. 버클리 POSTGRES 프로젝트의 역사 [#](https://www.postgresql.org/#HISTORY-BERKELEY)

Michael Stonebraker 교수가 이끈 POSTGRES 프로젝트는 미국 국방고등연구계획국(DARPA), 육군연구소(ARO), 국립과학재단(NSF), 그리고 ESL사의 지원을 받았다. 1986년에 POSTGRES 구현이 시작되었다. 시스템의 초기 개념은 [[ston86]](https://www.postgresql.org/docs/17/biblio.html#STON86)에서 처음 소개되었으며, 초기 데이터 모델의 정의는 [[rowe87]](https://www.postgresql.org/docs/17/biblio.html#ROWE87)에서 발표되었다. 당시 규칙 시스템의 설계는 [[ston87a]](https://www.postgresql.org/docs/17/biblio.html#STON87A)에서 설명되었고, 저장소 관리자의 설계 근거와 아키텍처는 [[ston87b]](https://www.postgresql.org/docs/17/biblio.html#STON87B)에서 자세히 다루었다.

이후 POSTGRES는 여러 차례 주요 릴리스를 거쳤다. 첫 번째 "데모웨어" 시스템은 1987년에 작동을 시작했고 1988년 ACM-SIGMOD 컨퍼런스에서 공개되었다. [[ston90a]](https://www.postgresql.org/docs/17/biblio.html#STON90A)에서 설명된 버전 1은 1989년 6월에 일부 외부 사용자에게 공개되었다. 첫 번째 규칙 시스템에 대한 비평([[ston89]](https://www.postgresql.org/docs/17/biblio.html#STON89))에 대응하여 규칙 시스템을 재설계했고([[ston90b]](https://www.postgresql.org/docs/17/biblio.html#STON90B)), 1990년 6월에 새로운 규칙 시스템을 탑재한 버전 2를 출시했다. 1991년에 등장한 버전 3은 다중 저장소 관리자 지원, 개선된 쿼리 실행기, 그리고 재작성된 규칙 시스템을 추가했다. Postgres95가 나오기 전까지 이후 릴리스들은 대부분 이식성과 안정성 향상에 집중했다.

POSTGRES는 다양한 연구와 실제 응용 프로그램 구현에 활용되었다. 금융 데이터 분석 시스템, 제트 엔진 성능 모니터링 패키지, 소행성 추적 데이터베이스, 의료 정보 데이터베이스, 여러 지리 정보 시스템 등이 그 예다. 또한 여러 대학에서 교육 도구로도 사용되었다. Illustra Information Technologies(후에 [Informix](https://www.ibm.com/analytics/informix)에 합병되었으며, 현재는 [IBM](https://www.ibm.com/)이 소유)는 이 코드를 가져가 상용화했다. 1992년 말, POSTGRES는 [Sequoia 2000 과학 컴퓨팅 프로젝트](http://meteora.ucsd.edu/s2k/s2k_home.html)의 주요 데이터 관리자가 되었다.

1993년에는 외부 사용자 커뮤니티가 거의 두 배로 성장했다. 프로토타입 코드의 유지보수와 지원에 많은 시간이 소요되어 데이터베이스 연구에 투자해야 할 시간이 줄어드는 것이 점점 더 분명해졌다. 이러한 지원 부담을 줄이기 위해 버클리 POSTGRES 프로젝트는 버전 4.2를 끝으로 공식적으로 종료되었다.

## 2.2. Postgres95의 역사 [#](https://www.postgresql.org/#HISTORY-POSTGRES95)

1994년 Andrew Yu와 Jolly Chen이 POSTGRES에 SQL 언어 해석기를 추가했다. 이후 Postgres95라는 새로운 이름으로 공개되어 Berkeley의 원본 POSTGRES 코드를 기반으로 한 오픈소스 프로젝트로 발전하기 시작했다.

Postgres95 코드는 완전한 ANSI C로 작성되었으며, 코드 크기가 25% 줄어들었다. 많은 내부 구조 변경을 통해 성능과 유지보수성이 크게 향상되었다. Wisconsin Benchmark 테스트 결과, Postgres95 1.0.x 버전은 POSTGRES 4.2 버전보다 30-50% 더 빠른 성능을 보여주었다. 버그 수정 외에도 다음과 같은 주요 기능이 개선되었다:

* PostQUEL 쿼리 언어를 SQL로 대체했다(서버에서 구현). (인터페이스 라이브러리 [libpq](https://www.postgresql.org/docs/17/libpq.html "32장. libpq - C 라이브러리")는 PostQUEL의 이름을 따왔다.) 서브쿼리는 PostgreSQL에서 지원될 때까지 구현되지 않았지만, 사용자 정의 SQL 함수를 통해 비슷한 기능을 구현할 수 있었다. 집계 함수를 새롭게 구현했으며, `GROUP BY` 쿼리절 지원도 추가했다.

* GNU Readline을 사용하는 새로운 프로그램(psql)을 도입하여 대화형 SQL 쿼리를 가능하게 했다. 이는 기존의 모니터 프로그램을 크게 개선했다.

* 새로운 프론트엔드 라이브러리인 `libpgtcl`을 도입하여 Tcl 기반 클라이언트를 지원했다. 샘플 셸인 `pgtclsh`는 Tcl 프로그램과 Postgres95 서버를 연동하는 새로운 Tcl 명령어를 제공했다.

* 대용량 객체 인터페이스를 전면 개편했다. 인버전 대용량 객체가 대용량 객체 저장을 위한 유일한 메커니즘이 되었다. (인버전 파일 시스템은 제거됐다.)

* 인스턴스 수준의 규칙 시스템을 제거했다. 규칙은 재작성 규칙으로만 사용할 수 있게 되었다.

* Postgres95의 기능과 함께 일반 SQL 기능을 소개하는 간단한 튜토리얼을 소스 코드와 함께 배포했다.

* 빌드 시스템을 BSD make 대신 GNU make를 사용하도록 변경했다. 또한 패치되지 않은 GCC로도 Postgres95를 컴파일할 수 있게 되었다(double 타입의 데이터 정렬 문제 해결).

## 2.3. PostgreSQL

1996년에 이르러 "Postgres95"라는 이름이 시대의 흐름에 맞지 않는다는 점이 분명해졌다. 원래의 POSTGRES와 SQL 기능이 추가된 최신 버전 사이의 관계를 표현하기 위해 PostgreSQL이라는 새로운 이름을 선택했다. 동시에 버전 번호를 6.0부터 시작하도록 설정했는데, 이는 버클리 POSTGRES 프로젝트에서 시작된 원래의 버전 순서를 이어가기 위한 결정이었다.

많은 사람들이 전통을 이어가거나 발음하기 쉽다는 이유로 PostgreSQL을 "Postgres"로 부르곤 한다(현재는 모두 대문자로 쓰는 경우가 드물다). 이러한 사용은 별칭이나 애칭으로 널리 인정받고 있다.

Postgres95 개발 과정에서는 서버 코드의 기존 문제점을 찾고 이해하는 데 중점을 두었다. PostgreSQL에서는 개발 중점이 기능과 성능 향상으로 옮겨갔지만, 모든 영역에서 개선 작업은 계속되고 있다.

그 이후 PostgreSQL에서 일어난 자세한 변화는 [부록 E](https://www.postgresql.org/docs/17/release.html "부록 E. 릴리스 노트")에서 확인할 수 있다.