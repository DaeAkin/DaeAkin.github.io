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


그래서 위에 있는 쿼리를 EXPLAIN을 해보면 2개의 id컬럼 값이 나온다.

#### 조인일때

**만약 하나의 SELETE 문장으로 여러 개의 테이블을 조인하면 조인되는 테이블의 개수만큼 실행 계획 레코드가 출력되지만 같은 id가 부여된다.**

```sql
EXPLAIN
SELECT e.emp_no, e.first_name, s.from_date, s.salary
FROM employees e, salaries s
WHERE e.emp_no=s.emp_no
LIMIT 10;
```

| id   | select\_type | table | type  | possible\_keys | key           | key\_len | ref                 | rows   | Extra       |
| :--- | :----------- | :---- | :---- | :------------- | :------------ | :------- | :------------------ | :----- | :---------- |
| 1    | SIMPLE       | e     | index | PRIMARY        | ix\_firstname | 16       | NULL                | 282916 | Using index |
| 1    | SIMPLE       | s     | ref   | PRIMARY        | PRIMARY       | 4        | employees.e.emp\_no | 4      |             |

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

일반적으로 서브 쿼리라고 하면 여러 가지를 통틀어서 이야기할 때가 많은데, 여기서  SUBQUERY라고 하는 것은 **<u>FROM 절 이외에서 사용되는 서브 쿼리</u>**만을 의미한다. 

```sql
EXPLAIN
select
    e.first_name,
    (select count(*) from dept_emp de, dept_manager dm where dm.dept_no = de.dept_no) as cnt
from employees e
where e.emp_no = 30000;
```

| id   | select\_type | table | partitions | type  | possible\_keys | key     | key\_len | ref                   | rows  | filtered | Extra       |
| :--- | :----------- | :---- | :--------- | :---- | :------------- | :------ | :------- | :-------------------- | :---- | :------- | :---------- |
| 1    | PRIMARY      | e     | NULL       | const | PRIMARY        | PRIMARY | 4        | const                 | 1     | 100      | NULL        |
| 2    | SUBQUERY     | dm    | NULL       | index | PRIMARY        | PRIMARY | 20       | NULL                  | 24    | 100      | Using index |
| 2    | SUBQUERY     | de    | NULL       | ref   | PRIMARY        | PRIMARY | 16       | employees.dm.dept\_no | 38278 | 100      | Using index |

FROM 절에서 사용된 서브쿼리는 select_type이 DERIVED 라고 표시되고, 그 밖의 위치에서 사용된 서브 쿼리는 전부 SUBQUERY라고 표시 된다.

> 서브 쿼리의 종류
>
> - 중첩된 쿼리(Nested Query)
>
>   SELECT 되는 칼럼에 사용된 서브 쿼리를 네스티드 쿼리라고 한다.
>
> - 서브 쿼리(Sub Query)
>
>   WHERE 절에 사용된 경우에는 일반적으로 그냥 서브 쿼리라고 한다.
>
> - 파생 테이블(Derived)
>
>   FROM 절에 사용된 서브 쿼리를 MySQL에서는 파생 테이블이라고 하며, 일반적으로 RDBMS 전체적으로 인라인 뷰 또는 서브 셀렉트라고 부르기도 한다.
>
> 또한 서브 쿼리가 반환하는 값의 특성에 따라 다음과 같이 구분하기도 한다.
>
> - 스칼라 서브 쿼리 (Scalar SubQuery)
>
>   하나의 값만(칼럼이 단 하나인 레코드 1건만) 반환하는 쿼리
>
> - 로우 서브 쿼리(Row Sub Query)
>
>   칼럼의 개수에 관계없이 하나의 레코드만 반환하는 쿼리



#### DEPENDENT SUBQUERY

서브 쿼리가 바깥쪽의 정의된 컬럼을 사용하는 경우를 DEPENDENT SUBQUERY 라고 한다.

```sql
EXPLAIN
select e.first_name,
    (select count(*)
        from dept_emp de, dept_manager dm
        where dm.dept_no=de.dept_no and de.emp_no=e.emp_no) AS cnt
from employees e
where e.emp_no = 30000;
```

