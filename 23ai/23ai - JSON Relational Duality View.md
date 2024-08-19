# Oracle Database 23ai : JSON Relational Duality Views

- [Oracle Database 23ai : JSON Relational Duality Views](#oracle-database-23ai--json-relational-duality-views)
  - [Introduction](#introduction)
  - [JSON-Relational Duality 기능 테스트](#json-relational-duality-기능-테스트)
    - [Setup](#setup)
    - [JSON-Relational Duality View 생성](#json-relational-duality-view-생성)
    - [DML 작업 (INSERT, UPDATE, DELETE)](#dml-작업-insert-update-delete)
  - [Car-Racing Duality Views Example](#car-racing-duality-views-example)
    - [Car-Racing Example - Tables](#car-racing-example---tables)
    - [Car-Racing Example - Duality Views](#car-racing-example---duality-views)
    - [Populating the Database](#populating-the-database)
  - [Working with JSON and the Duality Views](#working-with-json-and-the-duality-views)
  - [The Extreme Flexibility of JSON Duality Views](#the-extreme-flexibility-of-json-duality-views)

## Introduction

JSON-Relational Duality 기능은 관계형 데이터 또는 Document Model을 사용하여 애플리케이션을 구축할 때 개발자들이 만나는 문제들을 극복할 수 있게 해주는 Oracle Database 23ai 의 새로운 기능입니다.

이 기능은 Relational 데이터와 JSON Document 데이터를 통합하여 처리할 수 있는 방법을 제공합니다.  애플리케이션에서는 데이터 중복없이 JSON Document와 Relational 테이블이 동일한 데이터에 액세스할 수 있습니다.

관계형/SQL 개발자, 문서 데이터베이스 개발자, Java 또는 Python 앱 개발자라면 Duality View를 사용하여 앱을 손쉽게 구축할 수 있습니다.

JSON-Relational Duality 기능은 Document와 Relational 환경의 이점을 통합하는 데 도움이 됩니다.  개발자는 JSON Document Model의 유연성과 데이터 액세스 이점은 물론 관계형 모델의 저장 효율성과 다양한 기능들도 누릴 수 있습니다.

**JSON-Relation Duality의 주요 이점**

- Duality View를 사용하여 앱을 구축할 때 극도의 유연성을 제공합니다. 개발자는 사용 사례에 따라 관계형 또는 계층적 문서로 동일한 데이터에 액세스할 수 있습니다. 관계형 데이터를 활용하여 Document-centric 앱을 구축하거나 Document 데이터에 SQL 앱을 손쉽게 구축할 수 있습니다.

- 단일 데이터베이스 작업으로 앱에 필요한 모든 데이터를 손쉽게 검색하고 저장할 수 있습니다. Duality View는 데이터에 대해 업데이트 가능한 JSON View를 제공합니다. 앱은 기본 데이터 구조, 매핑, 일관성 또는 성능 조정에 대한 걱정 없이 문서를 읽고, 필요한 사항을 변경하고, 문서를 다시 작성할 수 있습니다.

- 동일한 데이터로 여러 앱을 구축할 때 유연성과 단순성을 지원합니다. 개발자는 Duality View 기능을 사용하여 겹치는 테이블 그룹에 걸쳐 여러 JSON View를 정의할 수 있습니다. 이러한 유연한 데이터 모델링을 통해 동일한 데이터에 대해 여러 앱을 쉽고 효율적으로 구축할 수 있습니다.

- Duality View는 문서 데이터베이스의 데이터 중복 및 데이터 불일치와 같은 전형적인 문제를 제거합니다. Duality View는 여러 문서와 테이블에 걸쳐 완전한 ACID(원자성, 일관성, 격리, 내구성) 트랜잭션을 제공합니다. 문서 데이터 간의 데이터 중복을 제거하는 동시에 일관성은 자동으로 유지됩니다.

- 높은 동시성 액세스 및 업데이트를 지원하는 앱을 구축할 수 있습니다. 기존 locking 기능은 최신 앱에서는 잘 작동하지 않습니다. Duality View와 함께 제공되는 새로운 lock-free 동시성 제어는 높은 동시성 업데이트를 지원합니다.

Oracle Database 23ai 의 JSON-Relational Duality Views 기능에 대한 상세한 자료는 다음의 문서를 참조하시기 바랍니다.  
[JSON-Relational Duality Developer's Guide](https://docs.oracle.com/en/database/oracle/oracle-database/23/jsnvu/)
  
  
## JSON-Relational Duality 기능 테스트

JSON-Relation Duality Views 기능은 Relational 데이터(테이블등)을 JSON Document 형태로 보여줍니다.  이를 통해 쿼리나 DML 작업을 기존 SQL 이나 JSON 을 사용하여 수행할 수 있습니다.

### Setup   

테스트를 위해 다음의 테이블을 생성합니다.

```sql
drop table if exists emp purge;
drop table if exists dept purge;

create table dept (
	deptno   number(2) constraint pk_dept primary key,
	dname    varchar2(14),
	loc      varchar2(13)
) ;

create table emp (
	empno    number(4) constraint pk_emp primary key,
	ename    varchar2(10),
	job      varchar2(9),
	mgr      number(4),
	hiredate date,
	sal      number(7,2),
	comm     number(7,2),
	deptno   number(2) constraint fk_deptno references dept
);

create index emp_dept_fk_i on emp(deptno);

insert into dept values (10,'ACCOUNTING','NEW YORK');
insert into dept values (20,'RESEARCH','DALLAS');
insert into dept values (30,'SALES','CHICAGO');
insert into dept values (40,'OPERATIONS','BOSTON');

insert into emp values (7369,'SMITH','CLERK',7902,to_date('17-12-1980','dd-mm-yyyy'),800,null,20);
insert into emp values (7499,'ALLEN','SALESMAN',7698,to_date('20-2-1981','dd-mm-yyyy'),1600,300,30);
insert into emp values (7521,'WARD','SALESMAN',7698,to_date('22-2-1981','dd-mm-yyyy'),1250,500,30);
insert into emp values (7566,'JONES','MANAGER',7839,to_date('2-4-1981','dd-mm-yyyy'),2975,null,20);
insert into emp values (7654,'MARTIN','SALESMAN',7698,to_date('28-9-1981','dd-mm-yyyy'),1250,1400,30);
insert into emp values (7698,'BLAKE','MANAGER',7839,to_date('1-5-1981','dd-mm-yyyy'),2850,null,30);
insert into emp values (7782,'CLARK','MANAGER',7839,to_date('9-6-1981','dd-mm-yyyy'),2450,null,10);
insert into emp values (7788,'SCOTT','ANALYST',7566,to_date('13-JUL-87','dd-mm-rr')-85,3000,null,20);
insert into emp values (7839,'KING','PRESIDENT',null,to_date('17-11-1981','dd-mm-yyyy'),5000,null,10);
insert into emp values (7844,'TURNER','SALESMAN',7698,to_date('8-9-1981','dd-mm-yyyy'),1500,0,30);
insert into emp values (7876,'ADAMS','CLERK',7788,to_date('13-JUL-87', 'dd-mm-rr')-51,1100,null,20);
insert into emp values (7900,'JAMES','CLERK',7698,to_date('3-12-1981','dd-mm-yyyy'),950,null,30);
insert into emp values (7902,'FORD','ANALYST',7566,to_date('3-12-1981','dd-mm-yyyy'),3000,null,20);
insert into emp values (7934,'MILLER','CLERK',7782,to_date('23-1-1982','dd-mm-yyyy'),1300,null,10);
commit;
```

### JSON-Relational Duality View 생성

테이블로 부터 문서 구조를 정의하여 JSON-Relational Duality View를 생성합니다.

다음 예에서는 부서 내의 모든 직원 정보를 포함하는 `DEPARTMENT_DV` 라는 View 를 생성합니다. View 에서 참조하는 기본 테이블들에 대해 가능한 DML 작업을 지정할 수 있습니다.  이 예에서는 `WITH INSERT UPDATE DELETE` 로 설정하였습니다.

Duality Views 로 생성하는 JSON 문서는 항상 최상위(Root) 레벨의 **document-identifier field** 인 `_id` 를 포함하여야 합니다.

```sql
create or replace json relational duality view department_dv as
	select json {
			'_id' : d.deptno,
			'departmentName' : d.dname,
		    'location' : d.loc,
		    'employees' :
				[ select json {
							'employeeNumber' : e.empno,
					        'employeeName' : e.ename,
							'job' : e.job,
							'salary' : e.sal 
						 }
				  from emp e with insert update delete
				  where d.deptno = e.deptno 
				] 
			}
	from dept d with insert update delete;
```

생성된 View 에는 데이터 타입이 JSON인 DATA 라는 이름의 컬럼이 생성됩니다. 

```sql
desc department_dv

Name Null? Type   
---- ----- ----   
DATA       JSON
```

생성된 Duality View 쿼리하면 참조된 테이블의 내용을 기반으로 시스템에서 생성된 여러 식별자와 JSON 데이터를 볼 수 있습니다.

```sql
select * from department_dv;

DATA                                                 
------------------------------------------------------------------------------------------
{"_id":10,"departmentName":"ACCOUNTING","location":"NEW YORK","employees":[{"employeeNumber":7782,"employeeName":"CLARK","job":"MANAGER","salary":2450},{"employeeNumber":7839,"employeeName":"KING","job":"PRESIDENT","salary":5000},{"employeeNumber":7934,"employeeName":"MILLER","job":"CLERK","salary":1300}],"_metadata":{"etag":"E546E2220E8F9620E36C2A7F8858D6F7","asof":"000000001F1C4FD4"}}
                                                                                                                                                              
{"_id":20,"departmentName":"RESEARCH","location":"DALLAS","employees":[{"employeeNumber":7369,"employeeName":"SMITH","job":"CLERK","salary":800},{"employeeNumber":7566,"employeeName":"JONES","job":"MANAGER","salary":2975},{"employeeNumber":7788,"employeeName":"SCOTT","job":"ANALYST","salary":3000},{"employeeNumber":7876,"employeeName":"ADAMS","job":"CLERK","salary":1100},{"employeeNumber":7902,"employeeName":"FORD","job":"ANALYST","salary":3000}],"_metadata":{"etag":"8DAFACC22EC949A2C54B9F7BBE79B171","asof":"000000001F1C4FD4"}}
                                                                                     
{"_id":30,"departmentName":"SALES","location":"CHICAGO","employees":[{"employeeNumber":7499,"employeeName":"ALLEN","job":"SALESMAN","salary":1600},{"employeeNumber":7521,"employeeName":"WARD","job":"SALESMAN","salary":1250},{"employeeNumber":7654,"employeeName":"MARTIN","job":"SALESMAN","salary":1250},{"employeeNumber":7698,"employeeName":"BLAKE","job":"MANAGER","salary":2850},{"employeeNumber":7844,"employeeName":"TURNER","job":"SALESMAN","salary":1500},{"employeeNumber":7900,"employeeName":"JAMES","job":"CLERK","salary":950}],"_metadata":{"etag":"72D95F921FBC3FFC59C269B80EFBA5CF","asof":"000000001F1C4FD4"}}   

{"_id":40,"departmentName":"OPERATIONS","location":"BOSTON","employees":[],"_metadata":{"etag":"6FAB9798FF405D87F0EB44456398A5D5","asof":"000000001F1C4FD4"}}  
```

JSON_SERIALIZE 함수를 사용하여 JSON 데이터를 사용자가 보기 좋은 형태로 출력할 수 있습니다.

```sql

select json_serialize(d.data pretty) from department_dv d;

JSON_SERIALIZE(D.DATAPRETTY)                    
---------------------------------------------------------------------------------------------------------------
{  
  "_id" : 10,  
  "_metadata" :  
  {  
    "etag" : "E546E2220E8F9620E36C2A7F8858D6F7",  
    "asof" : "000000001F1CB62B"  
  },  
  "departmentName" : "ACCOUNTING",  
  "location" : "NEW YORK",  
  "employees" :  
  [  
    {  
      "employeeNumber" : 7782,  
      "employeeName" : "CLARK",  
      "job" : "MANAGER",  
      "salary" : 2450  
    },  
    {  
      "employeeNumber" : 7839,  
      "employeeName" : "KING",  
      "job" : "PRESIDENT",  
      "salary" : 5000  
    },  
    {  
      "employeeNumber" : 7934,  
      "employeeName" : "MILLER",  
      "job" : "CLERK",  
      "salary" : 1300  
    }  
  ]  
}                                                                                                                                            
{  
  "_id" : 20,  
  "_metadata" :  
  {  
    "etag" : "8DAFACC22EC949A2C54B9F7BBE79B171",  
    "asof" : "000000001F1CB62B"  
  },  
  "departmentName" : "RESEARCH",  
  "location" : "DALLAS",  
  "employees" :  
  [  
    {  
      "employeeNumber" : 7369,  
      "employeeName" : "SMITH",  
      "job" : "CLERK",  
      "salary" : 800  
    },  
    {  
      "employeeNumber" : 7566,  
      "employeeName" : "JONES",  
      "job" : "MANAGER",  
      "salary" : 2975  
    },  
    {  
      "employeeNumber" : 7788,  
      "employeeName" : "SCOTT",  
      "job" : "ANALYST",  
      "salary" : 3000  
    },  
    {  
      "employeeNumber" : 7876,  
      "employeeName" : "ADAMS",  
      "job" : "CLERK",  
      "salary" : 1100  
    },  
    {  
      "employeeNumber" : 7902,  
      "employeeName" : "FORD",  
      "job" : "ANALYST",  
      "salary" : 3000  
    }  
  ]  
}                                                                                                                 

...
...
```

  

조회된 데이터를 보면, 각 행에 하나의 부서와 해당 부서의 모든 직원 정보가 포함되어 있습니다. 

`_etag` 속성은 레코드 컨텐츠로부터 생성된 해시 정보로, optimistic locking 을 지원하는 데 사용할 수 있으며 JSON 문서가 생성된 이후 레코드 내용이 변경되지 않았는지 확인하는 데 사용할 수 있습니다. 이 정보는 문서의 버전으로 생각할 수 있습니다.

dot notation을 사용하여 SQL 로 데이터를 조회할 수도 있습니다.

```sql
select d.data.departmentName,
	   d.data.location
from department_dv d
where d.data."_id" = 40;


DEPARTMENTNAME LOCATION   
-------------- --------   
OPERATIONS     BOSTON       
```

### DML 작업 (INSERT, UPDATE, DELETE)

**INSERT**

부서와 직원을 정의하는 JSON 문서를 사용하여 Duality View를 통해 새로운 부서정보를 생성할 수 있습니다.

```sql
-- 새로운 부서 INSERT : _id (departmentNumber) = 50

insert into department_dv d (data)
values (
	'{ "_id" : 50,
	   "departmentName" : "DBA",
	   "location" : "BIRMINGHAM",
	   "employees" : [ { 
			"employeeNumber" : 9999,
			"employeeName" : "HALL",
			"job" : "CLERK",
			"salary" : 500
		} ]
    }'
);

  
-- INSERT 한 정보 조회 - DEPARTMENT_DV 
select json_serialize(d.data pretty)
from department_dv d
where d."_id" = 50;

JSON_SERIALIZE(D.DATAPRETTY)
----------------------------------------------------------------------------------------------   
{  
  "_id" : 50,  
  "_metadata" :  
  {  
    "etag" : "77052B06E84B60749E410D5C2BA797DF",  
    "asof" : "000000001F1FB366"  
  },  
  "departmentName" : "DBA",  
  "location" : "BIRMINGHAM",  
  "employees" :  
  [  
    {  
      "employeeNumber" : 9999,  
      "employeeName" : "HALL",  
      "job" : "CLERK",  
      "salary" : 500  
    }  
  ]  
}
  

-- INSERT 된 데이터를 테이블에서 조회 - DEPT / EMP
select * from dept where deptno = 50;

DEPTNO DNAME LOC          
------ ----- ----------   
    50 DBA   BIRMINGHAM


select * from emp where deptno = 50;

EMPNO ENAME JOB    MGR HIREDATE     SAL COMM DEPTNO   
----- ----- ----- ---- ------------ --- ---- ------   
 9999 HALL  CLERK null null         500 null     50  
```


**UPDATE**

다음 예에서는 부서 40을 업데이트하여 해당 부서에 새 직원을 추가합니다.

```sql
-- UPDATE
update department_dv d
set d.data = (
'{
	"_id" : 40,
	"departmentName" : "OPERATIONS",
	"location" : "BOSTON",
	"employees" : [ {
		"employeeNumber" : 9998,
		"employeeName" : "HELLO",
		"job" : "CLERK",
		"salary" : 500
	} ]
}'
)
where d.data."_id" = 40;

  

-- 새롭게 추가된 직원 정보 조회
select json_serialize(d.data pretty)
from department_dv d
where d.data."_id" = 40;

  
JSON_SERIALIZE(D.DATAPRETTY)
--------------------------------------------------------------------------------------------------   
{  
  "_id" : 40,  
  "_metadata" :  
  {  
    "etag" : "20922F89E067A817FE430BCD79246271",  
    "asof" : "000000001F2691F8"  
  },  
  "departmentName" : "OPERATIONS",  
  "location" : "BOSTON",  
  "employees" :  
  [  
    {  
      "employeeNumber" : 9998,  
      "employeeName" : "HELLO",  
      "job" : "CLERK",  
      "salary" : 500  
    }  
  ]  
}
  

-- EMP 테이블에서 조회
select * from emp where empno = 9998;

EMPNO ENAME JOB    MGR HIREDATE     SAL COMM DEPTNO   
----- ----- ----- ---- ------------ --- ---- ------   
 9998 HELLO CLERK null Invalid Date 500 null     40

```

  
`JSON_TRANSFORM` 을 사용하여 테이블을 UPDATE 할 수도 있습니다. 마치 테이블을 JSON 테이블처럼 처리할 수 있습니다.

```sql

update department_dv d
set d.data = json_transform(d.data, set '$.location' = 'BOSTON2')
where d.data."_id" = 40;


select * from dept where deptno = 40;

DEPTNO DNAME      LOC       
------ ---------- -------   
    40 OPERATIONS BOSTON2

```

  

**DELETE**

부서 번호 50번 부서를 삭제합니다.

```sql
delete from department_dv d
where d.data."_id" = 50;


select json_serialize(d.data pretty)
from department_dv d
where d.data."_id" = 50;

no rows selected
```

  
## Car-Racing Duality Views Example

이 테스트 스크립트는 SQL을 통해 JSON Relational Duality View를 사용하는 예를 보여줍니다.

### Car-Racing Example - Tables
  
**테이블 성성**

```sql
DROP VIEW IF EXISTS team_dv;
DROP VIEW IF EXISTS race_dv;
DROP VIEW IF EXISTS driver_dv;
DROP TABLE IF EXISTS driver_race_map;
DROP TABLE IF EXISTS race;
DROP TABLE IF EXISTS driver;
DROP TABLE IF EXISTS team;

CREATE TABLE IF NOT EXISTS team (
	team_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
	name VARCHAR2(255) NOT NULL UNIQUE,
	points INTEGER NOT NULL,
	CONSTRAINT team_pk PRIMARY KEY(team_id)
);

CREATE TABLE IF NOT EXISTS driver (
	driver_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
	name VARCHAR2(255) NOT NULL UNIQUE,
	points INTEGER NOT NULL,
	team_id INTEGER,
	CONSTRAINT driver_pk PRIMARY KEY(driver_id),
	CONSTRAINT driver_fk FOREIGN KEY(team_id) REFERENCES team(team_id)
);

CREATE TABLE IF NOT EXISTS race (
	race_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
	name VARCHAR2(255) NOT NULL UNIQUE,
	laps INTEGER NOT NULL,
	race_date DATE,
	podium JSON,
	CONSTRAINT race_pk PRIMARY KEY(race_id)
);

CREATE TABLE IF NOT EXISTS driver_race_map (
	driver_race_map_id INTEGER GENERATED BY DEFAULT ON NULL AS IDENTITY,
	race_id INTEGER NOT NULL,
	driver_id INTEGER NOT NULL,
	position INTEGER,
	CONSTRAINT driver_race_map_pk PRIMARY KEY(driver_race_map_id),
	CONSTRAINT driver_race_map_fk1 FOREIGN KEY(race_id) REFERENCES race(race_id),
	CONSTRAINT driver_race_map_fk2 FOREIGN KEY(driver_id) REFERENCES driver(driver_id)
);

-- Create foreign-key indexes 
CREATE INDEX driver_fk_idx ON driver (team_id); 
CREATE INDEX driver_race_map_fk1_idx ON driver_race_map (race_id); 
CREATE INDEX driver_race_map_fk2_idx ON driver_race_map (driver_id);
```

<!--
**`DRIVER_RACE_MAP` 테이블에 대한 trigger 생성**

경기 결과에 따라 해당 팀과 드라이버의 포인터 정보를 변경하는 trigger 를 생성합니다.

```sql
CREATE OR REPLACE TRIGGER driver_race_map_trigger
BEFORE INSERT ON driver_race_map
FOR EACH ROW
DECLARE
	v_points INTEGER;
	v_team_id INTEGER;
BEGIN
	SELECT team_id INTO v_team_id FROM driver WHERE driver_id = :NEW.driver_id;

	IF :NEW.position = 1 THEN	
		v_points := 25;	
	ELSIF :NEW.position = 2 THEN	
		v_points := 18;	
	ELSIF :NEW.position = 3 THEN	
		v_points := 15;	
	ELSIF :NEW.position = 4 THEN	
		v_points := 12;	
	ELSIF :NEW.position = 5 THEN	
		v_points := 10;	
	ELSIF :NEW.position = 6 THEN	
		v_points := 8;	
	ELSIF :NEW.position = 7 THEN	
		v_points := 6;	
	ELSIF :NEW.position = 8 THEN	
		v_points := 4;	
	ELSIF :NEW.position = 9 THEN	
		v_points := 2;	
	ELSIF :NEW.position = 10 THEN	
		v_points := 1;	
	ELSE	
		v_points := 0;	
	END IF;	
  
	UPDATE driver SET points = points + v_points
	WHERE driver_id = :NEW.driver_id;

	UPDATE team SET points = points + v_points
	WHERE team_id = v_team_id;

END;
/

```
-->

### Car-Racing Example - Duality Views

**`RACE_DV` Duality View 생성**

이제 `RACE_DV` Duality View 를 생성합니다.  `DRIVER` 테이블에 대해서는 업데이트를 허용하고 `RACE` 테이블과 `DRIVER_RACE_MAP` 테이블에 대해서는 INSERT, UPDATE, DELETE 모두 허용하도록 설정합니다.

```sql
CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW race_dv AS
SELECT JSON {
            '_id' IS r.race_id,
            'name'   IS r.name,
            'laps'   IS r.laps WITH NOUPDATE,
            'date'   IS r.race_date,
            'podium' IS r.podium WITH NOCHECK,
            'result' IS
                [ SELECT JSON {'driverRaceMapId' IS drm.driver_race_map_id,
                                'position'        IS drm.position,
                                UNNEST
                                (SELECT JSON {'driverId' IS d.driver_id,
                                                'name'     IS d.name}
                                    FROM driver d WITH NOINSERT UPDATE NODELETE
                                    WHERE d.driver_id = drm.driver_id)}
                    FROM driver_race_map drm WITH INSERT UPDATE DELETE
                    WHERE drm.race_id = r.race_id ]}
    FROM race r WITH INSERT UPDATE DELETE;
```

**`DRIVER_DV` Duality View 생성**

`DRIVER_DV` 는 Driver를 위한 View 이므로, TEAM 테이블이나 RACE 테이블에 대해서는 `noinsert, noupdate, nodelete` 옵션을 지정하여 DML 을 허용하지 않습니다.  또한, DRIVER_RACE_MAP 테이블에 대해서는 INSERT, UPDATE 만 허용하고 DELETE는 허용하지 않도록 지정합니다.

```sql
CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW driver_dv AS
SELECT JSON {'_id' IS d.driver_id,
        'name'     IS d.name,
        'points'   IS d.points,
        UNNEST
            (SELECT JSON {'teamId' IS t.team_id,
                        'team'   IS t.name WITH NOCHECK}
                FROM team t WITH NOINSERT NOUPDATE NODELETE
                WHERE t.team_id = d.team_id),
        'race'     IS
            [ SELECT JSON {'driverRaceMapId' IS drm.driver_race_map_id,
                            UNNEST
                            (SELECT JSON {'raceId' IS r.race_id,
                                            'name'   IS r.name}
                                FROM race r WITH NOINSERT NOUPDATE NODELETE
                                WHERE r.race_id = drm.race_id),
                            'finalPosition'   IS drm.position}
                FROM driver_race_map drm WITH INSERT UPDATE NODELETE
                WHERE drm.driver_id = d.driver_id ]}
FROM driver d WITH INSERT UPDATE DELETE;
```

**`TEAM_DV` Duality View 생성**

TEAM 정보를 생성하거나 수정할 때 DRIVER 정보를 INSERT, UPDATE 가능하도록 설정합니다.  

```sql
CREATE OR REPLACE JSON RELATIONAL DUALITY VIEW team_dv AS
SELECT JSON {'_id'  IS t.team_id,
            'name'    IS t.name,
            'points'  IS t.points,
            'driver'  IS
                [ SELECT JSON {'driverId' IS d.driver_id,
                                'name'     IS d.name,
                                'points'   IS d.points WITH NOCHECK}
                    FROM driver d WITH INSERT UPDATE
                    WHERE d.team_id = t.team_id ]}
    FROM team t WITH INSERT UPDATE DELETE;
```

### Populating the Database

**`TEAM_DV` - Document 입력**

TEAM_DV Duality  View 에 document 를 입력하면 TEAM 테이블과 DRIVER 테이블에도 데이터가 자동으로 반영됩니다. 

```sql
INSERT INTO team_dv VALUES ('{"_id" : 301,
                        "name"   : "Red Bull",
                        "points" : 0,
                        "driver" : [ {"driverId" : 101,
                                        "name"     : "Max Verstappen",
                                        "points"   : 0},
                                    {"driverId" : 102,
                                        "name"     : "Sergio Perez",
                                        "points"   : 0} ]}');

INSERT INTO team_dv VALUES ('{"_id" : 302,
                            "name"   : "Ferrari",
                            "points" : 0,
                            "driver" : [ {"driverId" : 103,
                                            "name"     : "Charles Leclerc",
                                            "points"   : 0},
                                        {"driverId" : 104,
                                            "name"     : "Carlos Sainz Jr",
                                            "points"   : 0} ]}');

INSERT INTO team_dv VALUES ('{"_id" : 2,
                            "name"   : "Mercedes",
                            "points" : 0,
                            "driver" : [ {"driverId" : 105,
                                            "name"     : "George Russell",
                                            "points"   : 0},
                                        {"driverId" : 106,
                                            "name"     : "Lewis Hamilton",
                                            "points"   : 0} ]}');
COMMIT;
```

INSERT 완료 후 테이블 정보를 확인해 봅니다.

```sql
select * from team;

TEAM_ID NAME     POINTS   
------- -------- ------   
    301 Red Bull      0   
    302 Ferrari       0   
      2 Mercedes      0   


select * from driver;

DRIVER_ID NAME            POINTS TEAM_ID   
--------- --------------- ------ -------   
      101 Max Verstappen       0     301   
      102 Sergio Perez         0     301   
      103 Charles Leclerc      0     302   
      104 Carlos Sainz Jr      0     302   
      105 George Russell       0       2   
      106 Lewis Hamilton       0       2
```

**`RACE_DV` - Document 입력**

```sql
INSERT INTO race_dv VALUES ('{"_id" : 201,
                            "name"   : "Bahrain Grand Prix",
                            "laps"   : 57,
                            "date"   : "2022-03-20T00:00:00",
                            "podium" : {}}');

INSERT INTO race_dv VALUES ('{"_id" : 202,
                            "name"   : "Saudi Arabian Grand Prix",
                            "laps"   : 50,
                            "date"   : "2022-03-27T00:00:00",
                            "podium" : {}}');

INSERT INTO race_dv VALUES ('{"_id" : 203,
                            "name"   : "Australian Grand Prix",
                            "laps"   : 58,
                            "date"   : "2022-04-09T00:00:00",
                            "podium" : {}}');
COMMIT;
```

**`DRIVER_DV` 및 `RACE_DV` 조회**

```sql
SELECT json_serialize(data PRETTY) FROM driver_dv;

JSON_SERIALIZE(DATAPRETTY)                        
-------------------------------------------------------------------------------------------   
{  
  "_id" : 201,  
  "_metadata" :  
  {  
    "etag" : "2E8DC09543DD25DC7D588FB9734D962B",  
    "asof" : "000000001F40C8AC"  
  },  
  "name" : "Bahrain Grand Prix",  
  "laps" : 57,  
  "date" : "2022-03-20T00:00:00",  
  "podium" :  
  {  
  },  
  "result" :  
  [  
  ]  
}         
{  
  "_id" : 202,  
  "_metadata" :  
  {  
    "etag" : "7E056A845212BFDE19E0C0D0CD549EA0",  
    "asof" : "000000001F40C8AC"  
  },  
  "name" : "Saudi Arabian Grand Prix",  
  "laps" : 50,  
  "date" : "2022-03-27T00:00:00",  
  "podium" :  
  {  
  },  
  "result" :  
  [  
  ]  
}   
{  
  "_id" : 203,  
  "_metadata" :  
  {  
    "etag" : "EA6E1194C012970CA07116EE1EF167E8",  
    "asof" : "000000001F40C8AC"  
  },  
  "name" : "Australian Grand Prix",  
  "laps" : 58,  
  "date" : "2022-04-09T00:00:00",  
  "podium" :  
  {  
  },  
  "result" :  
  [  
  ]  
}      
  

SELECT json_serialize(data PRETTY) FROM race_dv;

JSON_SERIALIZE(DATAPRETTY)                          
-----------------------------------------------------------------------------------------   
{  
  "_id" : 201,  
  "_metadata" :  
  {  
    "etag" : "2E8DC09543DD25DC7D588FB9734D962B",  
    "asof" : "000000001F40CC48"  
  },  
  "name" : "Bahrain Grand Prix",  
  "laps" : 57,  
  "date" : "2022-03-20T00:00:00",  
  "podium" :  
  {  
  },  
  "result" :  
  [  
  ]  
}         
{  
  "_id" : 202,  
  "_metadata" :  
  {  
    "etag" : "7E056A845212BFDE19E0C0D0CD549EA0",  
    "asof" : "000000001F40CC48"  
  },  
  "name" : "Saudi Arabian Grand Prix",  
  "laps" : 50,  
  "date" : "2022-03-27T00:00:00",  
  "podium" :  
  {  
  },  
  "result" :  
  [  
  ]  
}   
{  
  "_id" : 203,  
  "_metadata" :  
  {  
    "etag" : "EA6E1194C012970CA07116EE1EF167E8",  
    "asof" : "000000001F40CC48"  
  },  
  "name" : "Australian Grand Prix",  
  "laps" : 58,  
  "date" : "2022-04-09T00:00:00",  
  "podium" :  
  {  
  },  
  "result" :  
  [  
  ]  
}
```


## Working with JSON and the Duality Views

**Find documents matching a filter (aka predicate)**

RACE 정보를 찾으려면 Duality View를 쿼리할 때 predicates 조건에 `json_value` 및` json_exists`와 같은 JSON 함수를 사용할 수 있습니다.  
혹은 predicates 에 단순화된 점 표기법을 사용할 수도 있습니다. 

```sql
SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$._id') = 201;

JSON_SERIALIZE(DATAPRETTY)                                                                      
------------------------------------------------------------------------ 
{
  "_id" : 201,
  "_metadata" :
  {
    "etag" : "2E8DC09543DD25DC7D588FB9734D962B",
    "asof" : "000000001F42183A"
  },
  "name" : "Bahrain Grand Prix",
  "laps" : 57,
  "date" : "2022-03-20T00:00:00",
  "podium" :
  {
  },
  "result" :
  [
  ]
} 


SELECT json_serialize(data PRETTY)
FROM race_dv r WHERE r.data."_id" = 201;

JSON_SERIALIZE(DATAPRETTY)                                                          
--------------------------------------------------------------------------------------------- 
{
  "_id" : 201,
  "_metadata" :
  {
    "etag" : "2E8DC09543DD25DC7D588FB9734D962B",
    "asof" : "000000001F426DD8"
  },
  "name" : "Bahrain Grand Prix",
  "laps" : 57,
  "date" : "2022-03-20T00:00:00",
  "podium" :
  {
  },
  "result" :
  [
  ]
} 
```


**Replace and fetch a document by ID**

RACE 완료 후 RACE 의 세부 정보 및 결과를 변경합니다.

"etag" 값은 동시 작업에서 발생할 수 있는 잘 알려진 "lost update" 문제를 방지하기 위해 "out-of-the-box" optimistic locking에 사용됩니다. 지정한 ID 의 다큐먼트를 Replace 하는 중에 데이터베이스는 교체 문서에 제공된 eTag가 대상 Duality View 문서의 최신 eTag와 일치하는지 확인합니다.

다른 동시 작업이 동일한 문서를 업데이트하는 경우 eTags 가 일치하지 않는 경우가 발생할 수 있으며, 이때는 오류가 발생합니다. 이러한 오류가 발생하는 경우 업데이트된 값(업데이트된 eTag 포함)을 다시 읽고 교체 작업을 다시 시도하여 업데이트된 값에 따라 조정(원하는 경우)할 수 있습니다.

```sql
UPDATE race_dv
SET data = ('{"_metadata": {"etag" : "2E8DC09543DD25DC7D588FB9734D962B"},
                "_id" : 201,
                "name"   : "Bahrain Grand Prix",
                "laps"   : 57,
                "date"   : "2022-03-20T00:00:00",
                "podium" :
                {"winner"         : {"name" : "Charles Leclerc",
                                    "time" : "01:37:33.584"},
                "firstRunnerUp"  : {"name" : "Carlos Sainz Jr",
                                    "time" : "01:37:39.182"},
                "secondRunnerUp" : {"name" : "Lewis Hamilton",
                                    "time" : "01:37:43.259"}},
                "result" : [ {"driverRaceMapId" : 3,
                            "position"        : 1,
                            "driverId"        : 103,
                            "name"            : "Charles Leclerc"},
                            {"driverRaceMapId" : 4,
                            "position"        : 2,
                            "driverId"        : 104,
                            "name"            : "Carlos Sainz Jr"},
                            {"driverRaceMapId" : 9,
                            "position"        : 3,
                            "driverId"        : 106,
                            "name"            : "Lewis Hamilton"},
                            {"driverRaceMapId" : 10,
                            "position"        : 4,
                            "driverId"        : 105,
                            "name"            : "George Russell"} ]}')
WHERE json_value(data, '$._id') = 201;

COMMIT;
```

RACE_DV 의 변경된 결과를 확인합니다.

```sql
SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$._id') = 201;

JSON_SERIALIZE(DATAPRETTY)                                                                      
------------------------------------------------------------------------------------------------------ 
{
  "_id" : 201,
  "_metadata" :
  {
    "etag" : "20F7D9F0C69AC5F959DCA819F9116848",
    "asof" : "000000001F451330"
  },
  "name" : "Bahrain Grand Prix",
  "laps" : 57,
  "date" : "2022-03-20T00:00:00",
  "podium" :
  {
    "winner" :
    {
      "name" : "Charles Leclerc",
      "time" : "01:37:33.584"
    },
    "firstRunnerUp" :
    {
      "name" : "Carlos Sainz Jr",
      "time" : "01:37:39.182"
    },
    "secondRunnerUp" :
    {
      "name" : "Lewis Hamilton",
      "time" : "01:37:43.259"
    }
  },
  "result" :
  [
    {
      "driverRaceMapId" : 3,
      "position" : 1,
      "driverId" : 103,
      "name" : "Charles Leclerc"
    },
    {
      "driverRaceMapId" : 4,
      "position" : 2,
      "driverId" : 104,
      "name" : "Carlos Sainz Jr"
    },
    {
      "driverRaceMapId" : 9,
      "position" : 3,
      "driverId" : 106,
      "name" : "Lewis Hamilton"
    },
    {
      "driverRaceMapId" : 10,
      "position" : 4,
      "driverId" : 105,
      "name" : "George Russell"
    }
  ]
} 
```

**Update specific fields in the document identified by a predicate**

`JSON_TRANSFORM` 을 사용하여 JSON 다큐먼트의 특정 필드를 변경합니다.

```sql
UPDATE race_dv dv
SET data = json_transform(data, SET '$.name' = 'Blue Air Bahrain Grand Prix')
WHERE dv.data.name LIKE 'Bahrain%';

COMMIT;
```

변경된 정보를 확인합니다.

```sql
SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$.name') LIKE 'Blue Air Bahrain%';

JSON_SERIALIZE(DATAPRETTY)                                                            
------------------------------------------------------------------------------------------ 
{
  "_id" : 201,
  "_metadata" :
  {
    "etag" : "F6906A8F7A131C127FAEF32CA43AF97A",
    "asof" : "000000001F45CB49"
  },
  "name" : "Blue Air Bahrain Grand Prix",
  "laps" : 57,
  "date" : "2022-03-20T00:00:00",
  "podium" :
  {
    "winner" :
    {
      "name" : "Charles Leclerc",
      "time" : "01:37:33.584"
    },
    "firstRunnerUp" :
    {
      "name" : "Carlos Sainz Jr",
      "time" : "01:37:39.182"
    },
    "secondRunnerUp" :
    {
      "name" : "Lewis Hamilton",
      "time" : "01:37:43.259"
    }
  },
  "result" :
  [
    {
      "driverRaceMapId" : 3,
      "position" : 1,
      "driverId" : 103,
      "name" : "Charles Leclerc"
    },
    {
      "driverRaceMapId" : 4,
      "position" : 2,
      "driverId" : 104,
      "name" : "Carlos Sainz Jr"
    },
    {
      "driverRaceMapId" : 9,
      "position" : 3,
      "driverId" : 106,
      "name" : "Lewis Hamilton"
    },
    {
      "driverRaceMapId" : 10,
      "position" : 4,
      "driverId" : 105,
      "name" : "George Russell"
    }
  ]
} 
```

**Re-parenting of sub-objects between two documents**

Team 의 Driver 를 교환합니다.  `TEAM_DV` 를 update 하여 수행합니다.

먼저, Team 정보를 확인해 봅니다.

```sql
-- Team/Driver 정보 확인
SELECT json_serialize(data PRETTY) FROM team_dv dv
WHERE dv.data.name LIKE 'Mercedes%';

JSON_SERIALIZE(DATAPRETTY)                                                                   
-------------------------------------------------------------------------------------------- 
{
  "_id" : 2,
  "_metadata" :
  {
    "etag" : "855840B905C8CAFA99FB9CBF813992E5",
    "asof" : "000000001F5109A3"
  },
  "name" : "Mercedes",
  "points" : 27,
  "driver" :
  [
    {
      "driverId" : 105,
      "name" : "George Russell",
      "points" : 12
    },
    {
      "driverId" : 106,
      "name" : "Lewis Hamilton",
      "points" : 15
    }
  ]
} 


SELECT json_serialize(data PRETTY) FROM team_dv dv
WHERE dv.data.name LIKE 'Ferrari%';

JSON_SERIALIZE(DATAPRETTY)                                       
----------------------------------------------------------------- 
{
  "_id" : 302,
  "_metadata" :
  {
    "etag" : "C5DD30F04DA1A6A390BFAB12B7D4F700",
    "asof" : "000000001F5109A3"
  },
  "name" : "Ferrari",
  "points" : 43,
  "driver" :
  [
    {
      "driverId" : 103,
      "name" : "Charles Leclerc",
      "points" : 25
    },
    {
      "driverId" : 104,
      "name" : "Carlos Sainz Jr",
      "points" : 18
    }
  ]
} 
```

Update 를 수행합니다.

```sql
UPDATE team_dv dv
SET data = ('{_metadata : {"etag" : "855840B905C8CAFA99FB9CBF813992E5"},
                "_id" : 2,
                "name"   : "Mercedes",
                "points" : 40,
                "driver" : [ {"driverId" : 106,
                            "name"     : "Lewis Hamilton",
                            "points"   : 15},
                            {"driverId" : 103,
                            "name"     : "Charles Leclerc",
                            "points"   : 25} ]}')
    WHERE dv.data.name LIKE 'Mercedes%';


UPDATE team_dv dv
SET data = ('{_metadata : {"etag" : "DA69DD103E8BAE95A0C09811B7EC9628"},
                "_id" : 302,
                "name"   : "Ferrari",
                "points" : 30,
                "driver" : [ {"driverId" : 105,
                            "name"     : "George Russell",
                            "points"   : 12},
                            {"driverId" : 104,
                            "name"     : "Carlos Sainz Jr",
                            "points"   : 18} ]}')
    WHERE dv.data.name LIKE 'Ferrari%';

COMMIT;
```

Team 정보를 확인하여 변경 Driver 정보를 확인합니다.

```sql
SELECT json_serialize(data PRETTY) FROM team_dv dv
WHERE dv.data.name LIKE 'Mercedes%';

JSON_SERIALIZE(DATAPRETTY)                                 
------------------------------------------------------------------- 
{
  "_id" : 2,
  "_metadata" :
  {
    "etag" : "9E266CD7554A89663B73B9977B1F967C",
    "asof" : "000025D77429A3D9"
  },
  "name" : "Mercedes",
  "points" : 40,
  "driver" :
  [
    {
      "driverId" : 103,
      "name" : "Charles Leclerc",
      "points" : 25
    },
    {
      "driverId" : 106,
      "name" : "Lewis Hamilton",
      "points" : 15
    }
  ]
} 


SELECT json_serialize(data PRETTY) FROM team_dv dv
WHERE dv.data.name LIKE 'Ferrari%';

JSON_SERIALIZE(DATAPRETTY)                                                       
----------------------------------------------------------- 
{
  "_id" : 302,
  "_metadata" :
  {
    "etag" : "DD9401D853765859714A6B8176BFC564",
    "asof" : "000025D77429A3DF"
  },
  "name" : "Ferrari",
  "points" : 30,
  "driver" :
  [
    {
      "driverId" : 104,
      "name" : "Carlos Sainz Jr",
      "points" : 18
    },
    {
      "driverId" : 105,
      "name" : "George Russell",
      "points" : 12
    }
  ]
} 

```

**Update a non-updatable field**

DRIVER_DV 뷰에서 Team 정보를 변경해 봅니다.  하지만, DRIVER_DV JSON Duality View 생성 시 TEAM 테이블에 대해 DML 을 허용하지 않아 에러가 발생합니다.

```sql
UPDATE driver_dv dv
SET DATA = ('{_metadata : {"etag" : "FCD4CEC63897F60DEA1EC2F64D3CE53A"},
                "_id" : 103,
                "name" : "Charles Leclerc",
                "points" : 25,
                "teamId" : 2,
                "team" : "Ferrari",
                "race" :
                [
                {
                    "driverRaceMapId" : 3,
                    "raceId" : 201,
                    "name" : "Blue Air Bahrain Grand Prix",
                    "finalPosition" : 1
                }
                ]
            }')
WHERE dv.data."_id" = 103;

[ORA-40940](https://docs.oracle.com/en/error-help/db/ora-40940/): Cannot update field 'team' corresponding to column 'NAME' of table 'TEAM' in JSON Relational Duality View 'DRIVER_DV': Missing UPDATE annotation or NOUPDATE annotation specified. 
```

**Delete by predicate**

`RACE_DV` 에서 특정 Document 를 삭제해 봅니다.  이 때 연관된 RACE 테이블과 DRIVER_RACE_MAP 테이블에서도 해당 데이터가 삭제됩니다.  

```sql
DELETE FROM race_dv dv WHERE dv.data."_id" = 201;

select * from race;

RACE_ID NAME                     LAPS RACE_DATE              PODIUM   
------- ------------------------ ---- ---------------------- ------   
    202 Saudi Arabian Grand Prix   50 3/27/2022, 12:00:00 AM {}       
    203 Australian Grand Prix      58 4/9/2022, 12:00:00 AM  {}

select * from driver_race_map;

No data found
```

## The Extreme Flexibility of JSON Duality Views

이번 테스트에서는 Oracle 23ai 데이터베이스에서 SQL 데이터와 JSON Document 를 동시에 작업하는 단계를 테스트합니다.  이를 통해 JSON Duality View 의 진정한 이중성을 살펴봅니다.

어떤 작업을 수행하든 데이터베이스의 데이터는 동일합니다.  따라서, 사용자는 SQL 액세스 및 JSON Documnt 액세스 시 동일한 데이터를 접근할 수 있습니다.

**Inserting into SQL tables and duality views**

JSON Duality View 에 데이터를 Insert 한 후, 베이스 테이블에도 데이터가 Insert 되는 지 확인합니다.

```sql
SELECT name FROM race where race_id = 204;

No data found

INSERT INTO race_dv VALUES ('{"_id" : 204,
                        "name"   : "Miami Grand Prix",
                        "laps"   : 57,
                        "date"   : "2022-05-08T00:00:00",
                        "podium" : {}}');

SELECT name FROM race where race_id = 204;

NAME               
----------------   
Miami Grand Prix
```

반대로, Relational 테이블에 데이터를 Insert 한 후, JSON Duality View 에서 변경된 데이터를 확인합니다.

```sql
SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$._id') = 205;

No data found


INSERT INTO race
VALUES(205, 'Japanese Grand Prix', 53, TO_DATE('2022-10-08','YYYY-MM-DD'), '{}');

SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$._id') = 205;

JSON_SERIALIZE(DATAPRETTY)                                      
--------------------------------------------------------- 
{
  "_id" : 205,
  "_metadata" :
  {
    "etag" : "BECAB2B6E186FEFB59EBD977418BA26F",
    "asof" : "000025D7743EE40C"
  },
  "name" : "Japanese Grand Prix",
  "laps" : 53,
  "date" : "2022-10-08T00:00:00",
  "podium" :
  {
  },
  "result" :
  [
  ]
} 
```

**Update and replace a document by ID**

테이블 변경 시에도 JSON Duality View 에서 변경된 정보를 확인할 수 있습니다.

```sql
SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$._id') = 204;

JSON_SERIALIZE(DATAPRETTY)                                                 
-------------------------------------------------------------------------------- 
{
  "_id" : 204,
  "_metadata" :
  {
    "etag" : "523BC0C711DCCDEB17A4A9435D95C560",
    "asof" : "000025D7744003F5"
  },
  "name" : "Miami Grand Prix",
  "laps" : 57,
  "date" : "2022-05-08T00:00:00",
  "podium" :
  {
  },
  "result" :
  [
  ]
} 


UPDATE race
SET name = 'Miami Grand Prix',
podium = '{"winner":{"name":"Charles Leclerc","time":"01:37:33.584"},
"firstRunnerUp":{"name":"Carlos Sainz Jr","time":"01:37:39.182"},
"secondRunnerUp":{"name":"Lewis Hamilton","time":"01:37:43.259"}}'
WHERE race_id = 204;

INSERT INTO driver_race_map
VALUES(3, 204, 103, 1),
(4, 204, 104, 2),
(9, 204, 106, 3),
(10, 204, 105, 4);

COMMIT;

SELECT json_serialize(data PRETTY)
FROM race_dv WHERE json_value(data, '$._id') = 204;

JSON_SERIALIZE(DATAPRETTY)                                                                 
---------------------------------------------------------------------------- 
{
  "_id" : 204,
  "_metadata" :
  {
    "etag" : "AE07A8E9B08C660D090808CA1EBA1FDC",
    "asof" : "000025D77440040B"
  },
  "name" : "Miami Grand Prix",
  "laps" : 57,
  "date" : "2022-05-08T00:00:00",
  "podium" :
  {
    "winner" :
    {
      "name" : "Charles Leclerc",
      "time" : "01:37:33.584"
    },
    "firstRunnerUp" :
    {
      "name" : "Carlos Sainz Jr",
      "time" : "01:37:39.182"
    },
    "secondRunnerUp" :
    {
      "name" : "Lewis Hamilton",
      "time" : "01:37:43.259"
    }
  },
  "result" :
  [
    {
      "driverRaceMapId" : 3,
      "position" : 1,
      "driverId" : 103,
      "name" : "Charles Leclerc"
    },
    {
      "driverRaceMapId" : 4,
      "position" : 2,
      "driverId" : 104,
      "name" : "Carlos Sainz Jr"
    },
    {
      "driverRaceMapId" : 9,
      "position" : 3,
      "driverId" : 106,
      "name" : "Lewis Hamilton"
    },
    {
      "driverRaceMapId" : 10,
      "position" : 4,
      "driverId" : 105,
      "name" : "George Russell"
    }
  ]
} 
```

