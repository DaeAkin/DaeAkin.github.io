---
layout: post
title: JPA 클래스와 기본키 맵핑
categories: [JPA]
comments: true
tags:
- JPA
---

# JPA 클래스와 기본키 맵핑

## 1. @Entity 

**JPA를 사용해서 테이블과 매핑할 클래스는 @Entity 어노테이션을 필수로 붙여야 합니다.**

**@Entity가 붙은 클래스는 JPA가 관리하는 것으로 엔티티라 부릅니다.**

### 속성 

- name : JPA에서 사용할 엔티티 이름을 지정한다. 기본 값으로는 클래스의 이름을 붙입니다.

### 주의사항

- 기본 생성자는 필수다.(파라미터가 없는 public 또는 protected 생정자)
- final 클래스 , enum,interface,inner 클래스에는 사용 불가능
- 저장할 필드에 final을 사용하면 안됩니다.

> 자바는 생성자가 없으면 기본생성자를 자동으로 만들어줍니다.

```java
@Entity
public class Member{
  
}
```



## 2. @Table

**@Table은 엔티티와 매핑할 테이블을 지정합니다.. 생략하면 매핑한 엔티티 이름을 테이블의 이름으로 사용합니다.**

### 속성

- name : 매핑할 테이블 이름으로 기본값은 엔티티 이름을 사용함.

- catalog : catalog 기능이 있는 데이터베이스에서 catalog를 매핑합니다.

- schema : schema 기능이 있는 데이터베이스에서 schema를 매핑합니다.

- uniqueConstraints(DDL) : DDL 생성 시에 유니크 제약조건을 만듭니다. 2개 이상의 복합 유니크 제약조건도 만들 수 있습니다.. 참고로 이 기능은 스키마 자동 생성 기능을 사용해서 DDL을 만들 때만 사용됩니다.

  ```java
  @Table(name="MEMBER", uniqueConstraints = {@UniqueConstraint(
  	name="NAME_AGE_UNIQUE",
  	columnNames = {"NAME","AGE"})})
  public class Member {
    @Id
    @Column(name="id")
    private String id;
    
    @Column(name="name")
    private String username;
    
    private Integer age;
  }
  ```

  **DDL**
  
  ```
  ALTER TABLE MEMBER ADD CONSTRAINT NAME_AGE_UNIQUE UNIQUE (NAME,AGE)
  ```

​	

> DDL(Data Definition Language)
>
> CREATE ALTER DROP 등 테이블에서 데이터 구조를 정의하는데 사용되는 명령어들입니다.



## hibernate.hbm2ddl.auto 속성

**SpringBoot jpa 설정**

```yaml
jpa:
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
#      ddl-auto:validate
      ddl-auto: update
    show-sql: true
```

- create : 기존 테이블을 삭제하고 새로 생성 DROP + CREATE
- create-drop : DROP + CREATE + DROP (로컬에 적합)
- update : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정 (개발서버에 적합)
- validate : 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항이 있으면 경고를 남기고 애플리케이션을 실행하지 않습니다.. 이 설정은 DDL을 수정하지 않습니다.. (운영서버에 적합)
- none : 아무것도 안함.  

JPA2.1부터는 update 및 validate를 지원하지 않음.



## 이름매핑전략

기본적으로 loginMethod -> login_method로 언더스코어를 주로 사용합니다.



## 기본키 맵핑 @Id , @GeneratedValue

**데이터베이스마다 기본 키를 생성하는 방식이 서로 다릅니다.**
**Oracle은 시퀀스를 사용하고 Mysql은 AUTO_INCREMENT 같은 기능을 사용합니다.**

JPA가 제공하는 데이터베이스 기본 키 생성 전략은 다음과 같습니다.

- 직접할당 : 기본 키를 애플리케이션에서 직접 할당합니다.
- 자동생성 : 대리 키 사용 방식 

## 1. IDENTITY 자동생성

**기본 키 생성을 데이터베이스에 위임한다. 이 전략은 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용합니다.**

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

> IDENTITY 전략과 최적화
>
> IDENTITY 전략은 데이터를 DB에 넣은 후에 기본 키 값을 조회 할 수 있어 SELECT문을 한번 더 호출해야 하지만,
> JDBC3에 추가된 Statement.getGeneratedKeys()를 사용하면 저장과 동시에 기본키 값을 얻어 올 수 있다.
> 하이버네이트는 이 메소드를 사용해서 DB와 통신을 한번만 한다.
>
> 또한 엔티티를 DB에 넣어야 기본키를 얻을 수 있기 때문에 쓰기 지연이 동작하지 않는다.

## 2. SEQUENCE 자동생성

**데이터베이스 시퀀스를 사용해서 기본 키를 할당합니다.이 전략은 주로 Oracle,PostgreSQL,DB2,H2에서 사용합니다.**

```java
@Entity
@SequenceGenerator(
	name="BOARD_SEQ_GENERATOR",
	sequenceName = "BOARD_SEQ", //매핑할 데이터베이스 시퀀스 이름 
	initialValue = 1, allocationSize = 1)
public class Board {
  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE,
                 	generator = "BOARD_SEQ_GENERATOR")
  private Long id;
}
```