| id   | select\_type       | table | partitions | type  | possible\_keys              | key                 | key\_len | ref                   | rows | filtered | Extra       |
| :--- | :----------------- | :---- | :--------- | :---- | :-------------------------- | :------------------ | :------- | :-------------------- | :--- | :------- | :---------- |
| 1    | PRIMARY            | e     | NULL       | const | PRIMARY                     | PRIMARY             | 4        | const                 | 1    | 100      | NULL        |
| 2    | DEPENDENT SUBQUERY | de    | NULL       | ref   | PRIMARY,ix\_empno\_fromdate | ix\_empno\_fromdate | 4        | const                 | 1    | 100      | Using index |
| 2    | DEPENDENT SUBQUERY | dm    | NULL       | ref   | PRIMARY                     | PRIMARY             | 16       | employees.de.dept\_no | 2    | 100      | Using index |

안쪽의 서브 쿼리가 바깥족에 있는 employess 테이블의 컬럼을 사용하고 있어 의존적이기 때문에 DEPENDENT라는 키워드가 붙는다. **DEPENDENT가 붙으면 외부 쿼리가 먼저 수행된 후 내부 쿼리를 실행돼야 하기 때문에 일반 서브 쿼리보다는 처리 속도가 느릴 때가 많다.**

#### DERIVED

**<u>서브 쿼리가 FROM 절에 사용된 경우</u>** MySQL은 항상 select_type이 DERIVED인 실행 계획을 만든다. DERIVED는 단위 SELECT 쿼리의 실행 결과를 메모리나 디스크에 임시 테이블을 생성하는 것을 의미 한다.

안타깝게도 MySQL은 FROM 절에 사용된 서브 쿼리를 제대로 최적화하지 못할 때가 대부분이다. **파생테이블은 인덱스가 전혀 없으므로 다른 테이블과 조인할 때 성능상 불리할 때가 많다.**

```sql
EXPLAIN
select *
from
(select de.emp_no from dept_emp de) tb,
     employees e
where e.emp_no = tb.emp_no;
```

**MySQL 5.4에서 결과**

| id   | select\_type | table            | type    | possible\_keys | key          | key\_len | ref        | rows   | Extra                           |
| :--- | :----------- | :--------------- | :------ | :------------- | :----------- | :------- | :--------- | :----- | :------------------------------ |
| 1    | PRIMARY      | &lt;derived2&gt; | ALL     | NULL           | NULL         | NULL     | NULL       | 306663 | Using temporary; Using filesort |
| 1    | PRIMARY      | e                | eq\_ref | PRIMARY        | PRIMARY      | 4        | tb.emp\_no | 1      |                                 |
| 2    | DERIVED      | de               | index   | NULL           | ix\_fromdate | 3        | NULL       | 309245 | Using index                     |

**MySQL 8.0에서 결과**

| id   | select\_type | table | partitions | type    | possible\_keys      | key          | key\_len | ref                  | rows   | filtered | Extra       |
| :--- | :----------- | :---- | :--------- | :------ | :------------------ | :----------- | :------- | :------------------- | :----- | :------- | :---------- |
| 1    | SIMPLE       | de    | NULL       | index   | ix\_empno\_fromdate | ix\_fromdate | 3        | NULL                 | 306228 | 100      | Using index |
| 1    | SIMPLE       | e     | NULL       | eq\_ref | PRIMARY             | PRIMARY      | 4        | employees.de.emp\_no | 1      | 100      | NULL        |

위의 SQL은 서브 쿼리를 간단히 제거하고 조인으로 처리할 수 있다. 실제로 다른 DBMS에서는 이렇게 쿼리를 재작성하는 형태의 최적화 기능도 제공한다.

MySQL 5.4에서는 FROM 절의 서브 쿼리를 임시테이블로 만들어서 처리한다.

MySQL 6.0이상 버전부터는 FROM 절의 서브 쿼리에 대한 최적화 부분이 많이 개선되어서 위에와 같이 타입이 DERIVED인 타입이 없다. 하지만 그전까지는 FROM 절의 서브 쿼리는 상당히 신경 써서 개발하고 튜닝 해야 한다.

#### UNCACHEABLE SUBQUERY

하나의 쿼리 문장에서 서브 쿼리가 하나만 있더라도 실제 그 서브 쿼리가 한 번만 실행되는 것은 아니다. 그런데 조건이 똑같은 서브 쿼리가 실행될 때는 다시 실행하지 않고 이전의 실행 결과를 그대로 사용할 수 있게 서브 쿼리의 결괴를 내부적인 캐시 공간에 담아둔다.

