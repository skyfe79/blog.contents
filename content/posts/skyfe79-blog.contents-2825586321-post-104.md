---
title: "[PostgreSQL] 제2부. SQL 언어 - 6장.데이터 조작"
date: 2025-02-02T03:53:37Z
author: "skyfe79"
draft: false
tags: ["postgresql"]
---

이전 장에서는 데이터를 저장하기 위한 테이블과 다른 구조를 만드는 방법에 대해 다뤘다. 이제는 테이블에 데이터를 채워 넣을 차례다. 이 장에서는 테이블 데이터를 삽입하고, 업데이트하고, 삭제하는 방법을 다룬다. 다음 장에서는 데이터베이스에서 오랫동안 잊혀진 데이터를 추출하는 방법을 설명할 것이다.

## 6.1. 데이터 삽입하기

테이블을 처음 생성하면 데이터가 없는 상태로 시작한다. 데이터베이스를 유용하게 사용하려면 가장 먼저 데이터를 삽입해야 한다. 데이터는 한 번에 한 행씩 삽입한다. 한 번에 여러 행을 삽입할 수도 있지만, 완전한 행이 아닌 부분적인 데이터를 삽입할 수는 없다. 일부 컬럼의 값만 알고 있어도, 반드시 완전한 행을 만들어야 한다.