### @SequenceGenerator 속성

- name : 식별자 생성기 이름이며 필수 값입니다.
- sequenceName : 데이터베이스에 등록되어 있는 시퀀스 이름이며 기본값은 `hibernate_sequence`입니다.
- initialValue : DDL 생성시에만 사용되며 처음 시작하는 수를 지정합니다. 기본값은 1 입니다.
- allocationSize : 시퀀스 한 번 호출에 증가하는 수이며, 성능 최적화를 위해 기본값은 50입니다.
- catalog, schema : 데이터베이스 catalog, schema 이름 입니다.

> SEQUENCE 전략과 최적화
>
> SEQUENCE 전략은 데이터베이스 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다. 따라서 데이터베이스와 2번 통신합니다.
>
> 1. 식별자를 구하려고 데이터베이스 시퀀스를 조회
> 2. 조회한 시퀀스를 기본 키 값으로 사용해 데이터베이스에 저장.
>
> JPA는 시퀀스에 접근하는 횟수를 줄이기 위해 @SequenceGenerator.allocationSize를 사용합니다. 
> 여기서 설정한 값만큼 한 번에 시퀀스 값을 증가시키고 나서 그만큼 메모리에 시퀀스 값을 할당합니다.
> 예를 들어 allocationSize 값이 50이면 시퀀스를 한 번에 50증가 시킨 다음에 1~50까지는 메모리에서 식별자를 할당합니다. 
> 그리고 51이 되면 시퀀스 값을 100으로 증가시킨 다음 51~100까지 메모리에서 식별자를 할당합니다.

## 3. TABLE 자동생성

**키 생성 테이블을 사용한다. 키 생성용 테이블을 하나 만들어두고 마치 시퀀스처럼 사용하는 방법입니다.**

**모든 데이터베이스에서 사용가능합니다.**

### TALBE 전략 용 테이블 생성 DDL

```
create table MY_SEQUENCES (
	sequence_name varchar(255) not null,
	next_val bigint,
	primary key (sequence_name)
)
```

### Entity 매핑

```java
@Entity
@TableGenerator(
	name = "BOARD_SEQ_GENERATOR",
  table = "MY_SEQUENCES",
  pkColumnValue = "BOARD_SEQ", allocationSize = 1)
public class Board {
  @Id
  @GeneratedValue(strategy = GenerationType.TABLE,
       generator = "BOARD_SEQ_GENERATOR")
  private Long id;
}
```



TALBE 전략은 시퀀스 대신에 테이블을 사용한다는 것만 제외한다면 SEQUENCE 전략과 내부 동작이 같습니다.



### TableGeneretor의 속성

- name : 식별자 생성기 이름이며 필수입니다.
- table : 키생성 테이블명이며 기본값은 hibernate_sequences 입니다.
- pkColumnName : 시퀀스 컬럼명이며, 기본값은 sequence_name 입니다.
- valueColumnName : 시퀀스 값 컬럼명이며 기본값은 next_val 입니다.
- pkColumnValue : 키로 사용할 값 이름이며 기본값은 엔티티 이름입니다.
- initialValue : 초기 값 입니다. 마지막으로 생성된 값이 기준입니다. 기본값은 0 입니다.
- allocationSize : 시퀀스 한 번 호출에 증가하는 수입니다. 이 성은 성능 최적화의 사용되며 기본 값은 50입니다.
- catalog,schema : 데이터베이스 catalog, schema 이름
- uniqueConstraints(DDL) : 유니크 제약 조건을 지정할 수 있습니다.

> TABLE 전략과 최적화
>
> TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고 다음 값으로 증가시키기 위해 UPDATE 쿼리를 사용합니다. 
> 이 전략은 SEQUENCE 전략과 비교해서 데이터베이스와 한 번 더 통신하는 단점이 있습니다.
> 최적화 하기 위해서는 SEQUENCE 전략처럼 @TableGenerator.allocationSize를 이용하면 됩니다. 



## 4. AUTO 전략

**GenerationType.AUTO는 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택합니다.** 

**데이터베이스 방언을 오라클을 선택하면 SEQUENCE를, MySQL이라면 IDENTITY를 사용합니다.**

```java
@Entity
public class Board {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Long id;
}
```



@GeneratedValue의 strategy 기본값은 AUTO이기 때문에 생략해도 됩니다.

```java
@Id @GeneratedValue
private Long id;
```



AUTO 전략의 장점은 데이터베이스를 변경해도 코드를 수정하지 않아도 됩니다.

특히 키 생성 전략이 아직 확정되지 않은 개발 초기 단계나 프로토타입 개발 시 편리하게 사용할 수 있습니다.

AUTO를 사용할 때 SEQUENCE나 TABLE 전략이 선택되면 시퀀스나 키 생성용 테이블을 미리 만들어 두어야 합니다. 
만약 스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본값을 사용해서 적절한 시퀀스나 키 생성용 테이블을 만들어 줍니다.



## 기본키 생성전략 정리

기본 키 매핑은 영속성 컨텍스트가 엔티티를 식별자 값으로 구분하므로 엔티티를 영속 상태로 만들려면 식별자 값이 반드시 있어야합니다.