**Subquery 경우**

서브쿼리는 바깥쪽 영향을 받지 않으므로 처음 한 번만 실행해서 그 결과를 캐시하고 필요할 때 캐시된 결과를 이용한다.

**Dependent Subquery 경우**

디펜던트 서브쿼리는 의존하는 바깥쪽 쿼리의 컬럼의 값 단위로 캐시해두고 사용한다.

unchacheable subquery는 캐시를 사용할 수 있냐 없느냐에 차이가 있다. 서브 쿼리에 포함된 요소에 의해 캐시 자체가 불가능할 수 있는데 이 경우 select_type이 UNCACHEABLE SUBQUERY로 표시 된다. 캐시를 사용하지 못하도록 하는 요소로는 대표적으로 다음과 같은 것 들이 있다.

- 사용자 변수가 서브 쿼리에 사용된 경우
- NOT-DETERMINISTIC 속성의 스토어드 루틴이 서브 쿼리내에 사용된 경우
- UUID()나 RAND()와 같이 결과값이 호출할 때마다 달라지는 함수가 서브 쿼리에 사용된 경우



**사용자 변수 @status를 사용한 쿼리**

```sql
EXPLAIN
select *
from employees e
where e.emp_no = (
    select @status from dept_emp de where de.dept_no='D005'
    );
```



| id   | select\_type         | table | type | possible\_keys | key     | key\_len | ref   | rows   | Extra                    |
| :--- | :------------------- | :---- | :--- | :------------- | :------ | :------- | :---- | :----- | :----------------------- |
| 1    | PRIMARY              | e     | ALL  | NULL           | NULL    | NULL     | NULL  | 282916 | Using where              |
| 2    | UNCACHEABLE SUBQUERY | de    | ref  | PRIMARY        | PRIMARY | 4        | const | 139228 | Using where; Using index |



#### UNCACHEABLE UNION 

UNCACHEABLE UNION이란 UNCACHEABLE 과 UNION 두 개의 키워드 조합된 select_type을 의미한다. MySQL 5.1부터 추가되었다.

### table 컬럼

MySQL의 실행계획은 단위 SELECT 쿼리 기준이 아니라 **테이블 기준으로 표시**된다. 만약 테이블의 이름에 별칭이 부여된 경우에는 별칭이 표시된다.

```sql
EXPLAIN select now();
```

| id   | select\_type | table | type | possible\_keys | key  | key\_len | ref  | rows | Extra          |
| :--- | :----------- | :---- | :--- | :------------- | :--- | :------- | :--- | :--- | :------------- |
| 1    | SIMPLE       | NULL  | NULL | NULL           | NULL | NULL     | NULL | NULL | No tables used |

### <derived N\>의 의미

Table 칼럼에 <derived\> 또는 <union\> 과 같이 "<>"로 둘러싸인 이름이 명시되는 경우가 많은데, 이 테이블은 임시 테이블을 의미한다. 또한 "<>"안에 항상 표시되는 숫자는 단위 SELECT 쿼리의 id를 지칭한다.

| id   | select\_type | table            | type    | possible\_keys | key          | key\_len | ref        | rows   | Extra                           |
| :--- | :----------- | :--------------- | :------ | :------------- | :----------- | :------- | :--------- | :----- | :------------------------------ |
| 1    | PRIMARY      | &lt;derived2&gt; | ALL     | NULL           | NULL         | NULL     | NULL       | 306663 | Using temporary; Using filesort |
| 1    | PRIMARY      | e                | eq\_ref | PRIMARY        | PRIMARY      | 4        | tb.emp\_no | 1      |                                 |
| 2    | DERIVED      | dept_emp         | index   | NULL           | ix\_fromdate | 3        | NULL       | 309245 | Using index                     |

위의 표를 보면 첫 번째의 table 컬럼의 값이 <drived2\> 인데 이 뜻은 단위 SELECT 쿼리의 아이디가 2번인 실행 계획으로부터 만들어진 파생 테이블을 가리킨다. detp_emp 테이블로부터  select 된 결과가 저장된 파생 테이블이라는 점을 알 수 있다.

**위에 테이블만 보고 분석해보기**

