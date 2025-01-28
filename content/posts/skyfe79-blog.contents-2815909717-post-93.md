---
title: "PostgreSQL 튜토리얼 - SQL 입문"
date: 2025-01-28T14:50:25Z
author: "skyfe79"
draft: false
tags: ["postgresql"]
---

## 2.1. SQL 입문

이 장에서는 SQL을 사용하여 기본적인 데이터베이스 작업을 수행하는 방법을 설명한다. 이 튜토리얼은 SQL의 기초 개념만을 다루며, SQL의 모든 내용을 포함하지는 않는다. SQL에 대한 더 자세한 내용은 [[melt93]](https://www.postgresql.org/docs/17/biblio.html#MELT93 "새로운 SQL의 이해")과 [[date97]](https://www.postgresql.org/docs/17/biblio.html#DATE97 "SQL 표준 가이드")와 같은 전문 서적을 참고한다. 또한 PostgreSQL은 표준 SQL에 없는 독자적인 기능을 포함한다는 점에 유의한다.

이어지는 예제들은 이전 장에서 설명한 대로 `mydb`라는 데이터베이스를 생성하고 psql을 실행할 수 있다는 전제 하에 진행한다.

PostgreSQL 소스 배포판의 `src/tutorial/` 디렉토리에서도 이 설명서의 예제를 찾을 수 있다. (단, PostgreSQL 바이너리 배포판에는 이 파일들이 포함되지 않을 수 있다.) 해당 파일들을 사용하려면 먼저 디렉토리로 이동한 후 다음과 같이 make 명령을 실행한다:

```bash
$ cd .../src/tutorial
$ make
```

이 명령은 스크립트를 생성하고 사용자 정의 함수와 타입이 포함된 C 파일을 컴파일한다. 그런 다음 튜토리얼을 시작하려면 다음 단계를 수행한다:

```bash
$ psql -s mydb
...
mydb=> i basics.sql
```

`i` 명령은 지정된 파일의 명령을 읽어들인다. `psql`의 `-s` 옵션을 사용하면 단계별 모드가 활성화되어 각 명령을 서버로 보내기 전에 일시 중지한다. 이 장에서 사용하는 모든 명령은 `basics.sql` 파일에 포함되어 있다.

## 2.2. 기본 개념

PostgreSQL은 관계형 데이터베이스 관리 시스템(RDBMS, Relational Database Management System)이다. 여기서 '관계형'이란 데이터를 '관계' 즉, 수학적 용어로 '테이블'에 저장하고 관리한다는 의미다. 오늘날 테이블을 사용해 데이터를 저장하는 방식은 매우 일반적이지만, 데이터베이스를 구성하는 다른 방식도 있다. 예를 들어, 유닉스 계열 운영체제의 파일과 디렉터리 구조는 계층형 데이터베이스의 한 예시다. 최근에는 객체 지향 데이터베이스라는 새로운 방식도 등장했다.

각 테이블은 이름이 있는 행(row)의 모음이다. 특정 테이블의 모든 행은 동일한 이름을 가진 열(column)로 구성되며, 각 열은 특정한 데이터 타입을 갖는다. 열은 각 행에서 고정된 순서를 가지지만, SQL은 테이블 내 행의 순서를 보장하지 않는다는 점을 기억해야 한다(단, 표시할 때는 명시적으로 정렬할 수 있다).

테이블은 데이터베이스로 묶이며, 하나의 PostgreSQL 서버 인스턴스가 관리하는 데이터베이스 모음을 데이터베이스 클러스터(cluster)라고 한다.

## 2.3. 새로운 테이블 생성하기

테이블을 새로 만들 때는 테이블 이름과 함께 모든 컬럼의 이름과 타입을 지정한다:

```sql
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,           -- 최저 기온
    temp_hi         int,           -- 최고 기온
    prcp            real,          -- 강수량
    date            date
);
```

이 명령문을 `psql`에 입력할 때 줄 바꿈을 포함해도 된다. `psql`은 세미콜론이 나올 때까지 명령이 끝나지 않았다고 인식한다.

SQL 명령문에서는 공백(스페이스, 탭, 줄바꿈)을 자유롭게 사용할 수 있다. 따라서 위 명령문을 다르게 정렬하거나 한 줄로 입력해도 된다. 두 개의 대시(`--`)는 주석을 의미한다. 대시 이후부터 해당 줄 끝까지의 내용은 무시된다. SQL은 식별자를 큰따옴표로 묶어 대소문자를 구분하도록 하지 않는 한(위 예제에서는 사용하지 않음), 키워드와 식별자의 대소문자를 구분하지 않는다.

`varchar(80)`은 최대 80자까지 저장할 수 있는 임의의 문자열 데이터 타입을 지정한다. `int`는 일반적인 정수 타입이다. `real`은 단정밀도 부동소수점 숫자를 저장하는 타입이다. `date`는 이름 그대로 날짜를 저장한다. (참고로, `date` 타입의 컬럼 이름도 `date`로 지정했다. 이는 상황에 따라 편리할 수도, 혼란스러울 수도 있다.)

PostgreSQL은 표준 SQL 타입인 `int`, `smallint`, `real`, `double precision`, `char(N)`, `varchar(N)`, `date`, `time`, `timestamp`, `interval`을 지원한다. 또한 다양한 일반 유틸리티 타입과 풍부한 기하학적 타입도 제공한다. PostgreSQL은 사용자 정의 데이터 타입을 무제한으로 추가할 수 있도록 확장 가능하다. 따라서 SQL 표준에서 특별한 경우를 지원하기 위해 필요한 경우를 제외하고는 타입 이름이 문법상 키워드가 아니다.

두 번째 예제는 도시와 해당 도시의 지리적 위치를 저장하는 테이블이다:

```sql
CREATE TABLE cities (
    name            varchar(80),
    location        point
);
```

`point` 타입은 PostgreSQL에서만 제공하는 특별한 데이터 타입의 예시이다.

마지막으로, 더 이상 필요하지 않은 테이블을 삭제하거나 다르게 재생성하고 싶을 때는 다음 명령어를 사용한다:

```sql
DROP TABLE tablename;
```

## 2.4. 테이블에 행 데이터 추가하기

테이블에 행을 추가할 때는 `INSERT` 문을 사용한다:

```sql
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```

모든 데이터 타입은 직관적인 입력 형식을 사용한다. 단순 숫자가 아닌 상수는 위 예제처럼 반드시 작은따옴표(`'`)로 묶어야 한다. date 타입은 다양한 형식을 받아들이지만, 이 튜토리얼에서는 명확성을 위해 위와 같은 형식만 사용한다.

point 타입은 아래 예제와 같이 좌표쌍을 입력값으로 받는다:

```sql
INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');
```

지금까지 본 문법은 컬럼 순서를 기억해야 하는 단점이 있다. 다음과 같이 컬럼을 명시적으로 나열하는 대체 문법도 있다:

```sql
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

원하는 경우 컬럼 순서를 다르게 지정할 수 있으며, 강수량처럼 알 수 없는 값이 있다면 해당 컬럼을 생략할 수도 있다:

```sql
INSERT INTO weather (date, city, temp_hi, temp_lo)
    VALUES ('1994-11-29', 'Hayward', 54, 37);
```

많은 개발자는 암묵적인 순서에 의존하는 것보다 컬럼을 명시적으로 나열하는 방식을 더 좋은 스타일로 본다.

다음 섹션에서 사용할 데이터를 준비하기 위해 위의 모든 명령어를 실행해보자.

대량의 데이터를 텍스트 파일에서 로드할 때는 `COPY` 명령어를 사용할 수 있다. `COPY` 명령어는 이러한 용도에 최적화되어 있어 `INSERT`보다 빠르지만, 유연성은 떨어진다. 사용 예는 다음과 같다:

```sql
COPY weather FROM '/home/user/weather.txt';
```

소스 파일은 백엔드 프로세스가 직접 읽기 때문에 클라이언트가 아닌 백엔드 프로세스가 실행되는 머신에서 접근 가능해야 한다. `COPY` 명령어에 대한 자세한 내용은 [COPY](https://www.postgresql.org/docs/17/sql-copy.html "COPY") 문서에서 확인할 수 있다.

## 2.5. 테이블 쿼리하기

테이블에서 데이터를 가져오려면 쿼리를 실행해야 한다. 이를 위해 SQL `SELECT` 문을 사용한다. SELECT 문은 세 부분으로 구성된다:
- 선택 목록 (반환할 컬럼을 나열하는 부분)
- 테이블 목록 (데이터를 가져올 테이블을 나열하는 부분)
- 선택적 조건절 (데이터 제한 조건을 지정하는 부분)

예를 들어, `weather` 테이블의 모든 행을 가져오려면 다음과 같이 입력한다:

```sql
SELECT * FROM weather;
```

여기서 `*`는 "모든 컬럼"을 의미하는 약자다. 따라서 아래 쿼리와 동일한 결과를 얻을 수 있다:

```sql
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```

실행 결과는 다음과 같다:

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29
(3 rows)
```

선택 목록에는 단순한 컬럼 참조뿐만 아니라 표현식도 작성할 수 있다. 예를 들면:

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

실행 결과:

```
     city      | temp_avg |    date
---------------+----------+------------
 San Francisco |       48 | 1994-11-27
 San Francisco |       50 | 1994-11-29
 Hayward       |       45 | 1994-11-29
(3 rows)
```

위 예제에서 `AS` 절은 출력 컬럼의 이름을 바꾸는 데 사용된다. `AS` 절은 선택사항이다.

`WHERE` 절을 추가하면 원하는 행만 선택적으로 가져올 수 있다. `WHERE` 절은 불리언(참/거짓) 표현식을 포함하며, 이 표현식이 참인 행만 반환된다. 일반적인 불리언 연산자(`AND`, `OR`, `NOT`)를 조건절에서 사용할 수 있다. 예를 들어, 다음 쿼리는 비가 오는 날의 샌프란시스코 날씨를 가져온다:

```sql
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;
```

실행 결과:

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
(1 row)
```

쿼리 결과를 정렬된 순서로 받을 수도 있다:

```sql
SELECT * FROM weather
    ORDER BY city;
```

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 Hayward       |      37 |      54 |      | 1994-11-29
 San Francisco |      43 |      57 |    0 | 1994-11-29
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
```

이 예제에서는 정렬 순서가 완전히 지정되지 않았기 때문에 샌프란시스코 행들이 어떤 순서로도 나타날 수 있다. 하지만 다음과 같이 하면 항상 위와 같은 결과를 얻을 수 있다:

```sql
SELECT * FROM weather
    ORDER BY city, temp_lo;
```

쿼리 결과에서 중복된 행을 제거할 수도 있다:

```sql
SELECT DISTINCT city
    FROM weather;
```

```
     city
---------------
 Hayward
 San Francisco
(2 rows)
```

여기서도 결과 행의 순서는 달라질 수 있다. `DISTINCT`와 `ORDER BY`를 함께 사용하면 일관된 결과를 보장할 수 있다:

```sql
SELECT DISTINCT city
    FROM weather
    ORDER BY city;
```

## 2.6. 테이블 간의 조인

지금까지는 한 번에 하나의 테이블만 조회하는 쿼리를 다뤘다. 하지만 여러 테이블을 동시에 조회하거나, 같은 테이블의 여러 행을 동시에 처리하는 쿼리도 작성할 수 있다. 여러 테이블(또는 같은 테이블의 여러 인스턴스)을 동시에 조회하는 쿼리를 *조인(join) 쿼리*라고 한다. 조인은 하나의 테이블에서 행을 가져와 다른 테이블의 행과 결합하는데, 이때 어떤 행을 짝지을지 결정하는 표현식을 사용한다. 예를 들어, 날씨 기록과 해당 도시의 위치 정보를 함께 조회하려면 데이터베이스는 `weather` 테이블의 각 행의 `city` 컬럼을 `cities` 테이블의 모든 행의 `name` 컬럼과 비교하여 이 값들이 일치하는 행들을 선택해야 한다. 다음 쿼리로 이를 수행할 수 있다:

```sql
SELECT * FROM weather JOIN cities ON city = name;
```

```sql
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(2 rows)
```

결과를 보면 두 가지 특징이 있다:

* Hayward 도시의 결과 행이 없다. 이는 `cities` 테이블에 Hayward와 일치하는 항목이 없어서 조인이 `weather` 테이블의 해당 행을 무시했기 때문이다. 이 문제는 곧 해결 방법을 알아볼 것이다.

* 도시 이름을 포함하는 컬럼이 두 개 있다. 이는 `weather`와 `cities` 테이블의 컬럼 목록이 연결되었기 때문에 정상이다. 하지만 실제로는 이렇게 중복된 정보를 보여주는 것이 바람직하지 않으므로, `*` 대신 출력할 컬럼을 명시적으로 나열하는 것이 좋다:

```sql
SELECT city, temp_lo, temp_hi, prcp, date, location
    FROM weather JOIN cities ON city = name;
```

모든 컬럼의 이름이 서로 달랐기 때문에 파서가 자동으로 각 컬럼이 어느 테이블에 속하는지 찾아냈다. 두 테이블에 같은 이름의 컬럼이 있다면 다음과 같이 컬럼 이름을 *한정*하여 어느 것을 의미하는지 명시해야 한다:

```sql
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.location
    FROM weather JOIN cities ON weather.city = cities.name;
```

조인 쿼리에서는 모든 컬럼 이름을 한정하는 것이 좋은 스타일로 여겨진다. 이렇게 하면 나중에 테이블 중 하나에 중복된 컬럼 이름이 추가되더라도 쿼리가 실패하지 않는다.

지금까지 본 종류의 조인 쿼리는 다음과 같은 형식으로도 작성할 수 있다:

```sql
SELECT *
    FROM weather, cities
    WHERE city = name;
```

이 문법은 SQL-92에서 도입된 `JOIN`/`ON` 문법보다 이전의 것이다. 테이블들을 단순히 `FROM` 절에 나열하고, 비교 표현식을 `WHERE` 절에 추가한다. 이전의 암시적 문법과 새로운 명시적 `JOIN`/`ON` 문법의 결과는 동일하다. 하지만 쿼리를 읽는 사람의 입장에서는 명시적 문법이 의미를 이해하기 더 쉽다. 조인 조건이 자체 키워드로 도입되는 반면, 이전에는 조건이 다른 조건들과 함께 `WHERE` 절에 섞여 있었기 때문이다.

이제 Hayward 기록을 어떻게 가져올 수 있는지 알아보자. 우리가 원하는 것은 `weather` 테이블을 검색하여 각 행에 대해 일치하는 `cities` 행을 찾되, 일치하는 행이 없을 경우 `cities` 테이블의 컬럼에 "빈 값"을 대신 넣는 것이다. 이러한 종류의 쿼리를 *외부 조인(outer join)*이라고 한다. (지금까지 본 조인은 *내부 조인(inner join)*이었다.) 명령은 다음과 같다:

```sql
SELECT *
    FROM weather LEFT OUTER JOIN cities ON weather.city = cities.name;
```

```sql
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location
---------------+---------+---------+------+------------+---------------+-----------
 Hayward       |      37 |      54 |      | 1994-11-29 |               |
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(3 rows)
```

이 쿼리를 *왼쪽 외부 조인*이라고 부르는 이유는 조인 연산자의 왼쪽에 있는 테이블의 모든 행이 적어도 한 번은 출력되는 반면, 오른쪽 테이블은 왼쪽 테이블의 행과 일치하는 행만 출력되기 때문이다. 왼쪽 테이블의 행에 대응하는 오른쪽 테이블의 행이 없을 때는 오른쪽 테이블의 컬럼에 빈 값(null)이 대신 들어간다.

**연습문제:** 오른쪽 외부 조인과 전체 외부 조인도 있다. 이들이 어떤 동작을 하는지 알아보자.

테이블을 자기 자신과 조인할 수도 있다. 이를 *자체 조인(self join)*이라고 한다. 예를 들어, 다른 날씨 기록의 온도 범위에 속하는 모든 날씨 기록을 찾고 싶다고 하자. 각 `weather` 행의 `temp_lo`와 `temp_hi` 컬럼을 다른 모든 `weather` 행의 `temp_lo`와 `temp_hi` 컬럼과 비교해야 한다. 다음 쿼리로 이를 수행할 수 있다:

```sql
SELECT w1.city, w1.temp_lo AS low, w1.temp_hi AS high,
       w2.city, w2.temp_lo AS low, w2.temp_hi AS high
    FROM weather w1 JOIN weather w2
        ON w1.temp_lo < w2.temp_lo AND w1.temp_hi > w2.temp_hi;
```

```sql
     city      | low | high |     city      | low | high
---------------+-----+------+---------------+-----+------
 San Francisco |  43 |   57 | San Francisco |  46 |   50
 Hayward       |  37 |   54 | San Francisco |  46 |   50
(2 rows)
```

여기서는 조인의 왼쪽과 오른쪽을 구분할 수 있도록 weather 테이블의 이름을 `w1`과 `w2`로 다시 지정했다. 이러한 종류의 별칭은 다른 쿼리에서도 타이핑을 줄이기 위해 사용할 수 있다. 예를 들면:

```sql
SELECT *
    FROM weather w JOIN cities c ON w.city = c.name;
```

이러한 약어 스타일은 매우 자주 사용된다.

## 2.7. 집계 함수

PostgreSQL은 다른 관계형 데이터베이스처럼 *집계 함수*를 지원한다. 집계 함수는 여러 행의 데이터를 입력받아 하나의 결과를 계산한다. 예를 들어 `count`(개수), `sum`(합계), `avg`(평균), `max`(최댓값), `min`(최솟값)과 같은 집계 함수가 있다.

예를 들어, 다음 쿼리로 전체 지역의 최저 기온 중 가장 높은 값을 찾을 수 있다:

```sql
SELECT max(temp_lo) FROM weather;
```
```
 max
-----
  46
(1 row)
```

이 최저 기온이 어느 도시에서 측정되었는지 알고 싶다면 다음과 같이 시도할 수 있다:

```sql
SELECT city FROM weather WHERE temp_lo = max(temp_lo);     -- 잘못된 쿼리
```

하지만 이 쿼리는 작동하지 않는다. `WHERE` 절에서는 집계 함수 `max`를 사용할 수 없기 때문이다. (`WHERE` 절은 집계 계산에 포함할 행을 결정하므로, 집계 함수보다 먼저 평가되어야 한다.) 그러나 대부분의 경우 *서브쿼리*를 사용하여 원하는 결과를 얻을 수 있다:

```sql
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);
```
```
     city
---------------
 San Francisco
(1 row)
```

이 방식이 가능한 이유는 서브쿼리가 외부 쿼리와 독립적으로 실행되어 자체적으로 집계를 계산하기 때문이다.

집계 함수는 `GROUP BY` 절과 함께 사용할 때 특히 유용하다. 예를 들어, 각 도시별로 관측 횟수와 최저 기온의 최댓값을 구할 수 있다:

```sql
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city;
```
```
     city      | count | max
---------------+-------+-----
 Hayward       |     1 |  37
 San Francisco |     2 |  46
(2 rows)
```

이 쿼리는 도시별로 한 행씩 출력한다. 각 집계 결과는 해당 도시에 해당하는 행들만 대상으로 계산된다. `HAVING`을 사용하면 이렇게 그룹화된 행들을 필터링할 수 있다:

```sql
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;
```
```
  city   | count | max
---------+-------+-----
 Hayward |     1 |  37
(1 row)
```

이 쿼리는 모든 `temp_lo` 값이 40 미만인 도시만 보여준다. 만약 "S"로 시작하는 도시만 보고 싶다면 다음과 같이 작성할 수 있다:

```sql
SELECT city, count(*), max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%'            -- (1)
    GROUP BY city;
```
```
     city      | count | max
---------------+-------+-----
 San Francisco |     2 |  46
(1 row)
```

(1) LIKE 연산자는 패턴 매칭을 수행하며, 9.7절에서 자세히 설명한다.

SQL의 `WHERE`절과 `HAVING`절이 집계 함수와 어떻게 상호작용하는지 이해하는 것이 중요하다. `WHERE`와 `HAVING`의 핵심적인 차이는 다음과 같다: `WHERE`는 그룹화와 집계가 계산되기 전에 입력 행을 선택한다(따라서 어떤 행이 집계 계산에 포함될지 결정한다). 반면 `HAVING`은 그룹화와 집계가 계산된 후에 그룹 행을 선택한다. 따라서 `WHERE` 절에는 집계 함수를 포함할 수 없다. 집계를 사용하여 집계의 입력이 될 행을 결정하려는 것은 논리적으로 맞지 않다. 반면 `HAVING` 절은 항상 집계 함수를 포함한다. (엄밀히 말하면, 집계를 사용하지 않는 `HAVING` 절도 작성할 수 있지만 거의 유용하지 않다. 같은 조건을 `WHERE` 단계에서 더 효율적으로 사용할 수 있다.)

이전 예제에서 도시 이름 제한을 `WHERE`에 적용할 수 있었던 이유는 집계가 필요하지 않았기 때문이다. 이 방식이 제한을 `HAVING`에 추가하는 것보다 더 효율적이다. `WHERE` 조건을 만족하지 않는 행들에 대해서는 그룹화와 집계 계산을 하지 않아도 되기 때문이다.

집계 계산에 포함될 행을 선택하는 다른 방법으로 `FILTER`를 사용할 수 있다. 이는 개별 집계에 적용되는 옵션이다:

```sql
SELECT city, count(*) FILTER (WHERE temp_lo < 45), max(temp_lo)
    FROM weather
    GROUP BY city;
```
```
     city      | count | max
---------------+-------+-----
 Hayward       |     1 |  37
 San Francisco |     1 |  46
(2 rows)
```

`FILTER`는 `WHERE`와 비슷하지만, 연결된 특정 집계 함수의 입력에서만 행을 제거한다는 점이 다르다. 위 예제에서 `count` 집계는 `temp_lo`가 45 미만인 행만 계산하지만, `max` 집계는 여전히 모든 행을 대상으로 적용되어 46이라는 값을 찾아낸다.

## 2.8. 데이터 갱신

기존 행의 데이터는 `UPDATE` 명령어로 변경할 수 있다. 예를 들어 11월 28일 이후의 모든 기온 측정값이 2도 높게 기록되었다는 것을 발견했다고 가정해보자. 다음과 같이 데이터를 수정할 수 있다:

```sql
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';
```

수정된 데이터의 상태를 확인해보자:

```sql
SELECT * FROM weather;
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      41 |      55 |    0 | 1994-11-29
 Hayward       |      35 |      52 |      | 1994-11-29
(3 rows)
```

## 2.9. 데이터 삭제하기

테이블에서 행을 삭제할 때는 `DELETE` 명령어를 사용한다. 예를 들어 Hayward 지역의 날씨 정보가 더 이상 필요하지 않다면 다음과 같이 해당 행을 테이블에서 삭제할 수 있다:

```sql
DELETE FROM weather WHERE city = 'Hayward';
```

이 명령어는 Hayward와 관련된 모든 날씨 기록을 삭제한다.

```sql
SELECT * FROM weather;
```

```
     city      | temp_lo | temp_hi | prcp |    date
---------------+---------+---------+------+------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      41 |      55 |    0 | 1994-11-29
(2 rows)
```

다음과 같은 형태의 명령어를 사용할 때는 특히 주의가 필요하다:

```sql
DELETE FROM tablename;
```

WHERE 절 없이 `DELETE`를 실행하면 지정한 테이블의 *모든* 행이 삭제되어 테이블이 비게 된다. 이때 시스템은 삭제 작업을 수행하기 전에 확인을 요청하지 않으니 각별히 주의해야 한다.