---
title: "[PostgreSQL] 제2부. SQL 언어 - 7장. 데이터 조회하기"
date: 2025-02-02T04:35:45Z
author: "skyfe79"
draft: false
tags: ["postgresql"]
---

이전 장에서는 테이블을 만드는 방법, 테이블에 데이터를 채우는 방법, 그리고 데이터를 조작하는 방법에 대해 설명했다. 이제 마침내 데이터베이스에서 데이터를 가져오는 방법에 대해 이야기할 차례이다. 

## 7.1. 개요

데이터베이스에서 데이터를 가져오는 과정 또는 그 명령을 *쿼리*라고 한다. SQL에서는 [`SELECT`](https://www.postgresql.org/docs/17/sql-select.html "SELECT") 명령어를 사용해 쿼리를 지정한다. `SELECT` 명령어의 일반적인 문법은 다음과 같다.

```sql
[WITH with_queries] SELECT select_list FROM table_expression [sort_specification]
```

이어지는 섹션에서는 선택 목록(`select_list`), 테이블 표현식(`table_expression`), 정렬 지정(`sort_specification`)에 대해 자세히 설명한다. `WITH` 쿼리는 고급 기능이므로 마지막에 다룬다.

가장 간단한 형태의 쿼리는 다음과 같다.

```sql
SELECT * FROM table1;
```

`table1`이라는 테이블이 존재한다고 가정할 때, 이 명령어는 `table1`의 모든 행과 사용자 정의 컬럼을 가져온다. (가져오는 방법은 클라이언트 애플리케이션에 따라 다르다. 예를 들어, psql 프로그램은 화면에 ASCII 아트 형태의 테이블을 표시하고, 클라이언트 라이브러리는 쿼리 결과에서 개별 값을 추출하는 함수를 제공한다.) select 목록에서 `*`는 테이블 표현식이 제공하는 모든 컬럼을 의미한다. 선택 목록을 사용해 사용 가능한 컬럼의 일부만 선택하거나 컬럼을 이용해 계산을 수행할 수도 있다. 예를 들어, `table1`에 `a`, `b`, `c`라는 컬럼이 있다면(그 외에 다른 컬럼이 있을 수도 있다), 다음과 같은 쿼리를 작성할 수 있다.

```sql
SELECT a, b + c FROM table1;
```

(단, `b`와 `c`는 숫자 타입이어야 한다.) 더 자세한 내용은 [7.3절](https://www.postgresql.org/docs/17/queries-select-lists.html "7.3. Select Lists")을 참고한다.

`FROM table1`은 가장 단순한 형태의 테이블 표현식이다. 이 표현식은 단일 테이블만 읽는다. 일반적으로 테이블 표현식은 기본 테이블, 조인, 서브쿼리 등으로 구성된 복잡한 구조일 수 있다. 하지만 테이블 표현식을 완전히 생략하고 `SELECT` 명령어를 계산기로 사용할 수도 있다.

```sql
SELECT 3 * 4;
```

이 방식은 select 목록의 표현식이 다양한 결과를 반환할 때 더 유용하다. 예를 들어, 다음과 같이 함수를 호출할 수도 있다.

```sql
SELECT random();
```

## 7.2. 테이블 표현식

테이블 표현식은 테이블을 계산한다. 테이블 표현식은 `FROM` 절을 포함하며, 선택적으로 `WHERE`, `GROUP BY`, `HAVING` 절이 뒤따를 수 있다. 단순한 테이블 표현식은 디스크에 있는 테이블, 즉 기본 테이블을 참조하지만, 더 복잡한 표현식을 사용하면 기본 테이블을 다양한 방식으로 수정하거나 결합할 수 있다.

테이블 표현식에서 선택적으로 사용할 수 있는 `WHERE`, `GROUP BY`, `HAVING` 절은 `FROM` 절에서 파생된 테이블에 대해 연속적인 변환을 수행하는 파이프라인을 지정한다. 이러한 모든 변환은 가상 테이블을 생성하며, 이 가상 테이블은 쿼리의 출력 행을 계산하기 위해 select 목록에 전달될 행을 제공한다.

### 7.2.1. `FROM` 절

`FROM` 절은 쉼표로 구분된 테이블 참조 목록에서 하나 이상의 테이블을 가져와 새로운 테이블을 생성한다.

```sql
FROM table_reference [, table_reference [, ...]]
```

테이블 참조는 테이블 이름(스키마가 지정될 수 있음)일 수도 있고, 서브쿼리, `JOIN` 구조, 또는 이들의 복잡한 조합과 같은 파생 테이블일 수도 있다. `FROM` 절에 여러 테이블 참조가 나열되면, 이 테이블들은 크로스 조인된다(즉, 각 테이블의 행들의 데카르트 곱이 생성된다. 아래 참조). `FROM` 목록의 결과는 중간 가상 테이블이며, 이 테이블은 `WHERE`, `GROUP BY`, `HAVING` 절에 의해 변환될 수 있고, 최종적으로 전체 테이블 표현식의 결과가 된다.

테이블 참조가 테이블 상속 계층의 부모 테이블을 가리키는 경우, 테이블 이름 앞에 `ONLY` 키워드를 사용하지 않으면 해당 테이블뿐만 아니라 모든 하위 테이블의 행도 생성된다. 그러나 이 참조는 명명된 테이블에 나타나는 컬럼만 생성하며, 하위 테이블에서 추가된 컬럼은 무시된다.

테이블 이름 앞에 `ONLY`를 쓰는 대신, 테이블 이름 뒤에 `*`를 써서 하위 테이블이 포함됨을 명시적으로 지정할 수도 있다. 이제 하위 테이블을 검색하는 것이 기본 동작이기 때문에 이 구문을 사용할 실질적인 이유는 없다. 그러나 이전 버전과의 호환성을 위해 여전히 지원된다.

#### 7.2.1.1. 조인된 테이블

조인된 테이블은 두 개의 다른 테이블(실제 테이블 또는 파생된 테이블)을 특정 조인 타입의 규칙에 따라 결합하여 생성된 테이블이다. 내부 조인, 외부 조인, 크로스 조인 등 다양한 조인 타입을 사용할 수 있다. 조인된 테이블의 일반적인 구문은 다음과 같다.

```sql
T1 join_type T2 [ join_condition ]
```

모든 타입의 조인은 연쇄적으로 연결하거나 중첩할 수 있다. 즉, `T1`과 `T2` 중 하나 또는 둘 다 조인된 테이블일 수 있다. `JOIN` 절 주위에 괄호를 사용하여 조인 순서를 제어할 수 있다. 괄호가 없으면 `JOIN` 절은 왼쪽에서 오른쪽으로 중첩된다.

**조인 타입**

**크로스 조인**

```sql
T1 CROSS JOIN T2
```

`T1`과 `T2`의 모든 가능한 행 조합(즉, 데카르트 곱)에 대해, 조인된 테이블은 `T1`의 모든 컬럼과 `T2`의 모든 컬럼으로 구성된 행을 포함한다. 두 테이블이 각각 N개와 M개의 행을 가지고 있다면, 조인된 테이블은 N * M개의 행을 가진다.

`FROM T1 CROSS JOIN T2`는 `FROM T1 INNER JOIN T2 ON TRUE`와 동일하다(아래 참조). 또한 `FROM T1, T2`와도 동일하다.

### 주의 사항

두 개 이상의 테이블이 등장할 때, `JOIN`이 쉼표보다 더 강하게 바인딩되기 때문에 동등성이 정확히 성립하지 않는다. 예를 들어, ``FROM *`T1`* CROSS JOIN *`T2`* INNER JOIN *`T3`* ON *`condition`*``와 ``FROM *`T1`*, *`T2`* INNER JOIN *`T3`* ON *`condition`*``는 동일하지 않다. 첫 번째 경우에는 *`condition`*이 *`T1`*을 참조할 수 있지만, 두 번째 경우에는 그렇지 않기 때문이다.

### 조건부 조인

```sql
T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 ON boolean_expression
T1 { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2 USING ( join column list )
T1 NATURAL { [INNER] | { LEFT | RIGHT | FULL } [OUTER] } JOIN T2
```

`INNER`와 `OUTER`는 모든 형태에서 선택 사항이다. `INNER`가 기본값이며, `LEFT`, `RIGHT`, `FULL`은 외부 조인을 의미한다.

조인 조건은 `ON` 또는 `USING` 절에서 명시하거나, `NATURAL` 키워드로 암시적으로 지정한다. 조인 조건은 두 소스 테이블에서 어떤 행이 "일치"하는지 결정하며, 이에 대한 자세한 설명은 아래에 나온다.

### 가능한 조건부 조인 타입

**`INNER JOIN`**

`T1`의 각 행 R1에 대해, 조인 조건을 만족하는 `T2`의 각 행이 조인된 테이블에 포함된다.

**`LEFT OUTER JOIN`**

먼저 내부 조인이 수행된다. 그런 다음, `T1`의 각 행 중 `T2`의 어떤 행과도 조인 조건을 만족하지 않는 경우, `T2`의 컬럼에 `NULL` 값을 가진 조인된 행이 추가된다. 따라서 조인된 테이블은 항상 `T1`의 각 행에 대해 최소 하나의 행을 가진다.

**`RIGHT OUTER JOIN`**

먼저 내부 조인이 수행된다. 그런 다음, `T2`의 각 행 중 `T1`의 어떤 행과도 조인 조건을 만족하지 않는 경우, `T1`의 컬럼에 `NULL` 값을 가진 조인된 행이 추가된다. 이는 왼쪽 조인의 반대 개념이다: 결과 테이블은 항상 `T2`의 각 행에 대해 하나의 행을 가진다.

**`FULL OUTER JOIN`**

먼저 내부 조인이 수행된다. 그런 다음, `T1`의 각 행 중 `T2`의 어떤 행과도 조인 조건을 만족하지 않는 경우, `T2`의 컬럼에 `NULL` 값을 가진 조인된 행이 추가된다. 또한, `T2`의 각 행 중 `T1`의 어떤 행과도 조인 조건을 만족하지 않는 경우, `T1`의 컬럼에 `NULL` 값을 가진 조인된 행이 추가된다.

### `ON` 절

`ON` 절은 가장 일반적인 조인 조건이다: `WHERE` 절에서 사용하는 것과 같은 종류의 불리언 값 표현식을 사용한다. *`T1`*과 *`T2`*의 행 쌍은 `ON` 표현식이 `true`로 평가될 때 일치한다.

### `USING` 절

`USING` 절은 조인 양쪽이 조인 컬럼에 대해 동일한 이름을 사용하는 특정 상황을 활용할 수 있는 단축 표현이다. 쉼표로 구분된 공유 컬럼 이름 목록을 받아 각각에 대해 동등 비교를 포함하는 조인 조건을 형성한다. 예를 들어, `USING (a, b)`로 *`T1`*과 *`T2`*를 조인하면 ``ON *`T1`*.a = *`T2`*.a AND *`T1`*.b = *`T2`*.b``와 같은 조인 조건이 생성된다.

또한, `JOIN USING`의 출력은 중복 컬럼을 제거한다: 일치하는 컬럼은 동일한 값을 가지므로 두 컬럼을 모두 출력할 필요가 없다. `JOIN ON`은 *`T1`*의 모든 컬럼 뒤에 *`T2`*의 모든 컬럼을 출력하지만, `JOIN USING`은 나열된 컬럼 쌍에 대해 하나의 출력 컬럼을 생성한 뒤, *`T1`*의 나머지 컬럼과 *`T2`*의 나머지 컬럼을 출력한다.

### `NATURAL` 키워드

`NATURAL`은 `USING`의 단축 형태이다: 두 입력 테이블에 모두 나타나는 모든 컬럼 이름으로 구성된 `USING` 목록을 형성한다. `USING`과 마찬가지로, 이 컬럼들은 출력 테이블에 한 번만 나타난다. 공통 컬럼 이름이 없는 경우, `NATURAL JOIN`은 `CROSS JOIN`처럼 동작한다.

### 주의 사항

`USING` 구문은 조인에 사용되는 컬럼을 명시적으로 지정하기 때문에, 조인 대상 테이블의 스키마가 변경되더라도 안전하게 동작한다. 반면 `NATURAL` 구문은 두 테이블에서 이름이 같은 모든 컬럼을 자동으로 조인에 사용하기 때문에, 스키마 변경으로 인해 새로운 컬럼이 추가되면 예상치 못한 결과를 초래할 수 있다.

이를 이해하기 위해, 다음과 같은 두 테이블 `t1`과 `t2`가 있다고 가정하자.

**t1 테이블:**

| num | name |
|-----|------|
| 1   | a    |
| 2   | b    |
| 3   | c    |

**t2 테이블:**

| num | value |
|-----|-------|
| 1   | xxx   |
| 3   | yyy   |
| 5   | zzz   |

이제 다양한 조인 방식의 결과를 살펴보자.

```sql
=> SELECT * FROM t1 CROSS JOIN t2;
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
| 1   | a    | 3   | yyy   |
| 1   | a    | 5   | zzz   |
| 2   | b    | 1   | xxx   |
| 2   | b    | 3   | yyy   |
| 2   | b    | 5   | zzz   |
| 3   | c    | 1   | xxx   |
| 3   | c    | 3   | yyy   |
| 3   | c    | 5   | zzz   |
(9 rows)

```sql
=> SELECT * FROM t1 INNER JOIN t2 ON t1.num = t2.num;
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
| 3   | c    | 3   | yyy   |
(2 rows)

```sql
=> SELECT * FROM t1 INNER JOIN t2 USING (num);
```

| num | name | value |
|-----|------|-------|
| 1   | a    | xxx   |
| 3   | c    | yyy   |
(2 rows)

```sql
=> SELECT * FROM t1 NATURAL INNER JOIN t2;
```

| num | name | value |
|-----|------|-------|
| 1   | a    | xxx   |
| 3   | c    | yyy   |
(2 rows)

```sql
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num;
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
| 2   | b    |     |       |
| 3   | c    | 3   | yyy   |
(3 rows)

```sql
=> SELECT * FROM t1 LEFT JOIN t2 USING (num);
```

| num | name | value |
|-----|------|-------|
| 1   | a    | xxx   |
| 2   | b    |       |
| 3   | c    | yyy   |
(3 rows)

```sql
=> SELECT * FROM t1 RIGHT JOIN t2 ON t1.num = t2.num;
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
| 3   | c    | 3   | yyy   |
|     |      | 5   | zzz   |
(3 rows)

```sql
=> SELECT * FROM t1 FULL JOIN t2 ON t1.num = t2.num;
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
| 2   | b    |     |       |
| 3   | c    | 3   | yyy   |
|     |      | 5   | zzz   |
(4 rows)

`ON` 절에 지정된 조인 조건은 조인과 직접적으로 관련이 없는 조건도 포함할 수 있다. 이는 특정 쿼리에서 유용할 수 있지만, 신중하게 고려해야 한다. 예를 들어:

```sql
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num AND t2.value = 'xxx';
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
| 2   | b    |     |       |
| 3   | c    |     |       |
(3 rows)

`WHERE` 절에 조건을 지정하면 다른 결과가 나온다:

```sql
=> SELECT * FROM t1 LEFT JOIN t2 ON t1.num = t2.num WHERE t2.value = 'xxx';
```

| num | name | num | value |
|-----|------|-----|-------|
| 1   | a    | 1   | xxx   |
(1 row)

이 차이는 `ON` 절의 조건이 조인 전에 처리되는 반면, `WHERE` 절의 조건은 조인 후에 처리되기 때문이다. 이 차이는 내부 조인에서는 중요하지 않지만, 외부 조인에서는 큰 영향을 미친다.

#### 7.2.1.2. 테이블과 컬럼 별칭

테이블과 복잡한 테이블 참조에 임시 이름을 부여할 수 있다. 이를 *테이블 별칭*이라고 한다. 이 별칭은 쿼리의 나머지 부분에서 파생된 테이블을 참조할 때 사용된다.

테이블 별칭을 생성하려면 다음과 같이 작성한다:

```sql
FROM table_reference AS alias
```

또는

```sql
FROM table_reference alias
```

`AS` 키워드는 선택 사항이며, 생략해도 된다. *`alias`*는 어떤 식별자든 가능하다.

테이블 별칭의 일반적인 사용 사례는 긴 테이블 이름에 짧은 식별자를 할당하여 조인 절을 읽기 쉽게 만드는 것이다. 예를 들어:

```sql
SELECT * FROM some_very_long_table_name s JOIN another_fairly_long_name a ON s.id = a.num;
```

별칭은 현재 쿼리에서 테이블 참조의 새로운 이름이 된다. 따라서 쿼리의 다른 부분에서 원래 이름으로 테이블을 참조할 수 없다. 따라서 다음은 유효하지 않다:

```sql
SELECT * FROM my_table AS m WHERE my_table.a > 5;    -- 잘못된 사용
```

테이블 별칭은 주로 표기상의 편의를 위해 사용되지만, 테이블을 자기 자신과 조인할 때는 반드시 필요하다. 예를 들어:

```sql
SELECT * FROM people AS mother JOIN people AS child ON mother.id = child.mother_id;
```

모호성을 해결하기 위해 괄호를 사용한다. 다음 예제에서 첫 번째 문장은 `my_table`의 두 번째 인스턴스에 `b`라는 별칭을 할당하지만, 두 번째 문장은 조인의 결과에 별칭을 할당한다:

```sql
SELECT * FROM my_table AS a CROSS JOIN my_table AS b ...
SELECT * FROM (my_table AS a CROSS JOIN my_table) AS b ...
```

테이블 별칭의 또 다른 형태는 테이블 자체뿐만 아니라 테이블의 컬럼에도 임시 이름을 부여하는 것이다:

```sql
FROM table_reference [AS] alias ( column1 [, column2 [, ...]] )
```

실제 테이블의 컬럼 수보다 적은 수의 컬럼 별칭을 지정하면, 나머지 컬럼은 이름이 바뀌지 않는다. 이 구문은 특히 자기 조인이나 서브쿼리에서 유용하다.

`JOIN` 절의 출력에 별칭을 적용하면, 별칭이 `JOIN` 내부의 원래 이름을 숨긴다. 예를 들어:

```sql
SELECT a.* FROM my_table AS a JOIN your_table AS b ON ...
```

는 유효한 SQL이지만,

```sql
SELECT a.* FROM (my_table AS a JOIN your_table AS b ON ...) AS c
```

는 유효하지 않다. 테이블 별칭 `a`는 별칭 `c` 외부에서 보이지 않는다.

#### 7.2.1.3. 서브쿼리

파생 테이블을 지정하는 서브쿼리는 반드시 괄호로 감싸야 한다. 서브쿼리에 테이블 별칭을 지정할 수 있으며, 선택적으로 컬럼 별칭도 지정할 수 있다([7.2.1.2절](https://www.postgresql.org/docs/17/queries-table-expressions.html#QUERIES-TABLE-ALIASES "7.2.1.2. 테이블 및 컬럼 별칭") 참조). 예를 들어:

```sql
FROM (SELECT * FROM table1) AS alias_name
```

이 예제는 `FROM table1 AS alias_name`과 동일하다. 서브쿼리가 그룹화나 집계를 포함하는 경우, 단순 조인으로 축소할 수 없는 더 흥미로운 사례가 발생한다.

서브쿼리는 `VALUES` 목록으로도 사용할 수 있다:

```sql
FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow'))
     AS names(first, last)
```

이 경우에도 테이블 별칭은 선택 사항이다. `VALUES` 목록의 컬럼에 별칭을 지정하는 것도 선택 사항이지만, 좋은 습관이다. 더 자세한 내용은 [7.7절](https://www.postgresql.org/docs/17/queries-values.html "7.7. VALUES 목록")을 참조한다.

SQL 표준에 따르면, 서브쿼리에 테이블 별칭을 반드시 제공해야 한다. PostgreSQL은 `AS`와 별칭을 생략할 수 있지만, 다른 시스템으로 이식될 가능성이 있는 SQL 코드에서는 별칭을 작성하는 것이 좋은 습관이다.


#### 7.2.1.4. 테이블 함수

테이블 함수는 기본 데이터 타입(스칼라 타입) 또는 복합 데이터 타입(테이블 행)으로 구성된 행 집합을 생성하는 함수다. 이 함수는 쿼리의 `FROM` 절에서 테이블, 뷰, 서브쿼리처럼 사용한다. 테이블 함수가 반환하는 컬럼은 `SELECT`, `JOIN`, `WHERE` 절에서 테이블, 뷰, 서브쿼리의 컬럼과 동일한 방식으로 활용할 수 있다.

테이블 함수는 `ROWS FROM` 구문을 사용해 결합할 수도 있다. 이 경우 결과는 병렬 컬럼으로 반환되며, 결과 행의 수는 가장 큰 함수 결과의 행 수와 동일하다. 더 작은 결과는 `null` 값으로 채워져 크기를 맞춘다.

```sql
function_call [WITH ORDINALITY] [[AS] table_alias [(column_alias [, ... ])]]
ROWS FROM( function_call [, ... ] ) [WITH ORDINALITY] [[AS] table_alias [(column_alias [, ... ])]]
```

`WITH ORDINALITY` 절을 지정하면 함수 결과 컬럼에 `bigint` 타입의 추가 컬럼이 생성된다. 이 컬럼은 함수 결과 집합의 행을 1부터 순서대로 번호를 매긴다. (이는 SQL 표준 구문인 `UNNEST ... WITH ORDINALITY`를 일반화한 것이다.) 기본적으로 이 컬럼은 `ordinality`로 명명되지만, `AS` 절을 사용해 다른 이름을 지정할 수 있다.

특수 테이블 함수 `UNNEST`는 여러 배열 매개변수를 받아 각 매개변수에 대해 `UNNEST`를 개별적으로 호출한 후 `ROWS FROM` 구문으로 결합한 것과 동일한 수의 컬럼을 반환한다.

```sql
UNNEST( array_expression [, ... ] ) [WITH ORDINALITY] [[AS] table_alias [(column_alias [, ... ])]]
```

`table_alias`를 지정하지 않으면 함수 이름이 테이블 이름으로 사용된다. `ROWS FROM()` 구문의 경우 첫 번째 함수의 이름이 사용된다.

컬럼 별칭을 제공하지 않으면 기본 데이터 타입을 반환하는 함수의 경우 컬럼 이름은 함수 이름과 동일하다. 복합 타입을 반환하는 함수의 경우 결과 컬럼은 타입의 개별 속성 이름을 가진다.

다음은 몇 가지 예제다:

```sql
CREATE TABLE foo (fooid int, foosubid int, fooname text);

CREATE FUNCTION getfoo(int) RETURNS SETOF foo AS $
    SELECT * FROM foo WHERE fooid = $1;
$ LANGUAGE SQL;

SELECT * FROM getfoo(1) AS t1;

SELECT * FROM foo
    WHERE foosubid IN (
                        SELECT foosubid
                        FROM getfoo(foo.fooid) z
                        WHERE z.fooid = foo.fooid
                      );

CREATE VIEW vw_getfoo AS SELECT * FROM getfoo(1);

SELECT * FROM vw_getfoo;
```

경우에 따라 호출 방식에 따라 다른 컬럼 집합을 반환하는 테이블 함수를 정의하는 것이 유용할 수 있다. 이를 지원하기 위해 테이블 함수는 `OUT` 매개변수 없이 `record`라는 의사 타입을 반환하도록 선언할 수 있다. 이러한 함수를 쿼리에서 사용할 때는 쿼리 자체에서 예상되는 행 구조를 지정해야 한다. 이렇게 하면 시스템이 쿼리를 파싱하고 계획을 세울 수 있다. 이 구문은 다음과 같다:

```sql
function_call [AS] alias (column_definition [, ... ])
function_call AS [alias] (column_definition [, ... ])
ROWS FROM( ... function_call AS (column_definition [, ... ]) [, ... ] )
```

`ROWS FROM()` 구문을 사용하지 않을 경우 `column_definition` 목록은 `FROM` 항목에 첨부될 수 있는 컬럼 별칭 목록을 대체한다. 컬럼 정의의 이름은 컬럼 별칭으로 사용된다. `ROWS FROM()` 구문을 사용할 경우 각 멤버 함수에 대해 별도로 `column_definition` 목록을 첨부할 수 있다. 또는 멤버 함수가 하나뿐이고 `WITH ORDINALITY` 절이 없는 경우 `ROWS FROM()` 뒤에 컬럼 별칭 목록 대신 `column_definition` 목록을 작성할 수 있다.

다음 예제를 살펴보자:

```sql
SELECT *
    FROM dblink('dbname=mydb', 'SELECT proname, prosrc FROM pg_proc')
      AS t1(proname name, prosrc text)
    WHERE proname LIKE 'bytea%';
```

`dblink` 함수(dblink 모듈의 일부)는 원격 쿼리를 실행한다. 이 함수는 모든 종류의 쿼리에 사용될 수 있으므로 `record`를 반환하도록 선언된다. 실제 컬럼 집합은 호출 쿼리에서 지정해야 한다. 이렇게 하면 파서가 예를 들어 `*`가 무엇으로 확장되어야 하는지 알 수 있다.

다음은 `ROWS FROM`을 사용한 예제다:

```sql
SELECT *
FROM ROWS FROM
    (
        json_to_recordset('[{"a":40,"b":"foo"},{"a":"100","b":"bar"}]')
            AS (a INTEGER, b TEXT),
        generate_series(1, 3)
    ) AS x (p, q, s)
ORDER BY p;

  p  |  q  | s
-----+-----+---
  40 | foo | 1
 100 | bar | 2
     |     | 3
```

이 예제는 두 함수를 단일 `FROM` 대상으로 결합한다. `json_to_recordset()`은 두 컬럼을 반환하도록 지시받는다. 첫 번째 컬럼은 `integer` 타입이고 두 번째 컬럼은 `text` 타입이다. `generate_series()`의 결과는 직접 사용된다. `ORDER BY` 절은 컬럼 값을 정수로 정렬한다.


#### 7.2.1.5. `LATERAL` 서브쿼리

`FROM` 절에 나타나는 서브쿼리는 `LATERAL` 키워드를 앞에 붙일 수 있다. 이렇게 하면 서브쿼리가 앞서 나온 `FROM` 항목의 컬럼을 참조할 수 있다. (`LATERAL` 없이는 각 서브쿼리가 독립적으로 평가되기 때문에 다른 `FROM` 항목을 참조할 수 없다.)

`FROM` 절에 나타나는 테이블 함수에도 `LATERAL` 키워드를 붙일 수 있지만, 함수의 경우 이 키워드는 선택 사항이다. 함수의 인자는 어떤 경우든 앞서 나온 `FROM` 항목의 컬럼을 참조할 수 있다.

`LATERAL` 항목은 `FROM` 리스트의 최상위 레벨에 나타날 수도 있고, `JOIN` 트리 내부에 나타날 수도 있다. 후자의 경우, `LATERAL` 항목은 자신이 오른쪽에 위치한 `JOIN`의 왼쪽에 있는 모든 항목을 참조할 수 있다.

`FROM` 항목이 `LATERAL` 교차 참조를 포함할 때, 평가는 다음과 같이 진행된다: 교차 참조된 컬럼을 제공하는 `FROM` 항목의 각 행 또는 여러 `FROM` 항목의 행 집합에 대해, `LATERAL` 항목은 해당 행 또는 행 집합의 컬럼 값을 사용하여 평가된다. 결과로 나온 행은 일반적으로 계산된 행과 조인된다. 이 과정은 컬럼 소스 테이블의 각 행 또는 행 집합에 대해 반복된다.

`LATERAL`의 간단한 예는 다음과 같다:

```sql
SELECT * FROM foo, LATERAL (SELECT * FROM bar WHERE bar.id = foo.bar_id) ss;
```

이 쿼리는 다음과 같은 더 전통적인 쿼리와 정확히 같은 결과를 반환하기 때문에 특별히 유용하지 않다:

```sql
SELECT * FROM foo, bar WHERE bar.id = foo.bar_id;
```

`LATERAL`은 주로 조인할 행을 계산하기 위해 교차 참조된 컬럼이 필요할 때 유용하다. 일반적인 응용 사례는 집합 반환 함수에 인자 값을 제공하는 것이다. 예를 들어, `vertices(polygon)`이 다각형의 꼭짓점 집합을 반환한다고 가정할 때, 테이블에 저장된 다각형의 가까운 꼭짓점을 다음과 같이 식별할 수 있다:

```sql
SELECT p1.id, p2.id, v1, v2
FROM polygons p1, polygons p2,
     LATERAL vertices(p1.poly) v1,
     LATERAL vertices(p2.poly) v2
WHERE (v1 <-> v2) < 10 AND p1.id != p2.id;
```

이 쿼리는 다음과 같이 작성할 수도 있다:

```sql
SELECT p1.id, p2.id, v1, v2
FROM polygons p1 CROSS JOIN LATERAL vertices(p1.poly) v1,
     polygons p2 CROSS JOIN LATERAL vertices(p2.poly) v2
WHERE (v1 <-> v2) < 10 AND p1.id != p2.id;
```

또는 여러 다른 동등한 형식으로 작성할 수 있다. (이미 언급했듯이, 이 예제에서는 `LATERAL` 키워드가 필요하지 않지만, 명확성을 위해 사용한다.)

`LATERAL` 서브쿼리에 `LEFT JOIN`을 사용하는 것은 특히 편리하다. 이렇게 하면 `LATERAL` 서브쿼리가 해당 행에 대해 아무런 행을 생성하지 않더라도 소스 행이 결과에 나타난다. 예를 들어, `get_product_names()`가 제조업체가 만든 제품의 이름을 반환하지만, 우리 테이블의 일부 제조업체가 현재 제품을 생산하지 않는 경우, 다음과 같이 어떤 제조업체가 해당되는지 찾을 수 있다:

```sql
SELECT m.name
FROM manufacturers m LEFT JOIN LATERAL get_product_names(m.id) pname ON true
WHERE pname IS NULL;
```

### 7.2.2. `WHERE` 절

`WHERE` 절의 문법은 다음과 같다.

```sql
WHERE search_condition
```

여기서 *`search_condition`*은 `boolean` 타입의 값을 반환하는 값 표현식이다. (자세한 내용은 [4.2장](https://www.postgresql.org/docs/17/sql-expressions.html "4.2. Value Expressions") 참조)

`FROM` 절의 처리가 끝나면, 파생된 가상 테이블의 각 행은 검색 조건에 대해 검사된다. 조건의 결과가 `true`이면 해당 행은 출력 테이블에 유지되고, `false` 또는 `null`이면 해당 행은 제외된다. 검색 조건은 일반적으로 `FROM` 절에서 생성된 테이블의 최소한 하나의 컬럼을 참조한다. 이는 필수는 아니지만, 그렇지 않으면 `WHERE` 절은 거의 쓸모가 없게 된다.

### 참고 사항

내부 조인의 조인 조건은 `WHERE` 절이나 `JOIN` 절 어디에 작성해도 된다. 예를 들어, 다음 테이블 표현식들은 모두 동일한 결과를 반환한다:

```sql
FROM a, b WHERE a.id = b.id AND b.val > 5
```

그리고:

```sql
FROM a INNER JOIN b ON (a.id = b.id) WHERE b.val > 5
```

또는:

```sql
FROM a NATURAL JOIN b WHERE b.val > 5
```

어떤 방식을 사용할지는 주로 스타일의 문제다. `FROM` 절의 `JOIN` 구문은 SQL 표준이지만, 다른 SQL 데이터베이스 관리 시스템에서는 호환성이 떨어질 수 있다. 외부 조인의 경우에는 선택의 여지가 없다. 반드시 `FROM` 절에서 처리해야 한다. 외부 조인의 `ON` 또는 `USING` 절은 `WHERE` 조건과 동일하지 않다. 왜냐하면 이들은 최종 결과에서 행을 제거할 뿐만 아니라, 일치하지 않는 입력 행도 추가하기 때문이다.

다음은 `WHERE` 절의 몇 가지 예제다:

```sql
SELECT ... FROM fdt WHERE c1 > 5

SELECT ... FROM fdt WHERE c1 IN (1, 2, 3)

SELECT ... FROM fdt WHERE c1 IN (SELECT c1 FROM t2)

SELECT ... FROM fdt WHERE c1 IN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10)

SELECT ... FROM fdt WHERE c1 BETWEEN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10) AND 100

SELECT ... FROM fdt WHERE EXISTS (SELECT c1 FROM t2 WHERE c2 > fdt.c1)
```

여기서 `fdt`는 `FROM` 절에서 파생된 테이블이다. `WHERE` 절의 검색 조건을 만족하지 않는 행은 `fdt`에서 제거된다. 스칼라 서브쿼리가 값 표현식으로 사용된 것을 주목하라. 다른 쿼리와 마찬가지로, 서브쿼리도 복잡한 테이블 표현식을 사용할 수 있다. 또한 서브쿼리에서 `fdt`가 어떻게 참조되는지 살펴보라. `c1`을 `fdt.c1`로 한정하는 것은 서브쿼리의 파생 입력 테이블에서도 `c1`이라는 열 이름이 존재할 때만 필요하다. 하지만 열 이름을 한정하면 필요하지 않은 경우에도 명확성을 높일 수 있다. 이 예제는 외부 쿼리의 열 이름 범위가 내부 쿼리로 어떻게 확장되는지를 보여준다.

### 7.2.3. `GROUP BY`와 `HAVING` 절

`WHERE` 필터를 통과한 후, 파생된 입력 테이블은 `GROUP BY` 절을 사용해 그룹화될 수 있으며, `HAVING` 절을 사용해 그룹 행을 제거할 수 있다.

```sql
SELECT select_list
    FROM ...
    [WHERE ...]
    GROUP BY grouping_column_reference [, grouping_column_reference]...
```

`GROUP BY` 절은 테이블에서 나열된 모든 열에 동일한 값을 가진 행들을 그룹화하는 데 사용된다. 열이 나열된 순서는 중요하지 않다. 이 절의 효과는 공통된 값을 가진 각 행 집합을 하나의 그룹 행으로 결합하는 것이다. 이는 출력에서 중복을 제거하거나 이러한 그룹에 적용되는 집계를 계산하기 위해 수행된다. 예를 들어:

```sql
=> SELECT * FROM test1;
 x | y
---+---
 a | 3
 c | 2
 b | 5
 a | 1
(4 rows)

=> SELECT x FROM test1 GROUP BY x;
 x
---
 a
 b
 c
(3 rows)
```

두 번째 쿼리에서 `SELECT * FROM test1 GROUP BY x`와 같이 작성할 수 없다. 왜냐하면 각 그룹과 연관될 수 있는 `y` 열의 단일 값이 존재하지 않기 때문이다. 그룹화된 열은 각 그룹에서 단일 값을 가지므로 `SELECT` 목록에서 참조할 수 있다.

일반적으로 테이블이 그룹화되면, `GROUP BY`에 나열되지 않은 열은 집계 표현식에서만 참조할 수 있다. 집계 표현식을 사용한 예제는 다음과 같다:

```sql
=> SELECT x, sum(y) FROM test1 GROUP BY x;
 x | sum
---+-----
 a |   4
 b |   5
 c |   2
(3 rows)
```

여기서 `sum`은 전체 그룹에 대해 단일 값을 계산하는 집계 함수이다. 사용 가능한 집계 함수에 대한 더 많은 정보는 [9.21절](https://www.postgresql.org/docs/17/functions-aggregate.html "9.21. Aggregate Functions")에서 찾을 수 있다.

### 팁

집계 표현식 없이 그룹화를 수행하면 특정 컬럼의 고유한 값 집합을 계산하는 효과를 얻을 수 있다. 이는 `DISTINCT` 절을 사용해서도 동일한 결과를 얻을 수 있다([7.3.3절](https://www.postgresql.org/docs/17/queries-select-lists.html#QUERIES-DISTINCT "7.3.3. DISTINCT") 참조).

다음은 또 다른 예제로, 모든 제품의 총 매출이 아니라 각 제품별 총 매출을 계산한다:

```sql
SELECT product_id, p.name, (sum(s.units) * p.price) AS sales
    FROM products p LEFT JOIN sales s USING (product_id)
    GROUP BY product_id, p.name, p.price;
```

이 예제에서 `product_id`, `p.name`, `p.price` 컬럼은 쿼리의 `SELECT` 목록에서 참조되기 때문에 `GROUP BY` 절에 포함되어야 한다(단, 아래 설명 참조). 반면 `s.units` 컬럼은 집계 표현식(`sum(...)`)에서만 사용되므로 `GROUP BY` 목록에 포함할 필요가 없다. 이 쿼리는 각 제품에 대해 해당 제품의 모든 판매를 요약한 행을 반환한다.

만약 `products` 테이블에서 `product_id`가 기본 키로 설정되어 있다면, 위 예제에서 `product_id`만으로 그룹화해도 충분하다. 이 경우 `name`과 `price`는 `product_id`에 **함수적으로 종속**되기 때문에, 각 `product_id` 그룹에 대해 어떤 `name`과 `price` 값을 반환할지 모호하지 않다.

엄격한 SQL 표준에서는 `GROUP BY` 절이 소스 테이블의 컬럼만을 그룹화할 수 있지만, PostgreSQL은 이를 확장하여 `SELECT` 목록의 컬럼도 그룹화할 수 있도록 지원한다. 또한 단순한 컬럼 이름 대신 값 표현식을 사용한 그룹화도 허용한다.

`GROUP BY`를 사용해 테이블을 그룹화한 후, 특정 그룹만을 선택하려면 `HAVING` 절을 사용할 수 있다. 이는 `WHERE` 절과 유사하게 동작하며, 결과에서 특정 그룹을 제외한다. 구문은 다음과 같다:

```sql
SELECT select_list FROM ... [WHERE ...] GROUP BY ... HAVING boolean_expression
```

`HAVING` 절의 표현식은 그룹화된 표현식과 그룹화되지 않은 표현식(반드시 집계 함수를 포함해야 함) 모두를 참조할 수 있다.

예제:

```sql
=> SELECT x, sum(y) FROM test1 GROUP BY x HAVING sum(y) > 3;
 x | sum
---+-----
 a |   4
 b |   5
(2 rows)

=> SELECT x, sum(y) FROM test1 GROUP BY x HAVING x < 'c';
 x | sum
---+-----
 a |   4
 b |   5
(2 rows)
```

다음은 더 현실적인 예제다:

```sql
SELECT product_id, p.name, (sum(s.units) * (p.price - p.cost)) AS profit
    FROM products p LEFT JOIN sales s USING (product_id)
    WHERE s.date > CURRENT_DATE - INTERVAL '4 weeks'
    GROUP BY product_id, p.name, p.price, p.cost
    HAVING sum(p.price * s.units) > 5000;
```

위 예제에서 `WHERE` 절은 그룹화되지 않은 컬럼을 기준으로 행을 선택한다(지난 4주 동안의 판매에 해당하는 행만 선택). 반면 `HAVING` 절은 총 매출이 5000을 초과하는 그룹만 결과에 포함한다. 여기서 주목할 점은 쿼리의 모든 부분에서 동일한 집계 표현식을 사용할 필요가 없다는 것이다.

쿼리에 집계 함수 호출이 포함되어 있지만 `GROUP BY` 절이 없는 경우에도 그룹화는 발생한다. 이 경우 결과는 단일 그룹 행이 된다(단, `HAVING` 절에 의해 해당 행이 제외될 수도 있다). 마찬가지로, 집계 함수 호출이나 `GROUP BY` 절이 없더라도 `HAVING` 절이 포함되어 있다면 그룹화가 발생한다.

### 7.2.4. `GROUPING SETS`, `CUBE`, 그리고 `ROLLUP`

앞서 설명한 기본적인 그룹화 작업보다 더 복잡한 그룹화를 수행하려면 *그룹화 집합(grouping sets)* 개념을 사용할 수 있다. `FROM`과 `WHERE` 절로 선택된 데이터는 각각 지정된 그룹화 집합에 따라 별도로 그룹화된다. 그런 다음, 각 그룹에 대해 집계 함수를 계산하고, 그 결과를 반환한다. 이 과정은 단순한 `GROUP BY` 절과 동일하게 작동한다. 예를 들어:

```sql
=> SELECT * FROM items_sold;
 brand | size | sales
-------+------+-------
 Foo   | L    |  10
 Foo   | M    |  20
 Bar   | M    |  15
 Bar   | L    |  5
(4 rows)

=> SELECT brand, size, sum(sales) FROM items_sold GROUP BY GROUPING SETS ((brand), (size), ());
 brand | size | sum
-------+------+-----
 Foo   |      |  30
 Bar   |      |  20
       | L    |  15
       | M    |  35
       |      |  50
(5 rows)
```

`GROUPING SETS`의 각 하위 목록은 0개 이상의 컬럼이나 표현식을 지정할 수 있으며, 이는 `GROUP BY` 절에 직접 사용된 것과 동일한 방식으로 해석된다. 빈 그룹화 집합은 모든 행을 단일 그룹으로 집계한다는 의미이다. 이는 `GROUP BY` 절 없이 집계 함수를 사용할 때와 동일한 방식으로 작동한다.

그룹화 집합에서 특정 컬럼이 나타나지 않는 경우, 해당 컬럼이나 표현식은 결과 행에서 `NULL` 값으로 대체된다. 특정 출력 행이 어떤 그룹화에서 나온 것인지 구분하려면 [표 9.64](https://www.postgresql.org/docs/17/functions-aggregate.html#FUNCTIONS-GROUPING-TABLE "표 9.64. 그룹화 작업")를 참고한다.

두 가지 일반적인 그룹화 집합 유형을 간단히 지정할 수 있는 약어 표기법이 제공된다. 다음과 같은 형식의 절:

```sql
ROLLUP ( e1, e2, e3, ... )
```

는 주어진 표현식 목록과 그 목록의 모든 접두사(빈 목록 포함)를 나타낸다. 따라서 이는 다음과 동일하다:

```sql
GROUPING SETS (
    ( e1, e2, e3, ... ),
    ...
    ( e1, e2 ),
    ( e1 ),
    ( )
)
```

이 방식은 계층적 데이터 분석에 자주 사용된다. 예를 들어, 부서별, 부문별, 회사 전체의 총 급여를 계산할 때 유용하다.

다음과 같은 형식의 절:

```sql
CUBE ( e1, e2, ... )
```

는 주어진 목록과 그 목록의 모든 가능한 부분집합(즉, 멱집합)을 나타낸다. 따라서:

```sql
CUBE ( a, b, c )
```

는 다음과 동일하다:

```sql
GROUPING SETS (
    ( a, b, c ),
    ( a, b    ),
    ( a,    c ),
    ( a       ),
    (    b, c ),
    (    b    ),
    (       c ),
    (         )
)
```

`CUBE` 또는 `ROLLUP` 절의 개별 요소는 단일 표현식일 수도 있고, 괄호로 묶인 하위 목록일 수도 있다. 후자의 경우, 하위 목록은 개별 그룹화 집합을 생성할 때 단일 단위로 처리된다. 예를 들어:

```sql
CUBE ( (a, b), (c, d) )
```

는 다음과 동일하다:

```sql
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b       ),
    (       c, d ),
    (            )
)
```

그리고:

```sql
ROLLUP ( a, (b, c), d )
```

는 다음과 동일하다:

```sql
GROUPING SETS (
    ( a, b, c, d ),
    ( a, b, c    ),
    ( a          ),
    (            )
)
```

`CUBE`와 `ROLLUP` 구조는 `GROUP BY` 절에 직접 사용하거나, `GROUPING SETS` 절 내에 중첩해서 사용할 수 있다. 하나의 `GROUPING SETS` 절이 다른 `GROUPING SETS` 절 내에 중첩된 경우, 내부 절의 모든 요소가 외부 절에 직접 작성된 것과 동일한 효과를 낸다.

단일 `GROUP BY` 절에 여러 그룹화 항목을 지정하면, 최종 그룹화 집합 목록은 각 항목의 데카르트 곱(Cartesian product)이 된다. 예를 들어:

```sql
GROUP BY a, CUBE (b, c), GROUPING SETS ((d), (e))
```

는 다음과 동일하다:

```sql
GROUP BY GROUPING SETS (
    (a, b, c, d), (a, b, c, e),
    (a, b, d),    (a, b, e),
    (a, c, d),    (a, c, e),
    (a, d),       (a, e)
)
```

여러 그룹화 항목을 함께 지정할 때, 최종 그룹화 집합 목록에 중복이 포함될 수 있다. 예를 들어:

```sql
GROUP BY ROLLUP (a, b), ROLLUP (a, c)
```

는 다음과 동일하다:

```sql
GROUP BY GROUPING SETS (
    (a, b, c),
    (a, b),
    (a, b),
    (a, c),
    (a),
    (a),
    (a, c),
    (a),
    ()
)
```

이러한 중복이 원하지 않는 경우, `GROUP BY` 절에 직접 `DISTINCT`를 사용해 제거할 수 있다. 따라서:

```sql
GROUP BY DISTINCT ROLLUP (a, b), ROLLUP (a, c)
```

는 다음과 동일하다:

```sql
GROUP BY GROUPING SETS (
    (a, b, c),
    (a, b),
    (a, c),
    (a),
    ()
)
```

이는 `SELECT DISTINCT`를 사용하는 것과는 다르다. 출력 행에는 여전히 중복이 포함될 수 있다. 그룹화되지 않은 컬럼 중 하나에 `NULL`이 포함된 경우, 해당 컬럼이 그룹화되었을 때 사용된 `NULL`과 구분할 수 없다.

### 참고 사항

일반적으로 `(a, b)`와 같은 구문은 표현식에서 [행 생성자](https://www.postgresql.org/docs/17/sql-expressions.html#SQL-SYNTAX-ROW-CONSTRUCTORS "4.2.13. Row Constructors")로 인식된다. 하지만 `GROUP BY` 절 내에서는 표현식의 최상위 수준에서 이 규칙이 적용되지 않는다. `GROUP BY` 절에서 `(a, b)`는 앞서 설명한 대로 표현식의 목록으로 해석된다. 만약 그룹화 표현식에서 행 생성자를 사용해야 하는 특별한 이유가 있다면, `ROW(a, b)`와 같은 구문을 사용해야 한다.

### 7.2.5. 윈도우 함수 처리

쿼리에 윈도우 함수가 포함되어 있다면(자세한 내용은 [3.5장](https://www.postgresql.org/docs/17/tutorial-window.html "3.5. Window Functions"), [9.22장](https://www.postgresql.org/docs/17/functions-window.html "9.22. Window Functions"), [4.2.8장](https://www.postgresql.org/docs/17/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS "4.2.8. Window Function Calls") 참조), 이 함수들은 그룹화, 집계, 그리고 `HAVING` 필터링이 수행된 후에 평가된다. 즉, 쿼리에서 집계 함수, `GROUP BY`, 또는 `HAVING`을 사용한다면, 윈도우 함수가 보는 행은 `FROM`/`WHERE` 절의 원본 테이블 행이 아니라 그룹화된 행이다.

여러 윈도우 함수를 사용할 때, 윈도우 정의에서 문법적으로 동일한 `PARTITION BY`와 `ORDER BY` 절을 가진 모든 윈도우 함수는 데이터를 한 번만 스캔하면서 평가된다. 따라서 `ORDER BY`가 고유한 순서를 결정하지 않더라도, 동일한 정렬 순서를 보장한다. 그러나 서로 다른 `PARTITION BY` 또는 `ORDER BY` 명세를 가진 함수의 평가에 대해서는 어떠한 보장도 하지 않는다. (이 경우, 윈도우 함수 평가 사이에 정렬 단계가 필요하며, 이 정렬은 `ORDER BY`가 동등하다고 판단한 행의 순서를 보존하지 않을 수 있다.)

현재, 윈도우 함수는 항상 미리 정렬된 데이터를 필요로 하므로, 쿼리 결과는 윈도우 함수의 `PARTITION BY`/`ORDER BY` 절 중 하나에 따라 정렬된다. 그러나 이에 의존하는 것은 권장하지 않는다. 특정 방식으로 결과를 정렬하고 싶다면 명시적인 최상위 `ORDER BY` 절을 사용해야 한다.

## 7.3. Select 목록의 역할

앞서 살펴본 내용에서 `SELECT` 명령어의 테이블 표현식은 여러 테이블과 뷰를 결합하거나, 행을 제거하거나, 그룹화하는 등의 작업을 통해 중간 가상 테이블을 생성한다. 이렇게 생성된 중간 테이블은 최종적으로 *select 목록*에 의해 처리된다. select 목록은 중간 테이블의 어떤 *컬럼*을 실제로 출력할지 결정한다.

### 7.3.1. Select 목록 항목

가장 간단한 형태의 select 목록은 `*`를 사용하는 것이다. 이는 테이블 표현식이 생성하는 모든 컬럼을 출력한다. 그 외의 경우, select 목록은 쉼표로 구분된 값 표현식의 목록이다(이 표현식은 [4.2절](https://www.postgresql.org/docs/17/sql-expressions.html "4.2. Value Expressions")에서 정의한 대로이다). 예를 들어, 컬럼 이름의 목록일 수 있다:

```sql
SELECT a, b, c FROM ...
```

여기서 `a`, `b`, `c`는 `FROM` 절에서 참조된 테이블의 실제 컬럼 이름이거나, [7.2.1.2절](https://www.postgresql.org/docs/17/queries-table-expressions.html#QUERIES-TABLE-ALIASES "7.2.1.2. Table and Column Aliases")에서 설명한 대로 부여된 별칭일 수 있다. select 목록에서 사용 가능한 이름 공간은 `WHERE` 절과 동일하다. 단, 그룹화를 사용하는 경우에는 `HAVING` 절과 동일한 이름 공간을 가진다.

여러 테이블에 동일한 이름의 컬럼이 존재하는 경우, 테이블 이름도 함께 지정해야 한다. 예를 들면 다음과 같다:

```sql
SELECT tbl1.a, tbl2.a, tbl1.b FROM ...
```

여러 테이블을 다룰 때, 특정 테이블의 모든 컬럼을 요청하는 것도 유용할 수 있다:

```sql
SELECT tbl1.*, tbl2.a FROM ...
```

`*` 표기법에 대한 더 자세한 내용은 [8.16.5절](https://www.postgresql.org/docs/17/rowtypes.html#ROWTYPES-USAGE "8.16.5. Using Composite Types in Queries")을 참고한다.

select 목록에서 임의의 값 표현식을 사용하면, 개념적으로 반환된 테이블에 새로운 가상 컬럼이 추가된다. 값 표현식은 각 결과 행에 대해 한 번씩 평가되며, 행의 값은 컬럼 참조를 대체한다. 그러나 select 목록의 표현식이 반드시 `FROM` 절의 테이블 표현식에 있는 컬럼을 참조할 필요는 없다. 예를 들어, 상수 산술 표현식일 수도 있다.

### 7.3.2. 컬럼 레이블

`SELECT` 목록에 있는 항목들은 `ORDER BY` 절에서 사용하거나 클라이언트 애플리케이션에서 표시하기 위해 이름을 지정할 수 있다. 예를 들어:

```sql
SELECT a AS value, b + c AS sum FROM ...
```

`AS`를 사용해 출력 컬럼 이름을 지정하지 않으면, 시스템이 기본 컬럼 이름을 할당한다. 단순한 컬럼 참조의 경우, 참조된 컬럼의 이름을 사용한다. 함수 호출의 경우, 함수 이름을 사용한다. 복잡한 표현식의 경우, 시스템이 일반적인 이름을 생성한다.

`AS` 키워드는 보통 선택 사항이지만, 원하는 컬럼 이름이 PostgreSQL 키워드와 일치하는 경우에는 `AS`를 작성하거나 컬럼 이름을 큰따옴표로 감싸야 모호함을 피할 수 있다. ([부록 C](https://www.postgresql.org/docs/17/sql-keywords-appendix.html "부록 C. SQL 키워드")에서는 `AS`를 컬럼 레이블로 사용해야 하는 키워드를 보여준다.) 예를 들어, `FROM`은 그러한 키워드 중 하나이므로 다음은 작동하지 않는다:

```sql
SELECT a from, b + c AS sum FROM ...
```

하지만 다음 중 하나는 작동한다:

```sql
SELECT a AS from, b + c AS sum FROM ...
SELECT a "from", b + c AS sum FROM ...
```

향후 키워드 추가에 대비해 가장 안전한 방법은 항상 `AS`를 작성하거나 출력 컬럼 이름을 큰따옴표로 감싸는 것이다.

### 주의 사항

여기서 출력 컬럼의 이름 지정은 `FROM` 절에서 사용한 방식과 다르다([7.2.1.2절](https://www.postgresql.org/docs/17/queries-table-expressions.html#QUERIES-TABLE-ALIASES "7.2.1.2. 테이블 및 컬럼 별칭") 참조). 동일한 컬럼을 두 번 이상 다른 이름으로 지정할 수 있지만, 최종적으로 전달되는 이름은 `select` 목록에서 할당한 이름이다.

### 7.3.3. `DISTINCT` 사용법

`SELECT` 목록을 처리한 후, 결과 테이블에서 중복된 행을 제거할 수 있다. `DISTINCT` 키워드를 `SELECT` 바로 뒤에 작성하여 이를 지정한다:

```sql
SELECT DISTINCT select_list ...
```

(`DISTINCT` 대신 `ALL` 키워드를 사용하면 모든 행을 유지하는 기본 동작을 지정할 수 있다.)

두 행은 적어도 하나의 컬럼 값이 다를 경우 서로 다른 것으로 간주한다. 이 비교에서 `NULL` 값은 동일한 것으로 처리한다.

또는, 임의의 표현식을 사용하여 어떤 행이 서로 다른 것으로 간주될지 결정할 수도 있다:

```sql
SELECT DISTINCT ON (expression [, expression ...]) select_list ...
```

여기서 *`expression`*은 모든 행에 대해 평가되는 임의의 값 표현식이다. 모든 표현식이 동일한 행 집합은 중복으로 간주되며, 출력에는 해당 집합의 첫 번째 행만 유지된다. 단, `DISTINCT` 필터에 도달하는 행의 순서가 고유하도록 충분한 컬럼으로 정렬하지 않으면 "첫 번째 행"이 예측 불가능할 수 있다. (`DISTINCT ON` 처리는 `ORDER BY` 정렬 이후에 발생한다.)

`DISTINCT ON` 절은 SQL 표준에 포함되지 않으며, 결과가 불확실할 수 있기 때문에 때로는 나쁜 스타일로 간주된다. `GROUP BY`와 `FROM` 절의 서브쿼리를 적절히 사용하면 이 구문을 피할 수 있지만, 종종 가장 편리한 대안이 되기도 한다.

## 7.4. 쿼리 결합 (UNION, INTERSECT, EXCEPT)

두 쿼리의 결과를 집합 연산자인 합집합(UNION), 교집합(INTERSECT), 차집합(EXCEPT)을 사용해 결합할 수 있다. 기본 구문은 다음과 같다.

```sql
query1 UNION [ALL] query2
query1 INTERSECT [ALL] query2
query1 EXCEPT [ALL] query2
```

여기서 `query1`과 `query2`는 지금까지 다룬 모든 기능을 사용할 수 있는 쿼리다.

`UNION`은 `query2`의 결과를 `query1`의 결과에 추가한다. 단, 행이 반환되는 순서는 보장하지 않는다. 또한 `UNION ALL`을 사용하지 않는 한, `DISTINCT`와 마찬가지로 중복 행을 제거한다.

`INTERSECT`는 `query1`과 `query2`의 결과에 모두 존재하는 행을 반환한다. `INTERSECT ALL`을 사용하지 않는 한, 중복 행을 제거한다.

`EXCEPT`는 `query1`의 결과에 있지만 `query2`의 결과에는 없는 행을 반환한다. 이 연산을 두 쿼리의 차집합이라고도 한다. 마찬가지로 `EXCEPT ALL`을 사용하지 않는 한, 중복 행을 제거한다.

두 쿼리의 합집합, 교집합, 차집합을 계산하려면 두 쿼리가 "합집합 호환성(union compatible)"을 가져야 한다. 이는 두 쿼리가 동일한 수의 컬럼을 반환하고, 해당 컬럼의 데이터 타입이 호환되어야 함을 의미한다. 자세한 내용은 [10.5절](https://www.postgresql.org/docs/17/typeconv-union-case.html "10.5. UNION, CASE, and Related Constructs")을 참고한다.

집합 연산자를 결합할 수도 있다. 예를 들어,

```sql
query1 UNION query2 EXCEPT query3
```

위 쿼리는 다음 쿼리와 동일하다.

```sql
(query1 UNION query2) EXCEPT query3
```

여기서 볼 수 있듯이, 괄호를 사용해 연산 순서를 제어할 수 있다. 괄호를 사용하지 않으면 `UNION`과 `EXCEPT`는 왼쪽에서 오른쪽으로 결합되지만, `INTERSECT`는 이 두 연산자보다 더 강하게 결합된다. 따라서,

```sql
query1 UNION query2 INTERSECT query3
```

위 쿼리는 다음 쿼리와 동일하다.

```sql
query1 UNION (query2 INTERSECT query3)
```

개별 `query`를 괄호로 감쌀 수도 있다. 이는 `query`가 `LIMIT`과 같은 절을 사용해야 할 때 중요하다. 괄호를 사용하지 않으면 구문 오류가 발생하거나, 해당 절이 집합 연산의 출력에 적용되는 것으로 해석될 수 있다. 예를 들어,

```sql
SELECT a FROM b UNION SELECT x FROM y LIMIT 10
```

위 쿼리는 허용되지만, 이는 다음 쿼리와 동일하다.

```sql
(SELECT a FROM b UNION SELECT x FROM y) LIMIT 10
```

다음 쿼리와는 다르다.

```sql
SELECT a FROM b UNION (SELECT x FROM y LIMIT 10)
```

## 7.5. 행 정렬하기 (ORDER BY)

쿼리가 결과 테이블을 생성한 후(select 목록이 처리된 후), 선택적으로 결과를 정렬할 수 있다. 정렬을 선택하지 않으면 행은 특정하지 않은 순서로 반환된다. 이 경우 실제 순서는 스캔 및 조인 계획 타입과 디스크 상의 순서에 따라 달라지지만, 이 순서에 의존해서는 안 된다. 특정 출력 순서를 보장하려면 명시적으로 정렬 단계를 선택해야 한다.

`ORDER BY` 절은 정렬 순서를 지정한다:

```sql
SELECT select_list
    FROM table_expression
    ORDER BY sort_expression1 [ASC | DESC] [NULLS { FIRST | LAST }]
             [, sort_expression2 [ASC | DESC] [NULLS { FIRST | LAST }] ...]
```

정렬 표현식은 쿼리의 select 목록에서 유효한 어떤 표현식이든 가능하다. 예를 들면 다음과 같다:

```sql
SELECT a, b FROM table1 ORDER BY a + b, c;
```

하나 이상의 표현식을 지정하면, 앞선 값에 따라 동일한 행을 나중 값으로 정렬한다. 각 표현식 뒤에는 선택적으로 `ASC` 또는 `DESC` 키워드를 붙여 오름차순 또는 내림차순 정렬 방향을 설정할 수 있다. `ASC` 순서가 기본값이다. 오름차순은 작은 값을 먼저 배치하며, "작은" 값은 `<` 연산자로 정의된다. 마찬가지로 내림차순은 `>` 연산자로 결정된다.

`NULLS FIRST`와 `NULLS LAST` 옵션은 정렬 순서에서 NULL 값이 비 NULL 값보다 앞에 오는지 뒤에 오는지 결정한다. 기본적으로 NULL 값은 비 NULL 값보다 큰 것으로 간주된다. 즉, `DESC` 순서에서는 `NULLS FIRST`가 기본값이고, 그렇지 않으면 `NULLS LAST`가 기본값이다.

정렬 옵션은 각 정렬 컬럼에 대해 독립적으로 고려된다. 예를 들어 `ORDER BY x, y DESC`는 `ORDER BY x ASC, y DESC`를 의미하며, 이는 `ORDER BY x DESC, y DESC`와 같지 않다.

*`sort_expression`*은 출력 컬럼의 레이블이나 번호일 수도 있다:

```sql
SELECT a + b AS sum, c FROM table1 ORDER BY sum;
SELECT a, max(b) FROM table1 GROUP BY a ORDER BY 1;
```

두 예제 모두 첫 번째 출력 컬럼으로 정렬한다. 출력 컬럼 이름은 단독으로 사용해야 하며, 표현식에서 사용할 수 없다. 예를 들어 다음은 올바르지 않다:

```sql
SELECT a + b AS sum, c FROM table1 ORDER BY sum + c;          -- 잘못된 사용
```

이 제한은 모호성을 줄이기 위해 존재한다. `ORDER BY` 항목이 출력 컬럼 이름이나 테이블 표현식의 컬럼과 일치할 수 있는 단순한 이름인 경우 여전히 모호성이 있다. 이러한 경우 출력 컬럼이 사용된다. 이는 `AS`를 사용해 출력 컬럼 이름을 다른 테이블 컬럼 이름과 일치하도록 변경할 때 혼란을 일으킬 수 있다.

`ORDER BY`는 `UNION`, `INTERSECT`, 또는 `EXCEPT` 조합의 결과에 적용할 수 있지만, 이 경우 출력 컬럼 이름이나 번호로만 정렬할 수 있으며 표현식으로는 정렬할 수 없다.

## 7.6. LIMIT과 OFFSET 활용

`LIMIT`과 `OFFSET`은 쿼리 결과에서 특정 부분만 가져오는 데 사용한다. 전체 쿼리 결과 중 일부 행만 선택적으로 반환할 수 있다.

```sql
SELECT select_list
    FROM table_expression
    [ ORDER BY ... ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start ]
```

`LIMIT`에 숫자를 지정하면 해당 숫자만큼의 행을 반환한다. 쿼리 결과가 지정한 숫자보다 적으면 그만큼만 반환한다. `LIMIT ALL`은 `LIMIT` 절을 생략한 것과 동일하다. `LIMIT`에 NULL을 지정해도 같은 효과를 얻는다.

`OFFSET`은 결과를 반환하기 전에 지정한 숫자만큼의 행을 건너뛴다. `OFFSET 0`은 `OFFSET` 절을 생략한 것과 같다. `OFFSET`에 NULL을 지정해도 동일하게 작동한다.

`OFFSET`과 `LIMIT`을 함께 사용하면 `OFFSET`으로 지정한 행을 건너뛴 후 `LIMIT`으로 지정한 행을 반환한다.

`LIMIT`을 사용할 때는 반드시 `ORDER BY` 절을 함께 사용해야 한다. `ORDER BY`로 결과 행의 순서를 명확히 지정하지 않으면 쿼리 결과의 일부를 예측할 수 없게 된다. 예를 들어, 10번째부터 20번째 행을 요청한다고 해도, 어떤 순서로 정렬된 10번째부터 20번째 행인지 알 수 없다. `ORDER BY`를 사용해야만 결과의 순서를 보장할 수 있다.

쿼리 최적화기는 `LIMIT`을 고려해 실행 계획을 생성한다. 따라서 `LIMIT`과 `OFFSET` 값에 따라 서로 다른 실행 계획이 만들어질 수 있다. 이는 결과 행의 순서가 달라질 수 있음을 의미한다. `ORDER BY`로 결과 순서를 명시하지 않으면, `LIMIT`과 `OFFSET`을 다르게 설정해 쿼리 결과의 다른 부분을 선택할 때 일관되지 않은 결과를 얻을 수 있다. 이는 버그가 아니라 SQL의 기본 동작 방식이다. SQL은 `ORDER BY`를 사용하지 않으면 특정 순서로 결과를 반환한다고 보장하지 않는다.

`OFFSET`으로 건너뛴 행도 서버 내부에서 계산해야 한다. 따라서 큰 `OFFSET` 값을 사용하면 성능이 저하될 수 있다.

## 7.7. VALUES 리스트

`VALUES` 구문은 실제로 디스크에 테이블을 생성하고 데이터를 채우지 않고도 쿼리에서 사용할 수 있는 "상수 테이블"을 생성하는 방법을 제공한다. 구문은 다음과 같다.

```sql
VALUES ( expression [, ...] ) [, ...]
```

각 괄호로 묶인 표현식 리스트는 테이블의 한 행을 생성한다. 모든 리스트는 동일한 수의 요소를 가져야 하며(즉, 테이블의 컬럼 수), 각 리스트의 해당 항목은 호환 가능한 데이터 타입을 가져야 한다. 결과 테이블의 각 컬럼에 할당되는 실제 데이터 타입은 `UNION`과 동일한 규칙을 사용하여 결정된다([10.5절](https://www.postgresql.org/docs/17/typeconv-union-case.html "10.5. UNION, CASE, and Related Constructs") 참조).

예를 들어, 다음 쿼리는 두 개의 컬럼과 세 개의 행을 가진 테이블을 반환한다.

```sql
VALUES (1, 'one'), (2, 'two'), (3, 'three');
```

이 쿼리는 다음과 동일한 효과를 가진다.

```sql
SELECT 1 AS column1, 'one' AS column2
UNION ALL
SELECT 2, 'two'
UNION ALL
SELECT 3, 'three';
```

기본적으로 PostgreSQL은 `VALUES` 테이블의 컬럼에 `column1`, `column2` 등의 이름을 할당한다. 컬럼 이름은 SQL 표준에 의해 지정되지 않으며, 데이터베이스 시스템마다 다르게 처리한다. 따라서 일반적으로 테이블 별칭 리스트를 사용해 기본 이름을 재정의하는 것이 좋다. 예를 들면 다음과 같다.

```sql
=> SELECT * FROM (VALUES (1, 'one'), (2, 'two'), (3, 'three')) AS t (num,letter);
 num | letter
-----+--------
   1 | one
   2 | two
   3 | three
(3 rows)
```

구문적으로, `VALUES` 뒤에 오는 표현식 리스트는 다음과 동일하게 처리된다.

```sql
SELECT select_list FROM table_expression
```

따라서 `SELECT`가 사용될 수 있는 모든 위치에서 `VALUES`를 사용할 수 있다. 예를 들어, `UNION`의 일부로 사용하거나, 정렬 명세(`ORDER BY`, `LIMIT`, `OFFSET`)를 추가할 수 있다. `VALUES`는 주로 `INSERT` 명령의 데이터 소스로 사용되며, 그 다음으로는 서브쿼리로 자주 활용된다.

더 자세한 정보는 [VALUES](https://www.postgresql.org/docs/17/sql-values.html "VALUES") 문서를 참고한다.

## 7.8. WITH 쿼리 (공통 테이블 표현식)

`WITH` 구문은 더 큰 쿼리 안에서 사용할 보조 문장을 작성하는 방법을 제공한다. 이 문장들은 일반적으로 공통 테이블 표현식(Common Table Expressions, CTE)이라고 불리며, 단일 쿼리에서만 존재하는 임시 테이블을 정의하는 것으로 생각할 수 있다. `WITH` 절 안의 각 보조 문장은 `SELECT`, `INSERT`, `UPDATE`, `DELETE`, 또는 `MERGE`가 될 수 있다. 그리고 `WITH` 절 자체는 `SELECT`, `INSERT`, `UPDATE`, `DELETE`, 또는 `MERGE`와 같은 주 문장에 연결된다.

### 7.8.1. `WITH` 절 안의 `SELECT` 사용

`WITH` 절 안에서 `SELECT`를 사용하는 기본적인 목적은 복잡한 쿼리를 더 단순한 부분으로 나누는 것이다. 다음은 그 예시다:

```sql
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
), top_regions AS (
    SELECT region
    FROM regional_sales
    WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```

이 쿼리는 상위 판매 지역에서만 제품별 판매 총액을 보여준다. `WITH` 절은 `regional_sales`와 `top_regions`라는 두 개의 보조 문을 정의한다. `regional_sales`의 출력은 `top_regions`에서 사용되고, `top_regions`의 출력은 주 `SELECT` 쿼리에서 사용된다. 이 예제는 `WITH` 없이도 작성할 수 있지만, 두 단계의 중첩된 하위 `SELECT`가 필요하다. 이 방식이 조금 더 이해하기 쉽다.

### 7.8.2. 재귀 쿼리

`RECURSIVE` 수정자는 `WITH` 구문을 단순한 문법적 편의 기능에서 표준 SQL로는 달성할 수 없는 기능을 수행하는 도구로 바꾼다. `RECURSIVE`를 사용하면 `WITH` 쿼리가 자신의 출력을 참조할 수 있다. 1부터 100까지의 정수를 더하는 매우 간단한 예제를 살펴보자:

```sql
WITH RECURSIVE t(n) AS (
    VALUES (1)
  UNION ALL
    SELECT n+1 FROM t WHERE n < 100
)
SELECT sum(n) FROM t;
```

재귀 `WITH` 쿼리의 일반적인 형태는 항상 *비재귀 항목*으로 시작하고, 그 뒤에 `UNION` (또는 `UNION ALL`)이 오며, 마지막으로 *재귀 항목*이 온다. 이때 재귀 항목만이 쿼리 자신의 출력을 참조할 수 있다. 이러한 쿼리는 다음과 같이 실행된다:

**재귀 쿼리 실행 과정**

1. 비재귀 항목을 평가한다. `UNION` (단, `UNION ALL`은 제외)의 경우 중복 행을 제거한다. 남은 모든 행을 재귀 쿼리의 결과에 포함시키고, 임시 *작업 테이블*에 저장한다.

2. 작업 테이블이 비어 있지 않은 동안 다음 단계를 반복한다:
    
    1. 재귀 항목을 평가한다. 이때 작업 테이블의 현재 내용을 재귀 자기 참조에 대체한다. `UNION` (단, `UNION ALL`은 제외)의 경우 중복 행과 이전 결과 행과 중복되는 행을 제거한다. 남은 모든 행을 재귀 쿼리의 결과에 포함시키고, 임시 *중간 테이블*에 저장한다.
        
    2. 작업 테이블의 내용을 중간 테이블의 내용으로 교체한 후, 중간 테이블을 비운다.

이 과정은 작업 테이블이 비워질 때까지 반복되며, 최종적으로 재귀 쿼리의 결과가 완성된다. 재귀 쿼리는 계층적 데이터나 반복적인 패턴을 처리할 때 매우 유용하다.

### 참고 사항

`RECURSIVE` 키워드를 사용하면 재귀적으로 쿼리를 지정할 수 있지만, 내부적으로는 이러한 쿼리를 반복적으로 평가한다.

위 예제에서 작업 테이블은 각 단계마다 단일 행만을 가지며, 연속적인 단계에서 1부터 100까지의 값을 차례로 취한다. 100번째 단계에서는 `WHERE` 절 때문에 출력이 발생하지 않으며, 이로 인해 쿼리가 종료된다.

재귀 쿼리는 일반적으로 계층적 또는 트리 구조의 데이터를 처리하는 데 사용된다. 유용한 예로, 특정 제품의 모든 직접 및 간접 하위 부품을 찾는 쿼리가 있다. 이 쿼리는 즉각적인 포함 관계만을 보여주는 테이블이 주어졌을 때 사용할 수 있다:

```sql
WITH RECURSIVE included_parts(sub_part, part, quantity) AS (
    SELECT sub_part, part, quantity FROM parts WHERE part = 'our_product'
  UNION ALL
    SELECT p.sub_part, p.part, p.quantity * pr.quantity
    FROM included_parts pr, parts p
    WHERE p.part = pr.sub_part
)
SELECT sub_part, SUM(quantity) as total_quantity
FROM included_parts
GROUP BY sub_part
```

#### 7.8.2.1. 탐색 순서

재귀 쿼리를 사용해 트리 탐색을 계산할 때, 결과를 깊이 우선 탐색(depth-first) 또는 너비 우선 탐색(breadth-first) 순서로 정렬하고 싶을 수 있다. 이를 위해 다른 데이터 컬럼과 함께 정렬 컬럼을 계산하고, 이를 사용해 최종 결과를 정렬할 수 있다. 이 방법은 쿼리 평가가 행을 방문하는 순서를 실제로 제어하지 않는다는 점에 유의해야 한다. 쿼리 평가 순서는 항상 SQL 구현에 의존적이다. 이 접근 방식은 단순히 결과를 나중에 정렬하는 편리한 방법을 제공할 뿐이다.

깊이 우선 탐색 순서를 만들기 위해, 각 결과 행에 대해 지금까지 방문한 행의 배열을 계산한다. 예를 들어, `link` 필드를 사용해 `tree` 테이블을 탐색하는 다음 쿼리를 고려해 보자:

```sql
WITH RECURSIVE search_tree(id, link, data) AS (
    SELECT t.id, t.link, t.data
    FROM tree t
  UNION ALL
    SELECT t.id, t.link, t.data
    FROM tree t, search_tree st
    WHERE t.id = st.link
)
SELECT * FROM search_tree;
```

깊이 우선 탐색 순서 정보를 추가하려면 다음과 같이 작성할 수 있다:

```sql
WITH RECURSIVE search_tree(id, link, data, path) AS (
    SELECT t.id, t.link, t.data, ARRAY[t.id]
    FROM tree t
  UNION ALL
    SELECT t.id, t.link, t.data, path || t.id
    FROM tree t, search_tree st
    WHERE t.id = st.link
)
SELECT * FROM search_tree ORDER BY path;
```

일반적으로 하나 이상의 필드를 사용해 행을 식별해야 하는 경우, 행의 배열을 사용한다. 예를 들어, `f1`과 `f2` 필드를 추적해야 한다면 다음과 같이 작성할 수 있다:

```sql
WITH RECURSIVE search_tree(id, link, data, path) AS (
    SELECT t.id, t.link, t.data, ARRAY[ROW(t.f1, t.f2)]
    FROM tree t
  UNION ALL
    SELECT t.id, t.link, t.data, path || ROW(t.f1, t.f2)
    FROM tree t, search_tree st
    WHERE t.id = st.link
)
SELECT * FROM search_tree ORDER BY path;
```

이 방법은 트리 구조의 데이터를 탐색하고 정렬할 때 유용하다. 깊이 우선 탐색 순서를 사용하면 트리의 각 가지를 끝까지 탐색한 후 다음 가지로 이동하는 방식으로 결과를 정렬할 수 있다. 이를 통해 트리 구조의 데이터를 더 직관적으로 이해하고 분석할 수 있다.

### 팁

추적해야 할 필드가 하나뿐인 일반적인 경우에는 `ROW()` 구문을 생략한다. 이렇게 하면 복합 타입 배열 대신 단순 배열을 사용할 수 있어 효율성을 높일 수 있다.

너비 우선 탐색 순서를 만들려면 탐색 깊이를 추적하는 컬럼을 추가하면 된다. 예를 들어 다음과 같이 작성할 수 있다:

```sql
WITH RECURSIVE search_tree(id, link, data, depth) AS (
    SELECT t.id, t.link, t.data, 0
    FROM tree t
  UNION ALL
    SELECT t.id, t.link, t.data, depth + 1
    FROM tree t, search_tree st
    WHERE t.id = st.link
)
SELECT * FROM search_tree ORDER BY depth;
```

안정적인 정렬을 얻으려면 데이터 컬럼을 보조 정렬 컬럼으로 추가한다.

### 팁

재귀 쿼리 평가 알고리즘은 너비 우선 탐색(BFS) 순서로 결과를 생성한다. 하지만 이는 구현 세부 사항에 불과하며, 이를 의존하는 것은 바람직하지 않을 수 있다. 각 레벨 내에서 행의 순서는 명확히 정의되지 않았기 때문에, 어떤 경우에는 명시적인 정렬이 필요할 수 있다.

깊이 우선 탐색(DFS)이나 너비 우선 탐색(BFS) 정렬 컬럼을 계산하기 위한 내장 구문이 존재한다. 예를 들어:

```sql
WITH RECURSIVE search_tree(id, link, data) AS (
    SELECT t.id, t.link, t.data
    FROM tree t
  UNION ALL
    SELECT t.id, t.link, t.data
    FROM tree t, search_tree st
    WHERE t.id = st.link
) SEARCH DEPTH FIRST BY id SET ordercol
SELECT * FROM search_tree ORDER BY ordercol;

WITH RECURSIVE search_tree(id, link, data) AS (
    SELECT t.id, t.link, t.data
    FROM tree t
  UNION ALL
    SELECT t.id, t.link, t.data
    FROM tree t, search_tree st
    WHERE t.id = st.link
) SEARCH BREADTH FIRST BY id SET ordercol
SELECT * FROM search_tree ORDER BY ordercol;
```

이 구문은 내부적으로 위의 수동으로 작성한 형태와 유사하게 확장된다. `SEARCH` 절은 깊이 우선 탐색 또는 너비 우선 탐색 중 어느 것을 원하는지 지정하고, 정렬을 위해 추적할 컬럼 목록과 정렬에 사용할 결과 데이터를 담을 컬럼 이름을 지정한다. 이 컬럼은 CTE의 출력 행에 암묵적으로 추가된다.

#### 7.8.2.2. 순환 탐지

재귀 쿼리를 다룰 때는 쿼리의 재귀 부분이 결국에는 더 이상 튜플을 반환하지 않도록 해야 한다. 그렇지 않으면 쿼리가 무한히 반복될 수 있다. 때로는 `UNION ALL` 대신 `UNION`을 사용하여 이전 출력 행과 중복되는 행을 제거함으로써 이를 달성할 수 있다. 그러나 순환이 완전히 동일한 출력 행을 포함하지 않는 경우도 있다. 이 경우에는 하나 또는 몇 개의 필드만 확인하여 동일한 지점에 도달했는지 판단해야 할 수도 있다. 이러한 상황을 처리하는 표준 방법은 이미 방문한 값의 배열을 계산하는 것이다. 예를 들어, `link` 필드를 사용하여 `graph` 테이블을 탐색하는 다음 쿼리를 다시 살펴보자:

```sql
WITH RECURSIVE search_graph(id, link, data, depth) AS (
    SELECT g.id, g.link, g.data, 0
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1
    FROM graph g, search_graph sg
    WHERE g.id = sg.link
)
SELECT * FROM search_graph;
```

이 쿼리는 `link` 관계에 순환이 포함되어 있으면 무한히 반복된다. "depth" 출력이 필요하기 때문에 단순히 `UNION ALL`을 `UNION`으로 변경하는 것만으로는 순환을 제거할 수 없다. 대신 특정 링크 경로를 따라가면서 동일한 행에 다시 도달했는지 인식해야 한다. 순환 가능성이 있는 쿼리에 `is_cycle`과 `path` 두 컬럼을 추가한다:

```sql
WITH RECURSIVE search_graph(id, link, data, depth, is_cycle, path) AS (
    SELECT g.id, g.link, g.data, 0,
      false,
      ARRAY[g.id]
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1,
      g.id = ANY(path),
      path || g.id
    FROM graph g, search_graph sg
    WHERE g.id = sg.link AND NOT is_cycle
)
SELECT * FROM search_graph;
```

순환을 방지하는 것 외에도, 배열 값은 특정 행에 도달하기 위해 거친 "경로"를 나타내는 데 유용하다.

순환을 인식하기 위해 둘 이상의 필드를 확인해야 하는 일반적인 경우에는 행의 배열을 사용한다. 예를 들어, `f1`과 `f2` 필드를 비교해야 한다면 다음과 같이 작성한다:

```sql
WITH RECURSIVE search_graph(id, link, data, depth, is_cycle, path) AS (
    SELECT g.id, g.link, g.data, 0,
      false,
      ARRAY[ROW(g.f1, g.f2)]
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1,
      ROW(g.f1, g.f2) = ANY(path),
      path || ROW(g.f1, g.f2)
    FROM graph g, search_graph sg
    WHERE g.id = sg.link AND NOT is_cycle
)
SELECT * FROM search_graph;
```

### 팁

사이클을 감지하기 위해 단일 필드만 확인하면 되는 일반적인 경우에는 `ROW()` 구문을 생략할 수 있다. 이렇게 하면 복합 타입 배열 대신 단순 배열을 사용할 수 있어 효율성을 높일 수 있다.

사이클 감지를 단순화하는 내장 구문이 존재한다. 위의 쿼리는 다음과 같이 작성할 수도 있다:

```sql
WITH RECURSIVE search_graph(id, link, data, depth) AS (
    SELECT g.id, g.link, g.data, 1
    FROM graph g
  UNION ALL
    SELECT g.id, g.link, g.data, sg.depth + 1
    FROM graph g, search_graph sg
    WHERE g.id = sg.link
) CYCLE id SET is_cycle USING path
SELECT * FROM search_graph;
```

이 쿼리는 내부적으로 위의 형태로 다시 작성된다. `CYCLE` 절은 먼저 사이클 감지를 위해 추적할 컬럼 목록을 지정하고, 사이클이 감지되었는지 여부를 나타낼 컬럼 이름을 지정하며, 마지막으로 경로를 추적할 컬럼 이름을 지정한다. 사이클과 경로 컬럼은 CTE의 출력 행에 암묵적으로 추가된다.

### 팁

사이클 경로 컬럼은 이전 섹션에서 설명한 깊이 우선 순서 컬럼과 동일한 방식으로 계산된다. 하나의 쿼리가 `SEARCH`와 `CYCLE` 절을 모두 포함할 수 있지만, 깊이 우선 탐색 명세와 사이클 탐지 명세를 함께 사용하면 중복 계산이 발생한다. 따라서 `CYCLE` 절을 사용하고 경로 컬럼으로 정렬하는 것이 더 효율적이다. 너비 우선 순서가 필요한 경우, `SEARCH`와 `CYCLE`을 모두 지정하는 것이 유용할 수 있다.

쿼리가 무한 루프에 빠질 가능성이 있을 때 테스트를 위한 유용한 방법은 상위 쿼리에 `LIMIT`을 추가하는 것이다. 예를 들어, 다음 쿼리는 `LIMIT`이 없으면 무한히 실행된다:

```sql
WITH RECURSIVE t(n) AS (
    SELECT 1
  UNION ALL
    SELECT n+1 FROM t
)
SELECT n FROM t LIMIT 100;
```

이 방법이 동작하는 이유는 PostgreSQL의 구현이 `WITH` 쿼리의 결과 중 상위 쿼리에서 실제로 가져오는 행만 평가하기 때문이다. 하지만 이 방법을 프로덕션 환경에서 사용하는 것은 권장하지 않는다. 다른 시스템에서는 다르게 동작할 수 있기 때문이다. 또한, 외부 쿼리가 재귀 쿼리의 결과를 정렬하거나 다른 테이블과 조인하는 경우에는 일반적으로 `WITH` 쿼리의 모든 출력을 가져오려고 시도하기 때문에 이 방법이 작동하지 않을 가능성이 크다.

### 7.8.3. 공통 테이블 표현식의 구체화

`WITH` 쿼리의 유용한 특징 중 하나는, 부모 쿼리나 동일한 `WITH` 쿼리에서 여러 번 참조되더라도 일반적으로 부모 쿼리 실행 당 한 번만 평가된다는 점이다. 따라서 여러 곳에서 필요한 비용이 큰 계산을 `WITH` 쿼리 안에 배치해 중복 작업을 피할 수 있다. 또 다른 활용 방법은 부작용이 있는 함수의 원치 않는 다중 평가를 방지하는 것이다. 그러나 이와 동시에, 최적화 프로그램은 부모 쿼리의 제약 조건을 여러 번 참조되는 `WITH` 쿼리로 밀어 넣을 수 없다. 왜냐하면 이는 `WITH` 쿼리의 출력을 사용하는 모든 경우에 영향을 미칠 수 있기 때문이다. 여러 번 참조되는 `WITH` 쿼리는 작성된 대로 평가되며, 부모 쿼리가 나중에 제거할 행을 미리 제거하지 않는다. (하지만 앞서 언급했듯이, 쿼리 참조가 제한된 수의 행만 요구할 경우 조기에 평가가 중단될 수 있다.)

그러나 `WITH` 쿼리가 재귀적이지 않고 부작용이 없는 경우 (즉, 휘발성 함수를 포함하지 않는 `SELECT`인 경우), 부모 쿼리로 병합되어 두 쿼리 수준의 공동 최적화가 가능하다. 기본적으로, 부모 쿼리가 `WITH` 쿼리를 한 번만 참조할 때 이 현상이 발생하지만, 여러 번 참조할 때는 발생하지 않는다. 이 결정을 재정의하려면 `MATERIALIZED`를 지정해 `WITH` 쿼리를 별도로 계산하도록 강제하거나, `NOT MATERIALIZED`를 지정해 부모 쿼리로 병합하도록 강제할 수 있다. 후자의 경우 `WITH` 쿼리의 중복 계산 위험이 있지만, `WITH` 쿼리의 전체 출력 중 일부만 필요한 경우에는 여전히 전체적으로 이점을 얻을 수 있다.

이 규칙의 간단한 예는 다음과 같다:

```sql
WITH w AS (
    SELECT * FROM big_table
)
SELECT * FROM w WHERE key = 123;
```

이 `WITH` 쿼리는 병합되어 다음과 동일한 실행 계획을 생성한다:

```sql
SELECT * FROM big_table WHERE key = 123;
```

특히, `key`에 인덱스가 있다면 `key = 123`인 행만 가져오는 데 사용될 가능성이 높다. 반면에, 다음 쿼리에서는:

```sql
WITH w AS (
    SELECT * FROM big_table
)
SELECT * FROM w AS w1 JOIN w AS w2 ON w1.key = w2.ref
WHERE w2.key = 123;
```

`WITH` 쿼리가 구체화되어 `big_table`의 임시 복사본이 생성된 후, 인덱스의 이점 없이 자기 자신과 조인된다. 이 쿼리는 다음과 같이 작성하면 훨씬 더 효율적으로 실행된다:

```sql
WITH w AS NOT MATERIALIZED (
    SELECT * FROM big_table
)
SELECT * FROM w AS w1 JOIN w AS w2 ON w1.key = w2.ref
WHERE w2.key = 123;
```

이렇게 하면 부모 쿼리의 제약 조건이 `big_table`의 스캔에 직접 적용될 수 있다.

`NOT MATERIALIZED`가 바람직하지 않은 경우의 예는 다음과 같다:

```sql
WITH w AS (
    SELECT key, very_expensive_function(val) as f FROM some_table
)
SELECT * FROM w AS w1 JOIN w AS w2 ON w1.f = w2.f;
```

여기서 `WITH` 쿼리의 구체화는 `very_expensive_function`이 테이블의 각 행당 한 번만 평가되도록 보장한다. 두 번 평가되지 않는다.

위 예제들은 `WITH`가 `SELECT`와 함께 사용되는 경우만 보여주지만, `INSERT`, `UPDATE`, `DELETE`, 또는 `MERGE`에도 동일한 방식으로 연결할 수 있다. 각 경우에 `WITH`는 기본 명령에서 참조할 수 있는 임시 테이블을 효과적으로 제공한다.

### 7.8.4. `WITH` 구문에서 데이터 수정 명령어 사용하기

`WITH` 구문 내에서 데이터를 수정하는 명령어(`INSERT`, `UPDATE`, `DELETE`, `MERGE`)를 사용할 수 있다. 이를 통해 하나의 쿼리 안에서 여러 가지 작업을 수행할 수 있다. 예를 들어 다음과 같은 쿼리를 작성할 수 있다:

```sql
WITH moved_rows AS (
    DELETE FROM products
    WHERE
        "date" >= '2010-10-01' AND
        "date" < '2010-11-01'
    RETURNING *
)
INSERT INTO products_log
SELECT * FROM moved_rows;
```

이 쿼리는 `products` 테이블에서 특정 조건에 맞는 행을 삭제하고, 그 결과를 `products_log` 테이블에 삽입한다. `WITH` 구문 안의 `DELETE` 명령어는 `products` 테이블에서 지정된 행을 삭제하며, `RETURNING` 절을 통해 삭제된 행의 내용을 반환한다. 이후 메인 쿼리는 이 결과를 읽어 `products_log` 테이블에 삽입한다.

위 예제에서 주목할 점은 `WITH` 절이 `INSERT` 명령어에 연결되어 있다는 것이다. 이는 데이터 수정 명령어가 최상위 명령어에 연결된 `WITH` 절에서만 허용되기 때문이다. 하지만 일반적인 `WITH` 가시성 규칙이 적용되므로, `INSERT` 내부의 하위 `SELECT`에서 `WITH` 구문의 출력을 참조할 수 있다.

`WITH` 구문에서 데이터 수정 명령어를 사용할 때는 일반적으로 `RETURNING` 절을 포함한다. 이는 위 예제에서 볼 수 있듯이, `RETURNING` 절의 출력이 임시 테이블을 형성하며, 이 임시 테이블은 쿼리의 나머지 부분에서 참조할 수 있다. 만약 `WITH` 구문 내의 데이터 수정 명령어에 `RETURNING` 절이 없다면, 임시 테이블이 생성되지 않으며 쿼리의 나머지 부분에서 참조할 수 없다. 그럼에도 불구하고 해당 명령어는 실행된다. 다음은 그다지 유용하지 않은 예제다:

```sql
WITH t AS (
    DELETE FROM foo
)
DELETE FROM bar;
```

이 예제는 `foo`와 `bar` 테이블에서 모든 행을 삭제한다. 클라이언트에게 보고되는 영향 받은 행의 수는 `bar` 테이블에서 삭제된 행만 포함한다.

데이터 수정 명령어에서 재귀적 자기 참조는 허용되지 않는다. 하지만 재귀적 `WITH` 구문의 출력을 참조하는 방식으로 이 제한을 우회할 수 있는 경우가 있다. 예를 들어:

```sql
WITH RECURSIVE included_parts(sub_part, part) AS (
    SELECT sub_part, part FROM parts WHERE part = 'our_product'
  UNION ALL
    SELECT p.sub_part, p.part
    FROM included_parts pr, parts p
    WHERE p.part = pr.sub_part
)
DELETE FROM parts
  WHERE part IN (SELECT part FROM included_parts);
```

이 쿼리는 특정 제품의 직접적 및 간접적 하위 부품을 모두 삭제한다.

`WITH` 구문 내의 데이터 수정 명령어는 정확히 한 번 실행되며, 메인 쿼리가 그 출력을 모두 읽는지 여부와 상관없이 항상 완료된다. 이는 `WITH` 구문 내의 `SELECT`와는 다른 규칙이다. 이전 섹션에서 설명했듯이, `SELECT`는 메인 쿼리가 그 출력을 요구하는 만큼만 실행된다.

`WITH` 구문 내의 하위 명령어는 서로 동시에 실행되며, 메인 쿼리와도 동시에 실행된다. 따라서 `WITH` 구문에서 데이터 수정 명령어를 사용할 때, 실제 업데이트가 발생하는 순서는 예측할 수 없다. 모든 명령어는 동일한 스냅샷을 기준으로 실행되므로(13장 참조), 각 명령어는 다른 명령어가 대상 테이블에 미치는 영향을 "볼" 수 없다. 이는 행 업데이트의 실제 순서가 예측 불가능한 문제를 완화하며, `RETURNING` 데이터가 다른 `WITH` 하위 명령어와 메인 쿼리 간의 변경 사항을 전달하는 유일한 방법임을 의미한다. 예를 들어:

```sql
WITH t AS (
    UPDATE products SET price = price * 1.05
    RETURNING *
)
SELECT * FROM products;
```

이 경우 외부 `SELECT`는 `UPDATE` 작업 이전의 원래 가격을 반환한다. 반면:

```sql
WITH t AS (
    UPDATE products SET price = price * 1.05
    RETURNING *
)
SELECT * FROM t;
```

이 경우 외부 `SELECT`는 업데이트된 데이터를 반환한다.

하나의 명령어 내에서 동일한 행을 두 번 업데이트하는 것은 지원되지 않는다. 두 수정 중 하나만 적용되며, 어떤 것이 적용될지 예측하기 어렵거나 불가능할 수 있다. 이는 동일한 명령어 내에서 이미 업데이트된 행을 삭제하는 경우에도 적용된다. 이 경우 업데이트만 수행된다. 따라서 일반적으로 하나의 명령어 내에서 동일한 행을 두 번 수정하려는 시도를 피해야 한다. 특히 메인 명령어나 다른 하위 명령어가 변경한 동일한 행에 영향을 미칠 수 있는 `WITH` 하위 명령어를 작성하지 않도록 주의해야 한다. 이러한 명령어의 결과는 예측할 수 없다.

현재 `WITH` 구문 내의 데이터 수정 명령어의 대상이 되는 테이블은 조건부 규칙, `ALSO` 규칙, 또는 여러 명령어로 확장되는 `INSTEAD` 규칙을 가질 수 없다.