1. 첫 번째 라인의 테이블이 <derived2\> 인것으로 보아 첫번 째 라인 보다는 id가 2인 쿼리가 먼저 실행 된 후 결과가 파생테이블에 넣어야 한다는걸 알 수 있다. 그럼 id 2번으로 가보자.
2. id 2번의 select_type 컬럼의 값이 DERIVED이다. 즉 이 라인은 table 컬럼에 표시된 dept_emp 테이블을 읽어서 파생 테이블을 생성한다. 
3. id 2번의 분석이 끝났으니 다시 첫번 째 라인으로 돌아가자.
4. 첫 번째라인과 두 번째라인은 같은 id 값을 가지고 있어서 2개 테이블은 서로 조인이 되는 쿼리라는 사실을 알 수 있다. 그런데 <derived2\> 테이블이 e 테이블보다 먼저 표시됐기 때문에 <derived2\>가 드라이빙 테이블이 되고 e 테이블이 드리븐 테이블이 된다는것을 알 수 있다. 즉 <derived2\> 테이블이 먼저 읽어서 e 테이블로 조인을 실행했다는 것을 알 수 있다.

이렇게 분석을하면 다음과 같은 쿼리가 나온다.

```sql
select *
from
	(select de.emp_no from dept_emp de) tb,
	emplyees e
where e.emp_no=tb.emp_no;
```

> MySQL은 다른 DBMS와 달리  FROM 절에 사용된 쿼리인 즉 type이 Derived인 파생테이블은 반드시 별칭을 가져야 한다. 그렇지 않으면 에러가 나온다.

### type 컬럼

쿼리의 실행 계획에서 type 이후의 컬럼은  MySQL 서버가 각 테이블의 레코드를 어떤 방식을 읽었는지 의미를 한다.

- 인덱스를 사용해 읽었는지?
- 테이블을 처음부터 끝까지 읽는 풀 테이블 스캔으로 읽었는지? 

일반적으로 쿼리를 튜닝할 때 인덱스를 효율적으로 사용하는지 확인하는 것이 중요하다.

> MySQL의 매뉴얼에서는  type 컬럼을 "조인 타입"으로 소개하고 있다.

type 컬럼에 표시되는 값은 버전마다 조금씩 다른데, MySQL5.0,5.1 기준으로 많이 사용되는 값들은 다음과 같다.

- system
- const 
- eq_ref
- ref
- fulltext
- ref_or_null
- unique_sbuquery
- index_subquery
- range
- index_merge
- index
- ALL

ALL을 제외한 나머지는 모두 인덱스를 사용하며, ALL은 풀 테이블 스캔을 사용한다.

index_merge를 제외한 나머지는 하나의 인덱스만 사용한다.

#### system

레코드가 1건만 존재하거나, 하나도 존재하지 않는 테이블에 접근하는 방법을 system이라고 한다. 이 접근 방식은 InnoDB 테이블에서는 나타나지 않고 MyISAM이나 MEMORY 테이블에서만 사용된다.

```sql
EXPLAIN
select * from tb_dual;
```

#### const

테이블의 레코드 건수 상관 없이 WHERE 조건으로 PK나 UNIQ 키 컬럼을 이용하여 조회를 하여 반드시 1건을 반환하는 쿼리 방식을 const라고 한다. 다른 DBMS에서는 유니크 인덱스 스캔이라고도 한다.

```sql
EXPLAIN
select * from employees where emp_no=30000;
```

| id   | select\_type | table     | type  | possible\_keys | key     | key\_len | ref   | rows | Extra |
| :--- | :----------- | :-------- | :---- | :------------- | :------ | :------- | :---- | :--- | :---- |
| 1    | SIMPLE       | employees | const | PRIMARY        | PRIMARY | 4        | const | 1    |       |

그러나 PK나 UNIQ 키가 여러개 있는데, **일부만 이용하면** const 타입의 접근을 이용할 수 없다. 이 경우에는 실제로 레코드가 1건만 저장돼있더라도 MySQL 엔진이 데이터를 읽어보지 않고서는 레코드가 1건이라는 것을 확신할 수 없기 때문이다.

```sql
EXPLAIN
select * from employees where emp_no=30000;
```

