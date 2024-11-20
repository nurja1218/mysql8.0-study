### 인비저블 인덱스

- 옵티마이저가 인덱스를 활용하지 못하게 설정할 수 있다.
- [`CREATE TABLE`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html), [`CREATE INDEX`](https://dev.mysql.com/doc/refman/8.0/en/create-index.html), [`ALTER TABLE`](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) 쿼리 문에 index를 정의하는 부분에서 사용할 수 있다

```sql
CREATE TABLE t1 (
  i INT,
  j INT,
  k INT,
  INDEX i_idx (i) INVISIBLE
) ENGINE = InnoDB;
CREATE INDEX j_idx ON t1 (j) INVISIBLE;
ALTER TABLE t1 ADD INDEX k_idx (k) INVISIBLE;
```

- 인비저블은 테이블에서 특정 인덱스를 제거했을 때의 퍼포먼스를 측정할 때 사용한다. 인덱스를 실제로 제거하고 다시 생성하는 것은 비용이 많이 들기 때문에 인비저블을 통해 간단하게 테스트를 해볼 수 있다.
- INVISIBLE 한 인덱스여도 [`use_invisible_indexes`](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html#optflag_use-invisible-indexes) flag 를 on으로 설정하면 옵티마이저가 해당 인덱스를 활용할 수 있다. 특정 쿼리문에서만 일시적으로 인비저블을 해제할 때만 사용하는 듯 하다(?)

```sql
mysql> EXPLAIN SELECT /*+ SET_VAR(optimizer_switch = 'use_invisible_indexes=on') */
     >     i, j FROM t1 WHERE j >= 50\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: range
possible_keys: j_idx
          key: j_idx
      key_len: 5
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: Using index condition

mysql> EXPLAIN SELECT i, j FROM t1 WHERE j >= 50\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 5
     filtered: 33.33
        Extra: Using where
```

### 스킵 스캔

- 스킵 스캔은 WHERE 조건에 인덱스가 포함되지 않을 때, 쿼리의 효율을 높이기 위한 최적화 전략이다

```sql
CREATE TABLE t1 (f1 INT NOT NULL, f2 INT NOT NULL, PRIMARY KEY(f1, f2));
INSERT INTO t1 VALUES
  (1,1), (1,2), (1,3), (1,4), (1,5),
  (2,1), (2,2), (2,3), (2,4), (2,5);
INSERT INTO t1 SELECT f1, f2 + 5 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 10 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 20 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 40 FROM t1;
ANALYZE TABLE t1;

EXPLAIN SELECT f1, f2 FROM t1 WHERE f2 > 40;
```

위와 같은 쿼리문에서 WHERE절에 f1에 대한 조건이 없기 때문에 인덱스를 활용할 수 없다. 이럴 경우 옵티마이저는 아래와 같은 과정을 거쳐서 최적화를 한다

1. Get the first distinct value of the first key part (`f1 = 1`).
2. Construct the range based on the first and second key parts (`f1 = 1 AND f2 > 40`).
3. Perform a range scan.
4. Get the next distinct value of the first key part (`f1 = 2`).
5. Construct the range based on the first and second key parts (`f1 = 2 AND f2 > 40`).
6. Perform a range scan.

### 해쉬 조인

- 8.0.18 부터 equi-join 일 경우(20버전 부터는 `*Inner non-equi-join` , `Semijoin`,`Antijoin`, `Left outer join` ,`Right outer join`\* 에도) 해쉬 조인을 사용한다(이전 버전에서는 Block Nested Loop Join 알고리즘 사용 - 해당 알고리즘은 20 버전에서부터 제거)

```sql
SELECT *
    FROM t1
    JOIN t2
        ON t1.c1=t2.c1;
```

- 해쉬조인은 레코드 건수가 적은 테이블을 기준으로 해쉬 테이블을 생성하고(빌드 단계) → 나머지 테이블을 읽어서 일치 레코드를 찾아서(프로브 단계) 조인한다.
- 해시 테이블을 만들때 제한된 용량(`join_buffer_size`)을 넘어가면 청크로 나눠서 디스크에 저장하고 순차적으로 불러와서 조인을 실행한다
- 해쉬조인이 항상 좋은 것은 아니다. 중첩 루프 조인은 최고 응답 속도 전략에 적합하고 해쉬조인은 최고 스루풋 전략에 적합하다. 전략에 따라 사용하는 것이 좋다(RDBMS의 목적을 생각하면 중첩 루프 조인 방식이 적절한 전략일 때가 많고 해쉬조인은 보조로 사용)
- 해쉬 조인 여부를 알기 위해서는 18버전에서는 EXPLAIN 뒤에`FORMAT=TREE` 붙여 줘야 한다(20버전부터는 필요없음)

```sql
mysql> EXPLAIN FORMAT=TREE
    -> SELECT *
    ->     FROM t1
    ->     JOIN t2
    ->         ON (t1.c1 = t2.c1 AND t1.c2 < t2.c2)
    ->     JOIN t3
    ->         ON (t2.c1 = t3.c1)\G
```

### 인덱스 정렬 선호
