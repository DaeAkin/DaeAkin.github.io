# Real MySQL

# 6장 내용 정리

## 쿼리의 실행 절차
MySQL 서버에서 쿼리가 실행되는 과정은 크게 3가지로 나눌 수 있습니다.
1. 사용자로부터 요청된 SQL 문장을 잘게 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리한다.
2. SQL의 파싱 정보(파스 트리)를 확인하면서 어떤 테이블로부터 읽고 어떤 인덱스를 이용해 테이블을 읽을지 선택한다.
3. 두 번째 단계에서 결정된 테이블의 읽기 순서나 선택된 인덱스를 이용해 스토리지 엔진으로부터 데이터를 가져온다.

첫 번째 단계를 “SQL 파싱(Parsing)”이라고 하며, MySQL서버의 “SQL 파서”라는 모듈로 처리합니다.
두 번째 단계는 첫 번째 단계에서 만들어진 SQL 파스 트리를 참조하면서 다음과 같은 내용을 처리합니다.

- 불필요한 조건의 제거 및 복잡한 연산의 단순화
- 여러 테이블의 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정
- 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스 결정
- 가져온 레코드들을 임시 테이블에 넣고 다시 한번 가공해야 하는지 결정

위와 같은 과정은 "최적화 및 실행 계획 수립" 단계이며, MySQL 서버의 "옵티마이저"에서 처리합니다.  또한 두 번째 단계가 완료되면 쿼리의 "실행 계획"이 만들어집니다.

세 번째 단계는 수립된 실행 계획대로 스토리지 엔진에 레코드를 읽어오도록 요청하고, MySQL 엔진에서는 스토리지 엔진으로부터 받은 레코드를 조인하거나 정렬하는 작업을 수행합니다. 

첫 번째 단계와 두 번 째 단계는 거의 MySQL 엔진에서 처리하며, 세 번째 단계는 MySQL 엔진과 스토리지 엔진(ex inno DB)으로부터 받은 레코드를 조인하거나 정렬하는 작업을 수행합니다.

## 옵티마이저의 종류

옵티마이저는 DB 서버에서 두뇌 같은 역할 입니다. 옵티마이저는 현재 대부분의 DBMS가 선택하고 있는 <u>비용 기반 최적화</u>(Cost-based optimizer, CBO) 방법과 예전 오라클에서 많이 사용됐던 <u>규칙 기반 최적화 방법</u>(Rule-based optimizer, RBO)으로 크게 나눠 볼 수 있습니다.

- 규칙 기반 최적화는 테이블의 레코드 건수나 선택도 등을 고려하지 않고, 옵티마이저에 내장된 우선순위에 따라 실행 계획을 하는 방법입니다, 요즘에는 지원이 되지않거나 업데이트 되지 않음.
- 비용 기반 최적화는 쿼리를 최적화 하기위한 여러가지 실행 계획별 비용을 산출하고 가장 비용이 적은 방법을 선택해 최종 쿼리를 실행합니다. 

현재는 거의 대부분의 RDBMS가 비용 기반의 옵티마이저를 채택하고 있으며, Mysql 역시 마찬가지 이다.



## 통계정보

비용 기반 최적화에서 가장 중요한 것은 통계 정보입니다. 통계 정보가 부정확하면 엉뚱한 방향으로 쿼리를 실행할 수 있기 때문입니다.  예를 들어 1억건의 레코드가 저장된 테이블의 통계 정보가 갱신되지 않아, 레코드가 10건 미만인 것처럼 돼 있다면, 옵티마이저는 쿼리를 실행 할 때 테이블을 처음부터 끝까지 읽는 방식(풀 테이블 스캔)으로 실행합니다. 

기본적으로 MySQL에서 관리되는 통계 정보는 대략의 `레코드 건수`와 `인덱스의 유니크한 값의 개수` 정도가 전부입니다.

오라클 같은 DBMS는 통계정보가 상당히 정적이고 수집에 많은 시간이 소요되서 통계 정보만 따로 백업하는 반면에 MySQL은 자동으로 변경되기 때문에 동적입니다. 하지만 레코드 건수가 얼마 되지 않는 개발용 MySQL 서버일 경우 레코드 건수가 많지 않으면 통계 정보가 부정확하기 때문에 "ANALYZE" 명령을 이용해 강제적으로 통계 정보를 갱신해야 할 때도 있습니다.