| id   | select\_type | table     | type | possible\_keys | key     | key\_len | ref   | rows   | Extra       |
| :--- | :----------- | :-------- | :--- | :------------- | :------ | :------- | :---- | :----- | :---------- |
| 1    | SIMPLE       | dept\_emp | ref  | PRIMARY        | PRIMARY | 4        | const | 139228 | Using where |

일부 조건만 사용하면 const가 아닌 ref로 표시된다.

type 컬럼이 const인 실행 계획은 MySQL의 옵티마이저가 쿼리를 최적화하는 단계에서 모두 상수화 한다. 그래서 실행 계획의 type 컬럼의 값이 상수(const)라고 표시 된다.

#### eq_ref

- 여러테이블이 조인이 되며
- **조인에서 처음 읽은 테이블의 컬럼 값이**
- 그 다음 읽어야할 테이블의 PK나 UNIQ 키 컬럼의 검색조건에 사용할 때

이 조건을 만족하는 두 번째 이후에 읽는 테이블의 type 컬럼에 eq_ref가 표시 된다. 또한 두 번째 이후에 읽히는 테이블을 유니크 키로 검색할 때 그 유니크 인덱스는 는  NOT NULL 이어야 하며, 다중 컬럼으로 만들어진 **PK나 UNIQ 인덱스라면 인덱스의 모든 컬럼이 비교 조건에 사용되어야 한다.** 

즉 조인에서 **두 번째 이후에 읽는 테이블에서 반드시 1건만 존재해야 한다는 보장이 있어야 한다.**

```sql
EXPLAIN
select * from dept_emp de, employees e
where e.emp_no=de.emp_no AND de.dept_no='d005';
```

| id   | select\_type | table | type    | possible\_keys              | key     | key\_len | ref                  | rows   | Extra       |
| :--- | :----------- | :---- | :------ | :-------------------------- | :------ | :------- | :------------------- | :----- | :---------- |
| 1    | SIMPLE       | de    | ref     | PRIMARY,ix\_empno\_fromdate | PRIMARY | 4        | const                | 139228 | Using where |
| 1    | SIMPLE       | e     | eq\_ref | PRIMARY                     | PRIMARY | 4        | employees.de.emp\_no | 1      |             |

#### ref

- 조인의 순서와 관계 없으며
- PK나  UNIQ 키 등의 제약 조건도 없으며
- 인덱스의 종류와 관계 없이 동등(Equal) 조건으로 검색할 때.

반환되는 레코드의 값이 반드시 1건이라는 보장이 없으므로 const나  eq_ref보다는 느리다.

```sql
EXPLAIN
select * from dept_emp WHERE dept_no='d005';
```

| id   | select\_type | table     | type | possible\_keys | key     | key\_len | ref   | rows   | Extra       |
| :--- | :----------- | :-------- | :--- | :------------- | :------ | :------- | :---- | :----- | :---------- |
| 1    | SIMPLE       | dept\_emp | ref  | PRIMARY        | PRIMARY | 4        | const | 139228 | Using where |

지금까지 살펴본 `const` , `eq_req` ,`ref` select_type은 모두 매우 좋은 접근 방법으로 인덱스의 분포도가 나쁘지 않다면 성능상의 문제를 일으키지 않기 때문에, 쿼리를 튜닝할 때 이 세가지 접근 방법에 대해서는 크게 신경쓰지 않고 넘어가도 무방하다.

#### fulltext

fulltext 접근 방법은 MySQL의 전문 검색 인덱스를 사용해 레코드를 읽는 접근 방법을 의미한다. 

비용 기반의 옵티마이저는 통계 정보를 이용해 비용을 계산하지만, 전문 검색 인덱스는 통계 정보가 관리되지 않으며, 전문 검색 인덱스를 사용하려면 전혀 다른 SQL 문법을 사용해야 한다. 그래서 MySQL 옵티마이저는 전문 인덱스를 사용할 수 있는 SQL에서는 쿼리의 비용과는 관계없이 매번 fulltext 접근 방법을 사용한다.

fulltext 접근 방법보다 명백히 빠른 const나 eq_ref 또는 ref 접근 방법을 사용할 수 있는 쿼리에서는 억지로 fulltext 접근 방법을 선택하지는 않는다.

