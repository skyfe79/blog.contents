---
title: "[PostgreSQL] 제2부. SQL 언어 - 8장. 데이터 타입"
date: 2025-02-07T11:44:35Z
author: "skyfe79"
draft: false
tags: ["postgresql"]
---

PostgreSQL은 사용자에게 다양한 내장 데이터 타입을 제공한다. 사용자는 [CREATE TYPE](https://www.postgresql.org/sql-createtype.html "CREATE TYPE") 명령어를 통해 새로운 타입을 추가할 수도 있다.

[표 8.1](https://www.postgresql.org/datatype.html#DATATYPE-TABLE "표 8.1. 데이터 타입")은 PostgreSQL에서 제공하는 모든 일반 목적의 내장 데이터 타입을 보여준다. "별칭" 컬럼에 나열된 대부분의 이름은 역사적인 이유로 PostgreSQL 내부에서 사용되는 이름이다. 또한, 내부적으로 사용되거나 더 이상 사용되지 않는 타입들도 있지만, 여기서는 다루지 않는다.

**표 8.1. 데이터 타입**

| 이름 | 별칭 | 설명 |
| --- | --- | --- |
| bigint | int8 | 8바이트 부호 있는 정수 |
| bigserial | serial8 | 자동 증가 8바이트 정수 |
| bit [ (n) ] |  | 고정 길이 비트 문자열 |
| bit varying [ (n) ] | varbit [ (n) ] | 가변 길이 비트 문자열 |
| boolean | bool | 논리 부울 값 (참/거짓) |
| box |  | 평면 위의 직사각형 |
| bytea |  | 이진 데이터 ("바이트 배열") |
| character [ (n) ] | char [ (n) ] | 고정 길이 문자열 |
| character varying [ (n) ] | varchar [ (n) ] | 가변 길이 문자열 |
| cidr |  | IPv4 또는 IPv6 네트워크 주소 |
| circle |  | 평면 위의 원 |
| date |  | 달력 날짜 (년, 월, 일) |
| double precision | float8 | 8바이트 배정밀도 부동소수점 숫자 |
| inet |  | IPv4 또는 IPv6 호스트 주소 |
| integer | int, int4 | 4바이트 부호 있는 정수 |
| interval [ fields ] [ (p) ] |  | 시간 간격 |
| json |  | 텍스트 형식의 JSON 데이터 |
| jsonb |  | 이진 형식의 JSON 데이터, 분해됨 |
| line |  | 평면 위의 무한 직선 |
| lseg |  | 평면 위의 선분 |
| macaddr |  | MAC (미디어 액세스 제어) 주소 |
| macaddr8 |  | MAC (미디어 액세스 제어) 주소 (EUI-64 형식) |
| money |  | 통화 금액 |
| numeric [ (p, s) ] | decimal [ (p, s) ] | 선택 가능한 정밀도의 정확한 숫자 |
| path |  | 평면 위의 기하학적 경로 |
| pg_lsn |  | PostgreSQL 로그 시퀀스 번호 |
| pg_snapshot |  | 사용자 수준 트랜잭션 ID 스냅샷 |
| point |  | 평면 위의 기하학적 점 |
| polygon |  | 평면 위의 닫힌 기하학적 경로 |
| real | float4 | 4바이트 단정밀도 부동소수점 숫자 |
| smallint | int2 | 2바이트 부호 있는 정수 |
| smallserial | serial2 | 자동 증가 2바이트 정수 |
| serial | serial4 | 자동 증가 4바이트 정수 |
| text |  | 가변 길이 문자열 |
| time [ (p) ] [ without time zone ] |  | 시간대 없는 시간 |
| time [ (p) ] with time zone | timetz | 시간대 포함 시간 |
| timestamp [ (p) ] [ without time zone ] |  | 시간대 없는 날짜와 시간 |
| timestamp [ (p) ] with time zone | timestamptz | 시간대 포함 날짜와 시간 |
| tsquery |  | 텍스트 검색 쿼리 |
| tsvector |  | 텍스트 검색 문서 |
| txid_snapshot |  | 사용자 수준 트랜잭션 ID 스냅샷 (더 이상 사용되지 않음; pg_snapshot 참조) |
| uuid |  | 범용 고유 식별자 |
| xml |  | XML 데이터 |

### 호환성

SQL은 다음과 같은 타입(또는 그 철자)을 명시한다: `bigint`, `bit`, `bit varying`, `boolean`, `char`, `character varying`, `character`, `varchar`, `date`, `double precision`, `integer`, `interval`, `numeric`, `decimal`, `real`, `smallint`, `time` (시간대 포함 또는 제외), `timestamp` (시간대 포함 또는 제외), `xml`.

각 데이터 타입은 입력 및 출력 함수에 의해 결정되는 외부 표현을 가진다. 내장 타입 중 많은 것들은 명확한 외부 형식을 가지고 있다. 그러나 PostgreSQL에만 존재하는 기하학적 경로와 같은 타입이나, 날짜 및 시간 타입처럼 여러 형식을 가질 수 있는 타입도 있다. 일부 입력 및 출력 함수는 역변환이 불가능할 수 있다. 즉, 출력 함수의 결과가 원래 입력과 비교했을 때 정확도를 잃을 수 있다.