MEMORY 테이블은 별도로 통계 정보가 없으며, MyISAM과 InnoDB의 테이블과 인덱스 통계정보는 다음과 같이 확인합니다.

ANALYZE 명령은 인덱스 키값의 분포도만 업데이트하며, 전체 테이블의 건수는 테이블의 전체 페이지 수를 이용해 예측합니다.

```sql
mysql> SHOW TABLE STATUS LIKE 'user'\g
+------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+---------+
| Name | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time | Check_time | Collation       | Checksum | Create_options | Comment |
+------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+---------+
| user | InnoDB |      10 | Dynamic    |    2 |           8192 |       16384 |               0 |        16384 |         0 |              3 | 2020-01-03 21:21:02 | NULL        | NULL       | utf8_general_ci |     NULL |                |         |
+------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+-------------+------------+-----------------+----------+----------------+---------+
1 row in set (0.01 sec)

```



```sql
mysql> SHOW INDEX FROM user;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| user  |          0 | PRIMARY  |            1 | id          | A         |           2 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
1 row in set (0.00 sec)
```

통계 정보를 갱신하려면 다음과 같이 ANALYZE를 실행하면 됩니다.

```sql
-- // 파티션을 사용하지 않는 일반 테이블의 통계 정보 수집하기
ANALYZE TABLE user;

-- // 파티션을 사용하는 일반 테이블의 통계 정보 수집하기
ALERT TABLE user ANALYZE PARTITION p3;
```

ANALYZE를 실행하는 동안 MyISAM 테이블은 읽기는 가능하지만 쓰기는 안 되며, 정확한 키 값 분포도를 위해 인덱스 전체를 스캔하므로 많은 시간이 걸린다.

InnoDB 테이블은 읽기 쓰기 모두 불가능해서 서비스 도중에 ANALYZE를 실행하는 건 좋지 않다. 인덱스 페이지 중에서 8개 정도만 랜덤하게 선택해서 분석한다.

> 5.1.38 미만에서는 랜덤하게 인덱스 페이지 8개만 읽어서 통계를 수집하지만. 5.1.38 이상부터는 `innodb_stats_sample_pages` 파라미터로 분석할 인덱스 페이지의 개수를 지정할 수 있다.
>
> 분석할 페이지 개수를 늘릴수록 더 정확한 통계 정보를 수집할 수 있겠지만 InnoDB의 통계정보는 다른 DBMS보다 훨씬 자주 수집되며 서비스 도중에도 통계 정보가 수집될 수 있다. InnoDB의 통계 수집을 위한 인덱스 페이지 개수는 기본값 8개에서 2~3배 이상을 벗어나지 않도록 설정하는 것이 좋다.



## 실행 계획 분석

EXPLAIN 명령을 사용하면 쿼리가 어떤 실행 계획을 갖고 있는지 알 수 있다. 아무 옵션 없이 사용하면 기본적인 쿼리 실행계획만 보이는데, `EXPLAIN EXTENDED`나 `EXPLAIN PARTITIONS` 명령을 이용해 더 상세한 실행 계획을 확인할 수 있다. 추가 옵션을 사용하는 경우에는 옵션 만큼 추가 정보가 더 표시 된다.

```sql
EXPLAIN
SELECT e.emp_no, e.first_name, s.from_date, s.salary FROM employees e, salaries s
WHERE e.emp_no=s.emp_no
LIMIT 10;
```

EXPLAIN을 실행하면 쿼리 문장의 특성에 따라 표 형태가 표시가 된다.

```json
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "s",
    "partitions": null,
    "type": "index",
    "possible_keys": "PRIMARY",
    "key": "ix_salary",
    "key_len": "4",
    "ref": null,
    "rows": 1,
    "filtered": 100,
    "Extra": "Using index"
  },
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "e",
    "partitions": null,
    "type": "eq_ref",
    "possible_keys": "PRIMARY",
    "key": "PRIMARY",
    "key_len": "4",
    "ref": "employees.s.emp_no",
    "rows": 1,
    "filtered": 100,
    "Extra": null
  }
]
```