MySQL의 전문 검색 조건은 우선순위가 상당히 높다. 쿼리에서 전문 인덱스를 사용하는 조건과 그 이외의 일반 인덱스를 사용하는 조건을 함께 사용하면 일반 인덱스의 접근 방법이 const나 eq_ref, 그리고 ref가 아니면 일반적으로 로  MySQL은 전문 인덱스를 사용하는 조건을 선택해서 처리한다.

전문 검색은 "MATCH ... AGAINST"  구문을 사용해서 실행하는데, 반드시 해당 테이블에 전문 검색용 인덱스가 있어야 한다. 만약 테이블에 전문 인덱스가 없다면 쿼리는 오류가 발생하고 중지 된다. 

```sql
EXPLAIN
select *
from employee_name
where emp_no = 30000
    AND emp_no BETWEEN 30000 AND 30005
AND MATCH(first_name, last_name) AGAINST('Matt'  IN BOOLEAN MODE);
```
여기 `where emp_no = 30000` 구문은 pk를 이용했기때문에 1건만 조회되는 const 타입의 조건이며 `emp_no BETWEEN 30000 AND 30005` 구문은 나중에 설명할 range 타입의 조건이고, 마지막으론 전문 검색 조건이다.

| id   | select\_type | table          | type  | possible\_keys   | key     | key\_len | ref   | rows | Extra       |
| :--- | :----------- | :------------- | :---- | :--------------- | :------ | :------- | :---- | :--- | :---------- |
| 1    | SIMPLE       | employee\_name | const | PRIMARY,fx\_name | PRIMARY | 4        | const | 1    | Using where |

위에 sql문의 실행계획을 살펴보면 다음과 같이 첫번 째 조건인 const 타입을 선택한다.

만약 첫번 째 조건을 제외하고 2번째 조건과 3번째 조건으로 조회하면 실행 계획은 다음과 같다.

```sql
select *
from employee_name
where emp_no BETWEEN 30000 AND 30005
AND MATCH(first_name, last_name) AGAINST('Matt'  IN BOOLEAN MODE);
```

| id   | select\_type | table          | type     | possible\_keys   | key      | key\_len | ref  | rows | Extra       |
| :--- | :----------- | :------------- | :------- | :--------------- | :------- | :------- | :--- | :--- | :---------- |
| 1    | SIMPLE       | employee\_name | fulltext | PRIMARY,fx\_name | fx\_name | 0        |      | 1    | Using where |

타입이 range가 아니라, 전문 검색 조건을 선택했다. 일반적으로 쿼리에 전문 검색 조건을 사용하면  MySQL은 아무런 주저 없이  fulltext 접근 방식을 사용하는 경향이 있다. 하지만 지금까지의 경험으로 보면 전문 검색 인덱스를 이용하는 fulltext보다 일반 인덱스를 이용하는 range 접근 방법이 더 빨리 처리되는 경우가 더 많았다. 전문 검색 쿼리를 사용할 때는 각 조건별로 성능을 확인해 보는 편이 좋다.

####  ref_or_null

이 접근 방식은 ref 접근 방식과 같은데 NULL 비교가 추가된 형태다. 

-  ref 방식 또는 NULL 비교(IS NULL) 접근 방식

실제 업무에서 많이는 쓰이지 않는다.

```sql
EXPLAIN
select *
from titles
where to_date = '1985-03-01'
OR to_date IS NULL;
```

| id   | select\_type | table  | type          | possible\_keys | key        | key\_len | ref   | rows | Extra                    |
| :--- | :----------- | :----- | :------------ | :------------- | :--------- | :------- | :---- | :--- | :----------------------- |
| 1    | SIMPLE       | titles | ref\_or\_null | ix\_todate     | ix\_todate | 4        | const | 2    | Using where; Using index |

#### unique_subquery

- WHERE 조건절에서 사용될 수 있는 IN 형태
- 서브 쿼리에서 중복되지 않는 유니크한 값을 반환할 때

```sql
EXPLAIN
select * from departments
where dept_no IN (
    select dept_no from dept_emp where emp_no=30000
    );
```

| id   | select\_type       | table       | type             | possible\_keys              | key          | key\_len | ref        | rows | Extra                    |
| :--- | :----------------- | :---------- | :--------------- | :-------------------------- | :----------- | :------- | :--------- | :--- | :----------------------- |
| 1    | PRIMARY            | departments | index            | NULL                        | ux\_deptname | 42       | NULL       | 9    | Using where; Using index |
| 2    | DEPENDENT SUBQUERY | dept\_emp   | unique\_subquery | PRIMARY,ix\_empno\_fromdate | PRIMARY      | 8        | func,const | 1    | Using index; Using where |