새로운 행을 생성하려면 `INSERT` 명령어를 사용한다. 이 명령어는 테이블 이름과 컬럼 값을 필요로 한다. 예를 들어, [5장](https://www.postgresql.org/docs/17/ddl.html "5장. 데이터 정의")에서 다룬 `products` 테이블을 생각해 보자:

```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);
```

여기에 한 행을 삽입하는 명령어는 다음과 같다:

```sql
INSERT INTO products VALUES (1, 'Cheese', 9.99);
```

데이터 값은 테이블에 정의된 컬럼 순서대로 쉼표로 구분하여 나열한다. 일반적으로 데이터 값은 리터럴(상수)이지만, 스칼라 표현식도 사용할 수 있다.

위의 구문은 테이블의 컬럼 순서를 알고 있어야 한다는 단점이 있다. 이를 피하기 위해 컬럼을 명시적으로 나열할 수도 있다. 예를 들어, 다음 두 명령어는 위의 명령어와 동일한 효과를 낸다:

```sql
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', 9.99);
INSERT INTO products (name, price, product_no) VALUES ('Cheese', 9.99, 1);
```

많은 사용자가 컬럼 이름을 항상 명시하는 것을 좋은 습관으로 여긴다.

모든 컬럼에 대한 값이 없을 경우, 일부 컬럼을 생략할 수 있다. 이 경우, 생략된 컬럼은 기본값으로 채워진다. 예를 들어:

```sql
INSERT INTO products (product_no, name) VALUES (1, 'Cheese');
INSERT INTO products VALUES (1, 'Cheese');
```

두 번째 형태는 PostgreSQL의 확장 기능이다. 주어진 값만큼 왼쪽부터 컬럼을 채우고, 나머지는 기본값으로 설정한다.

명확성을 위해 개별 컬럼이나 전체 행에 대해 기본값을 명시적으로 요청할 수도 있다:

```sql
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', DEFAULT);
INSERT INTO products DEFAULT VALUES;
```

한 번에 여러 행을 삽입할 수도 있다:

```sql
INSERT INTO products (product_no, name, price) VALUES
    (1, 'Cheese', 9.99),
    (2, 'Bread', 1.99),
    (3, 'Milk', 2.99);
```

쿼리의 결과를 삽입하는 것도 가능하다. 이 경우, 쿼리 결과가 없을 수도 있고, 한 행 또는 여러 행일 수도 있다:

```sql
INSERT INTO products (product_no, name, price)
  SELECT product_no, name, price FROM new_products
    WHERE release_date = 'today';
```

이 방법은 삽입할 행을 계산하기 위해 SQL 쿼리 메커니즘의 모든 기능을 활용할 수 있게 해준다. ([7장](https://www.postgresql.org/docs/17/queries.html "7장. 쿼리") 참조)

### 팁

동시에 많은 양의 데이터를 삽입해야 할 때는 [COPY](https://www.postgresql.org/docs/17/sql-copy.html "COPY") 명령어를 사용하는 것을 고려해보자. [INSERT](https://www.postgresql.org/docs/17/sql-insert.html "INSERT") 명령어만큼 유연하지는 않지만, 훨씬 더 효율적이다. 대량 데이터 로딩 성능을 향상시키는 방법에 대한 자세한 내용은 [14.4장](https://www.postgresql.org/docs/17/populate.html "14.4. Populating a Database")을 참고하자.

## 6.2. 데이터 업데이트

데이터베이스에 이미 존재하는 데이터를 수정하는 작업을 업데이트라고 한다. 업데이트는 특정 행 하나, 테이블의 모든 행, 또는 특정 조건을 만족하는 일부 행에 대해 수행할 수 있다. 각 컬럼은 개별적으로 업데이트할 수 있으며, 다른 컬럼은 영향을 받지 않는다.

기존 행을 업데이트하려면 [UPDATE](https://www.postgresql.org/docs/17/sql-update.html "UPDATE") 명령을 사용한다. 이 명령을 실행하려면 세 가지 정보가 필요하다:

1. 업데이트할 테이블과 컬럼의 이름  
2. 컬럼의 새로운 값  
3. 업데이트할 행을 지정하는 조건  

[5장](https://www.postgresql.org/docs/17/ddl.html "5장. 데이터 정의")에서 살펴봤듯이, SQL은 일반적으로 행에 대한 고유 식별자를 제공하지 않는다. 따라서 특정 행을 직접 지정할 수 없는 경우가 많다. 대신, 업데이트할 행이 충족해야 하는 조건을 지정한다. 테이블에 기본 키가 있는 경우(선언 여부와 상관없이) 기본 키와 일치하는 조건을 선택해 개별 행을 안정적으로 지정할 수 있다. 그래픽 데이터베이스 접근 도구는 이 사실을 활용해 사용자가 개별 행을 업데이트할 수 있도록 한다.

예를 들어, 다음 명령은 가격이 5인 모든 제품의 가격을 10으로 업데이트한다:

```sql
UPDATE products SET price = 10 WHERE price = 5;
```

이 명령은 0개, 1개, 또는 여러 행을 업데이트할 수 있다. 업데이트할 행이 없는 경우에도 오류가 발생하지 않는다.

이 명령을 자세히 살펴보자. 먼저 `UPDATE` 키워드와 테이블 이름이 온다. 일반적으로 테이블 이름은 스키마와 함께 지정할 수 있으며, 그렇지 않으면 경로에서 테이블을 찾는다. 다음으로 `SET` 키워드와 컬럼 이름, 등호, 새로운 컬럼 값이 온다. 새로운 컬럼 값은 상수뿐만 아니라 모든 스칼라 표현식이 될 수 있다. 예를 들어, 모든 제품의 가격을 10% 인상하려면 다음과 같이 할 수 있다:

```sql
UPDATE products SET price = price * 1.10;
```

보는 바와 같이, 새로운 값에 대한 표현식은 행의 기존 값을 참조할 수 있다. 또한 `WHERE` 절을 생략했다. `WHERE` 절을 생략하면 테이블의 모든 행이 업데이트된다. `WHERE` 절이 있으면 해당 조건을 만족하는 행만 업데이트된다. `SET` 절의 등호는 할당 연산자이고, `WHERE` 절의 등호는 비교 연산자이지만, 이로 인해 모호함이 발생하지 않는다. 물론 `WHERE` 조건은 등호 비교일 필요는 없다. 다양한 연산자를 사용할 수 있다([9장](https://www.postgresql.org/docs/17/functions.html "9장. 함수와 연산자") 참조). 단, 표현식은 불리언 결과를 반환해야 한다.

`UPDATE` 명령에서 여러 컬럼을 업데이트하려면 `SET` 절에 여러 할당을 나열하면 된다. 예를 들어:

```sql
UPDATE mytable SET a = 5, b = 3, c = 1 WHERE a > 0;
```

## 6.3. 데이터 삭제

지금까지 테이블에 데이터를 추가하고 변경하는 방법에 대해 설명했다. 이제 더 이상 필요하지 않은 데이터를 삭제하는 방법에 대해 알아보자. 데이터를 추가할 때 전체 행 단위로만 가능했던 것처럼, 데이터를 삭제할 때도 전체 행 단위로만 가능하다. 앞서 설명했듯이 SQL은 개별 행을 직접 지정하는 방법을 제공하지 않는다. 따라서 삭제할 행을 지정하려면 해당 행이 만족해야 하는 조건을 명시해야 한다. 테이블에 기본 키가 있다면 정확히 하나의 행을 지정할 수 있다. 하지만 조건에 맞는 여러 행을 한 번에 삭제하거나, 테이블의 모든 행을 한꺼번에 삭제할 수도 있다.

행을 삭제하려면 `DELETE` 명령어를 사용한다. 이 명령어의 구문은 `UPDATE` 명령어와 매우 유사하다. 예를 들어, `products` 테이블에서 가격이 10인 모든 행을 삭제하려면 다음과 같이 작성한다:

```sql
DELETE FROM products WHERE price = 10;
```

만약 다음과 같이 작성한다면:

```sql
DELETE FROM products;
```

테이블의 모든 행이 삭제된다! 프로그래머는 주의해야 한다.

## 6.4. 수정된 행에서 데이터 반환하기

데이터를 조작하는 동안 수정된 행에서 데이터를 가져오는 것이 유용할 때가 있다. `INSERT`, `UPDATE`, `DELETE`, 그리고 `MERGE` 명령어는 모두 이를 지원하는 `RETURNING` 절을 선택적으로 사용할 수 있다. `RETURNING`을 사용하면 데이터를 수집하기 위해 추가적인 데이터베이스 쿼리를 실행하지 않아도 되며, 특히 수정된 행을 신뢰할 수 있게 식별하기 어려운 경우에 매우 유용하다.

`RETURNING` 절에 허용되는 내용은 `SELECT` 명령어의 출력 목록과 동일하다([7.3절](https://www.postgresql.org/docs/17/queries-select-lists.html "7.3. Select Lists") 참조). 이 절은 명령어의 대상 테이블의 열 이름이나 해당 열을 사용한 값 표현식을 포함할 수 있다. 일반적으로 `RETURNING *`을 사용하면 대상 테이블의 모든 열을 순서대로 선택한다.

`INSERT` 명령어에서 `RETURNING`을 사용할 수 있는 데이터는 삽입된 행 그 자체이다. 단순한 삽입에서는 클라이언트가 제공한 데이터를 그대로 반환하므로 크게 유용하지 않을 수 있다. 하지만 계산된 기본값에 의존할 때는 매우 유용하다. 예를 들어, [`serial`](https://www.postgresql.org/docs/17/datatype-numeric.html#DATATYPE-SERIAL "8.1.4. Serial Types") 열을 사용해 고유 식별자를 제공할 때, `RETURNING`을 사용해 새 행에 할당된 ID를 반환할 수 있다:

```sql
CREATE TABLE users (firstname text, lastname text, id serial primary key);

INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;
```

`RETURNING` 절은 `INSERT ... SELECT`와 함께 사용할 때도 매우 유용하다.

`UPDATE` 명령어에서 `RETURNING`을 사용할 수 있는 데이터는 수정된 행의 새로운 내용이다. 예를 들어:

```sql
UPDATE products SET price = price * 1.10
  WHERE price <= 99.99
  RETURNING name, price AS new_price;
```

`DELETE` 명령어에서 `RETURNING`을 사용할 수 있는 데이터는 삭제된 행의 내용이다. 예를 들어:

```sql
DELETE FROM products
  WHERE obsoletion_date = 'today'
  RETURNING *;
```

`MERGE` 명령어에서 `RETURNING`을 사용할 수 있는 데이터는 소스 행의 내용과 삽입, 수정, 또는 삭제된 대상 행의 내용을 합친 것이다. 소스와 대상이 많은 동일한 열을 가지고 있는 경우가 흔하므로, `RETURNING *`을 지정하면 중복된 열이 많이 반환될 수 있다. 따라서 소스 행이나 대상 행만 반환하도록 지정하는 것이 더 유용할 때가 많다. 예를 들어:

```sql
MERGE INTO products p USING new_products n ON p.product_no = n.product_no
  WHEN NOT MATCHED THEN INSERT VALUES (n.product_no, n.name, n.price)
  WHEN MATCHED THEN UPDATE SET name = n.name, price = n.price
  RETURNING p.*;
```

대상 테이블에 트리거([37장](https://www.postgresql.org/docs/17/triggers.html "Chapter 37. Triggers"))가 있는 경우, `RETURNING`을 사용할 수 있는 데이터는 트리거에 의해 수정된 행이다. 따라서 트리거에 의해 계산된 열을 검사하는 것도 `RETURNING`의 일반적인 사용 사례 중 하나이다.