실행 순서는 위에서 아래로 표시 된다. 만약 UNION이나 상관 서브 쿼리 같은 경우 순서대로 표시되지 않을 수도 있다. 출력된 실행 계획에서 위쪽에서 출력된 결과일 수록(id 값이 작을수록) 쿼리의 바깥 부분이거나 먼저 접근한 테이블이고, 아래쪽에 출력된 결과일수록(id 값이 클수록) 쿼리의 안쪽 부분 또는 나중에 접근한 테이블에 해당 된다.

그러나 MySQL 경우 필요에 따라 실행 계획을 산출하기 위해 쿼리의 일부분을 직접 실행하기 때문에 쿼리 자체가 상당히 복잡하고 무거운 쿼리 일경우 실행 게획 조회가 느려질 가능성이 있다.

UPDATE나 INSERT, DELETE 문장에 대해서는 실행 계획을 확인할 방법은 없지만,  만약 확인하고 싶다면 WHERE 조건절만 같은 SELETE 문장을 만들어서 대략적으로 계획을 확인해 볼 수도 있다.

그럼 컬럼이 의미하는 바를 하나씩 알아보자.

### id 컬럼

실행 계획에서 가장 왼쪽에 표시되는 id 칼럼은 단위 select 쿼리별로 부여되는 식별자 값이다.

```sql
SELECT ...
FROM (SELECT ... FROM tb_test1) tb1,
	tb_test2 tb2
WHERE tb1.id = tb2.id;
```

위에 쿼리문장을 분리해보면 

```sql
SELECT ... FROM tb_test1;
SELECT ... FROM tb1, tb_test2 tb2 WHERE tb1.id=tb2.id;
```


그래서 위에 있는 쿼리를 EXPLAIN을 해보면 2개의 id컬럼 값이 나옵니다.

#### 조인일때

만약 하나의 SELETE 문장으로 여러 개의 테이블을 조인하면 조인되는 테이블의 개수만큼 실행 계획 레코드가 출력되지만 **같은 id**가 부여된다.

```sql
EXPLAIN
SELECT e.emp_no, e.first_name, s.from_date, s.salary
FROM employees e, salaries s
WHERE e.emp_no=s.emp_no
LIMIT 10;
```



```json
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "s",
    "partitions": null,
    "type": "index",
    "possible_keys": "PRIMARY",
    "key": "ix_salary",
    "key_len": "4",
    "ref": null,
    "rows": 1,
    "filtered": 100,
    "Extra": "Using index"
  },
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "e",
    "partitions": null,
    "type": "eq_ref",
    "possible_keys": "PRIMARY",
    "key": "PRIMARY",
    "key_len": "4",
    "ref": "employees.s.emp_no",
    "rows": 1,
    "filtered": 100,
    "Extra": null
  }
]
```

반대로 다음의 쿼리의 실행 게획에서는 쿼리 문장이 3개의 단위 SELECT 쿼리로 구성돼 있으므로 실행 계획의 각 레코드가 각기 다른  id를 갖는다.

```sql
EXPLAIN
select
    ((select count(*) from employees) + (select count(*) from departments)) as total_count;
```



```json
[
  {
    "id": 1,
    "select_type": "PRIMARY",
    "table": null,
    "partitions": null,
    "type": null,
    "possible_keys": null,
    "key": null,
    "key_len": null,
    "ref": null,
    "rows": null,
    "filtered": null,
    "Extra": "No tables used"
  },
  {
    "id": 3,
    "select_type": "SUBQUERY",
    "table": "departments",
    "partitions": null,
    "type": "index",
    "possible_keys": null,
    "key": "ux_deptname",
    "key_len": "162",
    "ref": null,
    "rows": 9,
    "filtered": 100,
    "Extra": "Using index"
  },
  {
    "id": 2,
    "select_type": "SUBQUERY",
    "table": "employees",
    "partitions": null,
    "type": "index",
    "possible_keys": null,
    "key": "ix_hiredate",
    "key_len": "3",
    "ref": null,
    "rows": 281387,
    "filtered": 100,
    "Extra": "Using index"
  }
]
```