위에 쿼리 문장의  IN 부분에서  서브쿼리를 살펴보자. emp_no=30000인 레코드 중에서 부서 번호는 중복이 없기 때문에(dept_emp 테이블에서 PK가 dept_no + emp_no 이니까) 실행 계획의 두번째 라인의  dept_emp 테이블의 접근 방식은  unique_subquery로 표시된 것이다.

> 질문
>
> 위에 있는 쿼리를 다음과 같이 두개로 쪼갤 수 잇지 않을까.
>
> 1. select * from departments where ??
> 2. select dept_no from dept_emp where emp_no = 30000
>
> 2번의 쿼리 결과로 2개 이상의 쿼리가 나올 수 있다고 생각한다. 왜냐하면  pk가 dept_no + emp_no 이기 때문이다. 그러면  unique 하지 않기 때문에  unique_subquery란 말은 어울리지 않지 않을까?
>
> 그런데 내가 임의로 insert 문을 실행해서 데이터를 1개 더 넣었는데, 위에 실행 계획에서 id가 2번인 실행계획의 row는 2가 나와야 하는데 1만 나왔다. 이상하다.

#### index_subquery

IN 연산자 특성상,  IN (subquery) 또는 IN (상수 나열) 형태의 조건은 괄호 안에 있는 값의 목록에서 **중복된 값이 먼저 제거돼야 한다.** 위에서 살펴본 unique_subquery 접근 방법은 IN 조건의 subquery가 **중복된 값을 만들어내지 않는다는 보장이 있으므로** 별도의 중복을 제거할 필요가 없었다. 하지만 IN (subquery)에서 **subquery가 중복된 값을 반환할 수는 있지만 중복된 값을** 인덱스를 이용해 제거할 수 있을 때  index_subquery 접근 방법이 사용 된다.

- unique_subquery

  IN 형태의 조건에서 subquery의 반환 값에는 중복이 없으므로 별도의 중복 제거가 필요하지 않음.

- index_subquery

  IN 형태의 조건에서  subquery의 반환 값에 중복된 값이 있을 수 있지만 인덱스를 이용해 중복된 값을 제거할 수 있음.

두개 모두 중복 값을 아주 낮은 비용으로 제거한다.

```sql
select * from departments where dept_no IN (
    select dept_no FROM dept_emp where dept_emp.dept_no BETWEEN 'd001' AND 'd003'
    );
```

| id   | select\_type       | table       | type            | possible\_keys | key          | key\_len | ref  | rows  | Extra                    |
| :--- | :----------------- | :---------- | :-------------- | :------------- | :----------- | :------- | :--- | :---- | :----------------------- |
| 1    | PRIMARY            | departments | index           | NULL           | ux\_deptname | 42       | NULL | 9     | Using where; Using index |
| 2    | DEPENDENT SUBQUERY | dept\_emp   | index\_subquery | PRIMARY        | PRIMARY      | 4        | func | 17175 | Using index; Using where |

#### range

range는 하나의 값이 아니라 범위로 검색하는 인덱스 레인지 스캔형태의 접근 방법이다. 주로 <,>, IS NULL, BETWEN , IN , LIKE 등의 연산자를 이용해 인덱스를 검색할 때 사용한다. 주로 애플리케이션이 많이 쓰는 쿼리이며, 우선순위는 낮지만, 이 접근방법도 상당히 빠르며, 어느정도 성능 보장도 된다.

```sql
select dept_no FROM dept_emp where dept_emp.dept_no BETWEEN 'd001' AND 'd003'
```

| id   | select\_type | table     | type  | possible\_keys | key     | key\_len | ref  | rows   | Extra                    |
| :--- | :----------- | :-------- | :---- | :------------- | :------ | :------- | :--- | :----- | :----------------------- |
| 1    | SIMPLE       | dept\_emp | range | PRIMARY        | PRIMARY | 4        | NULL | 105938 | Using where; Using index |

> 인덱스 레인지 스캔이라고 하면 const , ref, range 라는 세 가지 접근 방법을 모두 묶어서 칭한다.