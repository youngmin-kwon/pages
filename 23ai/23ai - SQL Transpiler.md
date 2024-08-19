# Oracle Database 23ai - SQL Transpiler - Automatic PL/SQL conversion into SQL

- [Oracle Database 23ai - SQL Transpiler - Automatic PL/SQL conversion into SQL](#oracle-database-23ai---sql-transpiler---automatic-plsql-conversion-into-sql)
  - [테스트 셋업](#테스트-셋업)
  - [SQL Transpiler 기능 테스트](#sql-transpiler-기능-테스트)
  - [Package-Level Function SQL Transpiler 기능 테스트](#package-level-function-sql-transpiler-기능-테스트)


SQL Transpiler 기능은 SQL 쿼리에서 호출되는 PL/SQL 함수를 자동으로 SQL 문으로 변환하여 수행해 주는 기능입니다.  
SQL 에서 PL/SQL 함수를 사용할 경우 SQL 엔진과 PL/SQL 엔진간의 Context Switching 으로 인해 오버헤드가 발생하여 쿼리 성능이 저하되는 경우가 있습니다. 
SQL Tranpiler 기능은 가능한 PL/SQL 함수 호출을 SQL 로 변환해서 Context Switching 으로 인한 오버헤드를 줄여줄 수 있습니다.

간단한 예를 통하여 SQL Transpiler 기능을 살펴보겠습니다.

## 테스트 셋업

- 테스트 용 테이블을 생성합니다.

    ```sql
    drop table if exists emp purge;
    create table emp (  
        empno    number(4,0),  
        ename    varchar2(10),  
        job      varchar2(9),  
        mgr      number(4,0),  
        hiredate date,  
        sal      number(7,2),  
        comm     number(7,2),  
        deptno   number(2,0)
    );

    INSERT INTO EMP VALUES 
        (7839,'KING','PRESIDENT',NULL,TO_DATE('1981-11-17','YYYY-MM-DD'),5000,NULL,10),
        (7698,'BLAKE','MANAGER',7839,TO_DATE('1981-05-01','YYYY-MM-DD'),2850,NULL,30),
        (7782,'CLARK','MANAGER',7839,TO_DATE('1981-05-09','YYYY-MM-DD'),2450,NULL,10),
        (7566,'JONES','MANAGER',7839,TO_DATE('1981-04-01','YYYY-MM-DD'),2975,NULL,20),
        (7654,'MARTIN','SALESMAN',7698,TO_DATE('1981-09-10','YYYY-MM-DD'),1250,1400,30),
        (7499,'ALLEN','SALESMAN',7698,TO_DATE('1981-02-11','YYYY-MM-DD'),1600,300,30),
        (7844,'TURNER','SALESMAN',7698,TO_DATE('1981-08-21','YYYY-MM-DD'),1500,0,30),
        (7900,'JAMES','CLERK',7698,TO_DATE('1981-12-11','YYYY-MM-DD'),950,NULL,30),
        (7521,'WARD','SALESMAN',7698,TO_DATE('1981-02-23','YYYY-MM-DD'),1250,500,30),
        (7902,'FORD','ANALYST',7566,TO_DATE('1981-12-11','YYYY-MM-DD'),3000,NULL,20),
        (7369,'SMITH','CLERK',7902,TO_DATE('1980-12-09','YYYY-MM-DD'),800,NULL,20),
        (7788,'SCOTT','ANALYST',7566,TO_DATE('1982-12-22','YYYY-MM-DD'),3000,NULL,20),
        (7876,'ADAMS','CLERK',7788,TO_DATE('1983-01-15','YYYY-MM-DD'),1100,NULL,20),
        (7934,'MILLER','CLERK',7782,TO_DATE('1982-01-11','YYYY-MM-DD'),1300,NULL,10)
    ;

    COMMIT;
    ```

## SQL Transpiler 기능 테스트

- 테스트 용 PL/SQL Function 을 생성합니다.
    ```sql
    create or replace function is_president(p_mgr number) 
    return number
    as
    begin
        return (case when p_mgr is null then 1 else 0 end);
    end;
    /
    ```

- SQL 쿼리의 WHERE 구문에 위에서 생성한 함수 사용합니다.
    ```sql
    select ename, job from emp where is_president(mgr) = 1;

    ENAME      JOB
    ---------- ---------
    KING       PRESIDENT
    ```

- 위에서 실행한 SQL 문의 실행 계획을 확인합니다.  `DBMS_XPLAN.DISPLAY_CURSOR` 를 사용하여 실행 계획을 확인할 수 있습니다.
    ```sql
    SET LINESIZE 130 PAGESIZE 0

    select * from dbms_xplan.display_cursor();

    SQL_ID  dpwht32hy7hpm, child number 0
    -------------------------------------
        select ename, job from emp where is_president(mgr) = 1

    Plan hash value: 3956160932

    ----------------------------------------------------------------------------------
    | Id  | Operation                 | Name | Rows  | Bytes | Cost (%CPU)| Time     |
    ----------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT          |      |       |       |     2 (100)|          |
    |*  1 |  TABLE ACCESS STORAGE FULL| EMP  |     1 |    26 |     2   (0)| 00:00:01 |
    ----------------------------------------------------------------------------------

    Predicate Information (identified by operation id):
    ---------------------------------------------------

    1 - filter("IS_PRESIDENT"("MGR")=1)

    Note
    -----
    -- dynamic statistics used: dynamic sampling (level=2)
    -- automatic DOP: Computed Degree of Parallelism is 1 because of no expensive parallel operation
    ```

- 실행 계획을 보면 `IS_PRESIDENT` 함수를 사용하여 filter 작업을 수행한 것을 확인할 수 있습니다. 이때 SQL 엔진과 PL/SQL 엔진 간에 Context Switch 가 발생합니다.   
    데이터 건수가 많을 경우 Context Switch 로 인해 성능 저하가 발생할 수 있습니다. 

- 참고로, 이런 문제를 해소하기 위하여 Oracle Database 21c 에서 SQL Macro 라는 기능을 제공합니다.  SQL Macro 기능을 사용하면 다음과 같이 함수를 작성할 수 있습니다.
    ```sql
    create function macro_is_president (p_mgr number) 
    return varchar2 SQL_MACRO(SCALAR) 
    as
    begin
    return('case when p_mgr is null then 1 else 0 end');
    end;
    /  

    select ename, job from emp where macro_is_president(mgr) = 1;

    select * from dbms_xplan.display_cursor();

    PLAN_TABLE_OUTPUT
    ----------------------------------------------------------------------------------------------------------------------------------
    SQL_ID  8zu1dcznbsqfa, child number 1
    -------------------------------------
    select ename, job from emp where macro_is_president(mgr) = 1

    Plan hash value: 3956160932

    ---------------------------------------------------------------------------------------------------------
    | Id  | Operation                  | Name                       | Rows  | Bytes | Cost (%CPU)| Time     |
    ---------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT           |                            |       |       |     2 (100)|          |
    |   1 |  RESULT CACHE              | g60sppbp3bc3s0vqyc0jfjpgzx |     1 |    18 |     2   (0)| 00:00:01 |
    |*  2 |   TABLE ACCESS STORAGE FULL| EMP                        |     1 |    18 |     2   (0)| 00:00:01 |
    ---------------------------------------------------------------------------------------------------------

    Predicate Information (identified by operation id):
    ---------------------------------------------------

    2 - storage(CASE  WHEN "MGR" IS NULL THEN 1 ELSE 0 END =1)
        filter(CASE  WHEN "MGR" IS NULL THEN 1 ELSE 0 END =1)

    Result Cache Information (identified by operation id):
    ------------------------------------------------------

    1 -

    Note
    -----
    - automatic DOP: Computed Degree of Parallelism is 1 because of no expensive parallel operation
    ```

    위의 실행계획에서 보는 바와 같이 함수를 호출하지 않고, 조건문이 바로 수행되는 것을 확인할 수 있습니다.

- Oracle Database 23ai 에서도 SQL Macro 기능을 사용할 수 있습니다.  하지만, 23ai 에서 제공하는 SQL Transpiler 는 자동으로 함수를 변환해 주는 기능을 제공합니다.  SQL Transpiler 기능을 위해 새롭게 도입된 `SQL_TRANSPILER` init parameter 를 설정하면 함수 작성 시 별도의 작업없이 자동으로 변환 작업을 처리해 줍니다.

- `SQL_TRANSPILER` init parameter 정보를 확인합니다.
    ```sql
    show parameter sql_transpiler

    NAME                                 TYPE        VALUE
    ------------------------------------ ----------- ------------------------------
    sql_transpiler                       string      OFF

    alter session set sql_transpiler = ON;
    ```
    `SQL_TRANSPILER` 파라미터는 `ALTER SYSTEM` 명령으로도 변경 가능합니다.

- 앞에서 작성한 `IS_PRESIDENT` 함수를 사용하는 쿼리를 재수행한 후 실행 계획을 확인합니다.
    ```sql
    select ename, job from emp where is_president(mgr) = 1; 

    ENAME      JOB
    ---------- ---------
    KING       PRESIDENT

    select * from  dbms_xplan.display_cursor ();

    PLAN_TABLE_OUTPUT
    ----------------------------------------------------------------------------------------------------------------------------------
    SQL_ID  47tydqpma54ww, child number 1
    -------------------------------------
    select ename, job from emp where is_president(mgr) = 1

    Plan hash value: 3956160932

    ---------------------------------------------------------------------------------------------------------
    | Id  | Operation                  | Name                       | Rows  | Bytes | Cost (%CPU)| Time     |
    ---------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT           |                            |       |       |     2 (100)|          |
    |   1 |  RESULT CACHE              | 07hs440nuknrra2m5kvtq2u8f1 |     1 |    18 |     2   (0)| 00:00:01 |
    |*  2 |   TABLE ACCESS STORAGE FULL| EMP                        |     1 |    18 |     2   (0)| 00:00:01 |
    ---------------------------------------------------------------------------------------------------------

    Predicate Information (identified by operation id):
    ---------------------------------------------------

    2 - storage(CASE  WHEN "MGR" IS NULL THEN 1 ELSE 0 END =1)
        filter(CASE  WHEN "MGR" IS NULL THEN 1 ELSE 0 END =1)

    Result Cache Information (identified by operation id):
    ------------------------------------------------------

    1 -

    Note
    -----
    - automatic DOP: Computed Degree of Parallelism is 1 because of no expensive parallel operation
    ```

    위의 실행계획에서 보면, IS_PRESIDENT(MGR) 함수가 SQL Transpiler 기능에 의해 CASE 문으로 치환되어 수행된 것을 확인할 수 있습니다.

> SQL Transpiler 기능은 모든 PL/SQL 함수에 적용되지는 않고, 제약 사항이 있습니다.  제약사항에 대한 정보는 [Oracle Document](https://docs.oracle.com/en/database/oracle/oracle-database/23/tgsql/introduction-to-sql-tuning.html#GUID-2E23C8E6-7FC6-406F-AE5F-BF2677BB8800) 를 참조하시기 바랍니다.

## Package-Level Function SQL Transpiler 기능 테스트

- SQL Transpiler 기능은 PL/SQL Package 내의 함수에 대해서도 적용됩니다.  테스트를 위해 위에서 작성한 함수를 Package 로 변환합니다.  
    ```sql
    CREATE OR REPLACE PACKAGE emp_pkg AS
        function is_president(p_mgr number) return number;
    end emp_pkg;
    /

    CREATE OR REPLACE PACKAGE BODY emp_pkg AS
        function is_president(p_mgr number) return number
        IS
        begin
            return (case when p_mgr is null then 1 else 0 end);
        end;
    end emp_pkg;
    /
    ```

- Package 내의 함수를 호출하는 SQL 문을 수행한 후 실행 계획을 확인합니다.
    ```sql
    select ename, job from emp where emp_pkg.is_president(mgr) = 1;

    ENAME      JOB
    ---------- ---------
    KING       PRESIDENT


    select * from dbms_xplan.display_cursor();

    PLAN_TABLE_OUTPUT
    -------------------------------------------------------------------------------------------------------------
    SQL_ID  2w3h4nxdmnmsp, child number 0
    -------------------------------------
        select ename, job from emp where emp_pkg.is_president(mgr) = 1

    Plan hash value: 3956160932

    ---------------------------------------------------------------------------------------------------------
    | Id  | Operation                  | Name                       | Rows  | Bytes | Cost (%CPU)| Time     |
    ---------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT           |                            |       |       |     2 (100)|          |
    |   1 |  RESULT CACHE              | 9cxr24cjb1pmh2t8fdptjxu8zt |     1 |    18 |     2   (0)| 00:00:01 |
    |*  2 |   TABLE ACCESS STORAGE FULL| EMP                        |     1 |    18 |     2   (0)| 00:00:01 |
    ---------------------------------------------------------------------------------------------------------

    Predicate Information (identified by operation id):
    ---------------------------------------------------

    2 - storage(CASE  WHEN "MGR" IS NULL THEN 1 ELSE 0 END =1)
        filter(CASE  WHEN "MGR" IS NULL THEN 1 ELSE 0 END =1)

    Result Cache Information (identified by operation id):
    ------------------------------------------------------

    1 -

    Note
    -----
    - automatic DOP: Computed Degree of Parallelism is 1 because of no expensive parallel operation    
   ```
    실행 계획을 보면 Package 에서도 SQL Transpiler 기능이 정상적으로 동작하는 것을 확인할 수 있습니다.