### select_type 칼럼

각 단위의 SELECT 쿼리가 어떤 타입의 쿼리인지 표시되는 칼럼이다.

#### SIMPLE 

**UNION이나 서브 쿼리를 사용하지 않는** 단순한 SELECT 쿼리인 경우 표시됩니다. 쿼리 문장이 아무리 복잡하더라도 실행 계획에서 select_type이 SIMPLE인 단위 쿼리는 반드시 하나만 존재한다. 일반적으로 제일 바깥 SELECT 쿼리가 SIMPLE 타입이다.

#### PRIMARY

**UNION이나 서브 쿼리가 포함된 SELECT 쿼리의 실행 계획에서** 가장 바깥쪽에 있는 단위 쿼리는 select_type이 PRIMARY로 표시된다. SIMPLE과 마찬가지로 select_type이 PRIMARY인 단위 SELECT **쿼리는 하나만 존재하**며, 쿼리의 제일 바깥 쪽에 있는 SELECT 단위 쿼리가 PRIMARY로 표시 됩니다.

#### UNION

UNION으로 결합하는 단위 SELECT 쿼리 가운데 첫 번째를 제외한 두 번째 이후 단위 SELECT 쿼리의 select_type은 UNION으로 표시된다. UNION의 첫 번째 단위 SELECT는 select_type이 UNION 쿼리로 결합된 전체 집합의 select_type이 표시된다.

```sql
EXPLAIN
SELECT * FROM (
	(SELECT emp_no FROM employees e1 LIMIT 10)
	UNION ALL
	(SELECT emp_no FROM employees e2 LIMIT 10)
	UNION ALL
	(SELECT emp_no FROM employees e3 LIMIT 10)
) tb;
```

|                    |                  |              |              |              |
| :----------------- | :--------------- | :----------- | :----------- | :----------- |
| **id**             | 1                | 2            | 3            | 4            |
| **select\_type**   | PRIMARY          | DERIVED      | UNION        | UNION        |
| **table**          | &lt;derived2&gt; | e1           | e2           | e3           |
| **partitions**     | NULL             | NULL         | NULL         | NULL         |
| **type**           | ALL              | index        | index        | index        |
| **possible\_keys** | NULL             | NULL         | NULL         | NULL         |
| **key**            | NULL             | ix\_hiredate | ix\_hiredate | ix\_hiredate |
| **key\_len**       | NULL             | 3            | 3            | 3            |
| **ref**            | NULL             | NULL         | NULL         | NULL         |
| **rows**           | 30               | 281387       | 281387       | 281387       |
| **filtered**       | 100              | 100          | 100          | 100          |
| **Extra**          | NULL             | Using index  | Using index  | Using index  |

UNION이 되는 단위 SELECT 쿼리 3개 중 첫번 째 테이블만 UNION이 아니고, 나머지 2개는 모두 UNION으로 표시되어 있다.

대신 UNION의 첫 번째 쿼리는 전체 UNION의 결과를 대표하는 DERIVED 타입으로 설정됐다.

#### DEPENDENT UNION

DEPENDENT UNION 타입 또한 UNION 타입과 같이 쿼리에  UNION이나 UNION ALL로 집합을 결합하는 쿼리에서 표시된다. 

```sql
select
    e.first_name,
    (select CONCAT('Salary change count :', count(*)) AS message
        FROM salaries s where s.emp_no = e.emp_no
    UNION
    select concat('Department change count : ', count(*)) AS message
        from dept_emp de where emp_no=e.emp_no
    ) as message
from employees e
where e.emp_no = 404688;
```

