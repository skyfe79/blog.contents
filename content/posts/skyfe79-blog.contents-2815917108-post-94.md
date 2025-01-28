---
title: "PostgreSQL 튜토리얼 - 고급 기능"
date: 2025-01-28T14:52:58Z
author: "skyfe79"
draft: false
tags: ["postgresql"]
---

## 3.1. 소개

앞 장에서는 PostgreSQL에서 SQL을 사용해 데이터를 저장하고 접근하는 기본적인 방법을 다루었다. 이번 장에서는 데이터 관리를 단순화하고 데이터의 손실이나 손상을 방지하는 SQL의 고급 기능을 살펴본다. 마지막으로 PostgreSQL의 확장 기능도 알아본다.

이번 장에서는 [2장: SQL 언어](https://www.postgresql.org/docs/17/tutorial-sql.html)의 예제를 참조하여 이를 수정하거나 개선할 것이다. 따라서 2장을 먼저 읽어두면 도움이 된다. 이번 장의 일부 예제는 튜토리얼 디렉터리의 `advanced.sql` 파일에서도 찾을 수 있다. 이 파일에는 여기서 반복하지 않은 샘플 데이터도 포함되어 있다. (파일 사용 방법은 [2.1절: 소개](https://www.postgresql.org/docs/17/tutorial-sql-intro.html)를 참조한다.)

## 3.2. 뷰(View)

[2.6절 테이블 간 조인](https://www.postgresql.org/docs/17/tutorial-join.html "2.6. Joins Between Tables")에서 다룬 쿼리를 다시 살펴보자. 날씨 기록과 도시 위치를 함께 보여주는 결과가 애플리케이션에서 자주 필요하다고 가정한다. 이때 매번 같은 쿼리를 입력하는 것은 비효율적이다. 이런 경우 *뷰*를 생성하면 쿼리에 이름을 부여할 수 있다. 이렇게 만든 뷰는 일반 테이블처럼 사용할 수 있다:

```sql
CREATE VIEW myview AS
    SELECT name, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;
SELECT * FROM myview;
```

뷰를 적극적으로 활용하는 것은 SQL 데이터베이스 설계의 핵심이다. 뷰는 테이블 구조의 세부 사항을 일관된 인터페이스 뒤에 캡슐화한다. 이는 애플리케이션이 발전하면서 테이블 구조가 변경되더라도 유연하게 대응할 수 있게 한다.

뷰는 실제 테이블을 사용할 수 있는 거의 모든 곳에서 활용할 수 있다. 다른 뷰를 기반으로 새로운 뷰를 만드는 것도 일반적인 방법이다.

## 3.3. 외래 키

[2장](https://www.postgresql.org/docs/17/tutorial-sql.html "2장. SQL 언어")에서 다룬 `weather`와 `cities` 테이블을 다시 살펴보자. 다음과 같은 문제 상황을 고려해보자: `cities` 테이블에 대응하는 데이터가 없는 행을 `weather` 테이블에 입력하지 못하도록 막고 싶다. 이를 데이터의 *참조 무결성*이라고 한다. 단순한 데이터베이스 시스템에서는 `cities` 테이블에서 일치하는 레코드가 있는지 먼저 확인한 다음, 새로운 `weather` 레코드를 삽입하거나 거부하는 방식으로 구현한다. 하지만 이 방식은 여러 가지 문제점이 있고 매우 불편하다. PostgreSQL은 이 작업을 자동으로 처리할 수 있다.

새로운 테이블 선언은 다음과 같다:

```sql
CREATE TABLE cities (
        name     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(name),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);
```

이제 잘못된 레코드를 삽입해보자:

```sql
INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28');
```

```
ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Berkeley) is not present in table "cities".
```

외래 키의 동작은 애플리케이션의 요구사항에 맞게 세밀하게 조정할 수 있다. 이 기초 튜토리얼에서는 이 간단한 예제만 다루고, 더 자세한 내용은 [5장](https://www.postgresql.org/docs/17/ddl.html "5장. 데이터 정의")을 참고하기 바란다. 외래 키를 올바르게 사용하면 데이터베이스 애플리케이션의 품질이 크게 향상되므로, 이에 대해 자세히 학습할 것을 강력히 권장한다.

## 3.4. 트랜잭션

트랜잭션은 모든 데이터베이스 시스템의 핵심 개념이다. 트랜잭션의 본질은 여러 단계를 하나의 '모 아니면 도' 작업으로 묶는 것이다. 각 단계 사이의 중간 상태는 다른 동시 실행 트랜잭션에서 볼 수 없으며, 트랜잭션 완료를 방해하는 오류가 발생하면 어떤 단계도 데이터베이스에 영향을 미치지 않는다.

예를 들어, 고객 계좌 잔액과 지점별 예금 총액을 보관하는 은행 데이터베이스를 살펴보자. Alice의 계좌에서 Bob의 계좌로 100달러를 이체하는 상황을 가정해보자. 매우 단순화하면, SQL 명령은 다음과 같이 구성된다:

```sql
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
UPDATE branches SET balance = balance - 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Alice');
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
UPDATE branches SET balance = balance + 100.00
    WHERE name = (SELECT branch_name FROM accounts WHERE name = 'Bob');
```

이 명령들의 세부 내용은 여기서 중요하지 않다. 중요한 점은 이 간단한 작업을 수행하는 데 여러 개의 독립적인 업데이트가 필요하다는 것이다. 은행 관리자는 이 모든 업데이트가 모두 실행되거나 전혀 실행되지 않아야 한다고 요구할 것이다. 시스템 오류로 인해 Alice의 계좌에서 차감되지 않은 100달러가 Bob의 계좌에 입금되는 상황은 절대 발생해서는 안 된다. 또한 Bob에게 입금되지 않은 금액이 Alice의 계좌에서 차감된다면 Alice는 만족스러운 고객이 될 수 없을 것이다. 작업 중간에 문제가 발생하면 이미 실행된 단계들이 전혀 영향을 미치지 않도록 보장해야 한다. 이러한 업데이트들을 트랜잭션으로 그룹화하면 이런 보장을 얻을 수 있다. 트랜잭션은 원자성을 가진다고 표현한다. 다른 트랜잭션의 관점에서 보면 트랜잭션은 완전히 실행되거나 전혀 실행되지 않는다.

또한 트랜잭션이 완료되고 데이터베이스 시스템이 이를 확인한 후에는 실제로 영구적으로 기록되어 바로 직후에 시스템이 중단되더라도 손실되지 않도록 보장해야 한다. 예를 들어, Bob의 현금 인출을 기록할 때 그가 은행 문을 나서자마자 시스템이 중단되어 그의 계좌에서 차감된 금액이 사라질 가능성이 있어서는 안 된다. 트랜잭션 데이터베이스는 트랜잭션의 모든 업데이트가 완료되었다고 보고하기 전에 영구 저장소(즉, 디스크)에 기록되도록 보장한다.

트랜잭션 데이터베이스의 또 다른 중요한 특성은 원자적 업데이트 개념과 밀접하게 연관되어 있다. 여러 트랜잭션이 동시에 실행될 때 각 트랜잭션은 다른 트랜잭션의 미완료 변경사항을 볼 수 없어야 한다. 예를 들어, 한 트랜잭션이 모든 지점의 잔액을 합산하는 중이라면, Alice 지점의 차감액은 포함하고 Bob 지점의 입금액은 포함하지 않는 상황이 발생해서는 안 된다. 따라서 트랜잭션은 데이터베이스에 영구적인 영향을 미치는 측면에서뿐만 아니라 진행 중인 변경사항의 가시성 측면에서도 '모 아니면 도'여야 한다. 진행 중인 트랜잭션의 업데이트는 트랜잭션이 완료될 때까지 다른 트랜잭션에서 볼 수 없으며, 완료되면 모든 업데이트가 동시에 가시화된다.

PostgreSQL에서 트랜잭션은 SQL 명령을 `BEGIN`과 `COMMIT` 명령으로 감싸서 설정한다. 따라서 앞서 설명한 은행 트랜잭션은 실제로 다음과 같이 구성된다:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
-- 기타 명령어들
COMMIT;
```

트랜잭션 중간에 커밋하지 않기로 결정한다면(예: Alice의 잔액이 마이너스가 되는 것을 발견한 경우), `COMMIT` 대신 `ROLLBACK` 명령을 실행하여 지금까지의 모든 업데이트를 취소할 수 있다.

PostgreSQL은 실제로 모든 SQL 문장을 트랜잭션 내에서 실행되는 것으로 처리한다. `BEGIN` 명령을 실행하지 않으면 각각의 개별 문장에 암시적인 `BEGIN`과 (성공 시) `COMMIT`이 자동으로 포함된다. `BEGIN`과 `COMMIT`으로 둘러싸인 문장 그룹을 트랜잭션 블록이라고 부른다.

### 참고사항

일부 클라이언트 라이브러리는 `BEGIN`과 `COMMIT` 커맨드를 자동으로 실행한다. 따라서 개발자가 직접 지정하지 않아도 트랜잭션 블록 효과를 얻을 수 있다. 사용하는 인터페이스의 문서를 확인하여 이러한 자동 실행 여부를 파악해야 한다.

세이브포인트를 활용하면 트랜잭션 내 구문을 더 세밀하게 제어할 수 있다. 세이브포인트는 트랜잭션의 일부분만 취소하고 나머지는 커밋할 수 있게 한다. `SAVEPOINT` 커맨드로 세이브포인트를 정의한 후, 필요한 경우 `ROLLBACK TO` 커맨드를 사용하여 해당 세이브포인트로 되돌릴 수 있다. 세이브포인트 정의 시점과 롤백 시점 사이의 모든 데이터베이스 변경사항은 취소되지만, 세이브포인트 이전의 변경사항은 유지된다.

세이브포인트는 롤백 후에도 정의된 상태로 남아있어 여러 번 롤백할 수 있다. 반대로 특정 세이브포인트로 더 이상 롤백할 필요가 없다면, 해당 세이브포인트를 해제하여 시스템 리소스를 확보할 수 있다. 세이브포인트를 해제하거나 롤백하면 해당 시점 이후에 정의된 모든 세이브포인트도 자동으로 해제된다는 점에 주의해야 한다.

이 모든 작업은 트랜잭션 블록 내에서 이루어지므로 다른 데이터베이스 세션에서는 변경사항을 볼 수 없다. 트랜잭션 블록을 커밋하면 커밋된 작업이 하나의 단위로 다른 세션에 표시되지만, 롤백된 작업은 전혀 표시되지 않는다.

은행 데이터베이스를 예로 들어보자. Alice의 계좌에서 100달러를 인출하여 Bob의 계좌에 입금하려다가, 나중에 Wally의 계좌에 입금해야 한다는 것을 알게 되었다. 이런 경우 세이브포인트를 다음과 같이 활용할 수 있다:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- 실수했으니 취소하고 Wally의 계좌를 사용하자
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;
```

이 예제는 매우 단순화된 것이지만, 세이브포인트를 통해 트랜잭션 블록 내에서 다양한 제어가 가능하다는 것을 보여준다. 더욱이 `ROLLBACK TO`는 시스템 오류로 인해 중단 상태가 된 트랜잭션 블록의 제어권을 다시 확보할 수 있는 유일한 방법이다. 그렇지 않으면 전체를 롤백하고 처음부터 다시 시작해야 한다.

## 3.5. 윈도우 함수

윈도우 함수(Window Function)는 현재 행과 연관된 테이블 행들의 집합에 대해 계산을 수행한다. 이는 집계 함수로 수행할 수 있는 계산과 유사하다. 하지만 일반적인 집계 함수와 달리 윈도우 함수는 여러 행을 하나의 출력 행으로 그룹화하지 않는다. 대신 각 행은 개별성을 유지한다. 내부적으로 윈도우 함수는 쿼리 결과의 현재 행뿐만 아니라 다른 행에도 접근할 수 있다.

다음 예제는 각 직원의 급여를 해당 부서의 평균 급여와 비교하는 방법을 보여준다:

```sql
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;
```

```
  depname  | empno | salary |          avg
-----------+-------+--------+-----------------------
 develop   |    11 |   5200 | 5020.0000000000000000
 develop   |     7 |   4200 | 5020.0000000000000000
 develop   |     9 |   4500 | 5020.0000000000000000
 develop   |     8 |   6000 | 5020.0000000000000000
 develop   |    10 |   5200 | 5020.0000000000000000
 personnel |     5 |   3500 | 3700.0000000000000000
 personnel |     2 |   3900 | 3700.0000000000000000
 sales     |     3 |   4800 | 4866.6666666666666667
 sales     |     1 |   5000 | 4866.6666666666666667
 sales     |     4 |   4800 | 4866.6666666666666667
(10 rows)
```

처음 세 개의 출력 컬럼은 `empsalary` 테이블에서 직접 가져온 것이며, 테이블의 각 행마다 하나의 출력 행이 있다. 네 번째 컬럼은 현재 행과 동일한 `depname` 값을 가진 모든 테이블 행의 평균을 나타낸다. (이는 실제로 일반적인 `avg` 집계 함수와 동일하지만, `OVER` 절로 인해 윈도우 함수로 처리되어 윈도우 프레임에 걸쳐 계산된다.)

윈도우 함수 호출은 항상 함수 이름과 인자 바로 뒤에 `OVER` 절을 포함한다. 이는 일반 함수나 비윈도우 집계와 구문적으로 구분되는 특징이다. `OVER` 절은 윈도우 함수가 처리할 쿼리의 행을 어떻게 분할할지 정확히 결정한다. `OVER` 내의 `PARTITION BY` 절은 `PARTITION BY` 표현식의 값이 같은 행들을 그룹 또는 파티션으로 나눈다. 각 행에 대해 윈도우 함수는 현재 행과 같은 파티션에 속하는 행들에 대해 계산을 수행한다.

`OVER` 내에서 `ORDER BY`를 사용하여 윈도우 함수가 행을 처리하는 순서도 제어할 수 있다. (윈도우 `ORDER BY`는 행이 출력되는 순서와 일치하지 않아도 된다.) 다음은 그 예시이다:

```sql
SELECT depname, empno, salary,
       rank() OVER (PARTITION BY depname ORDER BY salary DESC)
FROM empsalary;
```

```
  depname  | empno | salary | rank
-----------+-------+--------+------
 develop   |     8 |   6000 |    1
 develop   |    10 |   5200 |    2
 develop   |    11 |   5200 |    2
 develop   |     9 |   4500 |    4
 develop   |     7 |   4200 |    5
 personnel |     2 |   3900 |    1
 personnel |     5 |   3500 |    2
 sales     |     1 |   5000 |    1
 sales     |     4 |   4800 |    2
 sales     |     3 |   4800 |    2
(10 rows)
```

여기서 보듯이 `rank` 함수는 `ORDER BY` 절에서 정의된 순서에 따라 현재 행 파티션의 각 고유한 `ORDER BY` 값에 대해 순위를 매긴다. `rank` 함수는 동작이 전적으로 `OVER` 절에 의해 결정되므로 명시적인 매개변수가 필요하지 않다.

윈도우 함수가 고려하는 행들은 쿼리의 `FROM` 절이 생성한 "가상 테이블"의 행들이며, 이는 `WHERE`, `GROUP BY`, `HAVING` 절이 있다면 해당 절로 필터링된다. 예를 들어, `WHERE` 조건을 충족하지 않아 제거된 행은 어떤 윈도우 함수도 볼 수 없다. 하나의 쿼리는 서로 다른 `OVER` 절을 사용하여 데이터를 다양한 방식으로 분할하는 여러 윈도우 함수를 포함할 수 있지만, 이들은 모두 이 가상 테이블에 의해 정의된 동일한 행 집합에 대해 동작한다.

행의 순서가 중요하지 않은 경우 `ORDER BY`를 생략할 수 있다는 것을 앞서 보았다. `PARTITION BY`도 생략할 수 있으며, 이 경우 모든 행을 포함하는 단일 파티션이 생성된다.

윈도우 함수와 관련된 또 다른 중요한 개념은 *윈도우 프레임*이다. 각 행에 대해 해당 파티션 내에서 윈도우 프레임이라 불리는 행의 집합이 존재한다. 일부 윈도우 함수는 전체 파티션이 아닌 윈도우 프레임의 행에 대해서만 동작한다. 기본적으로 `ORDER BY`가 있는 경우 프레임은 파티션의 시작부터 현재 행까지의 모든 행과 `ORDER BY` 절에 따라 현재 행과 동일한 값을 가진 모든 후속 행으로 구성된다. `ORDER BY`가 생략된 경우 기본 프레임은 파티션의 모든 행으로 구성된다. 다음은 `sum`을 사용한 예시이다:

```sql
SELECT salary, sum(salary) OVER () FROM empsalary;
```

```
 salary |  sum
--------+-------
   5200 | 47100
   5000 | 47100
   3500 | 47100
   4800 | 47100
   3900 | 47100
   4200 | 47100
   4500 | 47100
   4800 | 47100
   6000 | 47100
   5200 | 47100
(10 rows)
```

위의 경우, `OVER` 절에 `ORDER BY`가 없으므로 윈도우 프레임은 파티션과 동일하며, `PARTITION BY`가 없으므로 전체 테이블이 하나의 파티션이 된다. 다시 말해 각 합계는 전체 테이블에 대해 계산되므로 모든 출력 행에 대해 동일한 결과를 얻는다. 그러나 `ORDER BY` 절을 추가하면 매우 다른 결과를 얻게 된다:

```sql
SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary;
```

```
 salary |  sum
--------+-------
   3500 |  3500
   3900 |  7400
   4200 | 11600
   4500 | 16100
   4800 | 25700
   4800 | 25700
   5000 | 30700
   5200 | 41100
   5200 | 41100
   6000 | 47100
(10 rows)
```

여기서는 첫 번째(가장 낮은) 급여부터 현재 급여까지의 합계를 계산하며, 현재 급여와 동일한 값을 가진 중복된 행도 포함한다(중복된 급여에 대한 결과를 주목하라).

윈도우 함수는 쿼리의 `SELECT` 목록과 `ORDER BY` 절에서만 허용된다. `GROUP BY`, `HAVING`, `WHERE` 절과 같은 다른 곳에서는 사용할 수 없다. 이는 이러한 절들의 처리 이후에 논리적으로 실행되기 때문이다. 또한 윈도우 함수는 비윈도우 집계 함수 이후에 실행된다. 이는 윈도우 함수의 인자로 집계 함수 호출을 포함할 수 있지만 그 반대는 불가능하다는 것을 의미한다.

윈도우 계산 이후에 행을 필터링하거나 그룹화할 필요가 있는 경우 서브쿼리를 사용할 수 있다. 예를 들어:

```sql
SELECT depname, empno, salary, enroll_date
FROM
  (SELECT depname, empno, salary, enroll_date,
          rank() OVER (PARTITION BY depname ORDER BY salary DESC, empno) AS pos
     FROM empsalary
  ) AS ss
WHERE pos < 3;
```

위의 쿼리는 `rank`가 3 미만인 내부 쿼리의 행만 보여준다.

쿼리가 여러 윈도우 함수를 포함할 때, 각각에 대해 별도의 `OVER` 절을 작성할 수 있지만, 여러 함수에 대해 동일한 윈도우 동작이 필요한 경우 이는 중복적이고 오류가 발생하기 쉽다. 대신 각 윈도우 동작에 `WINDOW` 절에서 이름을 지정하고 `OVER`에서 참조할 수 있다. 예를 들어:

```sql
SELECT sum(salary) OVER w, avg(salary) OVER w
  FROM empsalary
  WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

윈도우 함수에 대한 자세한 내용은 [4.2.8절](https://www.postgresql.org/docs/17/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS "4.2.8. 윈도우 함수 호출"), [9.22절](https://www.postgresql.org/docs/17/functions-window.html "9.22. 윈도우 함수"), [7.2.5절](https://www.postgresql.org/docs/17/queries-table-expressions.html#QUERIES-WINDOW "7.2.5. 윈도우 함수 처리"), 그리고 [SELECT](https://www.postgresql.org/docs/17/sql-select.html "SELECT") 참조 페이지에서 확인할 수 있다.

## 3.6. 상속

상속은 객체지향 데이터베이스에서 유래한 개념이다. 이를 통해 데이터베이스 설계에서 새롭고 흥미로운 가능성을 열 수 있다.

두 개의 테이블을 만들어보자: `cities` 테이블과 `capitals` 테이블이다. 수도는 도시이기도 하므로, 모든 도시 목록을 조회할 때 수도도 함께 표시하는 방법이 필요하다. 다음과 같은 방식을 생각해볼 수 있다:

```sql
CREATE TABLE capitals (
  name       text,
  population real,
  elevation  int,    -- (단위: 피트)
  state      char(2)
);

CREATE TABLE non_capitals (
  name       text,
  population real,
  elevation  int     -- (단위: 피트)
);

CREATE VIEW cities AS
  SELECT name, population, elevation FROM capitals
    UNION
  SELECT name, population, elevation FROM non_capitals;
```

이 방식은 데이터 조회에는 문제가 없지만, 여러 행을 한 번에 수정해야 할 때는 복잡해진다.

더 나은 해결책은 다음과 같다:

```sql
CREATE TABLE cities (
  name       text,
  population real,
  elevation  int     -- (단위: 피트)
);

CREATE TABLE capitals (
  state      char(2) UNIQUE NOT NULL
) INHERITS (cities);
```

이 경우 `capitals` 테이블의 각 행은 상위 테이블인 `cities`로부터 모든 컬럼(`name`, `population`, `elevation`)을 상속받는다. `name` 컬럼의 타입은 가변 길이 문자열을 저장하는 PostgreSQL 기본 타입인 `text`이다. `capitals` 테이블은 주 약자를 저장하는 추가 컬럼 `state`를 갖는다. PostgreSQL에서는 테이블이 0개 이상의 다른 테이블을 상속할 수 있다.

예를 들어, 다음 쿼리는 해발 500피트 이상에 위치한 모든 도시(주도 포함)의 이름을 찾는다:

```sql
SELECT name, elevation
  FROM cities
  WHERE elevation > 500;
```

실행 결과:
```
   name    | elevation
-----------+-----------
 Las Vegas |      2174
 Mariposa  |      1953
 Madison   |       845
(3 rows)
```

반면, 다음 쿼리는 해발 500피트 이상에 위치한 일반 도시(주도 제외)만 찾는다:

```sql
SELECT name, elevation
    FROM ONLY cities
    WHERE elevation > 500;
```

실행 결과:
```
   name    | elevation
-----------+-----------
 Las Vegas |      2174
 Mariposa  |      1953
(2 rows)
```

여기서 `cities` 앞의 `ONLY`는 상속 계층 구조에서 `cities` 아래에 있는 테이블은 제외하고 `cities` 테이블만 대상으로 쿼리를 실행하라는 의미다. 이미 살펴본 `SELECT`, `UPDATE`, `DELETE` 등 많은 명령어가 이 `ONLY` 표기법을 지원한다.

### 주의사항

상속은 많은 경우에 유용하지만, 고유 제약조건이나 외래 키와 통합되지 않아 활용에 제약이 있다. 자세한 내용은 [5.11절 상속](https://www.postgresql.org/docs/17/ddl-inherit.html "5.11. Inheritance")을 참조한다.

## 3.7. 맺음말

이 입문 튜토리얼은 SQL을 처음 접하는 사용자를 위해 작성했기 때문에 PostgreSQL의 많은 기능을 다루지 않았다. 나머지 기능에 대한 자세한 설명은 이 책의 다음 장들에서 확인할 수 있다.

더 많은 기초 자료가 필요하다면 PostgreSQL [웹사이트](https://www.postgresql.org/)를 방문하여 추가 학습 자료를 찾아볼 수 있다.