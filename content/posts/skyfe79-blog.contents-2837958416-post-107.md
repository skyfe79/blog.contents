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

## 8.1. 숫자 타입

숫자 타입은 2바이트, 4바이트, 8바이트 정수, 4바이트와 8바이트 부동소수점 숫자, 그리고 선택 가능한 정밀도의 소수를 포함한다. [표 8.2](https://www.postgresql.org/docs/current/datatype-numeric.html#DATATYPE-NUMERIC-TABLE "표 8.2. 숫자 타입")에서 사용 가능한 타입을 확인할 수 있다.

**표 8.2. 숫자 타입**

| 이름 | 저장 크기 | 설명 | 범위 |
| --- | --- | --- | --- |
| smallint | 2 바이트 | 작은 범위의 정수 | -32768 ~ +32767 |
| integer | 4 바이트 | 일반적인 정수 선택 | -2147483648 ~ +2147483647 |
| bigint | 8 바이트 | 큰 범위의 정수 | -9223372036854775808 ~ +9223372036854775807 |
| decimal | 가변 | 사용자 지정 정밀도, 정확 | 소수점 앞 최대 131072자리; 소수점 뒤 최대 16383자리 |
| numeric | 가변 | 사용자 지정 정밀도, 정확 | 소수점 앞 최대 131072자리; 소수점 뒤 최대 16383자리 |
| real | 4 바이트 | 가변 정밀도, 부정확 | 소수점 이하 6자리 정밀도 |
| double precision | 8 바이트 | 가변 정밀도, 부정확 | 소수점 이하 15자리 정밀도 |
| smallserial | 2 바이트 | 작은 자동 증가 정수 | 1 ~ 32767 |
| serial | 4 바이트 | 자동 증가 정수 | 1 ~ 2147483647 |
| bigserial | 8 바이트 | 큰 자동 증가 정수 | 1 ~ 9223372036854775807 |

숫자 타입의 상수 문법은 [4.1.2절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS "4.1.2. 상수")에서 설명한다. 숫자 타입은 해당하는 산술 연산자와 함수를 모두 제공한다. 더 자세한 내용은 [9장](https://www.postgresql.org/docs/current/functions.html "9장. 함수와 연산자")을 참고한다. 다음 절에서 각 타입을 자세히 설명한다.


### 8.1.1. 정수 타입

`smallint`, `integer`, `bigint` 타입은 소수 부분이 없는 정수를 저장한다. 각 타입은 허용 범위가 다르며, 이 범위를 벗어난 값을 저장하려고 하면 오류가 발생한다.

`integer` 타입은 범위, 저장 공간, 성능 측면에서 가장 균형이 잘 맞아 일반적으로 많이 사용된다. `smallint` 타입은 디스크 공간이 매우 제한된 경우에 주로 사용된다. `bigint` 타입은 `integer` 타입의 범위로는 부족할 때 사용하도록 설계되었다.

SQL 표준에서는 `integer`(또는 `int`), `smallint`, `bigint` 타입만 정의한다. `int2`, `int4`, `int8`과 같은 타입 이름은 확장된 형태로, 다른 SQL 데이터베이스 시스템에서도 사용된다.


### 8.1.2. 임의 정밀도 숫자

`numeric` 타입은 매우 많은 자릿수를 가진 숫자를 저장할 수 있다. 특히 금액이나 정확성이 요구되는 다른 수량을 저장할 때 권장된다. `numeric` 값을 사용한 계산은 가능한 한 정확한 결과를 제공한다. 예를 들어 덧셈, 뺄셈, 곱셈 등이 있다. 그러나 `numeric` 값에 대한 계산은 정수 타입이나 다음 섹션에서 설명할 부동소수점 타입에 비해 매우 느리다.

아래에서 사용할 용어를 정의한다: `numeric`의 *정밀도(precision)* 는 전체 숫자의 유효 자릿수를 의미하며, 소수점의 양쪽에 있는 모든 자릿수를 포함한다. *스케일(scale)* 은 소수점 오른쪽에 있는 소수 부분의 자릿수를 나타낸다. 따라서 숫자 23.5141은 정밀도 6, 스케일 4를 가진다. 정수는 스케일이 0인 것으로 간주할 수 있다.

`numeric` 컬럼의 최대 정밀도와 최대 스케일을 설정할 수 있다. `numeric` 타입의 컬럼을 선언하려면 다음 문법을 사용한다:

```
precision
```

정밀도는 양수여야 하지만, 스케일은 양수 또는 음수가 될 수 있다(아래 참조). 또는:

```
precision
```

이 문법은 스케일을 0으로 설정한다. 다음처럼:

```
NUMERIC
```

정밀도나 스케일을 지정하지 않으면 "제약 없는 numeric" 컬럼이 생성된다. 이 컬럼은 구현 한계까지 어떤 길이의 숫자 값도 저장할 수 있다. 이 타입의 컬럼은 입력 값을 특정 스케일로 강제 변환하지 않지만, 스케일이 선언된 `numeric` 컬럼은 입력 값을 해당 스케일로 강제 변환한다. (SQL 표준은 기본 스케일을 0으로 요구하며, 이는 정수 정밀도로 강제 변환한다는 의미다. 이는 다소 쓸모가 없다고 본다. 호환성을 고려한다면 항상 정밀도와 스케일을 명시적으로 지정하는 것이 좋다.)


### 참고

`numeric` 타입 선언에서 명시적으로 지정할 수 있는 최대 정밀도는 1000이다. 제약이 없는 `numeric` 컬럼은 [표 8.2](https://www.postgresql.org/docs/current/datatype-numeric.html#DATATYPE-NUMERIC-TABLE "표 8.2. Numeric 타입")에 설명된 제한을 따른다.

저장할 값의 소수 자릿수가 컬럼에 선언된 소수 자릿수보다 크면, 시스템은 해당 값을 지정된 소수 자릿수로 반올림한다. 그런 다음, 소수점 왼쪽의 자릿수가 선언된 정밀도에서 선언된 소수 자릿수를 뺀 값을 초과하면 오류가 발생한다. 예를 들어, 다음과 같이 선언된 컬럼은

```
NUMERIC(3, 1)
```

값을 소수점 첫째 자리로 반올림하며, -99.9부터 99.9까지의 값을 저장할 수 있다.

PostgreSQL 15부터는 `numeric` 컬럼을 음수 소수 자릿수로 선언할 수 있다. 이 경우 값은 소수점 왼쪽으로 반올림된다. 정밀도는 여전히 반올림되지 않은 최대 자릿수를 나타낸다. 따라서 다음과 같이 선언된 컬럼은

```
NUMERIC(2, -3)
```

값을 천 단위로 반올림하며, -99000부터 99000까지의 값을 저장할 수 있다. 또한 선언된 정밀도보다 큰 소수 자릿수를 선언할 수도 있다. 이러한 컬럼은 소수 값만 저장할 수 있으며, 소수점 바로 오른쪽에 선언된 소수 자릿수에서 선언된 정밀도를 뺀 값만큼의 0이 필요하다. 예를 들어, 다음과 같이 선언된 컬럼은

```
NUMERIC(3, 5)
```

값을 소수점 다섯째 자리로 반올림하며, -0.00999부터 0.00999까지의 값을 저장할 수 있다.


### 참고 사항

PostgreSQL은 `numeric` 타입 선언에서 소수점 이하 자릿수(scale)를 -1000에서 1000 사이의 값으로 허용한다. 그러나 SQL 표준은 scale이 0부터 *`precision`* 사이의 값이어야 한다고 규정한다. 이 범위를 벗어나는 scale을 사용하면 다른 데이터베이스 시스템과의 호환성이 떨어질 수 있다.

`numeric` 값은 물리적으로 저장될 때 추가적인 앞뒤의 0 없이 저장된다. 따라서 컬럼에 선언된 precision과 scale은 최대값일 뿐, 고정된 할당이 아니다. (이 점에서 `numeric` 타입은 ``char(n)``보다는 ``varchar(n)``에 더 가깝다.) 실제 저장 공간은 4자리 십진수마다 2바이트가 필요하며, 추가로 3~8바이트의 오버헤드가 발생한다.

일반적인 숫자 값 외에도 `numeric` 타입은 다음과 같은 특수한 값을 가진다:

  
`Infinity`  
`-Infinity`  
`NaN`  

이 값들은 IEEE 754 표준에서 차용한 것으로, 각각 "무한대", "음의 무한대", "숫자가 아님"을 나타낸다. SQL 명령에서 이 값을 상수로 사용할 때는 따옴표로 감싸야 한다. 예를 들어 `UPDATE table SET x = '-Infinity'`와 같이 작성한다. 입력 시 이 문자열은 대소문자를 구분하지 않고 인식된다. 무한대 값은 `inf`와 `-inf`로도 표현할 수 있다.

무한대 값은 수학적 기대에 따라 동작한다. 예를 들어, `Infinity`에 유한한 값을 더하면 `Infinity`가 되며, `Infinity`에 `Infinity`를 더해도 마찬가지다. 그러나 `Infinity`에서 `Infinity`를 빼면 `NaN`(숫자가 아님)이 된다. 이는 명확한 해석이 불가능하기 때문이다. 무한대 값은 유한한 precision 한계를 초과하기 때문에 제약이 없는 `numeric` 컬럼에만 저장할 수 있다.

`NaN`(숫자가 아님) 값은 정의되지 않은 계산 결과를 나타내는 데 사용된다. 일반적으로 `NaN`이 입력된 모든 연산은 또 다른 `NaN`을 반환한다. 단, 연산의 다른 입력값이 `NaN`을 유한한 값이나 무한대로 대체해도 동일한 결과가 나오는 경우에는 해당 결과값이 `NaN`에도 적용된다. (예를 들어, `NaN`을 0제곱하면 1이 된다.)


### 참고

대부분의 "숫자가 아님" 개념 구현에서 `NaN`은 다른 숫자 값(심지어 `NaN` 자체도 포함)과 동등하지 않다고 간주한다. 하지만 PostgreSQL은 `numeric` 값을 정렬하고 트리 기반 인덱스에서 사용할 수 있도록 `NaN` 값을 동등하게 처리하며, 모든 `NaN`이 아닌 값보다 크다고 간주한다.

`decimal`과 `numeric` 타입은 동일한 의미를 가진다. 두 타입 모두 SQL 표준에 포함되어 있다.

값을 반올림할 때, `numeric` 타입은 0에서 멀어지는 방향으로 반올림한다. 반면, (대부분의 시스템에서) `real`과 `double precision` 타입은 가장 가까운 짝수로 반올림한다. 예를 들면 다음과 같다:

```
SELECT x,
  round(x::numeric) AS num_round,
  round(x::double precision) AS dbl_round
FROM generate_series(-3.5, 3.5, 1) as x;
  x   | num_round | dbl_round
------+-----------+-----------
 -3.5 |        -4 |        -4
 -2.5 |        -3 |        -2
 -1.5 |        -2 |        -2
 -0.5 |        -1 |        -0
  0.5 |         1 |         0
  1.5 |         2 |         2
  2.5 |         3 |         2
  3.5 |         4 |         4
(8 rows)
```


### 8.1.3. 부동소수점 타입

`real`과 `double precision` 데이터 타입은 정확하지 않은 가변 정밀도 숫자 타입이다. 현재 지원되는 모든 플랫폼에서, 이 타입들은 각각 단정밀도와 배정밀도로 IEEE 표준 754 부동소수점 연산을 구현한다. 단, 이는 프로세서, 운영체제, 컴파일러가 이를 지원하는 범위 내에서이다.

'정확하지 않음'이란 일부 값이 내부 형식으로 정확히 변환되지 않고 근사치로 저장된다는 의미이다. 따라서 값을 저장하고 다시 불러올 때 약간의 차이가 발생할 수 있다. 이러한 오류를 관리하고 계산 과정에서 어떻게 전파되는지는 수학과 컴퓨터 과학의 한 분야로, 여기서는 자세히 다루지 않는다. 단, 다음과 같은 점은 주의해야 한다:

* 정확한 저장과 계산이 필요한 경우(예: 금액), `numeric` 타입을 사용한다.
* 중요한 계산을 위해 이러한 타입을 사용하려면, 특히 경계 조건(무한대, 언더플로우)에서 특정 동작에 의존하는 경우, 구현을 신중히 평가해야 한다.
* 두 부동소수점 값을 비교할 때, 동등성 검사가 항상 예상대로 동작하지 않을 수 있다.

현재 지원되는 모든 플랫폼에서, `real` 타입은 약 1E-37에서 1E+37 범위를 가지며, 최소 6자리 십진수 정밀도를 제공한다. `double precision` 타입은 약 1E-307에서 1E+308 범위를 가지며, 최소 15자리 정밀도를 제공한다. 값이 너무 크거나 작으면 오류가 발생한다. 입력 숫자의 정밀도가 너무 높으면 반올림이 발생할 수 있다. 0에 너무 가깝지만 0과 구별할 수 없는 숫자는 언더플로우 오류를 일으킨다.

기본적으로, 부동소수점 값은 가장 짧은 정확한 십진수 표현으로 텍스트 형태로 출력된다. 출력된 십진수 값은 실제 저장된 이진수 값에 더 가깝다. (단, 현재 출력 값은 두 표현 가능한 값의 정 중간에 위치하지 않도록 설계되어 있다. 이는 입력 루틴이 '가장 가까운 짝수로 반올림' 규칙을 제대로 준수하지 않는 버그를 방지하기 위함이다.) 이 값은 `float8` 값의 경우 최대 17자리 유효 십진수를, `float4` 값의 경우 최대 9자리를 사용한다.


### 참고

이 최소 정밀도 출력 형식은 과거의 반올림 형식보다 훨씬 빠르게 생성된다.

이전 버전의 PostgreSQL에서 생성된 출력과 호환성을 유지하고, 출력 정밀도를 낮추기 위해 [extra_float_digits](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-EXTRA-FLOAT-DIGITS) 매개변수를 사용해 반올림된 십진수 출력을 선택할 수 있다. 이 값을 0으로 설정하면 이전 기본값인 `float4`의 경우 6자리, `float8`의 경우 15자리로 값을 반올림한다. 음수 값을 설정하면 더 적은 자릿수로 반올림한다. 예를 들어 -2는 각각 4자리와 13자리로 출력을 반올림한다.

[extra_float_digits](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-EXTRA-FLOAT-DIGITS) 값을 0보다 크게 설정하면 최소 정밀도 형식을 선택한다.


### 참고

정확한 값을 얻기 위해 애플리케이션은 전통적으로 [extra_float_digits](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-EXTRA-FLOAT-DIGITS)를 3으로 설정해 왔다. 버전 간 최대 호환성을 위해 이 설정을 계속 사용하는 것이 좋다.

일반적인 숫자 값 외에도, 부동 소수점 타입은 몇 가지 특수 값을 가진다:

  
`Infinity`  
`-Infinity`  
`NaN`  

이 값들은 각각 IEEE 754 특수 값인 "무한대", "음의 무한대", "숫자가 아님"을 나타낸다. SQL 명령어에서 이 값을 상수로 사용할 때는 따옴표로 감싸야 한다. 예를 들어 `UPDATE table SET x = '-Infinity'`와 같이 작성한다. 입력 시 이 문자열은 대소문자를 구분하지 않고 인식된다. 무한대 값은 `inf`와 `-inf`로도 표기할 수 있다.


### 참고사항

IEEE 754 표준에 따르면, `NaN`은 다른 부동소수점 값(심지어 `NaN` 자체도 포함)과 동등하지 않아야 한다. 그러나 PostgreSQL은 부동소수점 값을 정렬하고 트리 기반 인덱스에 사용할 수 있도록 `NaN` 값을 동등하게 취급하며, 모든 `NaN`이 아닌 값보다 크다고 간주한다.

PostgreSQL은 또한 SQL 표준 표기법인 `float`와 ``float(p)``를 지원하여 부정확한 숫자 타입을 지정할 수 있다. 여기서 p는 최소 허용 정밀도를 *이진* 자릿수로 지정한다. PostgreSQL은 `float(1)`부터 `float(24)`까지를 `real` 타입으로 선택하고, `float(25)`부터 `float(53)`까지는 `double precision`으로 선택한다. 허용 범위를 벗어난 p 값은 오류를 발생시킨다. 정밀도를 지정하지 않은 `float`는 `double precision`을 의미한다.


### 8.1.4. Serial 타입

### 참고

이 섹션은 PostgreSQL에서 자동 증가 컬럼을 생성하는 특별한 방법에 대해 설명한다. 또 다른 방법으로는 SQL 표준의 identity 컬럼 기능을 사용할 수 있으며, 이는 [5.3절](https://www.postgresql.org/docs/current/ddl-identity-columns.html "5.3. Identity Columns")에서 자세히 다룬다.

`smallserial`, `serial`, `bigserial` 데이터 타입은 실제 타입이 아니라, 고유 식별자 컬럼을 생성하기 위한 편의 기능이다(다른 데이터베이스에서 지원하는 `AUTO_INCREMENT` 속성과 유사). 현재 구현에서는 다음과 같이 지정하는 것:

```
tablename
```

은 다음과 같이 지정하는 것과 동일하다:

```
tablename
```

따라서, 정수 타입 컬럼을 생성하고 기본값을 시퀀스 생성기에서 할당하도록 설정한다. 또한 `NOT NULL` 제약 조건을 적용해 NULL 값을 삽입할 수 없도록 한다. (대부분의 경우 `UNIQUE` 또는 `PRIMARY KEY` 제약 조건을 추가해 실수로 중복 값이 삽입되는 것을 방지하지만, 이는 자동으로 적용되지 않는다.) 마지막으로, 시퀀스는 해당 컬럼에 "소유됨"으로 표시되어 컬럼이나 테이블이 삭제될 때 함께 삭제된다.


### 참고

`smallserial`, `serial`, `bigserial` 타입은 시퀀스를 사용해 구현되기 때문에, 테이블에서 행이 삭제되지 않더라도 컬럼에 나타나는 값의 시퀀스에 "구멍"이나 간격이 생길 수 있다. 시퀀스에서 할당된 값은 해당 값을 포함하는 행이 테이블 컬럼에 성공적으로 삽입되지 않더라도 "사용된" 것으로 처리된다. 예를 들어, 삽입 트랜잭션이 롤백되는 경우 이런 현상이 발생할 수 있다. 자세한 내용은 [9.17장](https://www.postgresql.org/docs/current/functions-sequence.html "9.17. Sequence Manipulation Functions")의 `nextval()` 함수를 참고한다.

`serial` 컬럼에 시퀀스의 다음 값을 삽입하려면, `serial` 컬럼에 기본값을 할당하도록 지정해야 한다. 이는 `INSERT` 문에서 해당 컬럼을 제외하거나 `DEFAULT` 키워드를 사용해 수행할 수 있다.

`serial`과 `serial4` 타입 이름은 동일하다: 둘 다 `integer` 컬럼을 생성한다. `bigserial`과 `serial8`도 동일하게 작동하지만, `bigint` 컬럼을 생성한다. 테이블의 수명 동안 231개 이상의 식별자를 사용할 것으로 예상된다면 `bigserial`을 사용해야 한다. `smallserial`과 `serial2`도 동일하게 작동하지만, `smallint` 컬럼을 생성한다.

`serial` 컬럼을 위해 생성된 시퀀스는 해당 컬럼이 삭제될 때 자동으로 제거된다. 컬럼을 삭제하지 않고 시퀀스를 삭제할 수도 있지만, 이 경우 컬럼의 기본값 표현식이 강제로 제거된다.




## 8.2. 화폐 타입

`money` 타입은 고정된 소수점 정밀도로 화폐 금액을 저장한다. 소수점 정밀도는 데이터베이스의 `lc_monetary` 설정에 따라 결정된다. 표에 표시된 범위는 소수점 이하 두 자리를 가정한 것이다. 입력은 정수 및 부동소수점 리터럴뿐만 아니라 `'$1,000.00'`과 같은 일반적인 화폐 형식도 받아들인다. 출력은 일반적으로 후자의 형태지만 로케일에 따라 달라질 수 있다.

**표 8.3. 화폐 타입**

| 이름 | 저장 크기 | 설명 | 범위 |
| --- | --- | --- | --- |
| money | 8바이트 | 화폐 금액 | -92233720368547758.08 ~ +92233720368547758.07 |

이 데이터 타입의 출력은 로케일에 민감하므로, `lc_monetary` 설정이 다른 데이터베이스에 `money` 데이터를 로드할 때 문제가 발생할 수 있다. 문제를 방지하려면 덤프를 새로운 데이터베이스로 복원하기 전에 `lc_monetary`가 덤프된 데이터베이스와 동일하거나 동등한 값인지 확인해야 한다.

`numeric`, `int`, `bigint` 데이터 타입의 값은 `money`로 캐스팅할 수 있다. `real` 및 `double precision` 데이터 타입에서 변환할 때는 먼저 `numeric`으로 캐스팅해야 한다. 예를 들어:

```
SELECT '12.34'::float8::numeric::money;
```

그러나 이 방법은 권장하지 않는다. 부동소수점 숫자는 반올림 오류가 발생할 가능성이 있으므로 화폐를 처리하는 데 사용하지 않는 것이 좋다.

`money` 값은 정밀도 손실 없이 `numeric`으로 캐스팅할 수 있다. 다른 타입으로 변환할 때는 정밀도가 손실될 가능성이 있으며, 두 단계를 거쳐야 한다:

```
SELECT '52093.89'::money::numeric::float8;
```

`money` 값을 정수 값으로 나누면 소수 부분이 0 방향으로 잘린다. 반올림된 결과를 얻으려면 부동소수점 값으로 나누거나, `money` 값을 `numeric`으로 캐스팅한 후 나누고 다시 `money`로 캐스팅한다. (정밀도 손실을 방지하기 위해 후자가 더 좋은 방법이다.) `money` 값을 다른 `money` 값으로 나누면 결과는 `double precision`이 된다. (즉, 순수한 숫자이며 화폐 단위는 나눗셈 과정에서 상쇄된다.)




## 8.3. 문자 타입


**표 8.4. 문자 타입**

| 이름 | 설명 |
| --- | --- |
| character varying(n), varchar(n) | 길이 제한이 있는 가변 길이 |
| character(n), char(n), bpchar(n) | 고정 길이, 공백으로 채움 |
| bpchar | 길이 제한 없는 가변 길이, 공백 제거 |
| text | 길이 제한 없는 가변 길이 |

[표 8.4](https://www.postgresql.org/docs/current/datatype-character.html#DATATYPE-CHARACTER-TABLE "표 8.4. 문자 타입")는 PostgreSQL에서 사용할 수 있는 일반적인 문자 타입을 보여준다.

SQL은 두 가지 주요 문자 타입을 정의한다: `character varying(n)`와 `character(n)`. 여기서 n은 양의 정수이다. 이 두 타입은 최대 n개의 문자(바이트가 아님)를 저장할 수 있다. 더 긴 문자열을 저장하려고 하면 오류가 발생한다. 단, 초과된 문자가 모두 공백인 경우 문자열은 최대 길이로 잘린다. (이 다소 이상한 예외는 SQL 표준에 의해 요구된다.) 그러나 값을 명시적으로 `character varying(n)` 또는 `character(n)`로 캐스팅하면, 길이가 초과된 값은 오류 없이 n개의 문자로 잘린다. (이 또한 SQL 표준에 의해 요구된다.) 저장할 문자열이 선언된 길이보다 짧은 경우, `character` 타입의 값은 공백으로 채워지고, `character varying` 타입의 값은 짧은 문자열을 그대로 저장한다.

또한 PostgreSQL은 `text` 타입을 제공한다. 이 타입은 길이 제한 없이 문자열을 저장한다. `text` 타입은 SQL 표준에 포함되지 않지만, 여러 다른 SQL 데이터베이스 관리 시스템에서도 지원한다. `text`는 PostgreSQL의 기본 문자열 데이터 타입으로, 대부분의 내장 문자열 함수는 `character varying`이 아닌 `text`를 입력 또는 반환 타입으로 선언한다. 많은 경우 `character varying`은 `text` 위에 정의된 [도메인](https://www.postgresql.org/docs/current/domains.html "8.18. 도메인 타입")처럼 동작한다.

`varchar`는 `character varying`의 별칭이며, `bpchar`(길이 지정자 포함)와 `char`는 `character`의 별칭이다. `varchar`와 `char` 별칭은 SQL 표준에 정의되어 있고, `bpchar`는 PostgreSQL의 확장 기능이다.

길이 n을 지정할 경우, 이 값은 0보다 커야 하며 10,485,760을 초과할 수 없다. `character varying`(또는 `varchar`)을 길이 지정자 없이 사용하면, 이 타입은 길이 제한 없이 문자열을 허용한다. `bpchar`에 길이 지정자가 없으면, 이 또한 길이 제한 없이 문자열을 허용하지만, 뒤따르는 공백은 의미상 중요하지 않다. `character`(또는 `char`)에 길이 지정자가 없으면, `character(1)`과 동일하다.

`character` 타입의 값은 물리적으로 지정된 너비 n까지 공백으로 채워지며, 이렇게 저장되고 표시된다. 그러나 뒤따르는 공백은 의미상 중요하지 않으며, `character` 타입의 두 값을 비교할 때 무시된다. 공백이 중요한 정렬 규칙에서는 이 동작이 예상치 못한 결과를 초래할 수 있다. 예를 들어 `SELECT 'a '::CHAR(2) collate "C" < E'an'::CHAR(2)`는 `C` 로케일에서 공백이 줄바꿈보다 크다고 간주함에도 불구하고 true를 반환한다. `character` 값을 다른 문자열 타입으로 변환할 때는 뒤따르는 공백이 제거된다. `character varying`과 `text` 값에서는 뒤따르는 공백이 의미상 중요하며, 패턴 매칭(`LIKE` 및 정규 표현식)을 사용할 때도 마찬가지이다.

이러한 데이터 타입에 저장할 수 있는 문자는 데이터베이스 생성 시 선택된 문자 집합에 의해 결정된다. 특정 문자 집합과 관계없이 코드가 0인 문자(NUL이라고도 함)는 저장할 수 없다. 자세한 내용은 [23.3절](https://www.postgresql.org/docs/current/multibyte.html "23.3. 문자 집합 지원")을 참조한다.

짧은 문자열(최대 126바이트)의 저장 요구량은 실제 문자열에 1바이트를 더한 값이다. `character` 타입의 경우 공백으로 채우는 부분도 포함된다. 더 긴 문자열은 1바이트 대신 4바이트의 오버헤드가 있다. 긴 문자열은 시스템에 의해 자동으로 압축되므로 디스크의 물리적 요구량이 더 적을 수 있다. 매우 긴 값은 백그라운드 테이블에 저장되어 짧은 열 값에 대한 빠른 접근을 방해하지 않는다. 어쨌든 저장할 수 있는 가장 긴 문자열은 약 1GB이다. (데이터 타입 선언에서 n에 허용되는 최대값은 이보다 작다. 멀티바이트 문자 인코딩에서 문자 수와 바이트 수가 크게 다를 수 있으므로 이를 변경하는 것은 유용하지 않다. 특정 상한 없이 긴 문자열을 저장하려면 임의의 길이 제한을 만드는 대신 `text` 또는 길이 지정자 없이 `character varying`을 사용한다.)


### 팁

이 세 가지 타입 간에는 성능 차이가 거의 없다. 단, 빈 공간을 채우는 타입을 사용할 때 저장 공간이 더 필요하고, 길이 제한이 있는 컬럼에 저장할 때 길이를 확인하는 데 CPU 사이클이 약간 더 소요된다. 다른 데이터베이스 시스템에서는 `character(n)`가 성능상 이점이 있지만, PostgreSQL에서는 그렇지 않다. 오히려 `character(n)`는 추가 저장 비용으로 인해 세 가지 중 가장 느린 경우가 많다. 대부분의 경우 `text`나 `character varying`을 사용하는 것이 더 좋다.

문자열 리터럴 문법에 대한 자세한 내용은 [4.1.2.1절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS "4.1.2.1. String Constants")을 참고하고, 사용 가능한 연산자와 함수에 대한 정보는 [9장](https://www.postgresql.org/docs/current/functions.html "Chapter 9. Functions and Operators")을 참고하라.

**예제 8.1. 문자 타입 사용하기**

``
a   | char_length
------+-------------
 ok   |           2
``

PostgreSQL에는 [표 8.5](https://www.postgresql.org/docs/current/datatype-character.html#DATATYPE-CHARACTER-SPECIAL-TABLE "Table 8.5. Special Character Types")에 나온 두 가지 고정 길이 문자 타입이 더 있다. 이 타입들은 일반적인 용도로 사용하기 위한 것이 아니라, 내부 시스템 카탈로그에서만 사용된다. `name` 타입은 식별자를 저장하는 데 사용된다. 현재 길이는 64바이트(사용 가능한 문자 63개와 종결자)로 정의되어 있지만, `C` 소스 코드에서는 상수 `NAMEDATALEN`을 참조해야 한다. 길이는 컴파일 시점에 설정되며(따라서 특수한 용도에 맞게 조정 가능), 기본 최대 길이는 향후 릴리스에서 변경될 수 있다. `"char"` 타입(따옴표 주의)은 `char(1)`과 달리 저장 공간이 1바이트만 사용되므로 단일 ASCII 문자만 저장할 수 있다. 이 타입은 시스템 카탈로그에서 단순한 열거 타입으로 사용된다.

**표 8.5. 특수 문자 타입**

| 이름 | 저장 크기 | 설명 |
| --- | --- | --- |
| "char" | 1바이트 | 단일 바이트 내부 타입 |
| name | 64바이트 | 객체 이름을 위한 내부 타입 |




## 8.4. 바이너리 데이터 타입

`bytea` 데이터 타입은 바이너리 문자열을 저장할 수 있다. 자세한 내용은 [표 8.6](https://www.postgresql.org/docs/current/datatype-binary.html#DATATYPE-BINARY-TABLE "표 8.6. 바이너리 데이터 타입")을 참고한다.

**표 8.6. 바이너리 데이터 타입**

| 이름 | 저장 크기 | 설명 |
| --- | --- | --- |
| bytea | 1 또는 4 바이트 + 실제 바이너리 문자열 | 가변 길이 바이너리 문자열 |

바이너리 문자열은 옥텟(또는 바이트)의 연속이다. 바이너리 문자열은 문자열과 두 가지 측면에서 구별된다. 첫째, 바이너리 문자열은 값이 0인 옥텟과 기타 "출력 불가능한" 옥텟(일반적으로 32에서 126 사이의 10진수 범위를 벗어난 옥텟)을 명시적으로 저장할 수 있다. 문자열은 0 옥텟을 허용하지 않으며, 데이터베이스가 선택한 문자 집합 인코딩에 따라 유효하지 않은 다른 옥텟 값이나 옥텟 값 시퀀스도 허용하지 않는다. 둘째, 바이너리 문자열에 대한 연산은 실제 바이트를 처리하지만, 문자열에 대한 연산은 로케일 설정에 따라 달라진다. 간단히 말해, 바이너리 문자열은 프로그래머가 "원시 바이트"로 생각하는 데이터를 저장하는 데 적합하고, 문자열은 텍스트를 저장하는 데 적합하다.

`bytea` 타입은 입력과 출력을 위해 두 가지 형식을 지원한다: "헥스" 형식과 PostgreSQL의 전통적인 "이스케이프" 형식. 이 두 형식은 항상 입력 시 허용된다. 출력 형식은 [bytea_output](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-BYTEA-OUTPUT) 설정 매개변수에 따라 달라지며, 기본값은 헥스 형식이다. (헥스 형식은 PostgreSQL 9.0에서 도입되었으며, 이전 버전과 일부 도구는 이를 이해하지 못할 수 있다.)

SQL 표준은 `BLOB` 또는 `BINARY LARGE OBJECT`라는 다른 바이너리 문자열 타입을 정의한다. 입력 형식은 `bytea`와 다르지만, 제공되는 함수와 연산자는 대부분 동일하다.


### 8.4.1. `bytea` 16진수 형식

"16진수" 형식은 바이너리 데이터를 각 바이트당 2개의 16진수로 인코딩하며, 상위 니블이 먼저 표시된다. 전체 문자열 앞에는 `x` 시퀀스가 붙어 이스케이프 형식과 구분된다. 특정 상황에서는 초기 백슬래시를 두 번 써서 이스케이프해야 할 수도 있다([4.1.2.1절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS "4.1.2.1. 문자열 상수") 참조). 입력 시 16진수는 대문자 또는 소문자 모두 사용할 수 있으며, 숫자 쌍 사이에 공백을 넣을 수 있다(단, 숫자 쌍 내부나 시작 `x` 시퀀스에는 공백을 넣을 수 없다). 16진수 형식은 다양한 외부 애플리케이션 및 프로토콜과 호환되며, 이스케이프 형식보다 변환 속도가 빠르기 때문에 선호된다.

예시:

```
SET bytea_output = 'hex';

SELECT 'xDEADBEEF'::bytea;
   bytea
------------
 xdeadbeef
```


### 8.4.2. `bytea` 이스케이프 형식

"이스케이프" 형식은 PostgreSQL에서 전통적으로 사용해온 `bytea` 타입의 표현 방식이다. 이 방식은 바이너리 문자열을 ASCII 문자 시퀀스로 표현하며, ASCII 문자로 표현할 수 없는 바이트는 특수 이스케이프 시퀀스로 변환한다. 애플리케이션 관점에서 바이트를 문자로 표현하는 것이 의미가 있다면, 이 방식은 편리할 수 있다. 그러나 실제로는 바이너리 문자열과 문자열의 구분을 모호하게 만들며, 선택된 이스케이프 메커니즘이 다소 번거롭기 때문에 대부분의 새로운 애플리케이션에서는 이 형식을 피하는 것이 좋다.

이스케이프 형식으로 `bytea` 값을 입력할 때, 특정 값의 옥텟은 반드시 이스케이프해야 하며, 모든 옥텟 값은 이스케이프할 수 있다. 일반적으로 옥텟을 이스케이프하려면, 해당 옥텟을 3자리 8진수 값으로 변환하고 백슬래시를 앞에 붙인다. 백슬래시 자체(10진수 값 92)는 이중 백슬래시로 표현할 수도 있다. [표 8.7](https://www.postgresql.org/docs/current/datatype-binary.html#DATATYPE-BINARY-SQLESC "표 8.7. bytea 리터럴 이스케이프된 옥텟")은 반드시 이스케이프해야 하는 문자와 해당 이스케이프 시퀀스를 보여준다.

**표 8.7. `bytea` 리터럴 이스케이프된 옥텟**

| 10진수 옥텟 값 | 설명 | 이스케이프 입력 표현 | 예제 | 16진수 표현 |
| --- | --- | --- | --- | --- |
| 0 | 제로 옥텟 | '000' | '000'::bytea | x00 |
| 39 | 작은따옴표 | '''' or '047' | ''''::bytea | x27 |
| 92 | 백슬래시 | '\' or '134' | '\'::bytea | x5c |
| 0 ~ 31 및 127 ~ 255 | "비인쇄 가능" 옥텟 | 'xxx' (8진수 값) | '001'::bytea | x01 |

"비인쇄 가능" 옥텟의 이스케이프 요구 사항은 로케일 설정에 따라 달라진다. 경우에 따라 이스케이프하지 않아도 될 때도 있다.

[표 8.7](https://www.postgresql.org/docs/current/datatype-binary.html#DATATYPE-BINARY-SQLESC "표 8.7. bytea 리터럴 이스케이프된 옥텟")에서 보여주듯이, 작은따옴표를 반드시 두 번 입력해야 하는 이유는 SQL 명령에서 모든 문자열 리터럴에 적용되는 규칙이기 때문이다. 일반적인 문자열 리터럴 파서는 바깥쪽 작은따옴표를 소비하고, 작은따옴표 쌍을 하나의 데이터 문자로 축소한다. `bytea` 입력 함수가 보는 것은 단일 작은따옴표 하나이며, 이를 일반 데이터 문자로 처리한다. 그러나 `bytea` 입력 함수는 백슬래시를 특수 문자로 처리하며, [표 8.7](https://www.postgresql.org/docs/current/datatype-binary.html#DATATYPE-BINARY-SQLESC "표 8.7. bytea 리터럴 이스케이프된 옥텟")에 나온 다른 동작도 이 함수에 의해 구현된다.

일부 컨텍스트에서는 위에서 보여준 것보다 백슬래시를 더 많이 입력해야 한다. 일반 문자열 리터럴 파서가 백슬래시 쌍을 하나의 데이터 문자로 축소하기 때문이다. 자세한 내용은 [4.1.2.1절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS "4.1.2.1. 문자열 상수")을 참조한다.

`Bytea` 옥텟은 기본적으로 `hex` 형식으로 출력된다. [bytea_output](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-BYTEA-OUTPUT)을 `escape`로 변경하면, "비인쇄 가능" 옥텟은 해당하는 3자리 8진수 값으로 변환되고 앞에 하나의 백슬래시가 붙는다. 대부분의 "인쇄 가능" 옥텟은 클라이언트 문자 집합의 표준 표현으로 출력된다. 예를 들면:

```
SET bytea_output = 'escape';

SELECT 'abc 153154155 052251124'::bytea;
     bytea
----------------
 abc klm *251T
```

10진수 값 92(백슬래시)인 옥텟은 출력에서 두 번 표시된다. 자세한 내용은 [표 8.8](https://www.postgresql.org/docs/current/datatype-binary.html#DATATYPE-BINARY-RESESC "표 8.8. bytea 출력 이스케이프된 옥텟")을 참조한다.

**표 8.8. `bytea` 출력 이스케이프된 옥텟**

| 10진수 옥텟 값 | 설명 | 이스케이프 출력 표현 | 예제 | 출력 결과 |
| --- | --- | --- | --- | --- |
| 92 | 백슬래시 | \ | '134'::bytea | \ |
| 0 ~ 31 및 127 ~ 255 | "비인쇄 가능" 옥텟 | xxx (8진수 값) | '001'::bytea | 001 |
| 32 ~ 126 | "인쇄 가능" 옥텟 | 클라이언트 문자 집합 표현 | '176'::bytea | ~ |

사용하는 PostgreSQL 프런트엔드에 따라, `bytea` 문자열을 이스케이프하고 이스케이프 해제하는 추가 작업이 필요할 수 있다. 예를 들어, 인터페이스가 자동으로 줄 바꿈과 캐리지 리턴을 변환한다면, 이들도 이스케이프해야 할 수 있다.




## 8.5. 날짜/시간 타입

PostgreSQL은 SQL 날짜 및 시간 타입을 모두 지원한다. 이는 [표 8.9](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-TABLE "표 8.9. 날짜/시간 타입")에서 확인할 수 있다. 이러한 데이터 타입에 사용 가능한 연산은 [9.9절](https://www.postgresql.org/docs/current/functions-datetime.html "9.9. 날짜/시간 함수와 연산자")에서 설명한다. 날짜는 그레고리력을 기준으로 계산되며, 이 달력이 도입되기 이전의 연도에서도 동일하게 적용된다. 더 자세한 내용은 [B.6절](https://www.postgresql.org/docs/current/datetime-units-history.html "B.6. 단위의 역사")을 참고한다.

**표 8.9. 날짜/시간 타입**

| 이름 | 저장 크기 | 설명 | 최소값 | 최대값 | 정밀도 |
| --- | --- | --- | --- | --- | --- |
| timestamp [ (p) ] [ without time zone ] | 8 바이트 | 날짜와 시간 (시간대 없음) | 기원전 4713년 | 294276년 | 1 마이크로초 |
| timestamp [ (p) ] with time zone | 8 바이트 | 날짜와 시간 (시간대 포함) | 기원전 4713년 | 294276년 | 1 마이크로초 |
| date | 4 바이트 | 날짜 (시간 없음) | 기원전 4713년 | 5874897년 | 1일 |
| time [ (p) ] [ without time zone ] | 8 바이트 | 시간 (날짜 없음) | 00:00:00 | 24:00:00 | 1 마이크로초 |
| time [ (p) ] with time zone | 12 바이트 | 시간 (날짜 없음, 시간대 포함) | 00:00:00+1559 | 24:00:00-1559 | 1 마이크로초 |
| interval [ fields ] [ (p) ] | 16 바이트 | 시간 간격 | -178000000년 | 178000000년 | 1 마이크로초 |


### 참고

SQL 표준은 `timestamp`만 작성하는 것을 `timestamp without time zone`과 동일하게 취급하도록 요구한다. PostgreSQL은 이 동작을 준수한다. `timestamptz`는 `timestamp with time zone`의 축약형으로 사용할 수 있으며, 이는 PostgreSQL의 확장 기능이다.

`time`, `timestamp`, `interval` 타입은 초 단위 필드에 유지할 소수 자릿수를 지정하는 선택적 정밀도 값 `p`를 허용한다. 기본적으로 정밀도에는 명시적인 제한이 없다. `p`의 허용 범위는 0부터 6까지이다.

`interval` 타입은 저장할 필드 집합을 제한하는 추가 옵션을 제공한다. 다음 구문 중 하나를 작성하여 필드를 제한할 수 있다:

```
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND
```

`fields`와 `p`를 모두 지정하는 경우, 정밀도는 초 단위에만 적용되므로 `fields`에 `SECOND`가 포함되어야 한다.

`time with time zone` 타입은 SQL 표준에 정의되어 있지만, 그 정의는 의문스러운 유용성을 보인다. 대부분의 경우, `date`, `time`, `timestamp without time zone`, `timestamp with time zone`의 조합으로 애플리케이션에 필요한 모든 날짜/시간 기능을 제공할 수 있다.


### 8.5.1. 날짜/시간 입력

PostgreSQL은 ISO 8601, SQL 호환 형식, 전통적인 POSTGRES 형식 등 거의 모든 합리적인 형식의 날짜와 시간 입력을 지원한다. 일부 형식의 경우, 날짜 입력에서 일, 월, 년의 순서가 모호할 수 있으므로, 이러한 필드의 예상 순서를 지정할 수 있는 기능을 제공한다. [DateStyle](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DATESTYLE) 매개변수를 `MDY`로 설정하면 월-일-년 순서로 해석하고, `DMY`로 설정하면 일-월-년 순서로 해석하며, `YMD`로 설정하면 년-월-일 순서로 해석한다.

PostgreSQL은 SQL 표준에서 요구하는 것보다 더 유연하게 날짜/시간 입력을 처리한다. 정확한 날짜/시간 입력 파싱 규칙과 월, 요일, 시간대 등 인식 가능한 텍스트 필드에 대한 자세한 내용은 [부록 B](https://www.postgresql.org/docs/current/datetime-appendix.html "부록 B. 날짜/시간 지원")를 참조한다.

날짜나 시간 리터럴 입력은 텍스트 문자열과 마찬가지로 작은따옴표로 감싸야 한다. 자세한 내용은 [4.1.2.7절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS-GENERIC "4.1.2.7. 기타 타입의 상수")을 참조한다. SQL은 다음 구문을 요구한다.

```
type
```

여기서 `p`는 초 필드의 소수점 이하 자릿수를 지정하는 선택적 정밀도 명세이다. `time`, `timestamp`, `interval` 타입에 대해 정밀도를 지정할 수 있으며, 0에서 6까지의 범위를 가진다. 상수 명세에서 정밀도를 지정하지 않으면 리터럴 값의 정밀도가 기본값으로 사용된다(단, 최대 6자리까지).


#### 8.5.1.1. 날짜 [#](https://www.postgresql.org/#DATATYPE-DATETIME-INPUT-DATES)

[Table 8.10](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-DATE-TABLE "Table 8.10. Date Input")은 `date` 타입에 사용할 수 있는 다양한 입력 예제를 보여준다.

**Table 8.10. 날짜 입력**

| 예제 | 설명 |
| --- | --- |
| 1999-01-08 | ISO 8601; 모든 모드에서 1월 8일로 해석 (권장 형식) |
| January 8, 1999 | 모든 날짜 스타일 입력 모드에서 명확함 |
| 1/8/1999 | MDY 모드에서 1월 8일; DMY 모드에서 8월 1일 |
| 1/18/1999 | MDY 모드에서 1월 18일; 다른 모드에서는 거부됨 |
| 01/02/03 | MDY 모드에서 2003년 1월 2일; DMY 모드에서 2003년 2월 1일; YMD 모드에서 2001년 2월 3일 |
| 1999-Jan-08 | 모든 모드에서 1월 8일 |
| Jan-08-1999 | 모든 모드에서 1월 8일 |
| 08-Jan-1999 | 모든 모드에서 1월 8일 |
| 99-Jan-08 | YMD 모드에서 1월 8일, 다른 모드에서는 오류 |
| 08-Jan-99 | YMD 모드를 제외하고 1월 8일 |
| Jan-08-99 | YMD 모드를 제외하고 1월 8일 |
| 19990108 | ISO 8601; 모든 모드에서 1999년 1월 8일 |
| 990108 | ISO 8601; 모든 모드에서 1999년 1월 8일 |
| 1999.008 | 연도와 연중 일수 |
| J2451187 | 율리우스력 날짜 |
| January 8, 99 BC | 기원전 99년 |


#### 8.5.1.2. 시간 타입

시간을 표현하는 타입은 `time [ (p) ] without time zone`과 `time [ (p) ] with time zone`이다. 단순히 `time`이라고만 쓰면 `time without time zone`과 동일하다.

이 타입에 유효한 입력은 시간 뒤에 선택적으로 시간대를 붙인 형태다. (자세한 내용은 [표 8.11](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-TIME-TABLE "표 8.11. 시간 입력")과 [표 8.12](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-TIMEZONE-TABLE "표 8.12. 시간대 입력")를 참고하자.) `time without time zone` 타입에 시간대를 지정해도 무시된다. 날짜를 지정할 수도 있지만, `America/New_York`처럼 일광 절약 시간 규칙이 적용되는 시간대 이름을 사용하는 경우를 제외하고는 날짜도 무시된다. 이 경우 표준 시간대와 일광 절약 시간대를 구분하기 위해 날짜를 지정해야 한다. `time with time zone` 값에는 적절한 시간대 오프셋이 기록되며, 저장된 그대로 출력된다. 활성화된 시간대에 맞춰 조정되지 않는다.

**표 8.11. 시간 입력**

| 예시 | 설명 |
| --- | --- |
| 04:05:06.789 | ISO 8601 |
| 04:05:06 | ISO 8601 |
| 04:05 | ISO 8601 |
| 040506 | ISO 8601 |
| 04:05 AM | 04:05와 동일. AM은 값에 영향을 미치지 않음 |
| 04:05 PM | 16:05와 동일. 입력 시간은 12 이하여야 함 |
| 04:05:06.789-8 | ISO 8601, UTC 오프셋으로 시간대 지정 |
| 04:05:06-08:00 | ISO 8601, UTC 오프셋으로 시간대 지정 |
| 04:05-08:00 | ISO 8601, UTC 오프셋으로 시간대 지정 |
| 040506-08 | ISO 8601, UTC 오프셋으로 시간대 지정 |
| 040506+0730 | ISO 8601, UTC 오프셋으로 시간대 지정 (소수 시간 포함) |
| 040506+07:30:00 | UTC 오프셋을 초 단위로 지정 (ISO 8601에서는 허용되지 않음) |
| 04:05:06 PST | 시간대 약어로 지정 |
| 2003-04-12 04:05:06 America/New_York | 시간대 전체 이름으로 지정 |

**표 8.12. 시간대 입력**

| 예시 | 설명 |
| --- | --- |
| PST | 약어 (태평양 표준시) |
| America/New_York | 전체 시간대 이름 |
| PST8PDT | POSIX 스타일 시간대 지정 |
| -8:00:00 | PST에 대한 UTC 오프셋 |
| -8:00 | PST에 대한 UTC 오프셋 (ISO 8601 확장 형식) |
| -800 | PST에 대한 UTC 오프셋 (ISO 8601 기본 형식) |
| -8 | PST에 대한 UTC 오프셋 (ISO 8601 기본 형식) |
| zulu | UTC에 대한 군사 약어 |
| z | zulu의 짧은 형태 (ISO 8601에서도 사용) |

시간대를 지정하는 방법에 대한 자세한 내용은 [8.5.3절](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-TIMEZONES "8.5.3. 시간대")을 참고하자.


#### 8.5.1.3. 타임스탬프

타임스탬프 타입에 유효한 입력은 날짜와 시간이 결합된 형태이며, 선택적으로 시간대와 `AD` 또는 `BC`가 뒤따를 수 있다. (다른 방법으로 `AD`/`BC`가 시간대 앞에 올 수도 있지만, 이는 권장되는 순서는 아니다.) 따라서:

```
1999-01-08 04:05:06
```

그리고:

```
1999-01-08 04:05:06 -8:00
```

은 ISO 8601 표준을 따르는 유효한 값이다. 또한, 일반적인 형식인:

```
January 8 04:05:06 1999 PST
```

도 지원된다.

SQL 표준은 `timestamp without time zone`과 `timestamp with time zone` 리터럴을 시간 뒤에 "+" 또는 "-" 기호와 시간대 오프셋이 있는지 여부로 구분한다. 따라서 표준에 따르면,

```
TIMESTAMP '2004-10-19 10:23:54'
```

은 `timestamp without time zone`이며,

```
TIMESTAMP '2004-10-19 10:23:54+02'
```

은 `timestamp with time zone`이다. PostgreSQL은 리터럴 문자열의 내용을 타입을 결정하기 전에 검사하지 않으므로, 위의 두 경우 모두 `timestamp without time zone`으로 처리한다. 리터럴이 `timestamp with time zone`으로 처리되도록 하려면 명시적으로 올바른 타입을 지정해야 한다:

```
TIMESTAMP WITH TIME ZONE '2004-10-19 10:23:54+02'
```

`timestamp without time zone`으로 결정된 값에서 PostgreSQL은 시간대 표시를 무시한다. 즉, 결과 값은 입력 문자열의 날짜/시간 필드에서 도출되며, 시간대에 따라 조정되지 않는다.

`timestamp with time zone` 값의 경우, 명시적인 시간대가 포함된 입력 문자열은 해당 시간대의 적절한 오프셋을 사용해 UTC(Universal Coordinated Time)로 변환된다. 입력 문자열에 시간대가 명시되지 않은 경우, 시스템의 `TimeZone` 파라미터로 지정된 시간대로 간주되며, 해당 `timezone`의 오프셋을 사용해 UTC로 변환된다. 두 경우 모두 값은 내부적으로 UTC로 저장되며, 원래 명시되었거나 가정된 시간대는 유지되지 않는다.

`timestamp with time zone` 값이 출력될 때는 항상 UTC에서 현재 `timezone`으로 변환되어 해당 지역 시간으로 표시된다. 다른 시간대에서 시간을 확인하려면 `timezone`을 변경하거나 `AT TIME ZONE` 구문을 사용한다(섹션 9.9.4 참조).

`timestamp without time zone`과 `timestamp with time zone` 간의 변환은 일반적으로 `timestamp without time zone` 값을 `timezone` 지역 시간으로 간주하거나 제공한다. 변환 시 다른 시간대를 지정하려면 `AT TIME ZONE`을 사용한다.


#### 8.5.1.4. 특수 값

PostgreSQL은 편의를 위해 여러 특수한 날짜/시간 입력 값을 지원한다. [Table 8.13](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-SPECIAL-TABLE "Table 8.13. Special Date/Time Inputs")에서 확인할 수 있다. `infinity`와 `-infinity`는 시스템 내부에서 특별히 표현되며, 그대로 표시된다. 그러나 다른 값들은 단순히 표기법을 줄인 것으로, 읽을 때 일반 날짜/시간 값으로 변환된다. (특히, `now`와 관련된 문자열은 읽는 즉시 특정 시간 값으로 변환된다.) SQL 명령에서 상수로 사용할 때는 이 모든 값을 작은따옴표로 감싸야 한다.

**Table 8.13. 특수 날짜/시간 입력**

| 입력 문자열 | 유효한 타입 | 설명 |
| --- | --- | --- |
| epoch | date, timestamp | 1970-01-01 00:00:00+00 (유닉스 시스템 시간 기준) |
| infinity | date, timestamp, interval | 다른 모든 타임스탬프보다 이후 |
| -infinity | date, timestamp, interval | 다른 모든 타임스탬프보다 이전 |
| now | date, time, timestamp | 현재 트랜잭션의 시작 시간 |
| today | date, timestamp | 오늘 자정 (00:00) |
| tomorrow | date, timestamp | 내일 자정 (00:00) |
| yesterday | date, timestamp | 어제 자정 (00:00) |
| allballs | time | 00:00:00.00 UTC |

다음 SQL 호환 함수를 사용하여 해당 데이터 타입의 현재 시간 값을 얻을 수도 있다: `CURRENT_DATE`, `CURRENT_TIME`, `CURRENT_TIMESTAMP`, `LOCALTIME`, `LOCALTIMESTAMP`. ([Section 9.9.5](https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-CURRENT "9.9.5. Current Date/Time") 참조.) 이 함수들은 SQL 함수이며 데이터 입력 문자열에서 인식되지 않는다.


### 주의 사항

`now`, `today`, `tomorrow`, `yesterday`와 같은 입력 문자열은 대화형 SQL 명령어에서 사용해도 문제가 없다. 하지만 이러한 명령어를 나중에 실행하기 위해 저장하는 경우, 예를 들어 준비된 문장, 뷰, 함수 정의 등에서는 예상치 못한 동작을 일으킬 수 있다. 이 문자열은 특정 시간 값으로 변환될 수 있으며, 이 값은 오래된 후에도 계속 사용될 수 있다. 이러한 상황에서는 SQL 함수를 사용하는 것이 더 안전하다. 예를 들어, `'tomorrow'::date` 대신 `CURRENT_DATE + 1`을 사용하는 것이 더 적절하다.


### 8.5.2. 날짜/시간 출력

날짜/시간 타입의 출력 형식은 ISO 8601, SQL(Ingres), 전통적인 POSTGRES(Unix 날짜 형식), 또는 독일식 중 하나로 설정할 수 있다. 기본값은 ISO 형식이다. (SQL 표준은 ISO 8601 형식 사용을 요구한다. "SQL" 출력 형식의 이름은 역사적인 이유로 붙여졌다.) [표 8.14](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-OUTPUT-TABLE "표 8.14. 날짜/시간 출력 스타일")는 각 출력 스타일의 예제를 보여준다. `date`와 `time` 타입의 출력은 일반적으로 주어진 예제에 따라 날짜 또는 시간 부분만 포함한다. 그러나 POSTGRES 스타일은 날짜만 있는 값을 ISO 형식으로 출력한다.

**표 8.14. 날짜/시간 출력 스타일**

| 스타일 지정 | 설명 | 예제 |
| --- | --- | --- |
| ISO | ISO 8601, SQL 표준 | 1997-12-17 07:37:16-08 |
| SQL | 전통적인 스타일 | 12/17/1997 07:37:16.00 PST |
| Postgres | 원래 스타일 | Wed Dec 17 07:37:16 1997 PST |
| German | 지역 스타일 | 17.12.1997 07:37:16.00 PST |


### 참고

ISO 8601은 날짜와 시간을 구분하기 위해 대문자 `T`를 사용한다. PostgreSQL은 입력 시 이 형식을 허용하지만, 출력 시에는 `T` 대신 공백을 사용한다. 이는 가독성을 높이고 [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) 및 다른 데이터베이스 시스템과의 일관성을 유지하기 위함이다.

SQL 및 POSTGRES 스타일에서는 DMY 필드 순서가 지정된 경우 날짜가 월보다 앞에 오고, 그렇지 않으면 월이 날짜보다 앞에 온다. (이 설정이 입력 값의 해석에 어떻게 영향을 미치는지에 대해서는 [8.5.1절](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-INPUT "8.5.1. Date/Time Input")을 참고한다.) [표 8.15](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-DATETIME-OUTPUT2-TABLE "Table 8.15. Date Order Conventions")에서 예제를 확인할 수 있다.

**표 8.15. 날짜 순서 규칙**

| datestyle 설정 | 입력 순서 | 예제 출력 |
| --- | --- | --- |
| SQL, DMY | 일/월/년 | 17/12/1997 15:37:16.00 CET |
| SQL, MDY | 월/일/년 | 12/17/1997 07:37:16.00 PST |
| Postgres, DMY | 일/월/년 | Wed 17 Dec 07:37:16 1997 PST |

ISO 스타일에서는 시간대가 항상 UTC 기준의 부호 있는 숫자 오프셋으로 표시되며, 그리니치 동쪽의 시간대는 양수로 표시된다. 오프셋은 정수 시간인 경우 `hh` (시간만), 정수 분인 경우 `hh`:`mm`, 그 외의 경우 `hh`:`mm`:`ss`로 표시된다. (세 번째 경우는 현대 시간대 표준에서는 발생하지 않지만, 표준화된 시간대가 도입되기 전의 타임스탬프를 다룰 때 나타날 수 있다.) 다른 날짜 스타일에서는 시간대가 현재 시간대에서 일반적으로 사용되는 알파벳 약어로 표시된다. 그렇지 않으면 ISO 8601 기본 형식(`hh` 또는 `hhmm`)의 부호 있는 숫자 오프셋으로 나타난다.

날짜/시간 스타일은 사용자가 `SET datestyle` 명령, `postgresql.conf` 설정 파일의 [DateStyle](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DATESTYLE) 매개변수, 또는 서버나 클라이언트의 `PGDATESTYLE` 환경 변수를 통해 선택할 수 있다.

더 유연한 날짜/시간 출력을 위해 `to_char` 포맷팅 함수도 사용할 수 있다. (자세한 내용은 [9.8절](https://www.postgresql.org/docs/current/functions-formatting.html "9.8. Data Type Formatting Functions") 참고)


### 8.5.3. 시간대

시간대와 시간대 규칙은 지구의 기하학적 특성뿐만 아니라 정치적 결정에 의해 영향을 받는다. 전 세계의 시간대는 1900년대에 어느 정도 표준화되었지만, 특히 일광 절약 시간 규칙과 관련해 임의의 변경이 계속 발생할 가능성이 있다. PostgreSQL은 역사적인 시간대 규칙에 대한 정보를 위해 널리 사용되는 IANA(Olson) 시간대 데이터베이스를 사용한다. 미래의 시간에 대해서는 주어진 시간대의 가장 최근에 알려진 규칙이 무한히 먼 미래까지 계속 적용될 것이라고 가정한다.

PostgreSQL은 일반적인 사용 사례에서 SQL 표준 정의와 호환되도록 노력한다. 그러나 SQL 표준은 날짜와 시간 타입 및 기능에 대해 이상한 혼합을 가지고 있다. 두 가지 명백한 문제는 다음과 같다:

* `date` 타입은 시간대와 연관될 수 없지만, `time` 타입은 시간대와 연관될 수 있다. 실제 세계에서 시간대는 날짜와 시간 모두와 연관되지 않으면 거의 의미가 없다. 왜냐하면 오프셋이 일광 절약 시간 경계를 통해 연중에 따라 변할 수 있기 때문이다.

* 기본 시간대는 UTC에서 상수 숫자 오프셋으로 지정된다. 따라서 일광 절약 시간 경계를 넘는 날짜/시간 계산을 할 때 일광 절약 시간에 적응하는 것이 불가능하다.

이러한 문제를 해결하기 위해, 시간대를 사용할 때는 날짜와 시간을 모두 포함하는 날짜/시간 타입을 사용할 것을 권장한다. `time with time zone` 타입은 PostgreSQL에서 레거시 애플리케이션과 SQL 표준 준수를 위해 지원되지만, 사용을 권장하지 않는다. PostgreSQL은 날짜나 시간만 포함하는 타입에 대해 로컬 시간대를 가정한다.

시간대를 인식하는 모든 날짜와 시간은 내부적으로 UTC로 저장된다. 이들은 클라이언트에 표시되기 전에 [TimeZone](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TIMEZONE) 설정 매개변수로 지정된 시간대의 로컬 시간으로 변환된다.

PostgreSQL은 세 가지 다른 형식으로 시간대를 지정할 수 있다:

* 전체 시간대 이름, 예를 들어 `America/New_York`. 인식되는 시간대 이름은 `pg_timezone_names` 뷰에 나열되어 있다(섹션 [52.32](https://www.postgresql.org/docs/current/view-pg-timezone-names.html "52.32. pg_timezone_names") 참조). PostgreSQL은 이를 위해 널리 사용되는 IANA 시간대 데이터를 사용하므로, 동일한 시간대 이름이 다른 소프트웨어에서도 인식된다.

* 시간대 약어, 예를 들어 `PST`. 이러한 지정은 UTC에서 특정 오프셋만 정의한다. 전체 시간대 이름과 달리 일광 절약 시간 전환 규칙을 포함하지 않는다. 인식되는 약어는 `pg_timezone_abbrevs` 뷰에 나열되어 있다(섹션 [52.31](https://www.postgresql.org/docs/current/view-pg-timezone-abbrevs.html "52.31. pg_timezone_abbrevs") 참조). [TimeZone](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TIMEZONE) 또는 [log_timezone](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-TIMEZONE) 설정 매개변수를 시간대 약어로 설정할 수는 없지만, 날짜/시간 입력 값과 `AT TIME ZONE` 연산자에서 약어를 사용할 수 있다.

* 시간대 이름과 약어 외에도 PostgreSQL은 POSIX 스타일의 시간대 지정을 허용한다. 이는 섹션 [B.5](https://www.postgresql.org/docs/current/datetime-posix-timezone-specs.html "B.5. POSIX Time Zone Specifications")에 설명되어 있다. 이 옵션은 일반적으로 명명된 시간대를 사용하는 것보다 선호되지 않지만, 적합한 IANA 시간대 항목이 없는 경우 필요할 수 있다.

간단히 말해, 약어와 전체 이름의 차이점은 다음과 같다: 약어는 UTC에서 특정 오프셋을 나타내지만, 많은 전체 이름은 로컬 일광 절약 시간 규칙을 의미하므로 두 가지 가능한 UTC 오프셋을 가질 수 있다. 예를 들어, `2014-06-04 12:00 America/New_York`은 뉴욕의 현지 시간 정오를 나타내며, 이 특정 날짜에는 동부 일광 절약 시간(UTC-4)이었다. 따라서 `2014-06-04 12:00 EDT`는 동일한 시간을 지정한다. 그러나 `2014-06-04 12:00 EST`는 해당 날짜에 일광 절약 시간이 공식적으로 적용되었는지 여부에 관계없이 동부 표준 시간(UTC-5) 정오를 지정한다.

문제를 더 복잡하게 만드는 것은, 일부 관할 구역이 동일한 시간대 약어를 다른 시간에 다른 UTC 오프셋을 의미하는 데 사용했다는 것이다. 예를 들어, 모스크바에서 `MSK`는 어떤 해에는 UTC+3를 의미하고 다른 해에는 UTC+4를 의미했다. PostgreSQL은 지정된 날짜에 해당 약어가 의미했던(또는 가장 최근에 의미했던) 대로 해석한다. 그러나 위의 `EST` 예제와 마찬가지로, 이는 반드시 해당 날짜의 로컬 민간 시간과 동일하지는 않다.

모든 경우에, 시간대 이름과 약어는 대소문자를 구분하지 않고 인식된다. (이는 PostgreSQL 8.2 이전 버전과 달리, 일부 컨텍스트에서는 대소문자를 구분했지만 다른 컨텍스트에서는 그렇지 않았다.)

시간대 이름과 약어는 서버에 하드코딩되어 있지 않다. 이들은 설치 디렉토리의 `.../share/timezone/` 및 `.../share/timezonesets/` 아래에 저장된 구성 파일에서 얻어진다(섹션 [B.4](https://www.postgresql.org/docs/current/datetime-config-files.html "B.4. Date/Time Configuration Files") 참조).

[TimeZone](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-TIMEZONE) 설정 매개변수는 `postgresql.conf` 파일에서 설정하거나, [19장](https://www.postgresql.org/docs/current/runtime-config.html "Chapter 19. Server Configuration")에 설명된 다른 표준 방법 중 하나로 설정할 수 있다. 이를 설정하는 몇 가지 특별한 방법도 있다:

* SQL 명령 `SET TIME ZONE`은 세션의 시간대를 설정한다. 이는 `SET TIMEZONE TO`의 대체 문법으로, 더 SQL 표준과 호환되는 구문이다.

* `PGTZ` 환경 변수는 libpq 클라이언트가 서버에 연결할 때 `SET TIME ZONE` 명령을 보내는 데 사용된다.


### 8.5.4. Interval 입력 방식

`interval` 값은 다음과 같은 상세한 구문을 사용해 작성할 수 있다:

```
quantity
```

여기서 `quantity`는 숫자(부호가 있을 수 있음)이고, `unit`은 `microsecond`, `millisecond`, `second`, `minute`, `hour`, `day`, `week`, `month`, `year`, `decade`, `century`, `millennium` 또는 이 단위들의 약어나 복수형이다. `direction`은 `ago`이거나 비워둘 수 있다. `@` 기호는 선택적으로 사용할 수 있다. 다른 단위의 양은 적절한 부호 계산과 함께 암묵적으로 더해진다. `ago`는 모든 필드의 부호를 반전시킨다. 이 구문은 [IntervalStyle](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-INTERVALSTYLE)이 `postgres_verbose`로 설정된 경우 `interval` 출력에도 사용된다.

일, 시간, 분, 초의 양은 명시적인 단위 표시 없이 지정할 수 있다. 예를 들어, `'1 12:59:10'`은 `'1 day 12 hours 59 min 10 sec'`와 동일하게 읽힌다. 또한, 연도와 월의 조합은 대시를 사용해 지정할 수 있다. 예를 들어 `'200-10'`은 `'200 years 10 months'`와 동일하게 읽힌다. (이 짧은 형식은 사실 SQL 표준에서 허용하는 유일한 형식이며, `IntervalStyle`이 `sql_standard`로 설정된 경우 출력에도 사용된다.)

`interval` 값은 ISO 8601 시간 간격 형식으로도 작성할 수 있다. 이는 표준의 4.4.3.2절에 나오는 "지정자 형식" 또는 4.4.3.3절에 나오는 "대체 형식"을 사용한다. 지정자 형식은 다음과 같다:

```
quantity
```

문자열은 `P`로 시작해야 하며, 시간 단위를 나타내는 `T`를 포함할 수 있다. 사용 가능한 단위 약어는 [표 8.16](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-ISO8601-UNITS "표 8.16. ISO 8601 Interval 단위 약어")에 나와 있다. 단위는 생략할 수 있으며, 순서에 관계없이 지정할 수 있지만, 일보다 작은 단위는 `T` 뒤에 나타나야 한다. 특히, `M`의 의미는 `T` 앞에 있는지 뒤에 있는지에 따라 달라진다.

**표 8.16. ISO 8601 Interval 단위 약어**

| 약어 | 의미 |
| --- | --- |
| Y | 연도 |
| M | 월 (날짜 부분) |
| W | 주 |
| D | 일 |
| H | 시간 |
| M | 분 (시간 부분) |
| S | 초 |

대체 형식에서는:

```
years
```

문자열은 `P`로 시작해야 하며, `T`가 날짜와 시간 부분을 구분한다. 값은 ISO 8601 날짜와 유사한 숫자로 주어진다.

`fields` 지정과 함께 `interval` 상수를 작성하거나, `fields` 지정으로 정의된 `interval` 컬럼에 문자열을 할당할 때, 표시되지 않은 양의 해석은 `fields`에 따라 달라진다. 예를 들어, `INTERVAL '1' YEAR`은 1년으로 읽히지만, `INTERVAL '1'`은 1초로 읽힌다. 또한, `fields` 지정에서 허용하는 가장 낮은 중요도의 필드 "오른쪽"에 있는 필드 값은 조용히 버려진다. 예를 들어, `INTERVAL '1 day 2:03:04' HOUR TO MINUTE`를 작성하면 초 필드는 버려지지만, 일 필드는 버려지지 않는다.

SQL 표준에 따르면, `interval` 값의 모든 필드는 동일한 부호를 가져야 하므로, 앞에 오는 음수 부호는 모든 필드에 적용된다. 예를 들어, `'-1 2:03:04'`라는 `interval` 리터럴의 음수 부호는 일과 시간/분/초 부분 모두에 적용된다. PostgreSQL은 필드가 서로 다른 부호를 가질 수 있도록 허용하며, 전통적으로 텍스트 표현의 각 필드를 독립적으로 부호가 있는 것으로 처리한다. 따라서 이 예제에서는 시간/분/초 부분이 양수로 간주된다. `IntervalStyle`이 `sql_standard`로 설정된 경우, 앞에 오는 부호는 모든 필드에 적용된다(하지만 추가 부호가 나타나지 않는 경우에만). 그렇지 않으면 전통적인 PostgreSQL 해석이 사용된다. 모호함을 피하기 위해, 어떤 필드라도 음수인 경우 각 필드에 명시적으로 부호를 붙이는 것이 좋다.

내부적으로, `interval` 값은 세 개의 정수 필드(월, 일, 마이크로초)로 저장된다. 이 필드는 한 달의 일수가 변할 수 있고, 일광 절약 시간 전환과 관련해 하루가 23시간 또는 25시간일 수 있기 때문에 별도로 유지된다. 다른 단위를 사용하는 `interval` 입력 문자열은 이 형식으로 정규화된 후, 출력을 위해 표준화된 방식으로 재구성된다. 예를 들어:

```
SELECT '2 years 15 months 100 weeks 99 hours 123456789 milliseconds'::interval;
               interval
---------------------------------------
 3 years 3 mons 700 days 133:17:36.789
```

여기서 주는 "7일"로 이해되며 별도로 유지된 반면, 더 작고 큰 시간 단위는 결합되고 정규화되었다.

입력 필드 값은 소수 부분을 가질 수 있다. 예를 들어, `'1.5 weeks'` 또는 `'01:02:03.45'`와 같다. 그러나 `interval`은 내부적으로 정수 필드만 저장하므로, 소수 값은 더 작은 단위로 변환되어야 한다. 월보다 큰 단위의 소수 부분은 정수 개월 수로 반올림된다. 예를 들어, `'1.5 years'`는 `'1 year 6 mons'`가 된다. 주와 일의 소수 부분은 한 달을 30일, 하루를 24시간으로 가정하고 일과 마이크로초의 정수 값으로 계산된다. 예를 들어, `'1.75 months'`는 `1 mon 22 days 12:00:00`이 된다. 출력에서 소수로 표시되는 것은 초뿐이다.

[표 8.17](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-INPUT-EXAMPLES "표 8.17. Interval 입력 예제")은 유효한 `interval` 입력의 몇 가지 예를 보여준다.

**표 8.17. Interval 입력 예제**

| 예제 | 설명 |
| --- | --- |
| 1-2 | SQL 표준 형식: 1년 2개월 |
| 3 4:05:06 | SQL 표준 형식: 3일 4시간 5분 6초 |
| 1 year 2 months 3 days 4 hours 5 minutes 6 seconds | 전통적인 Postgres 형식: 1년 2개월 3일 4시간 5분 6초 |
| P1Y2M3DT4H5M6S | ISO 8601 "지정자 형식": 위와 동일한 의미 |
| P0001-02-03T04:05:06 | ISO 8601 "대체 형식": 위와 동일한 의미 |


### 8.5.5. Interval 출력

앞서 설명한 것처럼, PostgreSQL은 `interval` 값을 개월, 일, 마이크로초 단위로 저장한다. 출력 시, 개월 필드는 12로 나누어 연도와 개월로 변환된다. 일 필드는 그대로 표시된다. 마이크로초 필드는 시간, 분, 초, 소수 초로 변환된다. 따라서 개월, 분, 초는 각각 0–11, 0–59, 0–59 범위를 넘지 않도록 표시되지만, 표시되는 연도, 일, 시간 필드는 상당히 큰 값이 될 수 있다. (큰 일이나 시간 값을 다음 상위 필드로 전환하려면 [`justify_days`](https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTION-JUSTIFY-DAYS)와 [`justify_hours`](https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTION-JUSTIFY-HOURS) 함수를 사용할 수 있다.)

`interval` 타입의 출력 형식은 `SET intervalstyle` 명령을 사용해 `sql_standard`, `postgres`, `postgres_verbose`, `iso_8601` 네 가지 스타일 중 하나로 설정할 수 있다. 기본값은 `postgres` 형식이다. [표 8.18](https://www.postgresql.org/docs/current/datatype-datetime.html#INTERVAL-STYLE-OUTPUT-TABLE "표 8.18. Interval 출력 스타일 예제")은 각 출력 스타일의 예제를 보여준다.

`sql_standard` 스타일은 SQL 표준의 interval 리터럴 문자열 규격에 맞는 출력을 생성한다. 단, interval 값이 표준의 제한 사항(연도-개월만 또는 일-시간만, 양수와 음수 구성 요소가 혼합되지 않음)을 충족해야 한다. 그렇지 않으면 표준 연도-개월 리터럴 문자열 뒤에 일-시간 리터럴 문자열이 붙고, 혼합 부호 interval을 명확히 하기 위해 명시적 부호가 추가된다.

`postgres` 스타일의 출력은 [DateStyle](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DATESTYLE) 매개변수가 `ISO`로 설정된 PostgreSQL 8.4 이전 버전의 출력과 일치한다.

`postgres_verbose` 스타일의 출력은 `DateStyle` 매개변수가 `ISO`가 아닌 출력으로 설정된 PostgreSQL 8.4 이전 버전의 출력과 일치한다.

`iso_8601` 스타일의 출력은 ISO 8601 표준의 4.4.3.2 절에 설명된 "지정자 형식"과 일치한다.

**표 8.18. Interval 출력 스타일 예제**

| 스타일 지정 | 연도-개월 Interval | 일-시간 Interval | 혼합 Interval |
| --- | --- | --- | --- |
| sql_standard | 1-2 | 3 4:05:06 | -1-2 +3 -4:05:06 |
| postgres | 1 year 2 mons | 3 days 04:05:06 | -1 year -2 mons +3 days -04:05:06 |
| postgres_verbose | @ 1 year 2 mons | @ 3 days 4 hours 5 mins 6 secs | @ 1 year 2 mons -3 days 4 hours 5 mins 6 secs ago |
| iso_8601 | P1Y2M | P3DT4H5M6S | P-1Y-2M3D​T-4H-5M-6S |




## 8.6. 불리언 타입

PostgreSQL은 표준 SQL 타입인 `boolean`을 제공한다. 이 타입은 "참", "거짓", 그리고 "알 수 없음"이라는 세 가지 상태를 가질 수 있다. "알 수 없음"은 SQL의 null 값으로 표현된다.

**표 8.19. 불리언 데이터 타입**

| 이름 | 저장 크기 | 설명 |
| --- | --- | --- |
| boolean | 1바이트 | 참 또는 거짓 상태 |

불리언 상수는 SQL 쿼리에서 `TRUE`, `FALSE`, `NULL` 키워드로 표현할 수 있다.

`boolean` 타입의 입력 함수는 "참" 상태를 나타내는 다음과 같은 문자열 표현을 허용한다:

```
true
yes
on
1
```

"거짓" 상태를 나타내는 문자열 표현은 다음과 같다:

```
false
no
off
0
```

이 문자열의 고유한 접두사도 허용된다. 예를 들어 `t` 또는 `n`을 사용할 수 있다. 앞뒤 공백은 무시되며, 대소문자를 구분하지 않는다.

`boolean` 타입의 출력 함수는 항상 `t` 또는 `f`를 반환한다. 이는 [예제 8.2](https://www.postgresql.org/docs/current/datatype-boolean.html#DATATYPE-BOOLEAN-EXAMPLE "예제 8.2. 불리언 타입 사용")에서 확인할 수 있다.

**예제 8.2. `boolean` 타입 사용**

```
CREATE TABLE test1 (a boolean, b text);
INSERT INTO test1 VALUES (TRUE, 'sic est');
INSERT INTO test1 VALUES (FALSE, 'non est');
SELECT * FROM test1;
 a |    b
---+---------
 t | sic est
 f | non est

SELECT * FROM test1 WHERE a;
 a |    b
---+---------
 t | sic est
```

SQL 쿼리에서 불리언 상수를 작성할 때는 `TRUE`와 `FALSE` 키워드를 사용하는 것이 권장된다. 하지만 [4.1.2.7절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS-GENERIC "4.1.2.7. 기타 타입의 상수")에 설명된 일반 문자열 리터럴 상수 구문을 따라 문자열 표현을 사용할 수도 있다. 예를 들어 `'yes'::boolean`과 같이 작성할 수 있다.

파서는 `TRUE`와 `FALSE`가 `boolean` 타입임을 자동으로 이해하지만, `NULL`은 모든 타입이 될 수 있기 때문에 그렇지 않다. 따라서 일부 상황에서는 `NULL`을 명시적으로 `boolean`으로 캐스팅해야 할 수 있다. 예를 들어 `NULL::boolean`과 같이 작성한다. 반대로, 파서가 리터럴이 반드시 `boolean` 타입임을 추론할 수 있는 상황에서는 문자열 리터럴 불리언 값에서 캐스팅을 생략할 수 있다.




## 8.7. 열거형 타입

열거형(enum) 타입은 정적이며 순서가 정해진 값들의 집합으로 구성된 데이터 타입이다. 여러 프로그래밍 언어에서 지원하는 `enum` 타입과 동일한 개념이다. 예를 들어, 요일이나 데이터의 상태 값 집합 등이 열거형 타입의 대표적인 예시이다.


### 8.7.1. 열거형 타입 선언

열거형 타입은 `CREATE TYPE` 명령어를 사용해 생성한다. 예를 들어 다음과 같이 작성할 수 있다:

```
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
```

열거형 타입을 생성한 후에는 다른 타입과 마찬가지로 테이블이나 함수 정의에 사용할 수 있다:

```
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
CREATE TABLE person (
    name text,
    current_mood mood
);
INSERT INTO person VALUES ('Moe', 'happy');
SELECT * FROM person WHERE current_mood = 'happy';
 name | current_mood
------+--------------
 Moe  | happy
(1 row)
```


### 8.7.2. 순서 지정

열거형(enum) 타입의 값 순서는 해당 타입을 생성할 때 나열한 순서대로 정해진다. 열거형 타입은 모든 표준 비교 연산자와 관련 집계 함수를 지원한다. 예를 들면 다음과 같다:

```
INSERT INTO person VALUES ('Larry', 'sad');
INSERT INTO person VALUES ('Curly', 'ok');
SELECT * FROM person WHERE current_mood > 'sad';
 name  | current_mood
-------+--------------
 Moe   | happy
 Curly | ok
(2 rows)

SELECT * FROM person WHERE current_mood > 'sad' ORDER BY current_mood;
 name  | current_mood
-------+--------------
 Curly | ok
 Moe   | happy
(2 rows)

SELECT name
FROM person
WHERE current_mood = (SELECT MIN(current_mood) FROM person);
 name
-------
 Larry
(1 row)
```


### 8.7.3. 타입 안전성

각 열거형 데이터 타입은 독립적이며 다른 열거형 타입과 비교할 수 없다. 다음 예제를 살펴보자:

```
CREATE TYPE happiness AS ENUM ('happy', 'very happy', 'ecstatic');
CREATE TABLE holidays (
    num_weeks integer,
    happiness happiness
);
INSERT INTO holidays(num_weeks,happiness) VALUES (4, 'happy');
INSERT INTO holidays(num_weeks,happiness) VALUES (6, 'very happy');
INSERT INTO holidays(num_weeks,happiness) VALUES (8, 'ecstatic');
INSERT INTO holidays(num_weeks,happiness) VALUES (2, 'sad');
ERROR:  invalid input value for enum happiness: "sad"
SELECT person.name, holidays.num_weeks FROM person, holidays
  WHERE person.current_mood = holidays.happiness;
ERROR:  operator does not exist: mood = happiness
```

이와 같은 작업이 정말 필요하다면, 커스텀 연산자를 작성하거나 쿼리에 명시적 캐스트를 추가할 수 있다:

```
SELECT person.name, holidays.num_weeks FROM person, holidays
  WHERE person.current_mood::text = holidays.happiness::text;
 name | num_weeks
------+-----------
 Moe  |         4
(1 row)
```


### 8.7.4. 구현 세부 사항

Enum 레이블은 대소문자를 구분한다. 따라서 `'happy'`와 `'HAPPY'`는 서로 다른 값으로 간주된다. 또한 레이블 내의 공백도 의미가 있다.

Enum 타입은 주로 정적인 값 집합을 위해 설계되었지만, 기존 enum 타입에 새로운 값을 추가하거나 값을 이름 변경하는 기능을 지원한다. (자세한 내용은 [ALTER TYPE](https://www.postgresql.org/docs/current/sql-altertype.html "ALTER TYPE") 참조) 하지만 기존 값을 제거하거나 값의 정렬 순서를 변경하려면 enum 타입을 삭제하고 다시 생성해야 한다.

Enum 값은 디스크 상에서 4바이트를 차지한다. Enum 값의 텍스트 레이블 길이는 PostgreSQL에 컴파일된 `NAMEDATALEN` 설정에 의해 제한된다. 표준 빌드에서는 최대 63바이트까지 가능하다.

내부 enum 값과 텍스트 레이블 간의 변환은 시스템 카탈로그 [`pg_enum`](https://www.postgresql.org/docs/current/catalog-pg-enum.html "51.20. pg_enum")에 저장된다. 이 카탈로그를 직접 쿼리하면 유용한 정보를 얻을 수 있다.




## 8.8. 기하학적 타입

기하학적 데이터 타입은 2차원 공간 객체를 표현한다. [표 8.20](https://www.postgresql.org/docs/current/datatype-geometric.html#DATATYPE-GEO-TABLE "표 8.20. 기하학적 타입")은 PostgreSQL에서 사용 가능한 기하학적 타입을 보여준다.

**표 8.20. 기하학적 타입**

| 이름 | 저장 크기 | 설명 | 표현 |
| --- | --- | --- | --- |
| point | 16 bytes | 평면 위의 점 | (x,y) |
| line | 24 bytes | 무한 직선 | {A,B,C} |
| lseg | 32 bytes | 유한 선분 | [(x1,y1),(x2,y2)] |
| box | 32 bytes | 직사각형 박스 | (x1,y1),(x2,y2) |
| path | 16+16n bytes | 닫힌 경로 (다각형과 유사) | ((x1,y1),...) |
| path | 16+16n bytes | 열린 경로 | [(x1,y1),...] |
| polygon | 40+16n bytes | 다각형 (닫힌 경로와 유사) | ((x1,y1),...) |
| circle | 24 bytes | 원 | <(x,y),r> (중심점과 반지름) |

이 모든 타입에서 개별 좌표는 `double precision` (`float8`) 숫자로 저장된다.

확대, 이동, 회전, 교차점 결정과 같은 다양한 기하학적 연산을 수행하기 위한 풍부한 함수와 연산자가 제공된다. 이에 대한 자세한 설명은 [9.11절](https://www.postgresql.org/docs/current/functions-geometry.html "9.11. 기하학적 함수와 연산자")에서 확인할 수 있다.


### 8.8.1. 점

점은 기하학적 타입의 기본적인 2차원 구성 요소이다. `point` 타입의 값은 다음과 같은 구문 중 하나로 지정한다:

```
x
```

여기서 `x`와 `y`는 각각 부동 소수점 숫자로 표현된 좌표이다.

점은 첫 번째 구문을 사용해 출력된다.


### 8.8.2. 직선

직선은 선형 방정식 `A`x + `B`y + `C` = 0으로 표현된다. 여기서 `A`와 `B`는 동시에 0이 될 수 없다. `line` 타입의 값은 다음과 같은 형태로 입력되고 출력된다:

```
A
```

또는 입력 시 다음과 같은 형태 중 하나를 사용할 수 있다:

```
x1
```

여기서 `(x1, y1)`와 `(x2, y2)`는 직선 위의 서로 다른 두 점이다.


### 8.8.3. 선분

선분은 해당 선분의 양 끝점을 나타내는 점 쌍으로 표현된다. `lseg` 타입의 값은 다음 구문 중 하나를 사용하여 지정할 수 있다:

```
x1
```

여기서 `(x1, y1)`와 `(x2, y2)`는 선분의 양 끝점을 나타낸다.

선분은 첫 번째 구문을 사용하여 출력된다.


### 8.8.4. 박스(Boxes) [#](https://www.postgresql.org/#DATATYPE-GEOMETRIC-BOXES)

박스는 서로 반대쪽 모서리에 위치한 두 점으로 표현된다. `box` 타입의 값은 다음 구문 중 하나를 사용하여 지정할 수 있다:

```
x1
```

여기서 `(x1, y1)`와 `(x2, y2)`는 박스의 서로 반대쪽 모서리에 위치한 두 점이다.

박스는 두 번째 구문을 사용하여 출력된다.

입력 시에는 서로 반대쪽 모서리의 두 점을 제공할 수 있지만, 값은 오른쪽 상단과 왼쪽 하단 모서리의 순서로 재정렬되어 저장된다.


### 8.8.5. 경로

경로는 연결된 점들의 리스트로 표현된다. 경로는 *열린* 형태일 수 있으며, 이 경우 리스트의 첫 번째와 마지막 점은 연결되지 않은 것으로 간주된다. 반면 *닫힌* 형태의 경로는 첫 번째와 마지막 점이 연결된 것으로 간주된다.

`path` 타입의 값은 다음 구문 중 하나를 사용하여 지정할 수 있다:

```
x1
```

여기서 점들은 경로를 구성하는 선분의 끝점을 나타낸다. 대괄호(`[]`)는 열린 경로를 나타내고, 괄호(`()`)는 닫힌 경로를 나타낸다. 세 번째부터 다섯 번째 구문처럼 가장 바깥쪽 괄호가 생략된 경우, 닫힌 경로로 간주된다.

경로는 적절한 첫 번째 또는 두 번째 구문을 사용하여 출력된다.


### 8.8.6. 다각형(Polygons)

다각형은 점의 목록(다각형의 꼭짓점)으로 표현된다. 다각형은 닫힌 경로와 매우 유사하지만, 중요한 의미적 차이는 다각형은 내부 영역을 포함하는 것으로 간주되는 반면, 경로는 그렇지 않다는 점이다.

다각형과 경로 사이의 중요한 구현 차이점은 다각형의 저장된 표현이 가장 작은 경계 상자를 포함한다는 것이다. 이는 특정 검색 작업을 빠르게 하지만, 새로운 다각형을 생성할 때 경계 상자를 계산하는 데 오버헤드가 추가된다.

`polygon` 타입의 값은 다음 구문 중 하나를 사용하여 지정한다:

```
x1
```

여기서 점들은 다각형의 경계를 구성하는 선분의 끝점이다.

다각형은 첫 번째 구문을 사용하여 출력된다.


### 8.8.7. 원형 데이터 타입

원형은 중심점과 반지름으로 표현된다. `circle` 타입의 값은 다음 구문 중 하나를 사용해 지정할 수 있다:

```
x
```

여기서 `(x, y)`는 중심점을 나타내고, `r`은 원의 반지름을 의미한다.

원형은 첫 번째 구문을 사용해 출력된다.




## 8.9. 네트워크 주소 타입

PostgreSQL은 IPv4, IPv6, MAC 주소를 저장하기 위한 데이터 타입을 제공한다. [표 8.21](https://www.postgresql.org/docs/current/datatype-net-types.html#DATATYPE-NET-TYPES-TABLE "표 8.21. 네트워크 주소 타입")에서 확인할 수 있다. 네트워크 주소를 저장할 때 일반 텍스트 타입 대신 이러한 타입을 사용하는 것이 좋다. 이 타입들은 입력 오류 검사와 특화된 연산자 및 함수를 제공하기 때문이다. 자세한 내용은 [9.12절](https://www.postgresql.org/docs/current/functions-net.html "9.12. 네트워크 주소 함수와 연산자")을 참고한다.

**표 8.21. 네트워크 주소 타입**

| 이름 | 저장 크기 | 설명 |
| --- | --- | --- |
| cidr | 7 또는 19 바이트 | IPv4 및 IPv6 네트워크 |
| inet | 7 또는 19 바이트 | IPv4 및 IPv6 호스트와 네트워크 |
| macaddr | 6 바이트 | MAC 주소 |
| macaddr8 | 8 바이트 | MAC 주소 (EUI-64 형식) |

`inet` 또는 `cidr` 데이터 타입을 정렬할 때, IPv4 주소는 항상 IPv6 주소보다 앞에 위치한다. 이는 IPv4 주소가 IPv6 주소로 캡슐화되거나 매핑된 경우에도 적용된다. 예를 들어 `::10.2.3.4` 또는 `::ffff:10.4.3.2`와 같은 경우에도 IPv4 주소가 먼저 정렬된다.


### 8.9.1. `inet` 타입

`inet` 타입은 IPv4 또는 IPv6 호스트 주소와 선택적으로 서브넷을 하나의 필드에 저장한다. 서브넷은 호스트 주소에 포함된 네트워크 주소 비트 수(넷마스크)로 표현된다. 넷마스크가 32이고 주소가 IPv4인 경우, 이 값은 서브넷이 아닌 단일 호스트를 나타낸다. IPv6의 경우 주소 길이는 128비트이므로 128비트는 고유한 호스트 주소를 지정한다. 네트워크만 허용하려면 `inet` 대신 `cidr` 타입을 사용해야 한다.

이 타입의 입력 형식은 `address/y`이며, 여기서 `address`는 IPv4 또는 IPv6 주소이고 `y`는 넷마스크의 비트 수이다. `/y` 부분을 생략하면 넷마스크는 IPv4의 경우 32, IPv6의 경우 128로 간주되므로 값은 단일 호스트를 나타낸다. 표시 시 넷마스크가 단일 호스트를 지정하면 `/y` 부분은 생략된다.


### 8.9.2. `cidr` 타입

`cidr` 타입은 IPv4 또는 IPv6 네트워크 사양을 포함한다. 입력 및 출력 형식은 클래스 없는 인터넷 도메인 라우팅(CIDR) 규약을 따른다. 네트워크를 지정하는 형식은 `address/y`이다. 여기서 `address`는 IPv4 또는 IPv6 주소로 표현된 네트워크의 가장 낮은 주소이고, `y`는 네트마스크의 비트 수이다. `y`를 생략하면, 이전의 클래스 기반 네트워크 번호 체계를 기반으로 계산되지만, 입력에 작성된 모든 옥텟을 포함할 수 있을 만큼 충분히 큰 값이 된다. 지정된 네트마스크 오른쪽에 비트가 설정된 네트워크 주소를 지정하면 오류가 발생한다.

[표 8.22](https://www.postgresql.org/docs/current/datatype-net-types.html#DATATYPE-NET-CIDR-TABLE "표 8.22. cidr 타입 입력 예제")는 몇 가지 예제를 보여준다.

**표 8.22. `cidr` 타입 입력 예제**

| cidr 입력 | cidr 출력 | abbrev(cidr) |
| --- | --- | --- |
| 192.168.100.128/25 | 192.168.100.128/25 | 192.168.100.128/25 |
| 192.168/24 | 192.168.0.0/24 | 192.168.0/24 |
| 192.168/25 | 192.168.0.0/25 | 192.168.0.0/25 |
| 192.168.1 | 192.168.1.0/24 | 192.168.1/24 |
| 192.168 | 192.168.0.0/24 | 192.168.0/24 |
| 128.1 | 128.1.0.0/16 | 128.1/16 |
| 128 | 128.0.0.0/16 | 128.0/16 |
| 128.1.2 | 128.1.2.0/24 | 128.1.2/24 |
| 10.1.2 | 10.1.2.0/24 | 10.1.2/24 |
| 10.1 | 10.1.0.0/16 | 10.1/16 |
| 10 | 10.0.0.0/8 | 10/8 |
| 10.1.2.3/32 | 10.1.2.3/32 | 10.1.2.3/32 |
| 2001:4f8:3:ba::/64 | 2001:4f8:3:ba::/64 | 2001:4f8:3:ba/64 |
| 2001:4f8:3:ba:​2e0:81ff:fe22:d1f1/128 | 2001:4f8:3:ba:​2e0:81ff:fe22:d1f1/128 | 2001:4f8:3:ba:​2e0:81ff:fe22:d1f1/128 |
| ::ffff:1.2.3.0/120 | ::ffff:1.2.3.0/120 | ::ffff:1.2.3/120 |
| ::ffff:1.2.3.0/128 | ::ffff:1.2.3.0/128 | ::ffff:1.2.3.0/128 |


### 8.9.3. `inet` vs. `cidr`

`inet`과 `cidr` 데이터 타입의 가장 큰 차이점은 `inet`은 넷마스크 오른쪽에 비트가 0이 아닌 값을 허용하지만, `cidr`은 허용하지 않는다는 점이다. 예를 들어, `192.168.0.1/24`는 `inet`에서는 유효하지만 `cidr`에서는 유효하지 않다.


`inet` 또는 `cidr` 값의 출력 형식이 마음에 들지 않는다면, `host`, `text`, `abbrev` 함수를 사용해 보세요.


### 8.9.4. `macaddr` 타입

`macaddr` 타입은 MAC 주소를 저장한다. MAC 주소는 이더넷 카드 하드웨어 주소로 잘 알려져 있지만, 다른 용도로도 사용된다. 이 타입은 다음과 같은 형식의 입력을 허용한다:

| macaddr |
|---|
| '08:00:2b:01:02:03' |
| '08-00-2b-01-02-03' |
| '08002b:010203' |
| '08002b-010203' |
| '0800.2b01.0203' |
| '0800-2b01-0203' |
| '08002b010203' |

이 예제들은 모두 동일한 주소를 나타낸다. `a`부터 `f`까지의 숫자는 대소문자를 구분하지 않는다. 출력은 항상 첫 번째 형식으로 표시된다.

IEEE 표준 802-2001은 두 번째 형식(하이픈 사용)을 MAC 주소의 표준 형식으로 지정하고, 첫 번째 형식(콜론 사용)은 비트 반전된 MSB 우선 표기법으로 사용한다. 따라서 08-00-2b-01-02-03 = 10:00:D4:80:40:C0이 된다. 이 규칙은 현재 널리 무시되며, 오래된 네트워크 프로토콜(예: 토큰 링)에서만 관련이 있다. PostgreSQL은 비트 반전을 지원하지 않으며, 모든 허용 형식은 표준 LSB 순서를 사용한다.

나머지 다섯 가지 입력 형식은 어떤 표준에도 속하지 않는다.


### 8.9.5. `macaddr8`

`macaddr8` 타입은 EUI-64 형식으로 MAC 주소를 저장한다. 이 형식은 이더넷 카드의 하드웨어 주소에서 흔히 볼 수 있다(하지만 MAC 주소는 다른 용도로도 사용된다). 이 타입은 6바이트와 8바이트 길이의 MAC 주소를 모두 받아들일 수 있으며, 8바이트 길이 형식으로 저장한다. 6바이트 형식으로 제공된 MAC 주소는 4번째와 5번째 바이트가 각각 FF와 FE로 설정된 8바이트 형식으로 저장된다. IPv6는 수정된 EUI-64 형식을 사용하는데, EUI-48에서 변환할 때 7번째 비트를 1로 설정해야 한다. 이 변경을 위해 `macaddr8_set7bit` 함수가 제공된다. 일반적으로, 16진수 쌍(바이트 경계)으로 구성된 입력은 `':'`, `'-'`, `'.'` 중 하나로 일관되게 구분되어 있다면 허용된다. 16진수 숫자의 개수는 16개(8바이트) 또는 12개(6바이트)여야 한다. 앞뒤의 공백은 무시된다. 다음은 허용되는 입력 형식의 예시다:

| macaddr8 |
|---|
| '08:00:2b:01:02:03:04:05' |
| '08-00-2b-01-02-03-04-05' |
| '08002b:0102030405' |
| '08002b-0102030405' |
| '0800.2b01.0203.0405' |
| '0800-2b01-0203-0405' |
| '08002b01:02030405' |
| '08002b0102030405' |

이 예시들은 모두 동일한 주소를 지정한다. `a`부터 `f`까지의 숫자는 대소문자를 구분하지 않는다. 출력은 항상 첫 번째 형식으로 표시된다.

위에 나온 마지막 여섯 가지 입력 형식은 어떤 표준에도 속하지 않는다.

기존의 48비트 MAC 주소(EUI-48 형식)를 수정된 EUI-64 형식으로 변환하여 IPv6 주소의 호스트 부분으로 포함시키려면 다음과 같이 `macaddr8_set7bit`를 사용한다:

```
macaddr8_set7bit
-------------------------
 0a:00:2b:ff:fe:01:02:03
(1 row)
```




## 8.10. 비트 문자열 타입

비트 문자열은 1과 0으로 이루어진 문자열이다. 비트 마스크를 저장하거나 시각화하는 데 사용할 수 있다. SQL에는 두 가지 비트 타입이 있다: `bit(n)`와 `bit varying(n)`. 여기서 `n`은 양의 정수이다.

`bit` 타입 데이터는 길이 `n`과 정확히 일치해야 한다. 더 짧거나 더 긴 비트 문자열을 저장하려고 하면 오류가 발생한다. `bit varying` 데이터는 최대 길이 `n`까지 가변 길이를 가진다. 더 긴 문자열은 거부된다. 길이를 지정하지 않고 `bit`를 사용하면 `bit(1)`과 동일하며, `bit varying`에 길이를 지정하지 않으면 길이 제한이 없다는 의미이다.


### 주의 사항

비트 문자열 값을 `bit(n)`로 명시적으로 캐스팅하면, 오류를 발생시키지 않고 정확히 `n` 비트가 되도록 오른쪽에서 잘리거나 0으로 패딩된다. 마찬가지로, 비트 문자열 값을 `bit varying(n)`로 명시적으로 캐스팅하면, `n` 비트를 초과하는 경우 오른쪽에서 잘린다.

비트 문자열 상수의 문법에 대한 자세한 내용은 [4.1.2.5절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-BIT-STRINGS "4.1.2.5. Bit-String Constants")을 참고한다. 비트 논리 연산자와 문자열 조작 함수는 [9.6절](https://www.postgresql.org/docs/current/functions-bitstring.html "9.6. Bit String Functions and Operators")에서 확인할 수 있다.

**예제 8.3. 비트 문자열 타입 사용**

```
ERROR:  bit string length 2 does not match type bit(3)
```

비트 문자열 값은 8비트마다 1바이트가 필요하며, 문자열 길이에 따라 5바이트 또는 8바이트의 오버헤드가 추가된다. (단, 긴 값은 [8.3절](https://www.postgresql.org/docs/current/datatype-character.html "8.3. Character Types")에서 설명한 대로 압축되거나 외부로 이동될 수 있다.)




## 8.11. 텍스트 검색 타입

PostgreSQL은 자연어로 작성된 문서 집합을 검색해 쿼리와 가장 잘 일치하는 문서를 찾는 활동인 **전문 검색(full text search)**을 지원하기 위해 설계된 두 가지 데이터 타입을 제공한다. `tsvector` 타입은 텍스트 검색에 최적화된 형태로 문서를 표현하며, `tsquery` 타입은 텍스트 쿼리를 표현한다. [12장](https://www.postgresql.org/docs/current/textsearch.html "12장. 전문 검색")에서는 이 기능에 대한 자세한 설명을 제공하고, [9.13절](https://www.postgresql.org/docs/current/functions-textsearch.html "9.13. 텍스트 검색 함수와 연산자")에서는 관련 함수와 연산자를 요약한다.


### 8.11.1. `tsvector`

`tsvector` 값은 정렬된 고유 *어휘소(lexeme)* 목록이다. 어휘소는 동일한 단어의 다양한 변형을 통합하기 위해 *정규화(normalized)*된 단어를 의미한다. (자세한 내용은 [12장](https://www.postgresql.org/docs/current/textsearch.html "12장. 전체 텍스트 검색") 참조) 입력 시 자동으로 정렬과 중복 제거가 이루어지며, 다음 예제에서 이를 확인할 수 있다:

```
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector;
                      tsvector
----------------------------------------------------
 'a' 'and' 'ate' 'cat' 'fat' 'mat' 'on' 'rat' 'sat'
```

공백이나 구두점을 포함하는 어휘소를 표현하려면 따옴표로 감싸야 한다:

```
SELECT $$the lexeme '    ' contains spaces$$::tsvector;
                 tsvector
-------------------------------------------
 '    ' 'contains' 'lexeme' 'spaces' 'the'
```

(이 예제와 다음 예제에서는 리터럴 내부에서 따옴표를 두 번 사용해야 하는 혼란을 피하기 위해 달러 기호로 감싼 문자열 리터럴을 사용한다.) 내장된 따옴표와 백슬래시는 두 번 입력해야 한다:

```
SELECT $$the lexeme 'Joe''s' contains a quote$$::tsvector;
                    tsvector
------------------------------------------------
 'Joe''s' 'a' 'contains' 'lexeme' 'quote' 'the'
```

선택적으로, 어휘소에 정수 *위치(position)*를 첨부할 수 있다:

```
SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:9 a:10 fat:11 rat:12'::tsvector;
                                  tsvector
-------------------------------------------------------------------​------------
 'a':1,6,10 'and':8 'ate':9 'cat':3 'fat':2,11 'mat':7 'on':5 'rat':12 'sat':4
```

위치는 일반적으로 문서 내에서 원본 단어의 위치를 나타낸다. 위치 정보는 *근접 순위(proximity ranking)* 에 사용할 수 있다. 위치 값은 1부터 16383까지 가능하며, 이보다 큰 숫자는 자동으로 16383으로 설정된다. 동일한 어휘소에 대한 중복 위치는 제거된다.

위치가 지정된 어휘소는 추가로 *가중치(weight)* 를 부여할 수 있다. 가중치는 `A`, `B`, `C`, `D` 중 하나일 수 있으며, `D`가 기본값이므로 출력 시 표시되지 않는다:

```
SELECT 'a:1A fat:2B,4C cat:5D'::tsvector;
          tsvector
----------------------------
 'a':1A 'cat':5 'fat':2B,4C
```

가중치는 일반적으로 문서 구조를 반영하기 위해 사용된다. 예를 들어, 제목 단어와 본문 단어를 다르게 표시할 수 있다. 텍스트 검색 순위 함수는 다른 가중치 마커에 대해 다른 우선순위를 부여할 수 있다.

`tsvector` 타입 자체는 어떠한 단어 정규화도 수행하지 않는다는 점을 이해하는 것이 중요하다. 이 타입은 주어진 단어가 애플리케이션에 적합하게 정규화되었다고 가정한다. 예를 들어:

```
SELECT 'The Fat Rats'::tsvector;
      tsvector
--------------------
 'Fat' 'Rats' 'The'
```

대부분의 영어 텍스트 검색 애플리케이션에서 위 단어들은 정규화되지 않은 것으로 간주되지만, `tsvector`는 이를 신경 쓰지 않는다. 원본 문서 텍스트는 일반적으로 검색에 적합하게 단어를 정규화하기 위해 `to_tsvector`를 통해 전달되어야 한다:

```
SELECT to_tsvector('english', 'The Fat Rats');
   to_tsvector
-----------------
 'fat':2 'rat':3
```

더 자세한 내용은 [12장](https://www.postgresql.org/docs/current/textsearch.html "12장. 전체 텍스트 검색")을 참조하라.


### 8.11.2. `tsquery`

`tsquery`는 검색할 어휘를 저장하며, 이들 어휘를 불리언 연산자 `&` (AND), `|` (OR), `!` (NOT)와 구문 검색 연산자 `<->` (FOLLOWED BY)를 사용해 결합할 수 있다. 또한 FOLLOWED BY 연산자의 변형인 `<N>`도 존재하는데, 여기서 `N`은 검색할 두 어휘 간의 거리를 지정하는 정수 상수다. `<->`는 `<1>`과 동일하다.

괄호를 사용해 이러한 연산자의 그룹화를 강제할 수 있다. 괄호가 없는 경우, `!` (NOT)가 가장 강하게 결합되고, 그다음으로 `<->` (FOLLOWED BY), `&` (AND), 마지막으로 `|` (OR)가 가장 약하게 결합된다.

다음은 몇 가지 예제다:

```
SELECT 'fat & rat'::tsquery;
    tsquery
---------------
 'fat' & 'rat'

SELECT 'fat & (rat | cat)'::tsquery;
          tsquery
---------------------------
 'fat' & ( 'rat' | 'cat' )

SELECT 'fat & rat & ! cat'::tsquery;
        tsquery
------------------------
 'fat' & 'rat' & !'cat'
```

선택적으로, `tsquery`의 어휘에 하나 이상의 가중치 문자를 붙여 해당 가중치를 가진 `tsvector` 어휘와만 매치되도록 제한할 수 있다:

```
SELECT 'fat:ab & cat'::tsquery;
    tsquery
------------------
 'fat':AB & 'cat'
```

또한, `tsquery`의 어휘에 `*`를 붙여 접두사 매칭을 지정할 수 있다:

```
SELECT 'super:*'::tsquery;
  tsquery
-----------
 'super':*
```

이 쿼리는 "super"로 시작하는 모든 단어와 매치된다.

어휘에 대한 인용 규칙은 앞서 설명한 `tsvector`의 어휘와 동일하다. 또한 `tsvector`와 마찬가지로, `tsquery` 타입으로 변환하기 전에 필요한 단어 정규화를 수행해야 한다. `to_tsquery` 함수는 이러한 정규화를 수행하는 데 편리하다:

```
SELECT to_tsquery('Fat:ab & Cats');
    to_tsquery
------------------
 'fat':AB & 'cat'
```

`to_tsquery`는 접두사를 다른 단어와 동일한 방식으로 처리하므로, 다음 비교는 true를 반환한다:

```
SELECT to_tsvector( 'postgraduate' ) @@ to_tsquery( 'postgres:*' );
 ?column?
----------
 t
```

왜냐하면 `postgres`는 `postgr`로 어간 추출되기 때문이다:

```
SELECT to_tsvector( 'postgraduate' ), to_tsquery( 'postgres:*' );
  to_tsvector  | to_tsquery
---------------+------------
 'postgradu':1 | 'postgr':*
```

이 결과는 `postgraduate`의 어간 추출된 형태와 매치된다.




## 8.12. UUID 타입

`uuid` 데이터 타입은 [RFC 4122](https://datatracker.ietf.org/doc/html/rfc4122), ISO/IEC 9834-8:2005 및 관련 표준에 정의된 범용 고유 식별자(UUID)를 저장한다. (일부 시스템에서는 이 데이터 타입을 전역 고유 식별자, 즉 GUID라고 부르기도 한다.) 이 식별자는 128비트 크기의 값으로, 알고리즘을 통해 생성되며, 동일한 알고리즘을 사용해도 다른 누군가가 동일한 식별자를 생성할 가능성이 매우 낮다. 따라서 분산 시스템에서 이 식별자는 단일 데이터베이스 내에서만 고유성을 보장하는 시퀀스 생성자보다 더 나은 고유성을 제공한다.

UUID는 소문자 16진수로 표기되며, 하이픈으로 구분된 여러 그룹으로 나뉜다. 구체적으로, 8자리 그룹 하나, 4자리 그룹 세 개, 그리고 12자리 그룹 하나로 구성되어 총 32자리로 128비트를 나타낸다. 이 표준 형식의 UUID 예시는 다음과 같다:

```
a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11
```

PostgreSQL은 입력 시 다음과 같은 대체 형식도 허용한다: 대문자 사용, 표준 형식을 중괄호로 감싸기, 일부 또는 모든 하이픈 생략, 4자리 그룹 뒤에 하이픈 추가 등이다. 예시는 다음과 같다:

```
A0EEBC99-9C0B-4EF8-BB6D-6BB9BD380A11
{a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11}
a0eebc999c0b4ef8bb6d6bb9bd380a11
a0ee-bc99-9c0b-4ef8-bb6d-6bb9-bd38-0a11
{a0eebc99-9c0b4ef8-bb6d6bb9-bd380a11}
```

출력은 항상 표준 형식으로 제공된다.

PostgreSQL에서 UUID를 생성하는 방법은 [9.14절](https://www.postgresql.org/docs/current/functions-uuid.html "9.14. UUID Functions")을 참고한다.




## 8.13. XML 타입

`xml` 데이터 타입은 XML 데이터를 저장하는 데 사용된다. 이 타입은 XML 데이터를 `text` 필드에 저장하는 것과 비교해 몇 가지 장점이 있다. 입력 값이 올바른 형식인지 검사하고, 타입 안전한 연산을 수행할 수 있는 지원 함수를 제공한다. 자세한 내용은 [9.15절](https://www.postgresql.org/docs/current/functions-xml.html "9.15. XML 함수")을 참조한다. 이 데이터 타입을 사용하려면 `configure --with-libxml` 옵션을 사용해 설치해야 한다.

`xml` 타입은 XML 표준에 정의된 "문서" 뿐만 아니라, XQuery와 XPath 데이터 모델에서 더 유연한 ["문서 노드"](https://www.w3.org/TR/2010/REC-xpath-datamodel-20101214/#DocumentNode)를 참조해 정의된 "콘텐츠" 프래그먼트도 저장할 수 있다. 간단히 말해, 콘텐츠 프래그먼트는 여러 최상위 엘리먼트나 문자 노드를 가질 수 있다. ``*`xmlvalue`* IS DOCUMENT`` 표현식을 사용하면 특정 `xml` 값이 완전한 문서인지, 아니면 콘텐츠 프래그먼트인지 평가할 수 있다.

`xml` 데이터 타입의 제한 사항과 호환성 관련 정보는 [D.3절](https://www.postgresql.org/docs/current/xml-limits-conformance.html "D.3. XML 제한 사항 및 SQL/XML 준수")에서 확인할 수 있다.


### 8.13.1. XML 값 생성

문자 데이터를 `xml` 타입으로 변환하려면 `xmlparse` 함수를 사용한다:

```
value
```

예제:

```
XMLPARSE (DOCUMENT '<?xml version="1.0"?><book><title>Manual</title><chapter>...</chapter></book>')
XMLPARSE (CONTENT 'abc<foo>bar</foo><bar>foo</bar>')
```

이 방법은 SQL 표준에 따라 문자열을 XML 값으로 변환하는 유일한 방법이다. 하지만 PostgreSQL에서는 다음과 같은 특수 문법도 사용할 수 있다:

```
xml '<foo>bar</foo>'
'<foo>bar</foo>'::xml
```

`xml` 타입은 입력 값을 문서 타입 선언(DTD)에 대해 검증하지 않는다. 입력 값이 DTD를 지정하더라도 마찬가지다. 또한 현재는 XML 스키마 언어(예: XML Schema)에 대한 검증을 지원하지 않는다.

`xml` 타입을 문자열로 변환하려면 `xmlserialize` 함수를 사용한다:

```
value
```

`type`은 `character`, `character varying`, `text` 또는 이들의 별칭 중 하나가 될 수 있다. SQL 표준에 따르면 이 방법이 `xml` 타입과 문자 타입 간 변환의 유일한 방법이지만, PostgreSQL에서는 간단히 캐스팅하는 것도 허용한다.

`INDENT` 옵션을 사용하면 결과를 보기 좋게 출력할 수 있고, `NO INDENT`(기본값)는 원본 입력 문자열을 그대로 출력한다. 문자 타입으로 캐스팅할 때도 원본 문자열이 생성된다.

`XMLPARSE`나 `XMLSERIALIZE`를 거치지 않고 문자열을 `xml` 타입으로 캐스팅하거나 그 반대의 경우, `DOCUMENT`와 `CONTENT` 중 어떤 것을 사용할지는 "XML 옵션" 세션 설정 매개변수에 의해 결정된다. 이 매개변수는 다음 표준 명령어로 설정할 수 있다:

```
SET XML OPTION { DOCUMENT | CONTENT };
```

또는 PostgreSQL 스타일의 구문으로:

```
SET xmloption TO { DOCUMENT | CONTENT };
```

기본값은 `CONTENT`로, 모든 형태의 XML 데이터를 허용한다.


### 8.13.2. 인코딩 처리

클라이언트, 서버, 그리고 그 사이를 오가는 XML 데이터 간의 다양한 문자 인코딩을 다룰 때는 주의가 필요하다. 일반적으로 사용되는 텍스트 모드로 쿼리를 서버에 전달하고 쿼리 결과를 클라이언트에 반환할 때, PostgreSQL은 클라이언트와 서버 간에 전달되는 모든 문자 데이터를 각 측의 문자 인코딩으로 변환한다. 이는 위 예제에서와 같이 XML 값의 문자열 표현도 포함된다. 이 과정에서 XML 데이터에 포함된 인코딩 선언은 변경되지 않기 때문에, 클라이언트와 서버 간에 데이터가 이동하면서 문자 데이터가 다른 인코딩으로 변환될 때 인코딩 선언이 무효화될 수 있다. 이러한 동작을 해결하기 위해, `xml` 타입으로 입력되는 문자열에 포함된 인코딩 선언은 *무시*되며, 내용은 현재 서버 인코딩으로 간주된다. 따라서 올바른 처리를 위해 XML 데이터의 문자열은 클라이언트에서 현재 클라이언트 인코딩으로 전송되어야 한다. 클라이언트는 문서를 서버로 보내기 전에 현재 클라이언트 인코딩으로 변환하거나, 적절히 클라이언트 인코딩을 조정할 책임이 있다. 출력 시 `xml` 타입의 값에는 인코딩 선언이 포함되지 않으며, 클라이언트는 모든 데이터가 현재 클라이언트 인코딩으로 되어 있다고 가정해야 한다.

쿼리 매개변수를 서버에 전달하고 쿼리 결과를 클라이언트에 반환할 때 바이너리 모드를 사용하면 인코딩 변환이 수행되지 않으므로 상황이 다르다. 이 경우 XML 데이터의 인코딩 선언이 유효하며, 선언이 없으면 데이터는 UTF-8로 간주된다(XML 표준에 따라; PostgreSQL은 UTF-16을 지원하지 않는다는 점에 유의). 출력 시 데이터는 클라이언트 인코딩을 지정하는 인코딩 선언을 포함하지만, 클라이언트 인코딩이 UTF-8인 경우 생략된다.

당연히, XML 데이터 인코딩, 클라이언트 인코딩, 서버 인코딩이 모두 동일할 때 PostgreSQL에서 XML 데이터를 처리하는 것이 오류 발생 가능성이 적고 더 효율적이다. XML 데이터는 내부적으로 UTF-8로 처리되므로, 서버 인코딩도 UTF-8일 때 계산이 가장 효율적이다.


### 주의 사항

서버 인코딩이 UTF-8이 아닌 경우, 일부 XML 관련 함수는 비 ASCII 데이터에서 전혀 작동하지 않을 수 있다. 특히 `xmltable()`과 `xpath()` 함수에서 이러한 문제가 발생하는 것으로 알려져 있다.


### 8.13.3. XML 값에 접근하기

`xml` 데이터 타입은 비교 연산자를 제공하지 않는다는 점에서 특이하다. 이는 XML 데이터에 대해 잘 정의되고 보편적으로 유용한 비교 알고리즘이 없기 때문이다. 이로 인해 `xml` 컬럼을 검색 값과 비교해 행을 검색할 수 없다. 따라서 XML 값은 일반적으로 ID와 같은 별도의 키 필드와 함께 사용해야 한다. XML 값을 비교하는 또 다른 방법은 먼저 문자열로 변환하는 것이지만, 문자열 비교는 유용한 XML 비교 방법과는 거의 관련이 없다는 점에 유의해야 한다.

`xml` 데이터 타입에 비교 연산자가 없기 때문에, 이 타입의 컬럼에 직접 인덱스를 생성할 수 없다. XML 데이터에서 빠른 검색을 원한다면, 표현식을 문자열 타입으로 캐스팅한 후 인덱스를 생성하거나, XPath 표현식에 인덱스를 생성하는 방법을 고려할 수 있다. 물론 실제 쿼리는 인덱싱된 표현식을 기준으로 검색하도록 조정해야 한다.

PostgreSQL의 텍스트 검색 기능을 사용해 XML 데이터의 전체 문서 검색 속도를 높일 수도 있다. 하지만 필요한 전처리 지원은 아직 PostgreSQL 배포판에서 제공되지 않는다.




## 8.14. JSON 타입

JSON(JavaScript Object Notation) 데이터는 [RFC 7159](https://datatracker.ietf.org/doc/html/rfc7159)에 명시된 대로 저장할 수 있다. 이러한 데이터는 `text` 타입으로도 저장할 수 있지만, JSON 타입을 사용하면 저장된 값이 JSON 규칙에 따라 유효한지 검증할 수 있다는 장점이 있다. 또한 JSON 타입에 저장된 데이터를 처리하기 위한 다양한 함수와 연산자를 제공한다. 자세한 내용은 [9.16절](https://www.postgresql.org/docs/current/functions-json.html "9.16. JSON 함수와 연산자")을 참고한다.

PostgreSQL은 JSON 데이터를 저장하기 위해 `json`과 `jsonb` 두 가지 타입을 제공한다. 이 데이터 타입에 대한 효율적인 쿼리 메커니즘을 구현하기 위해 PostgreSQL은 [8.14.7절](https://www.postgresql.org/docs/current/datatype-json.html#DATATYPE-JSONPATH "8.14.7. jsonpath 타입")에서 설명하는 `jsonpath` 데이터 타입도 제공한다.

`json`과 `jsonb` 타입은 거의 동일한 값을 입력으로 받는다. 주요한 차이점은 효율성에 있다. `json` 타입은 입력 텍스트를 그대로 저장하며, 처리 함수는 매번 실행할 때마다 이를 다시 파싱해야 한다. 반면 `jsonb` 타입은 분해된 바이너리 형식으로 저장되기 때문에 변환 오버헤드로 인해 입력 속도가 약간 느리지만, 파싱이 필요 없어 처리 속도가 훨씬 빠르다. 또한 `jsonb`는 인덱싱을 지원하므로 큰 이점이 될 수 있다.

`json` 타입은 입력 텍스트를 그대로 저장하기 때문에 토큰 사이의 의미 없는 공백과 JSON 객체 내 키의 순서를 보존한다. 또한 값 내 JSON 객체에 동일한 키가 여러 번 포함된 경우 모든 키/값 쌍을 유지한다. (처리 함수는 마지막 값을 유효한 값으로 간주한다.) 반면 `jsonb`는 공백을 보존하지 않으며, 객체 키의 순서를 유지하지 않고 중복된 객체 키도 유지하지 않는다. 입력에 중복 키가 지정된 경우 마지막 값만 유지된다.

일반적으로 대부분의 애플리케이션은 객체 키의 순서와 같은 특수한 요구사항이 없는 한 JSON 데이터를 `jsonb`로 저장하는 것이 좋다.

RFC 7159은 JSON 문자열이 UTF8로 인코딩되어야 한다고 명시한다. 따라서 데이터베이스 인코딩이 UTF8이 아닌 경우 JSON 타입이 JSON 사양을 엄격히 준수할 수 없다. 데이터베이스 인코딩으로 표현할 수 없는 문자를 직접 포함하려고 하면 실패한다. 반대로 데이터베이스 인코딩으로는 표현할 수 있지만 UTF8로는 표현할 수 없는 문자는 허용된다.

RFC 7159은 JSON 문자열이 ``u*`XXXX`*``로 표시된 유니코드 이스케이프 시퀀스를 포함할 수 있도록 허용한다. `json` 타입의 입력 함수에서는 데이터베이스 인코딩과 관계없이 유니코드 이스케이프를 허용하며, 구문적 정확성(즉, `u` 뒤에 16진수 4자리가 오는지)만 검사한다. 그러나 `jsonb` 타입의 입력 함수는 더 엄격하다. 데이터베이스 인코딩으로 표현할 수 없는 문자에 대한 유니코드 이스케이프를 허용하지 않는다. 또한 `jsonb` 타입은 `u0000`을 거부한다(PostgreSQL의 `text` 타입으로는 표현할 수 없기 때문). 그리고 유니코드 기본 다국어 평면 외부의 문자를 지정하기 위해 유니코드 서로게이트 쌍을 사용할 때 정확해야 한다고 강조한다. 유효한 유니코드 이스케이프는 저장 시 단일 문자로 변환되며, 여기에는 서로게이트 쌍을 단일 문자로 변환하는 과정도 포함된다.


### 참고

[9.16절](https://www.postgresql.org/docs/current/functions-json.html "9.16. JSON Functions and Operators")에서 설명한 많은 JSON 처리 함수는 유니코드 이스케이프를 일반 문자로 변환한다. 따라서 입력 타입이 `json`이더라도 앞서 설명한 것과 동일한 유형의 오류를 발생시킬 수 있다. `json` 입력 함수가 이러한 검사를 수행하지 않는 것은 역사적 유산으로 간주될 수 있지만, 이는 데이터베이스 인코딩에서 지원하지 않는 문자를 포함한 JSON 유니코드 이스케이프를 처리 없이 간단히 저장할 수 있게 한다.

텍스트 형식의 JSON 입력을 `jsonb`로 변환할 때, RFC 7159에 설명된 기본 타입은 [표 8.23](https://www.postgresql.org/docs/current/datatype-json.html#JSON-TYPE-MAPPING-TABLE "표 8.23. JSON 기본 타입과 PostgreSQL 타입 매핑")과 같이 PostgreSQL의 기본 타입에 효과적으로 매핑된다. 따라서 `jsonb` 데이터로 유효한지 여부에 대한 몇 가지 추가 제약이 존재한다. 이러한 제약은 `json` 타입이나 추상적인 JSON에는 적용되지 않으며, 이는 기본 데이터 타입으로 표현할 수 있는 한계에 해당한다. 특히, `jsonb`는 PostgreSQL `numeric` 데이터 타입의 범위를 벗어나는 숫자를 거부하지만, `json`은 그렇지 않다. 이러한 구현 정의 제한은 RFC 7159에서 허용한다. 그러나 실제로는 JSON의 `number` 기본 타입을 IEEE 754 배정밀도 부동소수점으로 표현하는 것이 일반적이기 때문에(이는 RFC 7159에서 명시적으로 예상하고 허용한다), 다른 구현에서 이러한 문제가 발생할 가능성이 훨씬 더 높다. PostgreSQL에 원래 저장된 데이터와 비교해 숫자 정밀도를 잃을 위험을 고려해야 한다.

반대로, 표에서 언급한 것처럼 JSON 기본 타입의 입력 형식에 대해 PostgreSQL 타입에는 적용되지 않는 몇 가지 사소한 제약이 있다.

**표 8.23. JSON 기본 타입과 PostgreSQL 타입 매핑**

| JSON 기본 타입 | PostgreSQL 타입 | 참고 |
| --- | --- | --- |
| string | text | u0000은 허용되지 않으며, 데이터베이스 인코딩에서 사용할 수 없는 문자를 나타내는 유니코드 이스케이프도 허용되지 않음 |
| number | numeric | NaN 및 무한대 값은 허용되지 않음 |
| boolean | boolean | 소문자 true와 false만 허용됨 |
| null | (없음) | SQL NULL은 다른 개념임 |


### 8.14.1. JSON 입력 및 출력 문법

JSON 데이터 타입의 입력/출력 문법은 RFC 7159에 명시된 규격을 따른다.

다음은 모두 유효한 `json` (또는 `jsonb`) 표현식이다:

```
-- 단순 스칼라/기본 값
-- 기본 값은 숫자, 따옴표로 감싼 문자열, true, false, 또는 null이 될 수 있다
SELECT '5'::json;

-- 0개 이상의 요소로 구성된 배열 (요소들은 같은 타입일 필요는 없다)
SELECT '[1, 2, "foo", null]'::json;

-- 키와 값의 쌍으로 구성된 객체
-- 객체의 키는 항상 따옴표로 감싼 문자열이어야 한다
SELECT '{"bar": "baz", "balance": 7.77, "active": false}'::json;

-- 배열과 객체는 임의로 중첩될 수 있다
SELECT '{"foo": [true, "bar"], "tags": {"a": 1, "b": null}}'::json;
```

앞서 언급한 대로, JSON 값이 입력된 후 추가 처리 없이 출력될 때, `json`은 입력된 텍스트를 그대로 출력한다. 반면 `jsonb`는 공백과 같은 의미 없는 세부 사항을 보존하지 않는다. 예를 들어, 다음 차이점을 주목하라:

```
SELECT '{"bar": "baz", "balance": 7.77, "active":false}'::json;
                      json
-------------------------------------------------
 {"bar": "baz", "balance": 7.77, "active":false}
(1 row)

SELECT '{"bar": "baz", "balance": 7.77, "active":false}'::jsonb;
                      jsonb
--------------------------------------------------
 {"bar": "baz", "active": false, "balance": 7.77}
(1 row)
```

한 가지 주목할 만한 의미 없는 세부 사항은 `jsonb`에서 숫자가 기본 `numeric` 타입의 동작에 따라 출력된다는 점이다. 실제로 이는 `E` 표기법으로 입력된 숫자가 해당 표기법 없이 출력됨을 의미한다. 예를 들어:

```
SELECT '{"reading": 1.230e-5}'::json, '{"reading": 1.230e-5}'::jsonb;
         json          |          jsonb
-----------------------+-------------------------
 {"reading": 1.230e-5} | {"reading": 0.00001230}
(1 row)
```

그러나 `jsonb`는 소수점 이하의 의미 없는 0을 보존한다. 이 예제에서 볼 수 있듯이, 이러한 0은 동등성 검사와 같은 목적에서는 의미가 없지만, 여전히 출력에 포함된다.

JSON 값을 구성하고 처리하기 위해 사용할 수 있는 내장 함수와 연산자 목록은 [9.16절](https://www.postgresql.org/docs/current/functions-json.html "9.16. JSON 함수와 연산자")을 참조하라.


### 8.14.2. JSON 문서 설계

데이터를 JSON으로 표현하는 것은 전통적인 관계형 데이터 모델보다 훨씬 더 유연할 수 있다. 이는 요구사항이 유동적인 환경에서 특히 매력적이다. 두 접근 방식은 동일한 애플리케이션 내에서 공존하며 서로를 보완할 수 있다. 그러나 최대한의 유연성이 필요한 애플리케이션에서도 JSON 문서가 어느 정도 고정된 구조를 가지는 것이 권장된다. 이 구조는 일반적으로 강제되지는 않지만(일부 비즈니스 규칙을 선언적으로 강제하는 것은 가능), 예측 가능한 구조를 가지면 테이블 내의 "문서"(데이터) 집합을 효과적으로 요약하는 쿼리를 작성하기가 더 쉬워진다.

JSON 데이터는 테이블에 저장될 때 다른 데이터 타입과 동일한 동시성 제어 고려사항을 따른다. 큰 문서를 저장하는 것이 가능하지만, 업데이트 시 전체 행에 대한 행 수준 잠금이 발생한다는 점을 명심해야 한다. 업데이트 트랜잭션 간의 잠금 경합을 줄이기 위해 JSON 문서를 관리 가능한 크기로 제한하는 것을 고려해보자. 이상적으로, JSON 문서는 비즈니스 규칙에 따라 더 이상 분할할 수 없는 원자적 데이터를 표현해야 한다. 이는 독립적으로 수정될 수 있는 더 작은 데이터로 나눌 수 없는 데이터를 의미한다.


### 8.14.3. `jsonb` 포함 및 존재 여부 검사

`jsonb` 타입에서 *포함 여부*를 테스트하는 것은 중요한 기능이다. `json` 타입에는 이와 동일한 기능이 없다. 포함 여부 테스트는 하나의 `jsonb` 문서가 다른 `jsonb` 문서를 포함하고 있는지 확인한다. 아래 예제들은 특별히 언급된 경우를 제외하고 모두 true를 반환한다:

```
-- 단순 스칼라/기본 값은 동일한 값만 포함한다:
SELECT '"foo"'::jsonb @> '"foo"'::jsonb;

-- 오른쪽 배열이 왼쪽 배열에 포함된다:
SELECT '[1, 2, 3]'::jsonb @> '[1, 3]'::jsonb;

-- 배열 요소의 순서는 중요하지 않으므로 이 경우도 true이다:
SELECT '[1, 2, 3]'::jsonb @> '[3, 1]'::jsonb;

-- 배열 요소의 중복도 중요하지 않다:
SELECT '[1, 2, 3]'::jsonb @> '[1, 2, 2]'::jsonb;

-- 오른쪽 객체의 단일 키-값 쌍이 왼쪽 객체에 포함된다:
SELECT '{"product": "PostgreSQL", "version": 9.4, "jsonb": true}'::jsonb @> '{"version": 9.4}'::jsonb;

-- 오른쪽 배열이 왼쪽 배열에 포함되지 않는다. 비슷한 배열이 중첩되어 있더라도 포함되지 않는다:
SELECT '[1, 2, [1, 3]]'::jsonb @> '[1, 3]'::jsonb;  -- false 반환

-- 하지만 중첩된 배열은 포함된다:
SELECT '[1, 2, [1, 3]]'::jsonb @> '[[1, 3]]'::jsonb;

-- 마찬가지로, 이 경우 포함 여부는 확인되지 않는다:
SELECT '{"foo": {"bar": "baz"}}'::jsonb @> '{"bar": "baz"}'::jsonb;  -- false 반환

-- 최상위 키와 빈 객체는 포함된다:
SELECT '{"foo": {"bar": "baz"}}'::jsonb @> '{"foo": {}}'::jsonb;
```

일반적인 원칙은 포함되는 객체가 포함하는 객체의 구조와 데이터 내용과 일치해야 한다는 것이다. 이때 포함하는 객체에서 일치하지 않는 배열 요소나 객체 키/값 쌍을 제거할 수 있다. 하지만 배열 요소의 순서는 포함 여부 검사에서 중요하지 않으며, 중복된 배열 요소는 사실상 한 번만 고려된다.

구조가 일치해야 한다는 일반 원칙의 특별한 예외로, 배열은 기본 값을 포함할 수 있다:

```
-- 이 배열은 기본 문자열 값을 포함한다:
SELECT '["foo", "bar"]'::jsonb @> '"bar"'::jsonb;

-- 이 예외는 상호적이지 않다. 포함되지 않는다고 보고된다:
SELECT '"bar"'::jsonb @> '["bar"]'::jsonb;  -- false 반환
```

`jsonb`에는 *존재 여부* 연산자도 있다. 이 연산자는 포함 여부 테스트의 변형으로, 주어진 문자열(`text` 값)이 `jsonb` 값의 최상위 레벨에서 객체 키나 배열 요소로 존재하는지 확인한다. 아래 예제들은 특별히 언급된 경우를 제외하고 모두 true를 반환한다:

```
-- 문자열이 배열 요소로 존재한다:
SELECT '["foo", "bar", "baz"]'::jsonb ? 'bar';

-- 문자열이 객체 키로 존재한다:
SELECT '{"foo": "bar"}'::jsonb ? 'foo';

-- 객체 값은 고려되지 않는다:
SELECT '{"foo": "bar"}'::jsonb ? 'bar';  -- false 반환

-- 포함 여부와 마찬가지로, 존재 여부는 최상위 레벨에서 일치해야 한다:
SELECT '{"foo": {"bar": "baz"}}'::jsonb ? 'bar'; -- false 반환

-- 문자열이 기본 JSON 문자열과 일치하면 존재한다고 간주된다:
SELECT '"foo"'::jsonb ? 'foo';
```

많은 키나 요소가 관련된 경우, 포함 여부나 존재 여부를 테스트할 때 배열보다 JSON 객체가 더 적합하다. 배열과 달리 JSON 객체는 내부적으로 검색에 최적화되어 있어 선형 검색이 필요하지 않기 때문이다.


### 팁

JSON 포함 관계는 중첩 구조이기 때문에, 적절한 쿼리를 사용하면 하위 객체를 명시적으로 선택하지 않아도 된다. 예를 들어, `doc` 컬럼에 최상위 레벨 객체가 있고, 대부분의 객체가 `tags` 필드를 포함하며, 이 필드는 하위 객체의 배열을 담고 있다고 가정해 보자. 다음 쿼리는 `tags` 배열 내에서 `"term":"paris"`와 `"term":"food"`를 모두 포함하는 하위 객체가 있는 항목을 찾는다. 이때, `tags` 배열 외부에 있는 동일한 키는 무시한다:

```
SELECT doc->'site_name' FROM websites
  WHERE doc @> '{"tags":[{"term":"paris"}, {"term":"food"}]}';
```

동일한 결과를 얻기 위해 다음과 같은 쿼리를 사용할 수도 있다:

```
SELECT doc->'site_name' FROM websites
  WHERE doc->'tags' @> '[{"term":"paris"}, {"term":"food"}]';
```

하지만 이 방법은 덜 유연하고, 종종 효율성도 떨어진다.

반면, JSON 존재 연산자는 중첩되지 않는다. 이 연산자는 JSON 값의 최상위 레벨에서만 지정된 키나 배열 요소를 찾는다.

다양한 포함 및 존재 연산자와 기타 모든 JSON 연산자 및 함수는 [9.16절](https://www.postgresql.org/docs/current/functions-json.html "9.16. JSON Functions and Operators")에 문서화되어 있다.


### 8.14.4. `jsonb` 인덱싱

GIN 인덱스는 대량의 `jsonb` 문서(데이터) 내에서 키 또는 키/값 쌍을 효율적으로 검색하는 데 사용할 수 있다. 두 가지 GIN "연산자 클래스"가 제공되며, 각각 다른 성능과 유연성을 제공한다.

`jsonb`의 기본 GIN 연산자 클래스는 키 존재 연산자 `?`, `?|`, `?&`, 포함 연산자 `@>`, 그리고 `jsonpath` 매칭 연산자 `@?`와 `@@`를 지원한다. (이 연산자들이 구현하는 의미에 대한 자세한 내용은 [표 9.46](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-JSONB-OP-TABLE "표 9.46. 추가 jsonb 연산자")을 참조한다.) 이 연산자 클래스를 사용해 인덱스를 생성하는 예제는 다음과 같다:

```
CREATE INDEX idxgin ON api USING GIN (jdoc);
```

기본이 아닌 GIN 연산자 클래스인 `jsonb_path_ops`는 키 존재 연산자를 지원하지 않지만, `@>`, `@?`, `@@`는 지원한다. 이 연산자 클래스를 사용해 인덱스를 생성하는 예제는 다음과 같다:

```
CREATE INDEX idxginp ON api USING GIN (jdoc jsonb_path_ops);
```

다음은 제3자 웹 서비스에서 가져온 JSON 문서를 저장하는 테이블의 예제이다. 문서의 스키마 정의는 다음과 같다:

```
{
    "guid": "9c36adc1-7fb5-4d5b-83b4-90356a46061a",
    "name": "Angela Barton",
    "is_active": true,
    "company": "Magnafone",
    "address": "178 Howard Place, Gulf, Washington, 702",
    "registered": "2009-11-07T08:53:22 +08:00",
    "latitude": 19.793713,
    "longitude": 86.513373,
    "tags": [
        "enim",
        "aliquip",
        "qui"
    ]
}
```

이 문서들을 `api`라는 테이블의 `jdoc`이라는 `jsonb` 컬럼에 저장한다. 이 컬럼에 GIN 인덱스를 생성하면 다음과 같은 쿼리에서 인덱스를 활용할 수 있다:

```
-- "company" 키의 값이 "Magnafone"인 문서를 찾는다
SELECT jdoc->'guid', jdoc->'name' FROM api WHERE jdoc @> '{"company": "Magnafone"}';
```

그러나 다음과 같은 쿼리에서는 인덱스를 사용할 수 없다. 연산자 `?`는 인덱싱 가능하지만, 인덱싱된 컬럼 `jdoc`에 직접 적용되지 않기 때문이다:

```
-- "tags" 키가 "qui"라는 키 또는 배열 요소를 포함하는 문서를 찾는다
SELECT jdoc->'guid', jdoc->'name' FROM api WHERE jdoc -> 'tags' ? 'qui';
```

그러나 적절한 표현식 인덱스를 사용하면 위 쿼리도 인덱스를 활용할 수 있다. `"tags"` 키 내의 특정 항목을 자주 검색한다면 다음과 같은 인덱스를 정의하는 것이 유용할 수 있다:

```
CREATE INDEX idxgintags ON api USING GIN ((jdoc -> 'tags'));
```

이제 `WHERE` 절 `jdoc -> 'tags' ? 'qui'`는 인덱싱된 표현식 `jdoc -> 'tags'`에 인덱싱 가능한 연산자 `?`를 적용한 것으로 인식된다. (표현식 인덱스에 대한 자세한 내용은 [11.7절](https://www.postgresql.org/docs/current/indexes-expressional.html "11.7. 표현식 인덱스")을 참조한다.)

또 다른 쿼리 방식은 포함 관계를 활용하는 것이다. 예를 들어:

```
-- "tags" 키가 "qui"라는 배열 요소를 포함하는 문서를 찾는다
SELECT jdoc->'guid', jdoc->'name' FROM api WHERE jdoc @> '{"tags": ["qui"]}';
```

`jdoc` 컬럼에 단순한 GIN 인덱스를 생성하면 이 쿼리를 지원할 수 있다. 그러나 이러한 인덱스는 `jdoc` 컬럼의 모든 키와 값을 저장하는 반면, 앞선 예제의 표현식 인덱스는 `tags` 키 아래의 데이터만 저장한다. 단순 인덱스 방식은 어떤 키에 대한 쿼리도 지원하므로 훨씬 유연하지만, 특정 표현식 인덱스는 단순 인덱스보다 작고 검색 속도가 빠를 가능성이 높다.

GIN 인덱스는 `jsonpath` 매칭을 수행하는 `@?`와 `@@` 연산자도 지원한다. 예제는 다음과 같다:

```
SELECT jdoc->'guid', jdoc->'name' FROM api WHERE jdoc @? '$.tags[*] ? (@ == "qui")';
```
```
SELECT jdoc->'guid', jdoc->'name' FROM api WHERE jdoc @@ '$.tags[*] == "qui"';
```

이 연산자들에 대해 GIN 인덱스는 `jsonpath` 패턴에서 ``*`accessors_chain`* == *`constant`*`` 형태의 절을 추출하고, 이 절에서 언급된 키와 값을 기반으로 인덱스 검색을 수행한다. 액세서 체인은 ``.*`key`*``, `[*]`, ``[*`index`*]`` 액세서를 포함할 수 있다. `jsonb_ops` 연산자 클래스는 `.*`와 `.**` 액세서도 지원하지만, `jsonb_path_ops` 연산자 클래스는 지원하지 않는다.

`jsonb_path_ops` 연산자 클래스는 `@>`, `@?`, `@@` 연산자를 사용하는 쿼리만 지원하지만, 기본 연산자 클래스인 `jsonb_ops`보다 뛰어난 성능을 제공한다. `jsonb_path_ops` 인덱스는 동일한 데이터에 대한 `jsonb_ops` 인덱스보다 훨씬 작으며, 특히 데이터에서 자주 나타나는 키가 포함된 쿼리의 경우 검색 명확도가 더 높다. 따라서 검색 작업은 일반적으로 기본 연산자 클래스보다 더 나은 성능을 보인다.

`jsonb_ops`와 `jsonb_path_ops` GIN 인덱스의 기술적 차이는 전자는 데이터의 각 키와 값에 대해 독립적인 인덱스 항목을 생성하는 반면, 후자는 데이터의 각 값에 대해서만 인덱스 항목을 생성한다는 점이다. [[7]](https://www.postgresql.org/#ftn.id-1.5.7.22.18.9.3) 기본적으로 각 `jsonb_path_ops` 인덱스 항목은 값과 그 값으로 이어지는 키(들)의 해시이다. 예를 들어 `{"foo": {"bar": "baz"}}`를 인덱싱할 때, `foo`, `bar`, `baz` 모두를 해시 값에 포함하는 단일 인덱스 항목이 생성된다. 따라서 이 구조를 찾는 포함 쿼리는 매우 구체적인 인덱스 검색을 수행할 수 있다. 그러나 `foo`가 키로 존재하는지 여부를 확인할 방법은 전혀 없다. 반면 `jsonb_ops` 인덱스는 `foo`, `bar`, `baz`를 각각 나타내는 세 개의 인덱스 항목을 생성한다. 따라서 포함 쿼리를 수행할 때는 이 세 항목을 모두 포함하는 행을 찾는다. GIN 인덱스는 이러한 AND 검색을 상당히 효율적으로 수행할 수 있지만, 특히 세 인덱스 항목 중 하나라도 포함하는 행이 매우 많은 경우 동등한 `jsonb_path_ops` 검색보다 명확도가 떨어지고 속도가 느릴 수 있다.

`jsonb_path_ops` 방식의 단점은 `{"a": {}}`와 같이 값을 포함하지 않는 JSON 구조에 대한 인덱스 항목을 생성하지 않는다는 점이다. 이러한 구조를 포함하는 문서를 검색하려면 전체 인덱스 스캔이 필요하며, 이는 상당히 느리다. 따라서 `jsonb_path_ops`는 이러한 검색을 자주 수행하는 애플리케이션에는 적합하지 않다.

`jsonb`는 `btree`와 `hash` 인덱스도 지원한다. 이들은 일반적으로 완전한 JSON 문서의 동등성을 확인하는 것이 중요한 경우에만 유용하다. `jsonb` 데이터에 대한 `btree` 정렬은 흔히 큰 관심사가 아니지만, 완전성을 위해 다음과 같다:

```
Object
```

단, (역사적인 이유로) 빈 최상위 배열은 *`null`*보다 작게 정렬된다. 동일한 수의 쌍을 가진 객체는 다음 순서로 비교된다:

```
key-1
```

객체 키는 저장 순서로 비교된다. 특히 짧은 키가 긴 키보다 먼저 저장되므로 다음과 같은 직관적이지 않은 결과가 발생할 수 있다:

```
{ "aa": 1, "c": 1} > {"b": 1, "d": 1}
```

마찬가지로 동일한 수의 요소를 가진 배열은 다음 순서로 비교된다:

```
element-1
```

기본 JSON 값은 PostgreSQL 데이터 타입에 대한 비교 규칙을 사용해 비교된다. 문자열은 기본 데이터베이스 정렬 규칙을 사용해 비교된다.


### 8.14.5. `jsonb` 서브스크립팅

`jsonb` 데이터 타입은 배열 스타일의 서브스크립팅 표현식을 지원하여 요소를 추출하고 수정할 수 있다. 중첩된 값은 `jsonb_set` 함수의 `path` 인자와 동일한 규칙을 따라 서브스크립팅 표현식을 연결하여 지정할 수 있다. `jsonb` 값이 배열인 경우, 숫자 인덱스는 0부터 시작하며, 음수 정수는 배열의 마지막 요소부터 역순으로 계산된다. 슬라이스 표현식은 지원되지 않는다. 서브스크립팅 표현식의 결과는 항상 `jsonb` 데이터 타입이다.

`UPDATE` 문은 `SET` 절에서 서브스크립팅을 사용해 `jsonb` 값을 수정할 수 있다. 서브스크립트 경로는 모든 영향을 받는 값에 대해 순회 가능해야 한다. 예를 들어, `val['a']['b']['c']` 경로는 `val`, `val['a']`, `val['a']['b']`가 모두 객체인 경우에만 `c`까지 순회할 수 있다. 만약 `val['a']` 또는 `val['a']['b']`가 정의되지 않았다면, 빈 객체로 생성되고 필요에 따라 채워진다. 그러나 `val` 자체 또는 중간 값 중 하나가 문자열, 숫자, `jsonb` `null`과 같은 비객체로 정의된 경우, 순회가 불가능하므로 오류가 발생하고 트랜잭션이 중단된다.

서브스크립팅 구문 예제:

```sql
-- 키를 통해 객체 값 추출
SELECT ('{"a": 1}'::jsonb)['a'];

-- 중첩된 객체 값 추출
SELECT ('{"a": {"b": {"c": 1}}}'::jsonb)['a']['b']['c'];

-- 인덱스를 통해 배열 요소 추출
SELECT ('[1, "2", null]'::jsonb)[1];

-- 키를 통해 객체 값 업데이트. '1' 주의: 할당된 값도 jsonb 타입이어야 함
UPDATE table_name SET jsonb_field['key'] = '1';

-- jsonb_field['a']['b']가 객체가 아닌 경우 오류 발생. 예: {"a": 1}은 'a' 키의 값이 숫자임
UPDATE table_name SET jsonb_field['a']['b']['c'] = '1';

-- WHERE 절에서 서브스크립팅을 사용해 레코드 필터링. 서브스크립팅 결과는 jsonb이므로 비교 대상 값도 jsonb여야 함
SELECT * FROM table_name WHERE jsonb_field['key'] = '"value"';
```

서브스크립팅을 통한 `jsonb` 할당은 `jsonb_set`와는 몇 가지 예외 케이스를 다르게 처리한다. 소스 `jsonb` 값이 `NULL`인 경우, 서브스크립팅 키에 의해 암시된 타입(객체 또는 배열)의 빈 JSON 값인 것처럼 진행된다:

```sql
-- jsonb_field가 NULL이었다면, 이제 {"a": 1}이 됨
UPDATE table_name SET jsonb_field['a'] = '1';

-- jsonb_field가 NULL이었다면, 이제 [1]이 됨
UPDATE table_name SET jsonb_field[0] = '1';
```

배열에 지정된 인덱스가 요소 수보다 큰 경우, 인덱스에 도달할 때까지 `NULL` 요소가 추가되고 값이 설정된다.

```sql
-- jsonb_field가 []이었다면, 이제 [null, null, 2]가 됨
-- jsonb_field가 [0]이었다면, 이제 [0, null, 2]가 됨
UPDATE table_name SET jsonb_field[2] = '2';
```

`jsonb` 값은 마지막으로 순회할 요소가 객체 또는 배열인 한 존재하지 않는 서브스크립트 경로에 대한 할당을 허용한다(경로의 마지막 서브스크립트로 표시된 요소는 순회되지 않으며 어떤 값이든 될 수 있음). 중첩된 배열 및 객체 구조가 생성되며, 필요한 경우 `null`로 채워진다.

```sql
-- jsonb_field가 {}이었다면, 이제 {"a": [{"b": 1}]}이 됨
UPDATE table_name SET jsonb_field['a'][0]['b'] = '1';

-- jsonb_field가 []이었다면, 이제 [null, {"a": 1}]이 됨
UPDATE table_name SET jsonb_field[1]['a'] = '1';
```


### 8.14.6. JSONB 타입 변환

`jsonb` 타입을 다양한 절차적 언어로 변환하는 기능을 제공하는 추가 확장 기능이 있다.

PL/Perl용 확장은 `jsonb_plperl`과 `jsonb_plperlu`라고 한다. 이 확장을 사용하면 `jsonb` 값은 적절하게 Perl 배열, 해시, 스칼라로 매핑된다.

PL/Python용 확장은 `jsonb_plpython3u`라고 한다. 이 확장을 사용하면 `jsonb` 값은 적절하게 Python 딕셔너리, 리스트, 스칼라로 매핑된다.

이 확장들 중 `jsonb_plperl`은 "신뢰할 수 있는" 것으로 간주된다. 즉, 현재 데이터베이스에 `CREATE` 권한이 있는 일반 사용자도 설치할 수 있다. 나머지 확장은 슈퍼유저 권한이 필요하다.


### 8.14.7. jsonpath 타입

`jsonpath` 타입은 PostgreSQL에서 SQL/JSON 경로 언어를 지원하여 JSON 데이터를 효율적으로 쿼리할 수 있게 한다. 이 타입은 SQL/JSON 경로 엔진이 JSON 데이터에서 추출할 항목을 지정하는 파싱된 SQL/JSON 경로 표현식을 바이너리 형태로 제공하며, 이를 통해 SQL/JSON 쿼리 함수로 추가 처리를 수행한다.

SQL/JSON 경로 조건식과 연산자의 의미는 일반적으로 SQL을 따른다. 동시에 JSON 데이터를 자연스럽게 다루기 위해 SQL/JSON 경로 구문은 일부 JavaScript 규칙을 사용한다:

*   점(`.`)은 멤버 접근에 사용된다.
*   대괄호(`[]`)는 배열 접근에 사용된다.
*   SQL/JSON 배열은 0부터 시작하며, 일반 SQL 배열과 달리 1부터 시작하지 않는다.

SQL/JSON 경로 표현식의 숫자 리터럴은 JavaScript 규칙을 따르며, 이는 SQL과 JSON과 약간 다른 세부 사항을 가진다. 예를 들어, SQL/JSON 경로는 `.1`과 `1.`을 허용하지만, JSON에서는 유효하지 않다. 비십진수 정수 리터럴과 언더스코어 구분자를 지원하며, 예를 들어 `1_000_000`, `0x1EEE_FFFF`, `0o273`, `0b100101`과 같은 형태를 사용할 수 있다. SQL/JSON 경로(및 JavaScript)에서는 진수 접두사 바로 뒤에 언더스코어 구분자를 사용할 수 없다.

SQL/JSON 경로 표현식은 일반적으로 SQL 쿼리에서 SQL 문자열 리터럴로 작성되므로 작은따옴표로 감싸야 하며, 값 내부에서 작은따옴표를 사용하려면 두 번 입력해야 한다(자세한 내용은 [4.1.2.1절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS) 참조). 일부 경로 표현식은 내부에 문자열 리터럴을 요구한다. 이러한 내장 문자열 리터럴은 JavaScript/ECMAScript 규칙을 따르며, 큰따옴표로 감싸야 하고 백슬래시 이스케이프를 사용하여 입력하기 어려운 문자를 표현할 수 있다. 특히, 내장 문자열 리터럴 내에서 큰따옴표를 쓰려면 `"`를 사용하고, 백슬래시 자체를 쓰려면 `\`를 사용한다. 기타 특수 백슬래시 시퀀스는 JavaScript 문자열에서 인식되는 것과 동일하며, `b`, `f`, `n`, `r`, `t`, `v`는 다양한 ASCII 제어 문자를 나타내고, ``x*`NN`*``는 두 자리 16진수로 표현된 문자 코드, ``u*`NNNN`*``는 4자리 16진수 코드 포인트로 식별된 유니코드 문자, ``u{*`N...`*}``는 1~6자리 16진수로 표현된 유니코드 문자 코드 포인트를 나타낸다.

경로 표현식은 다음과 같은 경로 요소의 시퀀스로 구성된다:

*   JSON 기본 타입의 경로 리터럴: 유니코드 텍스트, 숫자, true, false, null.
*   [표 8.24](https://www.postgresql.org/docs/current/datatype-json.html#TYPE-JSONPATH-VARIABLES)에 나열된 경로 변수.
*   [표 8.25](https://www.postgresql.org/docs/current/datatype-json.html#TYPE-JSONPATH-ACCESSORS)에 나열된 접근자 연산자.
*   [9.16.2.3절](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-PATH-OPERATORS)에 나열된 `jsonpath` 연산자 및 메서드.
*   필터 표현식을 제공하거나 경로 평가 순서를 정의하기 위해 사용할 수 있는 괄호.

SQL/JSON 쿼리 함수와 함께 `jsonpath` 표현식을 사용하는 방법에 대한 자세한 내용은 [9.16.2절](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-PATH)을 참조한다.

**표 8.24. `jsonpath` 변수**

| 변수 | 설명 |
| --- | --- |
| $ | 쿼리 중인 JSON 값을 나타내는 변수(컨텍스트 항목). |
| $varname | 이름이 지정된 변수. 이 값은 여러 JSON 처리 함수의 vars 매개변수로 설정할 수 있으며, 자세한 내용은 표 9.49를 참조한다. |
| @ | 필터 표현식에서 경로 평가 결과를 나타내는 변수. |

  

**표 8.25. `jsonpath` 접근자**

| 접근자 연산자 | 설명 |
| --- | --- |
| .key."$varname" | 지정된 키를 가진 객체 멤버를 반환하는 멤버 접근자. 키 이름이 $로 시작하는 명명된 변수와 일치하거나 JavaScript 식별자 규칙을 충족하지 않으면 문자열 리터럴로 만들기 위해 큰따옴표로 감싸야 한다. |
| .* | 현재 객체의 최상위 수준에 있는 모든 멤버의 값을 반환하는 와일드카드 멤버 접근자. |
| .** | 현재 객체의 JSON 계층 구조의 모든 수준을 처리하고 중첩 수준에 관계없이 모든 멤버 값을 반환하는 재귀적 와일드카드 멤버 접근자. 이는 SQL/JSON 표준의 PostgreSQL 확장이다. |
| .**{level}.**{start_level to end_level} | .**와 유사하지만 JSON 계층 구조의 지정된 수준만 선택한다. 중첩 수준은 정수로 지정된다. 수준 0은 현재 객체에 해당한다. 가장 낮은 중첩 수준에 접근하려면 last 키워드를 사용할 수 있다. 이는 SQL/JSON 표준의 PostgreSQL 확장이다. |
| [subscript, ...] | 배열 요소 접근자. subscript는 두 가지 형태로 제공될 수 있다: 인덱스 또는 start_index to end_index. 첫 번째 형태는 인덱스로 단일 배열 요소를 반환한다. 두 번째 형태는 제공된 start_index와 end_index에 해당하는 요소를 포함하는 배열 슬라이스를 반환한다.지정된 인덱스는 정수일 수도 있고, 단일 숫자 값을 반환하는 표현식일 수도 있으며, 이는 자동으로 정수로 캐스팅된다. 인덱스 0은 첫 번째 배열 요소에 해당한다. 마지막 배열 요소를 나타내기 위해 last 키워드를 사용할 수도 있으며, 이는 길이를 알 수 없는 배열을 처리할 때 유용하다. |
| [*] | 모든 배열 요소를 반환하는 와일드카드 배열 요소 접근자. |




## 8.15. 배열

PostgreSQL은 테이블의 컬럼을 가변 길이 다차원 배열로 정의할 수 있게 한다. 내장 타입이나 사용자 정의 기본 타입, 열거형 타입, 복합 타입, 범위 타입, 도메인 타입 등 다양한 타입의 배열을 생성할 수 있다.


### 8.15.1. 배열 타입 선언

배열 타입의 사용법을 설명하기 위해 다음과 같은 테이블을 생성한다:

```
CREATE TABLE sal_emp (
    name            text,
    pay_by_quarter  integer[],
    schedule        text[][]
);
```

보여지는 것처럼, 배열 데이터 타입은 배열 요소의 데이터 타입 이름에 대괄호(`[]`)를 추가하여 명명한다. 위의 명령어는 `sal_emp`라는 이름의 테이블을 생성하며, `text` 타입의 컬럼(`name`), 사원의 분기별 급여를 나타내는 `integer` 타입의 1차원 배열(`pay_by_quarter`), 그리고 사원의 주간 스케줄을 나타내는 `text` 타입의 2차원 배열(`schedule`)을 포함한다.

`CREATE TABLE` 구문은 배열의 정확한 크기를 지정할 수 있도록 허용한다. 예를 들면:

```
CREATE TABLE tictactoe (
    squares   integer[3][3]
);
```

그러나 현재 구현에서는 제공된 배열 크기 제한을 무시한다. 즉, 크기가 지정되지 않은 배열과 동일하게 동작한다.

현재 구현은 선언된 차원 수도 강제하지 않는다. 특정 요소 타입의 배열은 크기나 차원 수에 관계없이 동일한 타입으로 간주된다. 따라서 `CREATE TABLE`에서 배열 크기나 차원 수를 선언하는 것은 단순히 문서화 역할을 할 뿐이며, 런타임 동작에는 영향을 미치지 않는다.

1차원 배열에 대해 SQL 표준을 준수하는 `ARRAY` 키워드를 사용하는 대체 구문도 사용할 수 있다. `pay_by_quarter`는 다음과 같이 정의될 수 있다:

```
pay_by_quarter  integer ARRAY[4],
```

또는 배열 크기를 지정하지 않을 경우:

```
pay_by_quarter  integer ARRAY,
```

그러나 이전과 마찬가지로 PostgreSQL은 어떤 경우에도 크기 제한을 강제하지 않는다.


### 8.15.2. 배열 값 입력

배열 값을 리터럴 상수로 작성하려면, 각 요소 값을 중괄호로 감싸고 쉼표로 구분한다. (C 언어를 알고 있다면, 구조체 초기화와 비슷한 구문이다.) 요소 값에 쉼표나 중괄호가 포함된 경우, 반드시 큰따옴표로 감싸야 한다. (자세한 내용은 아래에서 설명한다.) 따라서 배열 상수의 일반적인 형식은 다음과 같다:

```
val1
```

여기서 `delim`은 해당 타입의 구분자 문자로, `pg_type` 항목에 기록되어 있다. PostgreSQL 배포판에서 제공하는 표준 데이터 타입 중, `box` 타입은 세미콜론(`;`)을 사용하고, 나머지는 모두 쉼표(`,`)를 사용한다. 각 `val`은 배열 요소 타입의 상수이거나 하위 배열이다. 배열 상수의 예는 다음과 같다:

```
'{{1,2,3},{4,5,6},{7,8,9}}'
```

이 상수는 정수로 이루어진 3x3 크기의 2차원 배열로, 세 개의 하위 배열로 구성된다.

배열 상수의 요소를 NULL로 설정하려면, 요소 값에 `NULL`을 작성한다. (`NULL`의 대소문자 변형 모두 가능하다.) 만약 실제 문자열 값 "NULL"을 사용하려면, 큰따옴표로 감싸야 한다.

(이러한 배열 상수는 사실 [4.1.2.7절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS-GENERIC "4.1.2.7. 기타 타입의 상수")에서 다룬 일반적인 타입 상수의 특수한 경우다. 상수는 초기에 문자열로 처리되어 배열 입력 변환 루틴으로 전달된다. 명시적 타입 지정이 필요할 수 있다.)

이제 몇 가지 `INSERT` 문을 살펴보자:

```
INSERT INTO sal_emp
    VALUES ('Bill',
    '{10000, 10000, 10000, 10000}',
    '{{"meeting", "lunch"}, {"training", "presentation"}}');

INSERT INTO sal_emp
    VALUES ('Carol',
    '{20000, 25000, 25000, 25000}',
    '{{"breakfast", "consulting"}, {"meeting", "lunch"}}');
```

앞의 두 `INSERT` 문의 결과는 다음과 같다:

```
SELECT * FROM sal_emp;
 name  |      pay_by_quarter       |                 schedule
-------+---------------------------+-------------------------------------------
 Bill  | {10000,10000,10000,10000} | {{meeting,lunch},{training,presentation}}
 Carol | {20000,25000,25000,25000} | {{breakfast,consulting},{meeting,lunch}}
(2 rows)
```

다차원 배열은 각 차원의 크기가 일치해야 한다. 불일치 시 오류가 발생한다. 예를 들어:

```
INSERT INTO sal_emp
    VALUES ('Bill',
    '{10000, 10000, 10000, 10000}',
    '{{"meeting", "lunch"}, {"meeting"}}');
ERROR:  malformed array literal: "{{"meeting", "lunch"}, {"meeting"}}"
DETAIL:  다차원 배열은 하위 배열의 크기가 일치해야 합니다.
```

`ARRAY` 생성자 구문도 사용할 수 있다:

```
INSERT INTO sal_emp
    VALUES ('Bill',
    ARRAY[10000, 10000, 10000, 10000],
    ARRAY[['meeting', 'lunch'], ['training', 'presentation']]);

INSERT INTO sal_emp
    VALUES ('Carol',
    ARRAY[20000, 25000, 25000, 25000],
    ARRAY[['breakfast', 'consulting'], ['meeting', 'lunch']]);
```

배열 요소는 일반적인 SQL 상수나 표현식임에 유의한다. 예를 들어, 문자열 리터럴은 배열 리터럴과 달리 작은따옴표로 감싼다. `ARRAY` 생성자 구문에 대한 자세한 내용은 [4.2.12절](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-ARRAY-CONSTRUCTORS "4.2.12. 배열 생성자")에서 다룬다.


### 8.15.3. 배열 접근

이제 테이블에 대해 몇 가지 쿼리를 실행할 수 있다. 먼저 배열의 단일 요소에 접근하는 방법을 살펴본다. 다음 쿼리는 두 번째 분기 동안 급여가 변동된 직원의 이름을 조회한다:

```
SELECT name FROM sal_emp WHERE pay_by_quarter[1] <> pay_by_quarter[2];

 name
-------
 Carol
(1 row)
```

배열의 첨자는 대괄호 안에 작성한다. 기본적으로 PostgreSQL은 배열에 대해 1부터 시작하는 번호 매기기 규칙을 사용한다. 즉, `n`개의 요소를 가진 배열은 `array[1]`로 시작해 ``array[`n`]``로 끝난다.

다음 쿼리는 모든 직원의 세 번째 분기 급여를 조회한다:

```
SELECT pay_by_quarter[3] FROM sal_emp;

 pay_by_quarter
----------------
          10000
          25000
(2 rows)
```

또한 배열의 임의의 직사각형 부분 배열이나 하위 배열에 접근할 수도 있다. 배열 슬라이스는 하나 이상의 배열 차원에 대해 `하한:상한`형식으로 표기한다. 예를 들어, 다음 쿼리는 Bill의 일정에서 첫 주의 처음 두 날에 대한 첫 번째 항목을 조회한다:

```
SELECT schedule[1:2][1:1] FROM sal_emp WHERE name = 'Bill';

        schedule
------------------------
 {{meeting},{training}}
(1 row)
```

어떤 차원이 슬라이스로 작성되면, 즉 콜론을 포함하면 모든 차원이 슬라이스로 처리된다. 단일 숫자만 있는 차원(콜론 없음)은 1부터 지정된 숫자까지로 처리된다. 예를 들어, `[2]`는 `[1:2]`로 처리되며, 다음 예제와 같다:

```
SELECT schedule[1:2][2] FROM sal_emp WHERE name = 'Bill';

                 schedule
-------------------------------------------
 {{meeting,lunch},{training,presentation}}
(1 row)
```

비슬라이스 경우와의 혼동을 피하기 위해 모든 차원에 슬라이스 구문을 사용하는 것이 좋다. 예를 들어, `[1:2][1:1]`을 사용하고 `[2][1:1]`은 피한다.

슬라이스 지정자에서 `하한` 및/또는 `상한`을 생략할 수 있다. 누락된 한계는 배열 첨자의 하한 또는 상한으로 대체된다. 예를 들어:

```
SELECT schedule[:2][2:] FROM sal_emp WHERE name = 'Bill';

        schedule
------------------------
 {{lunch},{presentation}}
(1 row)

SELECT schedule[:][1:1] FROM sal_emp WHERE name = 'Bill';

        schedule
------------------------
 {{meeting},{training}}
(1 row)
```

배열 첨자 표현식은 배열 자체 또는 첨자 표현식 중 하나가 null이면 null을 반환한다. 또한, 첨자가 배열 범위를 벗어나면 null이 반환된다(이 경우 오류가 발생하지 않음). 예를 들어, `schedule`이 현재 `[1:3][1:2]` 차원을 가지고 있다면 `schedule[3][3]`을 참조하면 NULL이 반환된다. 마찬가지로, 잘못된 수의 첨자를 가진 배열 참조는 오류 대신 null을 반환한다.

배열 슬라이스 표현식도 배열 자체 또는 첨자 표현식 중 하나가 null이면 null을 반환한다. 그러나 현재 배열 범위를 완전히 벗어난 배열 슬라이스를 선택하는 경우와 같은 다른 경우에는 슬라이스 표현식이 null 대신 빈(0차원) 배열을 반환한다(이는 비슬라이스 동작과 일치하지 않으며 역사적인 이유로 인해 이렇게 동작한다). 요청된 슬라이스가 배열 범위와 부분적으로 겹치면 오류 없이 겹치는 영역만 반환된다.

어떤 배열 값의 현재 차원은 `array_dims` 함수로 조회할 수 있다:

```
SELECT array_dims(schedule) FROM sal_emp WHERE name = 'Carol';

 array_dims
------------
 [1:2][1:2]
(1 row)
```

`array_dims`는 `text` 결과를 생성하며, 이는 사람이 읽기에는 편리하지만 프로그램에는 불편할 수 있다. 차원은 `array_upper`와 `array_lower`로도 조회할 수 있으며, 이 함수들은 각각 지정된 배열 차원의 상한과 하한을 반환한다:

```
SELECT array_upper(schedule, 1) FROM sal_emp WHERE name = 'Carol';

 array_upper
-------------
           2
(1 row)
```

`array_length`는 지정된 배열 차원의 길이를 반환한다:

```
SELECT array_length(schedule, 1) FROM sal_emp WHERE name = 'Carol';

 array_length
--------------
            2
(1 row)
```

`cardinality`는 모든 차원에 걸쳐 배열의 총 요소 수를 반환한다. 이는 `unnest` 호출이 반환할 행의 수와 동일하다:

```
SELECT cardinality(schedule) FROM sal_emp WHERE name = 'Carol';

 cardinality
-------------
           4
(1 row)
```


### 8.15.4. 배열 수정

배열 값을 완전히 교체할 수 있다:

```
UPDATE sal_emp SET pay_by_quarter = '{25000,25000,27000,27000}'
    WHERE name = 'Carol';
```

또는 `ARRAY` 표현식 문법을 사용할 수도 있다:

```
UPDATE sal_emp SET pay_by_quarter = ARRAY[25000,25000,27000,27000]
    WHERE name = 'Carol';
```

배열의 단일 요소를 업데이트할 수도 있다:

```
UPDATE sal_emp SET pay_by_quarter[4] = 15000
    WHERE name = 'Bill';
```

또는 배열의 일부를 슬라이스로 업데이트할 수 있다:

```
UPDATE sal_emp SET pay_by_quarter[1:2] = '{27000,27000}'
    WHERE name = 'Carol';
```

`lower-bound` 또는 `upper-bound`를 생략한 슬라이스 문법도 사용할 수 있지만, 이는 NULL이 아니거나 0차원이 아닌 배열 값을 업데이트할 때만 가능하다 (그렇지 않으면 대체할 기존의 첨자 제한이 없다).

저장된 배열 값은 아직 존재하지 않는 요소에 값을 할당함으로써 확장할 수 있다. 기존 요소와 새로 할당된 요소 사이의 위치는 NULL로 채워진다. 예를 들어, 배열 `myarray`가 현재 4개의 요소를 가지고 있다면, `myarray[6]`에 값을 할당하는 업데이트 후에는 6개의 요소를 가지게 되며, `myarray[5]`는 NULL을 포함하게 된다. 현재 이 방식의 확장은 1차원 배열에서만 허용되며, 다차원 배열에서는 허용되지 않는다.

첨자 할당을 통해 1부터 시작하지 않는 첨자를 사용하는 배열을 생성할 수 있다. 예를 들어, `myarray[-2:7]`에 값을 할당하면 첨자 값이 -2부터 7까지인 배열을 생성할 수 있다.

새로운 배열 값은 연결 연산자 `||`를 사용하여 생성할 수도 있다:

```
SELECT ARRAY[1,2] || ARRAY[3,4];
 ?column?
-----------
 {1,2,3,4}
(1 row)

SELECT ARRAY[5,6] || ARRAY[[1,2],[3,4]];
      ?column?
---------------------
 {{5,6},{1,2},{3,4}}
(1 row)
```

연결 연산자는 1차원 배열의 시작 또는 끝에 단일 요소를 추가할 수 있다. 또한 두 개의 `N`차원 배열 또는 `N`차원과 `N+1`차원 배열을 연결할 수도 있다.

1차원 배열의 시작 또는 끝에 단일 요소를 추가하면, 결과 배열은 피연산자 배열과 동일한 하한 첨자를 가진다. 예를 들어:

```
SELECT array_dims(1 || '[0:1]={2,3}'::int[]);
 array_dims
------------
 [0:2]
(1 row)

SELECT array_dims(ARRAY[1,2] || 3);
 array_dims
------------
 [1:3]
(1 row)
```

동일한 차원 수를 가진 두 배열을 연결하면, 결과는 왼쪽 피연산자의 외부 차원 하한 첨자를 유지한다. 결과 배열은 왼쪽 피연산자의 모든 요소 뒤에 오른쪽 피연산자의 모든 요소가 따라오는 배열이다. 예를 들어:

```
SELECT array_dims(ARRAY[1,2] || ARRAY[3,4,5]);
 array_dims
------------
 [1:5]
(1 row)

SELECT array_dims(ARRAY[[1,2],[3,4]] || ARRAY[[5,6],[7,8],[9,0]]);
 array_dims
------------
 [1:5][1:2]
(1 row)
```

`N`차원 배열을 `N+1`차원 배열의 시작 또는 끝에 추가하면, 결과는 위의 요소-배열 경우와 유사하다. 각 `N`차원 하위 배열은 `N+1`차원 배열의 외부 차원의 요소와 같다. 예를 들어:

```
SELECT array_dims(ARRAY[1,2] || ARRAY[[3,4],[5,6]]);
 array_dims
------------
 [1:3][1:2]
(1 row)
```

배열은 `array_prepend`, `array_append`, 또는 `array_cat` 함수를 사용하여 생성할 수도 있다. 처음 두 함수는 1차원 배열만 지원하지만, `array_cat`은 다차원 배열을 지원한다. 몇 가지 예제:

```
SELECT array_prepend(1, ARRAY[2,3]);
 array_prepend
---------------
 {1,2,3}
(1 row)

SELECT array_append(ARRAY[1,2], 3);
 array_append
--------------
 {1,2,3}
(1 row)

SELECT array_cat(ARRAY[1,2], ARRAY[3,4]);
 array_cat
-----------
 {1,2,3,4}
(1 row)

SELECT array_cat(ARRAY[[1,2],[3,4]], ARRAY[5,6]);
      array_cat
---------------------
 {{1,2},{3,4},{5,6}}
(1 row)

SELECT array_cat(ARRAY[5,6], ARRAY[[1,2],[3,4]]);
      array_cat
---------------------
 {{5,6},{1,2},{3,4}}
```

단순한 경우에는 위에서 설명한 연결 연산자를 직접 사용하는 것이 더 선호된다. 그러나 연결 연산자는 세 가지 경우 모두를 처리하도록 오버로드되어 있기 때문에, 모호함을 피하기 위해 함수 중 하나를 사용하는 것이 도움이 되는 상황이 있다. 예를 들어:

```
SELECT ARRAY[1, 2] || '{3, 4}';  -- 타입이 지정되지 않은 리터럴은 배열로 간주됨
 ?column?
-----------
 {1,2,3,4}

SELECT ARRAY[1, 2] || '7';                 -- 이 경우도 마찬가지
ERROR:  malformed array literal: "7"

SELECT ARRAY[1, 2] || NULL;                -- 타입이 지정되지 않은 NULL도 마찬가지
 ?column?
----------
 {1,2}
(1 row)

SELECT array_append(ARRAY[1, 2], NULL);    -- 이렇게 의도했을 수도 있음
 array_append
--------------
 {1,2,NULL}
```

위 예제에서 파서는 연결 연산자의 한쪽에 정수 배열을, 다른 쪽에 타입이 결정되지 않은 상수를 본다. 상수의 타입을 결정하기 위해 사용하는 휴리스틱은 연산자의 다른 입력과 동일한 타입으로 간주하는 것이다. 이 경우 정수 배열로 간주한다. 따라서 연결 연산자는 `array_cat`이 아니라 `array_append`를 나타내는 것으로 추정된다. 이 선택이 잘못된 경우, 상수를 배열의 요소 타입으로 캐스팅하여 해결할 수 있지만, 명시적으로 `array_append`를 사용하는 것이 더 나은 해결책일 수 있다.


### 8.15.5. 배열 내 검색

배열에서 특정 값을 찾으려면 각 값을 하나씩 확인해야 한다. 배열의 크기를 알고 있다면 수동으로 검색할 수 있다. 예를 들어:

```
SELECT * FROM sal_emp WHERE pay_by_quarter[1] = 10000 OR
                            pay_by_quarter[2] = 10000 OR
                            pay_by_quarter[3] = 10000 OR
                            pay_by_quarter[4] = 10000;
```

하지만 배열이 크면 이 방법은 번거로워지며, 배열의 크기를 모를 경우에는 도움이 되지 않는다. 더 나은 방법은 [9.25절](https://www.postgresql.org/docs/current/functions-comparisons.html "9.25. Row and Array Comparisons")에서 설명한다. 위 쿼리는 다음과 같이 대체할 수 있다:

```
SELECT * FROM sal_emp WHERE 10000 = ANY (pay_by_quarter);
```

또한, 배열의 모든 값이 10000인 행을 찾으려면 다음과 같이 쿼리할 수 있다:

```
SELECT * FROM sal_emp WHERE 10000 = ALL (pay_by_quarter);
```

다른 방법으로 `generate_subscripts` 함수를 사용할 수도 있다. 예를 들어:

```
SELECT * FROM
   (SELECT pay_by_quarter,
           generate_subscripts(pay_by_quarter, 1) AS s
      FROM sal_emp) AS foo
 WHERE pay_by_quarter[s] = 10000;
```

이 함수는 [표 9.68](https://www.postgresql.org/docs/current/functions-srf.html#FUNCTIONS-SRF-SUBSCRIPTS "Table 9.68. Subscript Generating Functions")에서 자세히 설명한다.

`&&` 연산자를 사용해 배열을 검색할 수도 있다. 이 연산자는 왼쪽 피연산자가 오른쪽 피연산자와 겹치는지 확인한다. 예를 들어:

```
SELECT * FROM sal_emp WHERE pay_by_quarter && ARRAY[10000];
```

이와 같은 배열 연산자들은 [9.19절](https://www.postgresql.org/docs/current/functions-array.html "9.19. Array Functions and Operators")에서 더 자세히 설명한다. 적절한 인덱스를 사용하면 성능을 향상시킬 수 있으며, 이는 [11.2절](https://www.postgresql.org/docs/current/indexes-types.html "11.2. Index Types")에서 다룬다.

`array_position`과 `array_positions` 함수를 사용해 배열에서 특정 값을 검색할 수도 있다. `array_position`은 배열에서 값이 처음 나타나는 위치를 반환하고, `array_positions`는 값이 나타나는 모든 위치를 배열로 반환한다. 예를 들어:

```
SELECT array_position(ARRAY['sun','mon','tue','wed','thu','fri','sat'], 'mon');
 array_position
----------------
              2
(1 row)

SELECT array_positions(ARRAY[1, 4, 3, 1, 3, 4, 2, 1], 1);
 array_positions
-----------------
 {1,4,8}
(1 row)
```


### 팁

배열은 집합이 아니다. 특정 배열 요소를 검색하는 것은 데이터베이스 설계 오류의 신호일 수 있다. 각 배열 요소를 별도의 테이블 행으로 관리하는 방식을 고려해 보자. 이렇게 하면 검색이 더 쉬워지고, 요소 수가 많아져도 확장성이 좋아진다.


### 8.15.6. 배열 입력 및 출력 문법

배열 값의 외부 텍스트 표현은 배열 요소 타입에 대한 I/O 변환 규칙에 따라 해석되는 항목과 배열 구조를 나타내는 장식으로 구성된다. 장식은 배열 값을 둘러싼 중괄호(`{`와 `}`)와 인접한 항목 사이의 구분 문자로 이루어진다. 구분 문자는 보통 쉼표(`,`)이지만, 배열 요소 타입의 `typdelim` 설정에 따라 다른 문자일 수도 있다. PostgreSQL 배포판에서 제공하는 표준 데이터 타입 중 `box` 타입은 세미콜론(`;`)을 사용하며, 나머지는 모두 쉼표를 사용한다. 다차원 배열에서는 각 차원(행, 평면, 큐브 등)마다 중괄호가 한 단계씩 추가되며, 동일한 수준의 중괄호로 둘러싸인 항목 사이에 구분자를 작성해야 한다.

배열 출력 루틴은 요소 값이 빈 문자열이거나 중괄호, 구분 문자, 큰따옴표, 백슬래시, 공백을 포함하거나 `NULL`이라는 단어와 일치할 경우, 해당 요소 값을 큰따옴표로 둘러쌈. 요소 값에 포함된 큰따옴표와 백슬래시는 백슬래시로 이스케이프된다. 숫자 데이터 타입의 경우 큰따옴표가 나타나지 않는다고 가정해도 안전하지만, 텍스트 데이터 타입의 경우 큰따옴표가 있을 수도 있고 없을 수도 있으므로 이에 대비해야 한다.

기본적으로 배열 차원의 하한 인덱스 값은 1로 설정된다. 다른 하한을 가진 배열을 표현하려면 배열 내용을 작성하기 전에 배열 첨자 범위를 명시적으로 지정할 수 있다. 이 장식은 각 배열 차원의 하한과 상한을 대괄호(`[]`)로 둘러싸고, 그 사이에 콜론(`:`) 구분 문자를 사용한다. 배열 차원 장식 뒤에는 등호(`=`)가 온다. 예를 들어:

```
SELECT f1[1][-2][3] AS e1, f1[1][-1][5] AS e2
 FROM (SELECT '[1:1][-2:-1][3:5]={{{1,2,3},{4,5,6}}}'::int[] AS f1) AS ss;

 e1 | e2
----+----
  1 |  6
(1 row)
```

배열 출력 루틴은 하나 이상의 하한이 1과 다른 경우에만 명시적인 차원을 결과에 포함한다.

요소 값으로 `NULL`(대소문자 구분 없이)이 작성되면, 해당 요소는 NULL로 처리된다. 큰따옴표나 백슬래시가 있으면 이 기능이 비활성화되고, "NULL"이라는 문자열 값을 입력할 수 있다. 또한 PostgreSQL 8.2 이전 버전과의 호환성을 위해 `array_nulls` 설정 매개변수를 `off`로 설정하면 `NULL`을 NULL로 인식하지 않는다.

앞서 설명한 대로, 배열 값을 작성할 때 개별 배열 요소 주위에 큰따옴표를 사용할 수 있다. 요소 값이 배열 값 파서를 혼동시킬 수 있는 경우에는 반드시 큰따옴표를 사용해야 한다. 예를 들어, 중괄호, 쉼표(또는 데이터 타입의 구분 문자), 큰따옴표, 백슬래시, 앞뒤 공백을 포함하는 요소는 큰따옴표로 둘러싸야 한다. 빈 문자열과 `NULL`이라는 단어와 일치하는 문자열도 큰따옴표로 둘러싸야 한다. 큰따옴표나 백슬래시를 큰따옴표로 둘러싼 배열 요소 값에 포함시키려면 앞에 백슬래시를 추가한다. 또는 큰따옴표를 사용하지 않고 백슬래시 이스케이프를 사용해 배열 문법으로 해석될 수 있는 모든 데이터 문자를 보호할 수도 있다.

왼쪽 중괄호 앞이나 오른쪽 중괄호 뒤에 공백을 추가할 수 있다. 또한 개별 항목 문자열 앞뒤에 공백을 추가할 수도 있다. 이러한 경우 공백은 무시된다. 그러나 큰따옴표로 둘러싸인 요소 내부의 공백이나 요소의 비공백 문자 양쪽으로 둘러싸인 공백은 무시되지 않는다.


### 팁

SQL 명령에서 배열 값을 작성할 때, 배열 리터럴 문법보다 `ARRAY` 생성자 문법([4.2.12절](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-ARRAY-CONSTRUCTORS "4.2.12. Array Constructors") 참조)을 사용하는 것이 더 편리할 때가 많다. `ARRAY` 문법에서는 개별 요소 값을 배열의 일부가 아닌 일반 값처럼 작성할 수 있다.




## 8.16. 합성 타입

*합성 타입* 은 행이나 레코드의 구조를 나타낸다. 기본적으로 필드 이름과 해당 데이터 타입의 목록으로 구성된다. PostgreSQL은 단순 타입을 사용할 수 있는 다양한 방식으로 합성 타입을 사용할 수 있도록 허용한다. 예를 들어, 테이블의 컬럼을 합성 타입으로 선언할 수 있다.


### 8.16.1. 합성 타입 선언

다음은 합성 타입을 정의하는 간단한 예제다:

```
CREATE TYPE complex AS (
    r       double precision,
    i       double precision
);

CREATE TYPE inventory_item AS (
    name            text,
    supplier_id     integer,
    price           numeric
);
```

이 문법은 `CREATE TABLE`과 유사하지만, 필드 이름과 타입만 지정할 수 있다. 현재는 `NOT NULL`과 같은 제약 조건을 포함할 수 없다. `AS` 키워드는 필수적이다. 이 키워드가 없으면 다른 종류의 `CREATE TYPE` 명령으로 인식되어 이상한 구문 오류가 발생할 수 있다.

타입을 정의한 후, 이를 사용해 테이블을 생성할 수 있다:

```
CREATE TABLE on_hand (
    item      inventory_item,
    count     integer
);

INSERT INTO on_hand VALUES (ROW('fuzzy dice', 42, 1.99), 1000);
```

또는 함수를 생성할 수도 있다:

```
CREATE FUNCTION price_extension(inventory_item, integer) RETURNS numeric
AS 'SELECT $1.price * $2' LANGUAGE SQL;

SELECT price_extension(item, 10) FROM on_hand;
```

테이블을 생성할 때마다, 해당 테이블의 행 타입을 나타내는 동일한 이름의 합성 타입도 자동으로 생성된다. 예를 들어, 다음과 같이 테이블을 생성하면:

```
CREATE TABLE inventory_item (
    name            text,
    supplier_id     integer REFERENCES suppliers,
    price           numeric CHECK (price > 0)
);
```

위에서 보여준 것과 같은 `inventory_item` 합성 타입이 부산물로 생성되며, 동일한 방식으로 사용할 수 있다. 하지만 현재 구현의 중요한 제한 사항에 주의해야 한다: 합성 타입에는 제약 조건이 연결되지 않으므로, 테이블 정의에 표시된 제약 조건은 테이블 외부의 합성 타입 값에는 적용되지 않는다. (이 문제를 해결하려면 합성 타입 위에 [도메인](https://www.postgresql.org/docs/current/glossary.html#GLOSSARY-DOMAIN "Domain")을 생성하고, 원하는 제약 조건을 도메인의 `CHECK` 제약 조건으로 적용하면 된다.)


### 8.16.2. 복합 값 구성

복합 값을 리터럴 상수로 작성하려면, 필드 값을 괄호로 감싸고 쉼표로 구분한다. 필드 값에 쉼표나 괄호가 포함된 경우, 해당 값을 큰따옴표로 감싸야 한다. (자세한 내용은 [아래](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-IO-SYNTAX "8.16.6. 복합 타입 입력 및 출력 문법")에서 확인할 수 있다.) 따라서 복합 상수의 일반적인 형식은 다음과 같다:

```
val1
```

예를 들어:

```
'("fuzzy dice",42,1.99)'
```

이는 앞서 정의한 `inventory_item` 타입의 유효한 값이다. 필드를 NULL로 지정하려면, 리스트에서 해당 위치에 아무 문자도 쓰지 않는다. 예를 들어, 이 상수는 세 번째 필드를 NULL로 지정한다:

```
'("fuzzy dice",42,)'
```

NULL 대신 빈 문자열을 원한다면, 큰따옴표를 사용한다:

```
'("",42,)'
```

여기서 첫 번째 필드는 NULL이 아닌 빈 문자열이고, 세 번째 필드는 NULL이다.

(이러한 상수는 실제로 [4.1.2.7절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS-GENERIC "4.1.2.7. 기타 타입의 상수")에서 논의한 일반 타입 상수의 특수한 경우다. 상수는 초기에 문자열로 처리되며, 복합 타입 입력 변환 루틴에 전달된다. 상수를 어떤 타입으로 변환할지 명시적으로 지정해야 할 수도 있다.)

`ROW` 표현식 문법을 사용해 복합 값을 구성할 수도 있다. 대부분의 경우 이 방법은 문자열 리터럴 문법보다 훨씬 간단하며, 여러 겹의 따옴표를 신경 쓸 필요가 없다. 앞서 이 방법을 사용한 예는 다음과 같다:

```
ROW('fuzzy dice', 42, 1.99)
ROW('', 42, NULL)
```

표현식에 두 개 이상의 필드가 있는 경우, `ROW` 키워드는 생략할 수 있다. 따라서 위의 예는 다음과 같이 단순화할 수 있다:

```
('fuzzy dice', 42, 1.99)
('', 42, NULL)
```

`ROW` 표현식 문법에 대한 자세한 내용은 [4.2.13절](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-ROW-CONSTRUCTORS "4.2.13. 행 생성자")에서 다룬다.


### 8.16.3. 복합 타입 접근

복합 타입 컬럼의 필드에 접근하려면, 테이블 이름에서 필드를 선택하는 것처럼 점(`.`)과 필드 이름을 사용한다. 실제로 테이블 이름에서 선택하는 것과 매우 유사하기 때문에, 파서가 혼동하지 않도록 괄호를 사용해야 하는 경우가 많다. 예를 들어, `on_hand` 예제 테이블에서 일부 하위 필드를 선택하려고 다음과 같이 시도할 수 있다:

```
SELECT item.name FROM on_hand WHERE item.price > 9.99;
```

하지만 이 쿼리는 SQL 문법 규칙에 따라 `item`이 `on_hand`의 컬럼 이름이 아니라 테이블 이름으로 해석되기 때문에 작동하지 않는다. 대신 다음과 같이 작성해야 한다:

```
SELECT (item).name FROM on_hand WHERE (item).price > 9.99;
```

또는 테이블 이름도 함께 사용해야 하는 경우(예: 다중 테이블 쿼리), 다음과 같이 작성한다:

```
SELECT (on_hand.item).name FROM on_hand WHERE (on_hand.item).price > 9.99;
```

이제 괄호로 묶인 객체가 `item` 컬럼을 참조하는 것으로 올바르게 해석되고, 그 안에서 하위 필드를 선택할 수 있다.

복합 타입 값에서 필드를 선택할 때마다 유사한 문법 문제가 발생한다. 예를 들어, 복합 타입을 반환하는 함수의 결과에서 하나의 필드만 선택하려면 다음과 같이 작성해야 한다:

```
SELECT (my_func(...)).field FROM ...
```

추가 괄호 없이 이 쿼리를 실행하면 문법 오류가 발생한다.

특수 필드 이름 `*`는 "모든 필드"를 의미하며, 이에 대한 자세한 설명은 [8.16.5절](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-USAGE "8.16.5. 쿼리에서 복합 타입 사용")에서 확인할 수 있다.


### 8.16.4. 복합 타입 수정

복합 컬럼을 삽입하거나 업데이트할 때 사용하는 적절한 문법 예제를 살펴보자. 먼저, 전체 컬럼을 삽입하거나 업데이트하는 방법이다:

```
INSERT INTO mytab (complex_col) VALUES((1.1,2.2));

UPDATE mytab SET complex_col = ROW(1.1,2.2) WHERE ...;
```

첫 번째 예제는 `ROW`를 생략했고, 두 번째 예제는 `ROW`를 사용했다. 두 방법 모두 사용할 수 있다.

복합 컬럼의 개별 하위 필드를 업데이트할 수도 있다:

```
UPDATE mytab SET complex_col.r = (complex_col).r + 1 WHERE ...;
```

여기서 `SET` 바로 뒤에 오는 컬럼 이름 주위에 괄호를 사용할 필요가 없으며, 실제로 사용할 수도 없다. 하지만 등호 오른쪽의 표현식에서 동일한 컬럼을 참조할 때는 괄호가 필요하다.

`INSERT`에서도 하위 필드를 대상으로 지정할 수 있다:

```
INSERT INTO mytab (complex_col.r, complex_col.i) VALUES(1.1, 2.2);
```

컬럼의 모든 하위 필드에 값을 제공하지 않았다면, 나머지 하위 필드는 null 값으로 채워진다.


### 8.16.5. 쿼리에서 복합 타입 사용

쿼리에서 복합 타입을 사용할 때는 여러 특별한 문법 규칙과 동작이 적용된다. 이러한 규칙은 유용한 단축 기능을 제공하지만, 그 배경을 모르면 혼란스러울 수 있다.

PostgreSQL에서 쿼리 내의 테이블 이름(또는 별칭)은 해당 테이블의 현재 행에 대한 복합 값으로 간주된다. 예를 들어, 앞서 설명한 `inventory_item` 테이블이 있다면 다음과 같은 쿼리를 작성할 수 있다:

```
SELECT c FROM inventory_item c;
```

이 쿼리는 단일 복합 값 컬럼을 생성하며, 출력 결과는 다음과 같을 수 있다:

```
c
------------------------
 ("fuzzy dice",42,1.99)
(1 row)
```

하지만 단순한 이름은 테이블 이름보다 컬럼 이름과 먼저 매칭된다는 점에 유의해야 한다. 이 예제가 동작하는 이유는 쿼리 내의 테이블에 `c`라는 컬럼이 없기 때문이다.

일반적인 컬럼 이름 지정 문법인 `table_name.column_name`은 테이블의 현재 행에 대한 복합 값에 [필드 선택](https://www.postgresql.org/docs/current/sql-expressions.html#FIELD-SELECTION "4.2.4. Field Selection")을 적용하는 것으로 이해할 수 있다. (효율성 문제로 실제로는 그렇게 구현되지는 않는다.)

다음과 같은 쿼리를 작성하면:

```
SELECT c.* FROM inventory_item c;
```

SQL 표준에 따라 테이블의 내용이 별도의 컬럼으로 확장되어 출력된다:

```
name    | supplier_id | price
------------+-------------+-------
 fuzzy dice |          42 |  1.99
(1 row)
```

이는 마치 다음과 같은 쿼리를 실행한 것과 같다:

```
SELECT c.name, c.supplier_id, c.price FROM inventory_item c;
```

PostgreSQL은 모든 복합 값 표현식에 대해 이 확장 동작을 적용한다. 하지만 앞서 설명한 것처럼, `.*`를 적용할 값이 단순한 테이블 이름이 아닐 경우에는 해당 값 주위에 괄호를 써야 한다. 예를 들어, `myfunc()`가 `a`, `b`, `c` 컬럼을 가진 복합 타입을 반환하는 함수라면, 다음 두 쿼리는 동일한 결과를 생성한다:

```
SELECT (myfunc(x)).* FROM some_table;
SELECT (myfunc(x)).a, (myfunc(x)).b, (myfunc(x)).c FROM some_table;
```


### 팁

PostgreSQL은 컬럼 확장을 처리할 때 첫 번째 형태를 두 번째 형태로 변환한다. 따라서 이 예제에서는 `myfunc()`가 어떤 문법을 사용하든 행마다 세 번 호출된다. 만약 이 함수가 비용이 많이 드는 작업이라면, 다음과 같은 쿼리를 사용해 이를 피할 수 있다:

```sql
SELECT m.* FROM some_table, LATERAL myfunc(x) AS m;
```

함수를 `LATERAL` `FROM` 항목에 배치하면 행마다 한 번만 호출되도록 할 수 있다. `m.*`는 여전히 `m.a, m.b, m.c`로 확장되지만, 이제 이러한 변수들은 `FROM` 항목의 출력을 참조한다. (`LATERAL` 키워드는 여기서 선택 사항이지만, 함수가 `some_table`에서 `x`를 가져오는 것을 명확히 하기 위해 표시했다.)

`composite_value.*` 구문은 [`SELECT` 출력 목록](https://www.postgresql.org/docs/current/queries-select-lists.html "7.3. Select Lists"), `INSERT`/`UPDATE`/`DELETE`/`MERGE`의 [`RETURNING` 목록](https://www.postgresql.org/docs/current/dml-returning.html "6.4. Returning Data from Modified Rows"), [`VALUES` 절](https://www.postgresql.org/docs/current/queries-values.html "7.7. VALUES Lists"), 또는 [행 생성자](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-ROW-CONSTRUCTORS "4.2.13. Row Constructors")의 최상위 수준에 나타날 때 이러한 종류의 컬럼 확장을 유발한다. 다른 모든 맥락에서(이러한 구조 내부에 중첩된 경우 포함), `.*`를 복합 값에 붙여도 값은 변경되지 않는다. 왜냐하면 이는 "모든 컬럼"을 의미하며, 동일한 복합 값이 다시 생성되기 때문이다. 예를 들어, `somefunc()`가 복합 값을 인자로 받는다면, 다음 두 쿼리는 동일하다:

```sql
SELECT somefunc(c.*) FROM inventory_item c;
SELECT somefunc(c) FROM inventory_item c;
```

두 경우 모두 `inventory_item`의 현재 행이 단일 복합 값 인자로 함수에 전달된다. `.*`가 이러한 경우 아무런 역할을 하지 않더라도, 이를 사용하는 것은 좋은 스타일로 간주된다. 왜냐하면 복합 값이 의도되었다는 것을 명확히 할 수 있기 때문이다. 특히, 파서는 `c.*`에서 `c`를 테이블 이름이나 별칭으로 간주하며, 컬럼 이름으로 간주하지 않는다. 따라서 모호함이 없다. 반면 `.*` 없이는 `c`가 테이블 이름인지 컬럼 이름인지 명확하지 않으며, 실제로 `c`라는 이름의 컬럼이 있다면 컬럼 이름 해석이 우선시된다.

이러한 개념을 보여주는 또 다른 예로, 다음 쿼리들은 모두 동일한 의미를 가진다:

```sql
SELECT * FROM inventory_item c ORDER BY c;
SELECT * FROM inventory_item c ORDER BY c.*;
SELECT * FROM inventory_item c ORDER BY ROW(c.*);
```

이 모든 `ORDER BY` 절은 행의 복합 값을 지정하며, [Section 9.25.6](https://www.postgresql.org/docs/current/functions-comparisons.html#COMPOSITE-TYPE-COMPARISON "9.25.6. Composite Type Comparison")에 설명된 규칙에 따라 행을 정렬한다. 그러나 만약 `inventory_item`에 `c`라는 컬럼이 있다면, 첫 번째 경우는 다른 경우와 다르게 동작한다. 왜냐하면 이 경우 해당 컬럼만으로 정렬하게 되기 때문이다. 이전에 보여준 컬럼 이름을 고려할 때, 다음 쿼리들도 위의 쿼리들과 동일하다:

```sql
SELECT * FROM inventory_item c ORDER BY ROW(c.name, c.supplier_id, c.price);
SELECT * FROM inventory_item c ORDER BY (c.name, c.supplier_id, c.price);
```

(마지막 경우는 `ROW` 키워드를 생략한 행 생성자를 사용한다.)

복합 값과 관련된 또 다른 특별한 구문적 동작은 복합 값의 필드를 추출하기 위해 *함수 표기법*을 사용할 수 있다는 것이다. 이를 간단히 설명하면, `field(table)`와 `table.field` 표기법은 서로 교환 가능하다. 예를 들어, 다음 쿼리들은 동일하다:

```sql
SELECT c.name FROM inventory_item c WHERE c.price > 1000;
SELECT name(c) FROM inventory_item c WHERE price(c) > 1000;
```

또한, 복합 타입의 단일 인자를 받는 함수가 있다면, 이 함수를 두 표기법 중 어느 것으로도 호출할 수 있다. 다음 쿼리들은 모두 동일하다:

```sql
SELECT somefunc(c) FROM inventory_item c;
SELECT somefunc(c.*) FROM inventory_item c;
SELECT c.somefunc FROM inventory_item c;
```

함수 표기법과 필드 표기법 간의 이러한 동등성은 복합 타입에 대한 함수를 사용해 "계산된 필드"를 구현할 수 있게 한다. 위의 마지막 쿼리를 사용하는 애플리케이션은 `somefunc`가 테이블의 실제 컬럼이 아니라는 사실을 직접 알 필요가 없다.


### 팁

이러한 동작 때문에, 단일 복합 타입 인자를 받는 함수에 그 복합 타입의 필드와 동일한 이름을 붙이는 것은 좋지 않다. 모호한 상황에서 필드 이름 구문을 사용하면 필드 이름 해석이 선택되고, 함수 호출 구문을 사용하면 함수가 선택된다. 하지만 PostgreSQL 11 이전 버전에서는 호출 구문이 함수 호출을 요구하지 않는 한 항상 필드 이름 해석을 선택했다. 이전 버전에서 함수 해석을 강제하려면 스키마 한정 함수 이름을 사용하면 된다. 즉, `schema.func(compositevalue)`와 같이 작성한다.


### 8.16.6. 복합 타입의 입력 및 출력 문법

복합 값의 외부 텍스트 표현은 각 필드 타입에 대한 I/O 변환 규칙에 따라 해석되는 항목들로 구성되며, 복합 구조를 나타내는 장식 요소가 추가된다. 이 장식 요소는 전체 값을 감싸는 괄호(`(`와 `)`)와 항목들 사이에 위치하는 쉼표(`,`)로 이루어진다. 괄호 바깥의 공백은 무시되지만, 괄호 안의 공백은 필드 값의 일부로 간주되며, 필드 데이터 타입의 입력 변환 규칙에 따라 의미가 있을 수도 있고 없을 수도 있다. 예를 들어:

```
'(  42)'
```

위 예제에서 필드 타입이 정수형이라면 공백은 무시되지만, 텍스트 타입이라면 무시되지 않는다.

앞서 설명한 것처럼, 복합 값을 작성할 때는 개별 필드 값 주위에 큰따옴표를 사용할 수 있다. 필드 값이 복합 값 파서를 혼동시킬 가능성이 있다면 큰따옴표를 반드시 사용해야 한다. 특히, 괄호, 쉼표, 큰따옴표, 백슬래시가 포함된 필드는 반드시 큰따옴표로 감싸야 한다. 큰따옴표로 감싸진 복합 필드 값 안에 큰따옴표나 백슬래시를 넣으려면 해당 문자 앞에 백슬래시를 추가해야 한다. (또한, 큰따옴표로 감싸진 필드 값 안의 큰따옴표 쌍은 SQL 문자열 리터럴에서 작은따옴표를 처리하는 규칙과 유사하게 큰따옴표 문자를 나타낸다.) 또는, 따옴표를 사용하지 않고 백슬래시 이스케이프를 통해 복합 문법으로 해석될 수 있는 모든 데이터 문자를 보호할 수도 있다.

완전히 비어 있는 필드 값(쉼표나 괄호 사이에 아무 문자도 없는 경우)은 NULL을 나타낸다. 빈 문자열을 나타내려면 `""`를 작성해야 한다.

복합 출력 루틴은 필드 값이 빈 문자열이거나 괄호, 쉼표, 큰따옴표, 백슬래시, 공백을 포함하는 경우 해당 필드 값 주위에 큰따옴표를 추가한다. (공백에 대해 큰따옴표를 사용하는 것은 필수는 아니지만 가독성을 높이는 데 도움이 된다.) 필드 값에 포함된 큰따옴표와 백슬래시는 두 배로 처리된다.


### 주의사항

SQL 명령어에 작성한 내용은 먼저 문자열 리터럴로 해석된 후, 복합 타입으로 해석된다. 이 과정에서 백슬래시의 수가 두 배로 늘어난다(이스케이프 문자열 구문을 사용한다고 가정할 때). 예를 들어, 복합 값 안에 쌍따옴표와 백슬래시가 포함된 `text` 필드를 삽입하려면 다음과 같이 작성해야 한다:

```
INSERT ... VALUES ('(""\")');
```

문자열 리터럴 프로세서가 한 단계의 백슬래시를 제거하기 때문에, 복합 값 파서에 도달하는 문자열은 `(""\")`처럼 보인다. 이어서 `text` 데이터 타입의 입력 루틴에 전달된 문자열은 `"`가 된다. (백슬래시를 특별하게 처리하는 데이터 타입, 예를 들어 `bytea`를 사용한다면, 저장된 복합 필드에 하나의 백슬래시를 넣기 위해 커맨드에서 최대 여덟 개의 백슬래시가 필요할 수 있다.) 달러 인용부호([4.1.2.4절](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-DOLLAR-QUOTING "4.1.2.4. Dollar-Quoted String Constants") 참조)를 사용하면 백슬래시를 두 배로 늘릴 필요가 없어진다.


### 팁

SQL 커맨드에서 복합 값을 작성할 때, 복합 리터럴 구문보다 `ROW` 생성자 구문을 사용하는 것이 일반적으로 더 쉽다. `ROW`에서는 개별 필드 값을 복합체의 멤버가 아닌 경우와 동일한 방식으로 작성할 수 있다.




## 8.17. 범위 타입

범위 타입은 특정 요소 타입(범위의 *하위 타입*이라고 함)의 값 범위를 나타내는 데이터 타입이다. 예를 들어, `timestamp`의 범위는 회의실이 예약된 시간 범위를 나타내는 데 사용될 수 있다. 이 경우 데이터 타입은 `tsrange`("timestamp range"의 약어)이고, `timestamp`가 하위 타입이다. 하위 타입은 전체 순서를 가져야 하며, 이는 요소 값이 범위 내에 있는지, 앞에 있는지, 뒤에 있는지 명확하게 정의할 수 있도록 한다.

범위 타입은 하나의 범위 값으로 여러 요소 값을 표현할 수 있고, 범위가 겹치는 것과 같은 개념을 명확하게 표현할 수 있기 때문에 유용하다. 시간과 날짜 범위를 사용해 스케줄링하는 것이 가장 명확한 예시다. 하지만 가격 범위, 기기에서 측정한 값의 범위 등도 유용하게 사용할 수 있다.

모든 범위 타입에는 해당하는 다중 범위 타입이 있다. 다중 범위는 연속적이지 않고 비어 있지 않으며 null이 아닌 범위들로 이루어진 순서가 있는 목록이다. 대부분의 범위 연산자는 다중 범위에서도 작동하며, 다중 범위만의 몇 가지 함수도 있다.


### 8.17.1. 내장된 범위 및 다중 범위 타입

PostgreSQL은 다음과 같은 내장 범위 타입을 제공한다:

*   `int4range` — `integer` 범위, `int4multirange` — 해당 다중 범위
    
*   `int8range` — `bigint` 범위, `int8multirange` — 해당 다중 범위
    
*   `numrange` — `numeric` 범위, `nummultirange` — 해당 다중 범위
    
*   `tsrange` — `timestamp without time zone` 범위, `tsmultirange` — 해당 다중 범위
    
*   `tstzrange` — `timestamp with time zone` 범위, `tstzmultirange` — 해당 다중 범위
    
*   `daterange` — `date` 범위, `datemultirange` — 해당 다중 범위
    

또한, 여러분은 자신만의 범위 타입을 정의할 수 있다. 자세한 내용은 [CREATE TYPE](https://www.postgresql.org/docs/current/sql-createtype.html "CREATE TYPE")을 참고한다.


### 8.17.2. 예제

```
CREATE TABLE reservation (room int, during tsrange);
INSERT INTO reservation VALUES
    (1108, '[2010-01-01 14:30, 2010-01-01 15:30)');

-- 포함 여부 확인
SELECT int4range(10, 20) @> 3;

-- 겹침 여부 확인
SELECT numrange(11.1, 22.2) && numrange(20.0, 30.0);

-- 상한 값 추출
SELECT upper(int8range(15, 25));

-- 교집합 계산
SELECT int4range(10, 20) * int4range(15, 25);

-- 범위가 비어 있는지 확인
SELECT isempty(numrange(1, 5));
```

범위 타입에 대한 연산자와 함수의 전체 목록은 [Table 9.56](https://www.postgresql.org/docs/current/functions-range.html#RANGE-OPERATORS-TABLE "Table 9.56. Range Operators")과 [Table 9.58](https://www.postgresql.org/docs/current/functions-range.html#RANGE-FUNCTIONS-TABLE "Table 9.58. Range Functions")을 참조한다.


### 8.17.3. 포함적 경계와 배타적 경계

비어 있지 않은 모든 범위는 하한 경계와 상한 경계 두 가지를 갖는다. 이 두 값 사이의 모든 점은 범위에 포함된다. 포함적 경계는 경계점 자체도 범위에 포함된다는 의미이며, 배타적 경계는 경계점이 범위에 포함되지 않는다는 의미다.

범위를 텍스트로 표현할 때, 하한 경계가 포함적이면 “`[`”로 표시하고, 배타적이면 “`(`”로 표시한다. 마찬가지로 상한 경계가 포함적이면 “`]`”로, 배타적이면 “`)`”로 표시한다. (자세한 내용은 [8.17.5절](https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-IO "8.17.5. Range Input/Output")을 참고한다.)

`lower_inc`와 `upper_inc` 함수는 각각 범위 값의 하한 경계와 상한 경계가 포함적인지 여부를 확인한다.


### 8.17.4. 무한(무한정) 범위

범위의 하한을 생략하면 상한보다 작은 모든 값이 범위에 포함된다. 예를 들어 `(,3]`과 같은 형태다. 마찬가지로 범위의 상한을 생략하면 하한보다 큰 모든 값이 범위에 포함된다. 하한과 상한을 모두 생략하면 해당 요소 타입의 모든 값이 범위에 포함된다. 포함으로 지정된 누락된 경계는 자동으로 제외로 변환된다. 예를 들어 `[,]`는 `(,)`로 변환된다. 이러한 누락된 값을 +/-무한대로 생각할 수 있지만, 이들은 특수한 범위 타입 값이며 범위 요소 타입의 +/-무한대 값보다 더 큰 개념이다.

"무한대" 개념을 가진 요소 타입은 이를 명시적 경계 값으로 사용할 수 있다. 예를 들어, 타임스탬프 범위에서 `[today,infinity)`는 특수한 `timestamp` 값 `infinity`를 제외하지만, `[today,infinity]`와 `[today,)`, `[today,]`는 이를 포함한다.

`lower_inf`와 `upper_inf` 함수는 각각 범위의 하한과 상한이 무한한지 테스트한다.


### 8.17.5. 범위 입력/출력

범위 값을 입력할 때는 다음 패턴 중 하나를 따라야 한다:

```
lower-bound
```

괄호 또는 대괄호는 하한과 상한이 포함되는지 여부를 나타낸다. 마지막 패턴은 `empty`로, 아무런 점도 포함하지 않는 빈 범위를 의미한다.

*`lower-bound`*는 하위 타입에 유효한 문자열이거나, 하한이 없음을 나타내기 위해 비워둘 수 있다. 마찬가지로, *`upper-bound`*는 상위 타입에 유효한 문자열이거나, 상한이 없음을 나타내기 위해 비워둘 수 있다.

각 경계 값은 `"`(쌍따옴표)로 묶을 수 있다. 경계 값에 괄호, 대괄호, 쉼표, 쌍따옴표, 백슬래시가 포함된 경우 이 문자들이 범위 문법의 일부로 해석되지 않도록 쌍따옴표로 묶어야 한다. 쌍따옴표로 묶은 경계 값 안에 쌍따옴표나 백슬래시를 넣으려면 앞에 백슬래시를 추가한다. (또한, 쌍따옴표로 묶은 경계 값 안에 쌍따옴표 두 개를 연속해서 사용하면 SQL 문자열 리터럴에서의 작은따옴표 규칙과 유사하게 쌍따옴표 문자로 해석된다.) 또는, 쌍따옴표를 사용하지 않고 백슬래시 이스케이프를 통해 범위 문법으로 해석될 수 있는 모든 데이터 문자를 보호할 수도 있다. 또한, 빈 문자열을 경계 값으로 사용하려면 `""`로 작성해야 한다. 아무것도 작성하지 않으면 무한한 경계를 의미한다.

범위 값 앞뒤에 공백을 넣을 수 있지만, 괄호나 대괄호 사이에 있는 공백은 하한 또는 상한 값의 일부로 간주된다. (요소 타입에 따라 이 공백이 의미가 있을 수도 있고 없을 수도 있다.)


### 참고

이 규칙들은 복합 타입 리터럴에서 필드 값을 작성할 때의 규칙과 매우 유사하다. 추가 설명은 [8.16.6절](https://www.postgresql.org/docs/current/rowtypes.html#ROWTYPES-IO-SYNTAX "8.16.6. 복합 타입 입력 및 출력 문법")을 참고한다.

예제:

```
-- 3을 포함하고, 7은 포함하지 않으며, 그 사이의 모든 점을 포함한다.
SELECT '[3,7)'::int4range;

-- 3과 7을 포함하지 않지만, 그 사이의 모든 점을 포함한다.
SELECT '(3,7)'::int4range;

-- 단일 점 4만 포함한다.
SELECT '[4,4]'::int4range;

-- 어떤 점도 포함하지 않는다(그리고 'empty'로 정규화된다).
SELECT '[4,4)'::int4range;
```

다중 범위(multirange)의 입력은 중괄호(`{`와 `}`)로 감싸고, 그 안에 쉼표로 구분된 하나 이상의 유효한 범위를 포함한다. 중괄호와 쉼표 주변에는 공백을 허용한다. 이는 배열 문법을 연상시키도록 의도되었지만, 다중 범위는 훨씬 단순하다. 단일 차원만 가지고 있으며, 내용을 따옴표로 감쌀 필요가 없다. (단, 범위의 경계는 위와 같이 따옴표로 감쌀 수 있다.)

예제:

```
SELECT '{}'::int4multirange;
SELECT '{[3,7)}'::int4multirange;
SELECT '{[3,7), [8,9)}'::int4multirange;
```


### 8.17.6. 범위와 다중 범위 생성

각 범위 타입은 해당 범위 타입과 동일한 이름을 가진 생성자 함수를 제공한다. 생성자 함수를 사용하면 범위 리터럴 상수를 작성할 때보다 더 편리한데, 이는 경계 값에 추가적인 따옴표를 사용할 필요가 없기 때문이다. 생성자 함수는 두 개 또는 세 개의 인자를 받는다. 두 개의 인자를 받는 형태는 표준 형식(하한 포함, 상한 제외)으로 범위를 생성하며, 세 개의 인자를 받는 형태는 세 번째 인자로 지정된 형식에 따라 범위를 생성한다. 세 번째 인자는 반드시 "`()`", "`(]`", "`[)`", "`[]`" 중 하나여야 한다. 예를 들면:

```
-- 전체 형식은 다음과 같다: 하한, 상한, 그리고 경계의 포함/제외 여부를 나타내는 텍스트 인자.
SELECT numrange(1.0, 14.0, '(]');

-- 세 번째 인자를 생략하면 '[)' 형식으로 가정한다.
SELECT numrange(1.0, 14.0);

-- 여기서 '(]'를 지정했지만, int8range는 이산 범위 타입이므로 표시 시 표준 형식으로 변환된다.
SELECT int8range(1, 14, '(]');

-- 하한 또는 상한에 NULL을 사용하면 해당 측면이 무한한 범위가 생성된다.
SELECT numrange(NULL, 2.2);
```

각 범위 타입은 해당 다중 범위 타입과 동일한 이름을 가진 다중 범위 생성자도 제공한다. 생성자 함수는 적절한 타입의 범위를 인자로 받거나, 인자를 받지 않을 수도 있다. 예를 들면:

```
SELECT nummultirange();
SELECT nummultirange(numrange(1.0, 14.0));
SELECT nummultirange(numrange(1.0, 14.0), numrange(20.0, 25.0));
```


### 8.17.7. 이산 범위 타입

이산 범위는 `integer`나 `date`처럼 요소 타입이 명확한 "단계"를 가지고 있는 범위를 말한다. 이러한 타입에서는 두 요소가 서로 인접해 있을 때, 그 사이에 유효한 값이 존재하지 않는다. 이는 연속 범위와 대조적이다. 연속 범위에서는 두 값 사이에 항상(혹은 거의 항상) 다른 요소 값을 식별할 수 있다. 예를 들어, `numeric` 타입의 범위는 연속적이며, `timestamp` 타입의 범위도 마찬가지이다. (`timestamp`는 정밀도가 제한되어 있기 때문에 이론적으로는 이산적으로 취급할 수 있지만, 일반적으로 단계 크기가 중요하지 않으므로 연속적인 것으로 간주하는 것이 더 적절하다.)

이산 범위 타입을 이해하는 또 다른 방법은 각 요소 값에 대해 "다음" 또는 "이전" 값에 대한 명확한 개념이 있다는 것이다. 이를 통해 범위의 경계를 포함적(inclusive) 또는 배제적(exclusive) 표현으로 변환할 수 있다. 예를 들어, 정수 범위 타입에서 `[4,8]`과 `(3,9)`는 동일한 값 집합을 나타낸다. 그러나 `numeric` 타입의 범위에서는 이렇게 되지 않는다.

이산 범위 타입은 요소 타입의 단계 크기를 고려한 *정규화(canonicalization)* 함수를 가져야 한다. 정규화 함수는 동등한 범위 타입의 값을 동일한 표현으로 변환하는 역할을 한다. 특히, 일관되게 포함적이거나 배제적인 경계를 가지도록 한다. 정규화 함수가 지정되지 않으면, 서로 다른 형식의 범위는 실제로 동일한 값 집합을 나타내더라도 항상 다른 것으로 처리된다.

내장된 범위 타입인 `int4range`, `int8range`, `daterange`는 모두 하한을 포함하고 상한을 배제하는 정규 형식을 사용한다. 즉, `[)` 형식을 따른다. 그러나 사용자 정의 범위 타입은 다른 규칙을 사용할 수도 있다.


### 8.17.8. 새로운 범위 타입 정의

사용자는 자신만의 범위 타입을 정의할 수 있다. 가장 일반적인 이유는 내장된 범위 타입에서 제공하지 않는 하위 타입을 사용하기 위해서다. 예를 들어, `float8` 하위 타입을 사용하는 새로운 범위 타입을 정의하려면 다음과 같이 작성한다:

```
CREATE TYPE floatrange AS RANGE (
    subtype = float8,
    subtype_diff = float8mi
);

SELECT '[1.234, 5.678]'::floatrange;
```

`float8`은 의미 있는 "단계"가 없기 때문에 이 예제에서는 정규화 함수를 정의하지 않았다.

사용자가 자신만의 범위 타입을 정의하면 자동으로 해당하는 다중 범위 타입도 함께 생성된다.

자신만의 범위 타입을 정의하면 다른 하위 타입 B-tree 연산자 클래스나 정렬 규칙을 지정할 수 있다. 이를 통해 주어진 범위에 속하는 값을 결정하는 정렬 순서를 변경할 수 있다.

하위 타입이 연속적인 값이 아니라 이산적인 값을 가진다면, `CREATE TYPE` 명령어에 `canonical` 함수를 지정해야 한다. 정규화 함수는 입력 범위 값을 받아 동일한 값 집합을 나타내지만 다른 경계와 형식을 가진 범위 값을 반환해야 한다. 예를 들어, 정수 범위 `[1, 7]`과 `[1, 8)`은 동일한 값 집합을 나타내므로 정규화된 출력이 동일해야 한다. 어떤 표현을 정규화된 것으로 선택하는지는 중요하지 않지만, 서로 다른 형식을 가진 동등한 값이 항상 동일한 형식의 동일한 값으로 매핑되어야 한다. 정규화 함수는 포함/제외 경계 형식을 조정하는 것 외에도, 원하는 단계 크기가 하위 타입이 저장할 수 있는 것보다 큰 경우 경계 값을 반올림할 수 있다. 예를 들어, `timestamp`를 기반으로 한 범위 타입을 시간 단위로 정의할 수 있으며, 이 경우 정규화 함수는 시간의 배수가 아닌 경계 값을 반올림하거나 오류를 발생시켜야 한다.

또한, GiST 또는 SP-GiST 인덱스와 함께 사용할 범위 타입은 하위 타입 차이 함수, 즉 `subtype_diff` 함수를 정의해야 한다. (인덱스는 `subtype_diff` 없이도 동작하지만, 차이 함수를 제공한 경우보다 효율성이 크게 떨어질 수 있다.) 하위 타입 차이 함수는 하위 타입의 두 입력 값을 받아 그 차이(즉, *`X`*에서 *`Y`*를 뺀 값)를 `float8` 값으로 반환한다. 위의 예제에서는 일반 `float8` 뺄셈 연산자를 기반으로 하는 `float8mi` 함수를 사용할 수 있지만, 다른 하위 타입의 경우 일부 타입 변환이 필요할 수 있다. 또한, 차이를 숫자로 표현하는 방법에 대해 창의적인 생각이 필요할 수 있다. 가능한 한 `subtype_diff` 함수는 선택한 연산자 클래스와 정렬 규칙에 의해 암시되는 정렬 순서와 일치해야 한다. 즉, 첫 번째 인수가 두 번째 인수보다 크면 결과가 양수여야 한다.

`subtype_diff` 함수의 덜 단순화된 예는 다음과 같다:

```
CREATE FUNCTION time_subtype_diff(x time, y time) RETURNS float8 AS
'SELECT EXTRACT(EPOCH FROM (x - y))' LANGUAGE sql STRICT IMMUTABLE;

CREATE TYPE timerange AS RANGE (
    subtype = time,
    subtype_diff = time_subtype_diff
);

SELECT '[11:10, 23:00]'::timerange;
```

범위 타입 생성에 대한 더 많은 정보는 [CREATE TYPE](https://www.postgresql.org/docs/current/sql-createtype.html "CREATE TYPE")을 참조한다.


### 8.17.9. 인덱싱

범위 타입(range types)을 가진 테이블 컬럼에 대해 GiST와 SP-GiST 인덱스를 생성할 수 있다. 또한, 멀티범위 타입(multirange types)을 가진 테이블 컬럼에도 GiST 인덱스를 생성할 수 있다. 예를 들어, GiST 인덱스를 생성하려면 다음과 같이 작성한다:

```
CREATE INDEX reservation_idx ON reservation USING GIST (during);
```

범위에 대한 GiST 또는 SP-GiST 인덱스는 `=`, `&&`, `<@`, `@>`, `<<`, `>>`, `-|-`, `&<`, `&>`와 같은 범위 연산자를 포함한 쿼리의 성능을 향상시킬 수 있다. 멀티범위에 대한 GiST 인덱스도 동일한 멀티범위 연산자를 사용하는 쿼리의 성능을 높일 수 있다. 또한, 범위에 대한 GiST 인덱스와 멀티범위에 대한 GiST 인덱스는 범위와 멀티범위 간의 교차 타입 연산자인 `&&`, `<@`, `@>`, `<<`, `>>`, `-|-`, `&<`, `&>`를 포함한 쿼리의 성능을 개선할 수 있다. 더 자세한 정보는 [표 9.56](https://www.postgresql.org/docs/current/functions-range.html#RANGE-OPERATORS-TABLE "표 9.56. 범위 연산자")을 참고한다.

또한, 범위 타입을 가진 테이블 컬럼에 대해 B-tree와 해시 인덱스를 생성할 수도 있다. 이러한 인덱스 타입의 경우, 기본적으로 유용한 범위 연산자는 동등 연산자(`=`)뿐이다. 범위 값에 대해 정의된 B-tree 정렬 순서가 있으며, 이에 대응하는 `<`와 `>` 연산자가 있지만, 이 순서는 다소 임의적이며 실제로는 거의 유용하지 않다. 범위 타입의 B-tree와 해시 지원은 주로 쿼리 내부에서 정렬과 해싱을 가능하게 하기 위한 것이지, 실제 인덱스를 생성하기 위한 것은 아니다.


### 8.17.10. 범위 타입의 제약 조건

스칼라 값에는 `UNIQUE` 제약 조건이 적합하지만, 범위 타입에는 일반적으로 적합하지 않다. 대신 배제 제약 조건(exclusion constraint)이 더 적절한 경우가 많다([CREATE TABLE ... CONSTRAINT ... EXCLUDE](https://www.postgresql.org/docs/current/sql-createtable.html#SQL-CREATETABLE-EXCLUDE) 참조). 배제 제약 조건을 사용하면 범위 타입에 대해 "겹치지 않음"과 같은 제약 조건을 지정할 수 있다. 예를 들어:

```
CREATE TABLE reservation (
    during tsrange,
    EXCLUDE USING GIST (during WITH &&)
);
```

이 제약 조건은 테이블에 동시에 겹치는 값이 존재하지 않도록 방지한다:

```
INSERT INTO reservation VALUES
    ('[2010-01-01 11:30, 2010-01-01 15:00)');
INSERT 0 1

INSERT INTO reservation VALUES
    ('[2010-01-01 14:45, 2010-01-01 15:45)');
ERROR:  conflicting key value violates exclusion constraint "reservation_during_excl"
DETAIL:  Key (during)=(["2010-01-01 14:45:00","2010-01-01 15:45:00")) conflicts
with existing key (during)=(["2010-01-01 11:30:00","2010-01-01 15:00:00")).
```

[`btree_gist`](https://www.postgresql.org/docs/current/btree-gist.html "F.8. btree_gist — GiST operator classes with B-tree behavior") 확장 기능을 사용하면 일반 스칼라 데이터 타입에 대해 배제 제약 조건을 정의할 수 있다. 이를 범위 배제와 결합하면 더 큰 유연성을 얻을 수 있다. 예를 들어, `btree_gist`를 설치한 후 다음 제약 조건은 회의실 번호가 동일한 경우에만 겹치는 범위를 거부한다:

```
CREATE EXTENSION btree_gist;
CREATE TABLE room_reservation (
    room text,
    during tsrange,
    EXCLUDE USING GIST (room WITH =, during WITH &&)
);

INSERT INTO room_reservation VALUES
    ('123A', '[2010-01-01 14:00, 2010-01-01 15:00)');
INSERT 0 1

INSERT INTO room_reservation VALUES
    ('123A', '[2010-01-01 14:30, 2010-01-01 15:30)');
ERROR:  conflicting key value violates exclusion constraint "room_reservation_room_during_excl"
DETAIL:  Key (room, during)=(123A, ["2010-01-01 14:30:00","2010-01-01 15:30:00")) conflicts
with existing key (room, during)=(123A, ["2010-01-01 14:00:00","2010-01-01 15:00:00")).

INSERT INTO room_reservation VALUES
    ('123B', '[2010-01-01 14:30, 2010-01-01 15:30)');
INSERT 0 1
```




## 8.18. 도메인 타입

도메인은 사용자가 정의한 데이터 타입으로, 다른 *기본 타입*을 기반으로 한다. 선택적으로, 도메인은 기본 타입이 허용하는 값의 일부만 유효한 값으로 제한하는 조건을 가질 수 있다. 그렇지 않으면 도메인은 기본 타입과 동일하게 동작한다. 예를 들어, 기본 타입에 적용할 수 있는 모든 연산자나 함수는 도메인 타입에도 적용된다. 기본 타입은 내장 타입이나 사용자 정의 기본 타입, 열거형 타입, 배열 타입, 복합 타입, 범위 타입, 또는 다른 도메인이 될 수 있다.

예를 들어, 양의 정수만 허용하는 도메인을 정수 타입 위에 생성할 수 있다:

```
CREATE DOMAIN posint AS integer CHECK (VALUE > 0);
CREATE TABLE mytable (id posint);
INSERT INTO mytable VALUES(1);   -- 성공
INSERT INTO mytable VALUES(-1);  -- 실패
```

기본 타입의 연산자나 함수가 도메인 값에 적용되면, 도메인은 자동으로 기본 타입으로 다운캐스트된다. 따라서 예를 들어, `mytable.id - 1`의 결과는 `posint` 타입이 아니라 `integer` 타입으로 간주된다. 결과를 다시 `posint`로 캐스트하려면 `(mytable.id - 1)::posint`와 같이 작성할 수 있으며, 이 경우 도메인의 조건이 다시 검사된다. 이 예제에서, `id` 값이 1인 경우에는 오류가 발생할 것이다. 기본 타입의 값을 도메인 타입의 필드나 변수에 할당할 때는 명시적인 캐스트 없이도 허용되지만, 도메인의 조건은 검사된다.

추가 정보는 [CREATE DOMAIN](https://www.postgresql.org/docs/current/sql-createdomain.html "CREATE DOMAIN")을 참고한다.




## 8.19. 객체 식별자 타입

객체 식별자(OID)는 PostgreSQL에서 다양한 시스템 테이블의 기본 키로 내부적으로 사용된다. `oid` 타입은 객체 식별자를 나타낸다. 또한 `oid`에 대한 여러 별칭 타입이 있으며, 각각의 이름은 ``reg*`something`*`` 형식이다. [표 8.26](https://www.postgresql.org/docs/current/datatype-oid.html#DATATYPE-OID-TABLE "표 8.26. 객체 식별자 타입")에서 개요를 확인할 수 있다.

`oid` 타입은 현재 부호 없는 4바이트 정수로 구현되어 있다. 따라서 대규모 데이터베이스나 개별 테이블에서 데이터베이스 전반에 걸쳐 고유성을 보장하기에는 크기가 충분하지 않다.

`oid` 타입 자체는 비교 연산 외에 별다른 연산을 제공하지 않는다. 그러나 정수로 캐스팅한 후에는 표준 정수 연산자를 사용해 조작할 수 있다. (이때 부호 있는 정수와 부호 없는 정수 간의 혼동에 주의해야 한다.)

OID 별칭 타입은 특수화된 입력 및 출력 루틴 외에는 자체적인 연산을 제공하지 않는다. 이러한 루틴은 시스템 객체에 대한 숫자 값 대신 심볼릭 이름을 입력하고 표시할 수 있다. 별칭 타입을 사용하면 객체의 OID 값을 더 간단하게 조회할 수 있다. 예를 들어, `mytable` 테이블과 관련된 `pg_attribute` 행을 조회하려면 다음과 같이 작성할 수 있다:

```
SELECT * FROM pg_attribute WHERE attrelid = 'mytable'::regclass;
```

이것은 다음과 같은 쿼리보다 간단하다:

```
SELECT * FROM pg_attribute
  WHERE attrelid = (SELECT oid FROM pg_class WHERE relname = 'mytable');
```

이 쿼리 자체는 나쁘지 않아 보이지만, 여전히 단순화된 형태이다. 서로 다른 스키마에 동일한 이름의 테이블이 여러 개 있는 경우, 올바른 OID를 선택하려면 훨씬 더 복잡한 하위 쿼리가 필요하다. `regclass` 입력 변환기는 스키마 경로 설정에 따라 테이블 조회를 처리하므로, 자동으로 "올바른 작업"을 수행한다. 마찬가지로, 테이블의 OID를 `regclass`로 캐스팅하면 숫자 OID를 심볼릭으로 표시하는 데 유용하다.

**표 8.26. 객체 식별자 타입**

| 이름 | 참조 | 설명 | 값 예시 |
| --- | --- | --- | --- |
| oid | 모든 | 숫자 객체 식별자 | 564182 |
| regclass | pg_class | 관계 이름 | pg_type |
| regcollation | pg_collation | 정렬 규칙 이름 | "POSIX" |
| regconfig | pg_ts_config | 텍스트 검색 구성 | english |
| regdictionary | pg_ts_dict | 텍스트 검색 사전 | simple |
| regnamespace | pg_namespace | 네임스페이스 이름 | pg_catalog |
| regoper | pg_operator | 연산자 이름 | + |
| regoperator | pg_operator | 인자 타입이 있는 연산자 | *(integer,​integer) 또는 -(NONE,​integer) |
| regproc | pg_proc | 함수 이름 | sum |
| regprocedure | pg_proc | 인자 타입이 있는 함수 | sum(int4) |
| regrole | pg_authid | 역할 이름 | smithee |
| regtype | pg_type | 데이터 타입 이름 | integer |

네임스페이스로 그룹화된 객체에 대한 모든 OID 별칭 타입은 스키마 한정 이름을 입력으로 받아들이며, 현재 검색 경로에서 객체를 찾을 수 없는 경우 출력 시 스키마 한정 이름을 표시한다. 예를 들어, `myschema.mytable`은 `regclass`에 대한 유효한 입력이다(해당 테이블이 존재하는 경우). 이 값은 현재 검색 경로에 따라 `myschema.mytable` 또는 단순히 `mytable`로 출력될 수 있다. `regproc` 및 `regoper` 별칭 타입은 고유한(오버로드되지 않은) 이름만 입력으로 받아들이므로 사용이 제한적이다. 대부분의 경우 `regprocedure` 또는 `regoperator`가 더 적합하다. `regoperator`의 경우, 단항 연산자는 사용되지 않는 피연산자에 `NONE`을 작성하여 식별한다.

이러한 타입의 입력 함수는 토큰 사이에 공백을 허용하며, 큰따옴표 안에 있는 경우를 제외하고 대문자를 소문자로 변환한다. 이는 SQL에서 객체 이름을 작성하는 방식과 유사한 구문 규칙을 만들기 위함이다. 반대로, 출력 함수는 출력이 유효한 SQL 식별자가 되도록 필요한 경우 큰따옴표를 사용한다. 예를 들어, 두 개의 정수 인자를 받는 `Foo`(대문자 `F`)라는 함수의 OID는 `' "Foo" ( int, integer ) '::regprocedure`로 입력될 수 있다. 출력은 `"Foo"(integer,integer)`와 같이 표시된다. 함수 이름과 인자 타입 이름 모두 스키마 한정 이름일 수도 있다.

PostgreSQL의 많은 내장 함수는 테이블이나 다른 종류의 데이터베이스 객체의 OID를 입력으로 받으며, 편의를 위해 `regclass`(또는 적절한 OID 별칭 타입)을 받도록 선언된다. 이는 객체의 OID를 수동으로 조회할 필요 없이 이름을 문자열 리터럴로 입력할 수 있음을 의미한다. 예를 들어, `nextval(regclass)` 함수는 시퀀스 관계의 OID를 받으므로 다음과 같이 호출할 수 있다:

```
foo
```


### 참고

함수의 인자로 단순한 문자열 리터럴을 사용하면, 이는 `regclass` 타입(또는 적절한 타입)의 상수로 간주된다. 이 값은 실제로 OID이므로, 이후에 이름이 변경되거나 스키마가 재할당되더라도 원래 식별된 객체를 계속 추적한다. 이러한 "초기 바인딩" 동작은 일반적으로 컬럼 기본값이나 뷰에서 객체를 참조할 때 바람직하다. 하지만 때로는 런타임에 객체 참조를 해결하는 "늦은 바인딩"이 필요할 수 있다. 늦은 바인딩 동작을 원한다면, 상수를 `regclass`가 아닌 `text` 상수로 저장하도록 강제하면 된다.

```
foo
```

`to_regclass()` 함수와 그 유사 함수들은 런타임 조회를 수행하는 데에도 사용할 수 있다. 자세한 내용은 [표 9.74](https://www.postgresql.org/docs/current/functions-info.html#FUNCTIONS-INFO-CATALOG-TABLE "표 9.74. 시스템 카탈로그 정보 함수")를 참고한다.

`regclass`를 사용하는 또 다른 실용적인 예는 `information_schema` 뷰에 나열된 테이블의 OID를 조회하는 것이다. 이 뷰는 직접적으로 OID를 제공하지 않는다. 예를 들어, 테이블 OID가 필요한 `pg_relation_size()` 함수를 호출하고자 할 때, 위 규칙을 고려하면 올바른 방법은 다음과 같다.

```
SELECT table_schema, table_name,
       pg_relation_size((quote_ident(table_schema) || '.' ||
                         quote_ident(table_name))::regclass)
FROM information_schema.tables
WHERE ...
```

`quote_ident()` 함수는 필요한 경우 식별자를 이중 따옴표로 감싸준다. 다음처럼 간단히 보이는 방법은

```
SELECT pg_relation_size(table_name)
FROM information_schema.tables
WHERE ...
```

**추천하지 않는다**. 이 방법은 검색 경로 밖에 있거나 따옴표가 필요한 이름을 가진 테이블에서 실패할 수 있기 때문이다.

대부분의 OID 별칭 타입은 의존성을 생성한다는 추가적인 특성을 가진다. 저장된 표현식(예: 컬럼 기본값 표현식이나 뷰)에 이러한 타입의 상수가 나타나면, 참조된 객체에 대한 의존성이 생성된다. 예를 들어, 컬럼에 `nextval('my_seq'::regclass)`라는 기본값 표현식이 있다면, PostgreSQL은 이 기본값 표현식이 시퀀스 `my_seq`에 의존한다는 것을 이해한다. 따라서 시스템은 기본값 표현식을 제거하지 않고는 시퀀스를 삭제할 수 없게 된다. 반면 `nextval('my_seq'::text)`는 의존성을 생성하지 않는다. (`regrole` 타입은 이 특성의 예외다. 이 타입의 상수는 저장된 표현식에서 허용되지 않는다.)

시스템에서 사용하는 또 다른 식별자 타입은 `xid`, 즉 트랜잭션 식별자(줄여서 xact)이다. 이는 시스템 컬럼 `xmin`과 `xmax`의 데이터 타입이다. 트랜잭션 식별자는 32비트 값이다. 일부 컨텍스트에서는 64비트 변형인 `xid8`이 사용된다. `xid` 값과 달리, `xid8` 값은 엄격하게 단조 증가하며 데이터베이스 클러스터의 수명 동안 재사용되지 않는다. 자세한 내용은 [66.1절](https://www.postgresql.org/docs/current/transaction-id.html "66.1. 트랜잭션과 식별자")을 참고한다.

시스템에서 사용하는 세 번째 식별자 타입은 `cid`, 즉 커맨드 식별자이다. 이는 시스템 컬럼 `cmin`과 `cmax`의 데이터 타입이다. 커맨드 식별자 역시 32비트 값이다.

시스템에서 사용하는 마지막 식별자 타입은 `tid`, 즉 튜플 식별자(행 식별자)이다. 이는 시스템 컬럼 `ctid`의 데이터 타입이다. 튜플 ID는 (블록 번호, 블록 내 튜플 인덱스) 쌍으로, 테이블 내에서 행의 물리적 위치를 식별한다.

(시스템 컬럼에 대한 자세한 설명은 [5.6절](https://www.postgresql.org/docs/current/ddl-system-columns.html "5.6. 시스템 컬럼")을 참고한다.)




## 8.20. pg_lsn 타입

`pg_lsn` 데이터 타입은 WAL(Write-Ahead Log) 내 위치를 가리키는 LSN(Log Sequence Number) 데이터를 저장하는 데 사용된다. 이 타입은 PostgreSQL의 내부 시스템 타입인 `XLogRecPtr`를 표현한다.

내부적으로 LSN은 64비트 정수로, WAL 스트림 내 바이트 위치를 나타낸다. 이 값은 슬래시로 구분된 최대 8자리의 두 개의 16진수로 출력된다. 예를 들어 `16/B374D848`과 같은 형태다. `pg_lsn` 타입은 `=`와 `>`와 같은 표준 비교 연산자를 지원한다. 두 LSN은 `-` 연산자를 사용해 뺄 수 있으며, 그 결과는 두 WAL 위치 사이의 바이트 수가 된다. 또한 `+(pg_lsn,numeric)`과 `-(pg_lsn,numeric)` 연산자를 사용해 LSN에 바이트 수를 더하거나 뺄 수 있다. 단, 계산된 LSN은 `pg_lsn` 타입의 범위 내에 있어야 한다. 즉, `0/0`부터 `FFFFFFFF/FFFFFFFF` 사이여야 한다.




## 8.21. 의사 타입

PostgreSQL 타입 시스템에는 *의사 타입*이라고 불리는 특수 목적의 항목들이 있다. 의사 타입은 컬럼 데이터 타입으로 사용할 수 없지만, 함수의 인자나 결과 타입을 선언할 때 사용할 수 있다. 각 의사 타입은 함수의 동작이 특정 SQL 데이터 타입의 값을 단순히 받거나 반환하는 것과는 다른 경우에 유용하게 활용된다. [표 8.27](https://www.postgresql.org/docs/current/datatype-pseudo.html#DATATYPE-PSEUDOTYPES-TABLE "표 8.27. 의사 타입")은 현재 존재하는 의사 타입 목록을 보여준다.

**표 8.27. 의사 타입**

| 이름 | 설명 |
| --- | --- |
| any | 함수가 모든 입력 데이터 타입을 허용함을 나타낸다. |
| anyelement | 함수가 모든 데이터 타입을 허용함을 나타낸다 (36.2.5절 참조). |
| anyarray | 함수가 모든 배열 데이터 타입을 허용함을 나타낸다 (36.2.5절 참조). |
| anynonarray | 함수가 모든 비배열 데이터 타입을 허용함을 나타낸다 (36.2.5절 참조). |
| anyenum | 함수가 모든 열거형 데이터 타입을 허용함을 나타낸다 (36.2.5절과 8.7절 참조). |
| anyrange | 함수가 모든 범위 데이터 타입을 허용함을 나타낸다 (36.2.5절과 8.17절 참조). |
| anymultirange | 함수가 모든 다중 범위 데이터 타입을 허용함을 나타낸다 (36.2.5절과 8.17절 참조). |
| anycompatible | 함수가 모든 데이터 타입을 허용하며, 여러 인자를 공통 데이터 타입으로 자동 변환함을 나타낸다 (36.2.5절 참조). |
| anycompatiblearray | 함수가 모든 배열 데이터 타입을 허용하며, 여러 인자를 공통 데이터 타입으로 자동 변환함을 나타낸다 (36.2.5절 참조). |
| anycompatiblenonarray | 함수가 모든 비배열 데이터 타입을 허용하며, 여러 인자를 공통 데이터 타입으로 자동 변환함을 나타낸다 (36.2.5절 참조). |
| anycompatiblerange | 함수가 모든 범위 데이터 타입을 허용하며, 여러 인자를 공통 데이터 타입으로 자동 변환함을 나타낸다 (36.2.5절과 8.17절 참조). |
| anycompatiblemultirange | 함수가 모든 다중 범위 데이터 타입을 허용하며, 여러 인자를 공통 데이터 타입으로 자동 변환함을 나타낸다 (36.2.5절과 8.17절 참조). |
| cstring | 함수가 널 종료 C 문자열을 받거나 반환함을 나타낸다. |
| internal | 함수가 서버 내부 데이터 타입을 받거나 반환함을 나타낸다. |
| language_handler | 프로시저 언어 호출 핸들러는 language_handler를 반환하도록 선언된다. |
| fdw_handler | 외부 데이터 래퍼 핸들러는 fdw_handler를 반환하도록 선언된다. |
| table_am_handler | 테이블 접근 방법 핸들러는 table_am_handler를 반환하도록 선언된다. |
| index_am_handler | 인덱스 접근 방법 핸들러는 index_am_handler를 반환하도록 선언된다. |
| tsm_handler | 테이블 샘플링 방법 핸들러는 tsm_handler를 반환하도록 선언된다. |
| record | 함수가 지정되지 않은 행 타입을 받거나 반환함을 나타낸다. |
| trigger | 트리거 함수는 trigger를 반환하도록 선언된다. |
| event_trigger | 이벤트 트리거 함수는 event_trigger를 반환하도록 선언된다. |
| pg_ddl_command | 이벤트 트리거에서 사용할 수 있는 DDL 명령의 표현을 나타낸다. |
| void | 함수가 값을 반환하지 않음을 나타낸다. |
| unknown | 아직 해결되지 않은 타입을 나타낸다. 예를 들어, 데코레이션되지 않은 문자열 리터럴의 타입이다. |

C로 작성된 함수(내장 함수나 동적으로 로드된 함수 모두)는 이러한 의사 타입을 받거나 반환하도록 선언할 수 있다. 함수 작성자는 의사 타입이 인자 타입으로 사용될 때 함수가 안전하게 동작하도록 보장해야 한다.

프로시저 언어로 작성된 함수는 해당 언어의 구현에 따라 의사 타입을 사용할 수 있다. 현재 대부분의 프로시저 언어는 의사 타입을 인자 타입으로 사용하는 것을 금지하며, 결과 타입으로는 `void`와 `record`만 허용한다(함수가 트리거나 이벤트 트리거로 사용될 때는 `trigger`나 `event_trigger`도 허용). 일부 언어는 위에 나열된 다형성 의사 타입을 사용한 다형성 함수도 지원한다. 이에 대한 자세한 내용은 [36.2.5절](https://www.postgresql.org/docs/current/extend-type-system.html#EXTEND-TYPES-POLYMORPHIC "36.2.5. 다형성 타입")에서 다룬다.

`internal` 의사 타입은 데이터베이스 시스템 내부에서만 호출되도록 의도된 함수를 선언할 때 사용한다. SQL 쿼리에서 직접 호출할 수 없다. 함수가 하나 이상의 `internal` 타입 인자를 가지면 SQL에서 호출할 수 없다. 이 제한의 타입 안전성을 보장하려면 다음과 같은 코딩 규칙을 준수해야 한다: `internal` 타입 인자가 하나 이상 없는 한 `internal`을 반환하도록 선언된 함수를 만들지 않는다.