|                    |         |                    |                                          |                  |
| :----------------- | :------ | :----------------- | :--------------------------------------- | :--------------- |
| **id**             | 1       | 2                  | 3                                        | NULL             |
| **select\_type**   | PRIMARY | DEPENDENT SUBQUERY | DEPENDENT UNION                          | UNION RESULT     |
| **table**          | e       | s                  | de                                       | &lt;union2,3&gt; |
| **partitions**     | NULL    | NULL               | NULL                                     | NULL             |
| **type**           | const   | ref                | ref                                      | ALL              |
| **possible\_keys** | PRIMARY | PRIMARY,ix\_salary | PRIMARY,ix\_fromdate,ix\_empno\_fromdate | NULL             |
| **key**            | PRIMARY | PRIMARY            | ix\_empno\_fromdate                      | NULL             |
| **key\_len**       | 4       | 4                  | 4                                        | NULL             |
| **ref**            | const   | const              | const                                    | NULL             |
| **rows**           | 1       | 1                  | 1                                        | NULL             |
| **filtered**       | 100     | 100                | 100                                      | NULL             |
| **Extra**          | NULL    | Using index        | Using index                              | Using temporary  |

여기서 DEPENDENT는  UNION이나 UNION ALL로 결합된 단위 쿼리가 외부의 영향에 의해 영향을 받는 것을 의미한다. 

두 개의 SELECT 쿼리가 UNION으로 결합됐으므로 select_type에  UNION이 표시된 것이다. 그런데 UNION으로 결합되는 각 쿼리를 보면 이 **서브 쿼리의 외부에 정의된 employees 테이블의 emp_no 컬럼을 사용하고 있다**. 이렇게 내부 쿼리가 외부 값을 참조해서 처리 될 때 의존한다는 뜻으로 **dependent** 키워드가 타입에 붙게 된다.

> 하나의 단위 select 쿼리가 다른 단위 select를 포함하고 있으면 이를 서브쿼리라고 한다. 이처럼 서브 쿼리가 사용된 경우에는 외부 쿼리보다 서브 쿼리가 먼저 실행되는게 일반적이며, 대부분 이 방식이 반대의 경우보다 빠르게 처리 된다. 하지만 select_type이 dependent 키워드가 포함된 서브 쿼리는 외부 쿼리에 의존적이므로 절대 외부 쿼리보다 먼저 실행 될 수가 없어서 비효율적인 경우가 많다.



#### UNION RESULT

UNION RESULT는 결과를 담아두는 테이블을 의미한다.

```sql
EXPLAIN
select emp_no from salaries where salary > 100000
UNION
select emp_no from dept_emp where from_date > '2001-01-01';
```



|                    |                          |                                  |                  |
| :----------------- | :----------------------- | :------------------------------- | :--------------- |
| **id**             | 1                        | 2                                | NULL             |
| **select\_type**   | PRIMARY                  | UNION                            | UNION RESULT     |
| **table**          | salaries                 | dept\_emp                        | &lt;union1,2&gt; |
| **partitions**     | NULL                     | NULL                             | NULL             |
| **type**           | index                    | range                            | ALL              |
| **possible\_keys** | ix\_salary               | ix\_fromdate,ix\_empno\_fromdate | NULL             |
| **key**            | ix\_salary               | ix\_fromdate                     | NULL             |
| **key\_len**       | 4                        | 3                                | NULL             |
| **ref**            | NULL                     | NULL                             | NULL             |
| **rows**           | 1                        | 4948                             | NULL             |
| **filtered**       | 100                      | 100                              | NULL             |
| **Extra**          | Using where; Using index | Using where; Using index         | Using temporary  |

MySQL에서 UNION 쿼리는 임시테이블을 생성하게 되는데, 실행 계획에서 이 임시 테이블을 가리키는 select_type은 UNION RESULT다. UNION RESULT는 실제 쿼리에서 단위 쿼리가 아니기 때문에 별도로 id 값은 부여되지 않는다.

`<union1,2>` 의 뜻은 id값이 1과 2를 UNION 했다는 것을 의미한다.

> MySQL 5.7.3 이전에는 UNION ALL도 임시테이블을 만들었다. 하지만 5.7.3 이후부터는 UNION (DISTINCT) 만 임시테이블을 생성한다.

#### SUBQUERY

일반적으로 서브 쿼리라고 하면 여러 가지를 통틀어서 이야기할 때가 많은데, 여기서  SUBQUERY라고 하는 것은 FROM 절 이외에서 사용되는 서브 쿼리만을 의미한다. 