# Oracle Database 23ai - SQL Domains - Data Use Case Domain

- [Oracle Database 23ai - SQL Domains - Data Use Case Domain](#oracle-database-23ai---sql-domains---data-use-case-domain)
	- [Single-Column Domain](#single-column-domain)
	- [Monitor SQL Domains](#monitor-sql-domains)
	- [Multi Column Domain](#multi-column-domain)
	- [Enumeration Domain](#enumeration-domain)
	- [Flexible Domain](#flexible-domain)
	- [Built-in Domains](#built-in-domains)


Oracle Database 23ai 의 신기능 중 하나인 SQL Domain 별도의 Data Dictionary Object 로 테이블 컬럼에 대한 의도된 용도를 정의하여 애플리케이션 개발을 쉽고 편하게 할 수 있게 해 줍니다.  이를 통해 domain-specific 지식을 쉽게 재사용할 수 있게 해 줍니다.

SQL Domain 기능을 사용하여 컬럼에 대한 추가 정보를 제공하고, 이를 사용하여 데이터를 정의하고 검증할 수 있습니다.  또한, 테이블 정의를 변경하지 않고도 데이터 검증 논리나 정보를 변경할 수 있습니다.

SQL Domain 은 일반적인 값에 대한 속성이나 제약 조건을 정의하는 Data Dictionary Object 이고 스키마에 포함됩니다.  SQL Domain 은 제약 조건, 디스플레이, 순서 및 주석 정보등을 정의할 수 있습니다.  SQL Domain 을 정의한 후 테이블 컬럼에 해당 Domain 을 지정하여 컬럼에 Domain의 속성이나 제약 조건등을 명시적으로 적용할 수 있습니다.  따라서, SQL Domain 은 컬럼(관계형 또는 JSON)에 추가 정보를 제공하는데 사용되므로 데이터를 정의하고 검증하는데 사용할 수 있습니다.

SQL Domain 은  Single Point of Definition(SPOD)를 제공하여 애플리케이션 전체에 일관성을 제공합니다.  컬럼과 관련된 속성과 제약 조건을 한번 정의한 후 애플리케이션 전반에 걸쳐 해당 정의를 사용할 수 있습니다.

사용자는 Domain 에 대해 Create, Alter, Drop 작업을 수행할 수 있습니다.  Domain 을 사용하려면 `CREATE DOMAIN` 권한이 필요하며, 이는 `RESOURCE` 및 `DB_DEVELOPER_ROLE` Role 에 포함되어 있습니다.

Domain 은 Single-Column, Multi-Column, Flexible Domain 의 3가지 타입이 있습니다.  또한, 사용자가 바로 사용할 수 있는 Built-in Domain 도 제공합니다.

이제 간단한 테스트를 통해 SQL Domain 기능에 대해 살펴보겠습니다.

## Single-Column Domain

- 먼저, Domain 생성 Syntax 는 다음과 같습니다.  SQL Domain 생성을 위해 `CREATE DOMAIN` 구문을 사용합니다:

    ```sql
    CREATE DOMAIN [IF NOT EXISTS]  DomainName  AS  <Type> [STRICT]
    [ DEFAULT [ON NULL..] <expression>]
    [ [NOT] NULL]
    [ CONSTRAINT [Name] CHECK (<expression>) [ ENABLE | DISABLE] ..]*
    [ VALIDATE USING <json_schema_string>]
    [ COLLATE collation ]
    [ DISPLAY <expression> ]
    [ ORDER <expression> ]
    ```
    
    Domain drop 구문은 다음과 같습니다:
    ```
    DROP DOMAIN [IF EXISTS] domain_name [FORCE [PRESERVE]]
    ```

	**참고 사항** : `PRESERVE` 대신 `FORCE` 를 사용하면 Domain 과 종속인 모든 컬럼의 연결이 해제됩니다.  또한, Domain 에서 상속된 모든 제약 조건이 컬럼에서 삭제됩니다.  컬럼에 명시적으로 default 값을 지정하지 않았을 경우, Domain 에서 상속된 default 값도 삭제됩니다.
	
    SQL Domain 구문에 대한 자세한 내용은 [SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/23/sqlrf/create-domain.html#GUID-17D3A9C6-D993-4E94-BF6B-CACA56581F41)에서 확인할 수 있습니다.
    
- 간단한 예 - Email Domain - 를 통해 SQL Domain 사용법에 대해 살펴보겠습니다.  

	```sql
	-- Database 접속 - ADMIN User
	
	-- Domain 생성
	CREATE DOMAIN IF NOT EXISTS myemail_domain AS VARCHAR2(100) 
		default on null 'XXXX' || '@missingmail.com' 
		constraint email_c CHECK (regexp_like (myemail_domain, '^(\S+)\@(\S+)\.(\S+)$')) 
		display substr(myemail_domain, instr(myemail_domain, '@') + 1);
	```

	`myemail_domain` 이라는 Domain 을 생성하였습니다.   Check Constraint `email_c` 는 유효한 이메일이 저장되는 지 검사하고, `DISPLAY` 는 출력 시 Domain 이 적용된 컬럼을 반환하는 방법을 지정합니다.  지정된 컬럼에 대해 `DOMAIN_DISPLAY` 라는 SQL 함수를 사용하여 컬럼을 표시할 수 있습니다.
	
- 테이블 생성 시 앞에서 생성한 Domain 을 사용해 보겠습니다.

	```sql
	-- PERSON Table 생성
	drop table if exists PERSON;
	
	create table PERSON
    ( p_id number(5),
      p_name varchar2(50),
      p_sal number,
      p_email varchar2(100) domain myemail_domain      -- Domain 지정
    )
	annotations (display 'person_table');
	```
    
- Domain 조건(Check Constraint)에 맞는 데이터를 INSERT 합니다.

	```sql
	insert into PERSON values 
		(1,'Bold',3000,null),
		(2,'Schulte',1000, 'user-schulte@gmx.net'),
		(3,'Walter',1000,'user_walter@t_online.de'),
		(4,'Schwinn',1000, 'UserSchwinn@oracle.com'),
		(5,'King',1000, 'user-king@aol.com');

	commit;

	select * from PERSON;
	
	      P_ID P_NAME          P_SAL P_EMAIL
	---------- ---------- ---------- -------------------------
	         1 Bold             3000 XXXX@missingmail.com
	         2 Schulte          1000 user-schulte@gmx.net
	         3 Walter           1000 user_walter@t_online.de
	         4 Schwinn          1000 UserSchwinn@oracle.com
	         5 King             1000 user-king@aol.com
	```
    
- Domain 조건에 맞지않는 데이터를 INSERT 합니다.  이 경우 에러가 발생합니다. 

	```sql
	INSERT INTO person values (1,'Schulte',3000, 'user-schulte%gmx.net');
	
	*
	ERROR at line 1:
	ORA-11534: check constraint (ADMIN.SYS_C0024157) involving column P_EMAIL due
	to domain constraint ADMIN.EMAIL_C of domain ADMIN.MYEMAIL_DOMAIN violated
	```
    

## Monitor SQL Domains

- SQL Domain 사용 여부를 확인해 보겠습니다.   
  `DESCRIBE` 를 사용하여 컬럼에 적용된 Domain 과 Null Constraint 를 확인할 수 있습니다. (19c SQLPlus 에서는 Domain 정보가 표시되지 않습니다.)

	```sql
	desc PERSON
	
	Name    Null?    Type                                  
	------- -------- -----------------------------------   
	P_ID             NUMBER(5)                             
	P_NAME           VARCHAR2(50)                          
	P_SAL            NUMBER                                
	P_EMAIL NOT NULL VARCHAR2(100) DOMAIN MYEMAIL_DOMAIN   
	```

- 앞에서 잠깐 언급했듯이 Domain 속성에 대한 더 많은 정보를 얻기 위해 테이블 컬럼에 적용할 수 있는 새로운 Domain 관련 함수들이 있습니다.  예를 들어, `DOMAIN_NAME` 함수는 Domain 이름을 반환하고, `DOMAIN_DISPLAY` 함수는 Domain 에 정의된 DISPLAY 를 반환합니다.

	```sql
	select p_name, domain_name(p_email), domain_display(p_email) from person;
	
	P_NAME	   DOMAIN_NAME(P_EMAIL)   DOMAIN_DISPLAY(P_EMAIL)
	---------- ---------------------- -------------------------
	Bold	   ADMIN.MYEMAIL_DOMAIN   missingmail.com
	Schulte    ADMIN.MYEMAIL_DOMAIN   gmx.net
	Walter	   ADMIN.MYEMAIL_DOMAIN   t_online.de
	Schwinn    ADMIN.MYEMAIL_DOMAIN   oracle.com
	King	   ADMIN.MYEMAIL_DOMAIN   aol.com
	```

- 다양한 Data dictionary View 를 통해 SQL Domain 정보를 확인할 수 있습니다. - `USER_DOMAIN, USER_DOMAIN_COLS, USER_DOMAIN_CONSTRAINTS (ALL/DBA)`

	```sql
	SELECT owner, name, data_display FROM user_domains;
	
	OWNER NAME           DATA_DISPLAY                                           
	----- -------------- ------------------------------------------------------ 
	ADMIN MYEMAIL_DOMAIN substr(myemail_domain, instr(myemail_domain, '@') + 1) 
	
    
	SELECT * FROM user_domain_constraints WHERE domain_name='MYEMAIL_DOMAIN';
	
	NAME             DOMAIN_OWNER DOMAIN_NAME    CONSTRAINT_TYPE SEARCH_CONDITION                                      STATUS  DEFERRABLE     DEFERRED  VALIDATED GENERATED      BAD  RELY INVALID ORIGIN_CON_ID 
	---------------- ------------ -------------- --------------- ----------------------------------------------------- ------- -------------- --------- --------- -------------- ---- ---- ------- ------------- 
	EMAIL_C          ADMIN        MYEMAIL_DOMAIN C               regexp_like (myemail_domain, '^(\S+)\@(\S+)\.(\S+)$') ENABLED NOT DEFERRABLE IMMEDIATE VALIDATED USER NAME      null null null               50 
	SYS_DOMAIN_C0044 ADMIN        MYEMAIL_DOMAIN C               "MYEMAIL_DOMAIN" IS NOT NULL                          ENABLED NOT DEFERRABLE IMMEDIATE VALIDATED GENERATED NAME null null null               50 
	
	```

- `DBMS_METADATA` 패키지를 사용하여 DDL 문을 추출할 수도 있습니다.
	```sql
	SELECT dbms_metadata.get_ddl('SQL_DOMAIN', 'MYEMAIL_DOMAIN') ;

	DBMS_METADATA.GET_DDL('SQL_DOMAIN','MYEMAIL_DOMAIN')                                                  
	------------------------------------------------------------------------------------------------------
	 CREATE DOMAIN "ADMIN"."MYEMAIL_DOMAIN" AS VARCHAR2(100) DEFAULT ON NULL 'XXXX' || '@missingmail.com'  
	 CONSTRAINT "EMAIL_C" CHECK (regexp_like (myemail_domain, '^(\S+)\@(\S+)\.(\S+)$')) ENABLE  
	 DISPLAY substr(myemail_domain, instr(myemail_domain, '@') + 1)
	```

## Multi Column Domain

Multi Column Domain 을 사용하여 여러 컬럼에 걸친 Domain 을 생성할 수 있습니다.

- Multi Column Domain 을 생성합니다.  주소 정보에 대한 Domain 을 생성합니다.

	```sql
	drop domain if exists address_dom;	
	create domain address_dom as (
	  address_line_1  as varchar2(50),
	  address_line_2  as varchar2(50),
	  city            as varchar2(50),
	  country_code    as varchar2(5),
	  postcode        as varchar2(10)
	)
	constraint address_chk check (address_line_1 is not null and
	                              city is not null and
	                              country_code is not null and
	                              postcode is not null)
	display address_line_1||','||address_line_2||','||city||','||country_code||','||postcode;
	```

- Address Domain 을 사용하는 테이블을 생성합니다.

	```sql
	drop table if exists addresses purge;
	create table addresses (
	  id              number,
	  address_line_1  varchar2(50),
	  address_line_2  varchar2(50),
	  city            varchar2(50),
	  country_code    varchar2(5),
	  postcode        varchar2(10),  
	  domain address_dom(address_line_1, address_line_2, city, country_code, postcode)
	);	
	```

- 테이블 정보를 확인합니다.

	```sql
	desc addresses
	
	Name           Null? Type                              
	-------------- ----- -------------------------------   
	ID                   NUMBER                            
	ADDRESS_LINE_1       VARCHAR2(50) DOMAIN ADDRESS_DOM   
	ADDRESS_LINE_2       VARCHAR2(50) DOMAIN ADDRESS_DOM   
	CITY                 VARCHAR2(50) DOMAIN ADDRESS_DOM   
	COUNTRY_CODE         VARCHAR2(5) DOMAIN ADDRESS_DOM    
	POSTCODE             VARCHAR2(10) DOMAIN ADDRESS_DOM
	```

- 다음 테스트를 위해 생성한 객체들을 삭제합니다. 

	```sql
	drop domain if exists address_dom;
	drop table if exists addresses purge;
	```

## Enumeration Domain 

- Oracle SQL 로 days of the week 숫자(1,2,..,7)를 요일 정보로 표시하려면 일반적으로 다음과 같은 쿼리를 작성하여야 합니다.
	```sql
	select case day_number
				when 1 then 'Monday'
				when 2 then 'Tuesday'
				...
		end day_of_week
	from ...
	```

	마찬가지로 월 정보를 표시하기 위해서도 비슷한 형태의 SQL 문을 작성해야 합니다.

- Oracle Database 23ai 의 **Enumeration Domain** 기능을 사용하면 list of values 를 아주 쉽게 생성하고 사용할 수 있습니다.

- 다음의 SQL 문으로 Enumeration Domain 을 생성할 수 있습니다.  시작번호를 지정하면 이후의 값은 1씩 증가된 값으로 설정됩니다.

	```sql
	create domain days_of_week as enum (
		monday = 0, tuesday, wednesday, 
		thursday, friday, saturday, 
		sunday
	);
	```

- 요일에 대한 숫자값 대신 두 글자 약어(MO, TU, ..)를 값으로 설정할 수 있습니다.  마찬자기로 세글자 값도 동시에 설정하려면 다음과 같이 Enumeration Domain 을 생성합니다.

	```sql
	drop domain if exists days_of_week;
	create domain days_of_week as enum ( 
	monday    = mon = 'MO', 
	tuesday   = tue = 'TU', 
	wednesday = wed = 'WE', 
	thursday  = thu = 'TH', 
	friday    = fri = 'FR', 
	saturday  = sat = 'SA', 
	sunday    = sun = 'SU'
	);
	```

- 생성한 Eunumeration Domain 은 일반적인 테이블처럼 조회할 수 있습니다. (조회만 가능, DML 불가)

	```sql
	select * from days_of_week;

	ENUM_NAME EN
	--------- --
	MONDAY    MO
	MON       MO
	TUESDAY   TU
	TUE       TU
	WEDNESDAY WE
	WED       WE
	THURSDAY  TH
	THU       TH
	FRIDAY    FR
	FRI       FR
	SATURDAY  SA
	SAT       SA
	SUNDAY    SU
	SUN       SU
	```

- 앞에서 생성한 Domain 을 적용하여 테이블을 생성합니다.  컬럼의 데이터 타입으로 앞에서 생성한 Enumeration Domain 을 지정합니다.

	```sql
	create table class_schedules (
  		class_id      int, 
  		school_year   int,
  		scheduled_day days_of_week
	);

	desc class_schedules
	
	Name                                      Null?    Type
	----------------------------------------- -------- ----------------------------
	CLASS_ID                                           NUMBER(38)
	SCHOOL_YEAR                                        NUMBER(38)
	SCHEDULED_DAY                                      VARCHAR2(4)
	```

- Enumeration Domain 을 컬럼에 적용하면 암묵적으로 Check Constraints 가 적용됩니다.  데이터베이스는 컬럼을 Domain 과 연결할 때 해당 컬럼에 대해 제약 조건을 적용합니다.
	이를 통해, Enumeration Domain 에 지정된 값만 컬럼 값으로 저장할 수 있습니다.

	```sql
	insert into class_schedules values ( 1, 2024, 'ZZ' );
	*
	ERROR at line 1:
	ORA-11534: check constraint (ADMIN.SYS_C0024218) involving column SCHEDULED_DAY
	due to domain constraint ADMIN.SYS_DOMAIN_C0057 of domain ADMIN.DAYS_OF_WEEK
	violated


	insert into class_schedules values ( 1, 2024, 'MO' );

	1 row created.

	commit;

	select * from class_schedules;

	CLASS_ID SCHOOL_YEAR SCHE
	---------- ----------- ----
			1        2024 MO
	```

- 또한, 지정한 상수값('MO', 'TU',..) 뿐만 Enumeration Domain 명을 SQL 문에서 사용할 수 있습니다.  이렇게 하면 키와 연결된 값이 반환됩니다.

	```sql
	insert into class_schedules values ( 1, 2024, days_of_week.tue );
	insert into class_schedules values ( 1, 2024, days_of_week.wednesday );

	select * from class_schedules;

		CLASS_ID SCHOOL_YEAR SCHE
	---------- ----------- ----
			1        2024 MO
			1        2024 TU
			1        2024 WE
	```

## Flexible Domain

Flexible Domain 은 데이터의 조건에 따라 여러 Domain 중 하나를 적용할 수 있는 기능을 제공합니다.  

아래 테스트에서는 'UK', 'US', 및 기타 주소를 나타내기 위한 세 개의 Domain 을 생성합니다.  각 국의 주소 조건에 따라, 영국 주소에는 6~8 자의 postcode를 정의하고, 미국 주소에는 5 또는 9 자의 postcode를 정의합니다.  기본 주소에는 postcode 의 길이를 체크하지 않도록 정의합니다.   이러한 문자열 크기 체크는 각 Domaind 의 Check Constraints 에 추가됩니다.

- 국가 별 주소 Domain 을 생성합니다.

	```sql
	-- UK address.
	drop domain if exists address_uk_dom;
	create domain address_uk_dom as (
	  address_line_1  as varchar2(50),
	  address_line_2  as varchar2(50),
	  city            as varchar2(50),
	  postcode        as varchar2(10)
	)
	constraint address_uk_chk check (address_line_1 is not null and
	                                 city is not null and
	                                 postcode is not null and
	                                 length(postcode) between 6 and 8);
	
	
	-- US address.
	drop domain if exists address_us_dom;
	create domain address_us_dom as (
	  address_line_1  as varchar2(50),
	  address_line_2  as varchar2(50),
	  city            as varchar2(50),
	  postcode        as varchar2(10)
	)
	constraint address_us_chk check (address_line_1 is not null and
	                                 city is not null and
	                                 postcode is not null and
	                                 (length(postcode) = 5 or length(postcode) = 9));
	
	-- Default address.
	drop domain if exists address_dom;
	create domain address_dom as (
	  address_line_1  as varchar2(50),
	  address_line_2  as varchar2(50),
	  city            as varchar2(50),
	  postcode        as varchar2(10)
	)
	constraint address_chk check (address_line_1 is not null and
	                              city is not null and
	                              postcode is not null);
	```

- 데이터의 조건에 따라 Domain 을 적용하는 Flexible Domain 을 생성합니다.  조건을 결정할 컬럼을 지정한 후(Discriminant Column), DECODE 나 CASE 구문을 사용하여 적용할 Domain 을 결정할 수 있습니다.
	```sql
	drop domain if exists address_flex_dom;
	
	create flexible domain address_flex_dom (address_line_1, address_line_2, city, postcode)
	choose domain using (country_code varchar2(5))
	from case
	       when country_code in ('GB','GBR') 
		       then address_uk_dom(address_line_1, address_line_2, city, postcode)
	       when country_code in ('US','USA') 
		       then address_us_dom(address_line_1, address_line_2, city, postcode)
	       else address_dom(address_line_1, address_line_2, city, postcode)
	     end;
	```

- 위에서 생성한 Flexible Domain 을 사용하는 Address 테이블을 생성합니다.   조건 컬럼(Discriminant Column) 을 `USING` 키워드를 사용하여 지정합니다.
	```sql
	drop table if exists addresses purge;
	create table addresses (
	id              number,
	address_line_1  varchar2(50),
	address_line_2  varchar2(50),
	city            varchar2(50),
	country_code    varchar2(5),
	postcode        varchar2(10),  
	domain address_flex_dom(address_line_1, address_line_2, city, postcode) using (country_code)
	);
	```

- 테이블에 새로운 데이터를 Insert 하며 Flexible Domain 을 테스트합니다.  
	```sql
	- Domain 조건에 맞는 데이터는 정상적으로 INSERT
	insert into addresses 
	values (1, '1 my street', null, 'birmingham', 'GB', 'A12 BCD');
	
	insert into addresses 
	values (2, '2 my street', null, 'boston', 'US', '12345');
	
	insert into addresses 
	values (3, '3 my street', null, 'dublin', 'IRE', '1234567890');


	-- Domain 조건에 맞지 않는 데이터는 오류 발생
	insert into addresses
	values (4, '4 my street', null, 'birmingham', 'GB', '12345');
	*
	ERROR at line 1:
	ORA-11534: check constraint (ADMIN.SYS_C0024159) involving columns
	ADDRESS_LINE_1, CITY, COUNTRY_CODE, POSTCODE due to domain constraint
	ADMIN.SYS_DOMAIN_C0054 of domain ADMIN.ADDRESS_FLEX_DOM violated
	Help: https://docs.oracle.com/error-help/db/ora-11534/
	
	
	insert into addresses
	values (5, '5 my street', null, 'boston', 'US', 'A12 BCD');
	*
	ERROR at line 1:
	ORA-11534: check constraint (ADMIN.SYS_C0024160) involving columns
	ADDRESS_LINE_1, CITY, COUNTRY_CODE, POSTCODE due to domain constraint
	ADMIN.SYS_DOMAIN_C0053 of domain ADMIN.ADDRESS_FLEX_DOM violated
	Help: https://docs.oracle.com/error-help/db/ora-11534/
	```


## Built-in Domains

Oracle 은 Domain 기능을 사용하기 쉽게 하기 위해 내장 Domain 을 제공합니다 - 예> email, ssn, credit_card 등.  사용자는 내장 Domain 을 바로 테이블 컬럼에 사용할 수 있습니다.

- `ALL_DOMAINS` 뷰에서 Oracle 에서 제공하는 내장 Domain 정보를 확인할 수 있습니다.

	```sql
	SELECT name FROM all_domains where owner='SYS';
	
	NAME
	------------------------------
	MEAN_D
	MEDIAN_D
	MODE_D
	VARIANCE_D
	STANDARD_DEVIATION_D
	MEASURE_VALUE_D
	MEASURE_TYPE_D
	...
	...
	
	109 rows selected.
	```

- 내장 Domain 중 이메일 주소 검증을 위한 `EMAIL_D` Domain 을 살펴보겠습니다.  `DBMS_METADATA` 패키지를 사용하여 DDL 문을 확인해 보겠습니다.

	```sql
	SELECT dbms_metadata.get_ddl('SQL_DOMAIN', 'EMAIL_D','SYS') domain_ddl ;

	DOMAIN_DDL
	--------------------------------------------------------------------------------
	  CREATE DOMAIN "SYS"."EMAIL_D" AS VARCHAR2(4000) CHECK (REGEXP_LIKE (email_d
	, '^([a-zA-Z0-9!#$%&*+=?^_`{|}~-]+(\.[A-Za-z0-9!#$%&
	*+=?^_`{|}~-]+)*)@(([a-zA-Z0-9]([a-zA-Z0-9-]*[a-zA-Z
	0-9])?\.)+[a-zA-Z0-9]([a-zA-Z0-9-]*[a-zA-Z0-9])?)$')
	) ENABLE ANNOTATIONS("PERSON_INFO")
	```

- 내장 Domain `EMAIL_D` 를 사용하여 테이블을 다시 생성해 보겠습니다.

	```sql
	DROP TABLE IF EXISTS person;
	
	CREATE TABLE person
    (   p_id number(5),
        p_name varchar2(50),
        p_sal number,
        p_email varchar2(4000) domain EMAIL_D
    )
    annotations (display 'person_table');
    
	```

	- 현재 Autonomous Database 에서는 위의 테이블 생성 작업 시 `ORA-01031: insufficient privileges` 에러 발생
	- `EXECUTE ANY DOMAIN` 권한 부여 시 다른 스키마의 Domain 을 사용할 수 있다고 문서에 표시되어 있지만, 권한 부여 후에도 에러가 발생함.
	- 확인 필요

