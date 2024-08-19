# Oracle Database 23ai : Database Firewall

- [Oracle Database 23ai : Database Firewall](#oracle-database-23ai--database-firewall)
  - [Database User 생성](#database-user-생성)
  - [SQL Firewall 구성](#sql-firewall-구성)
  - [SQL 테스트](#sql-테스트)
  - [접속 테스트](#접속-테스트)
  - [Maintenance](#maintenance)

Oracle Database Vault의 새로운 기능인 SQL Firewall 이 Oracle Database에 내장되었습니다.  

SQL Firewall 기능은 데이터베이스로 들어오는 모든 SQL 문을 검사하고 명시적으로 허가된 SQL만 실행되도록 하여 데이터베이스에서 처리할 수 있는 SQL 문을 제어할 수 있습니다.  허가되지 않은 SQL은 기록하고 차단할 수 있습니다.  또한, 데이터베이스 접속도 제한할 수 있습니다. 

SQL Firewall 은 허가된 SQL 문 또는 접속으로만 데이터베이스 접근를 제한하여 일반적인 데이터베이스 공격으로부터 실시간 보호를 제공합니다. SQL Injection, 비정상적인 액세스, Credential 도용 또는 남용으로 인한 위험을 완화해 줍니다.

SQL Firewall 은 IP address, OS Username, OS Program name 등과 같은 Session Context 데이터를 사용하여 데이터베이스 계정이 데이터베이스에 연결하는 방법을 제한할 수 있습니다.

- 참고문서:
	- [Oracle Documentation - SQL Firewall](https://www.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23&id=DBSEG-GUID-F53EAE01-CE78-47F4-80AD-A0091BA3C434)
	- [LiveLabs - DB Security - SQL Firewall](https://apexapps.oracle.com/pls/apex/r/dbpm/livelabs/view-workshop?wid=3875)

## Database User 생성

- 테스트를 위해 다음의 유저를 생성합니다.
	- `FWADMIN` : SQL Firewall 관리자 계정.  관리자는 `SQL_FIREWALL_ADMIN` role 필요
	- `APPUSER` : Database 사용자

- `FWADMIN` User 생성

  ```sql
  -- ADW Admin User 로 접속
  $ sqlplus amdin/WElcome12345__@adb23ai_low

  -- Drop & Create FWADMIN
  drop user if exists FWADMIN cascade;

  create user FWADMIN identified by WElcome12345__;
  grant connect, resource to FWADMIN;

  -- SQL_FIREWALL_ADMIN Role 부여
  grant SQL_FIREWALL_ADMIN to FWADMIN with admin option;

  -- Enable REST
  BEGIN
      ORDS_ADMIN.ENABLE_SCHEMA(
          p_enabled => TRUE,
          p_schema => 'FWADMIN',
          p_url_mapping_type => 'BASE_PATH',
          p_url_mapping_pattern => 'FWADMIN',
          p_auto_rest_auth=> TRUE
      );

      -- Enable data sharing
      C##ADP$SERVICE.DBMS_SHARE.ENABLE_SCHEMA(
              schema_name => 'FWADMIN',
              enabled => TRUE
      );
      commit;
  END;
  /

  -- Set quota
  alter user FWADMIN quota unlimited on data;
  ```

- `APPUSER` 사용자 및 테이블 생성
  ```sql
  -- Drop & Create APPUSER
  drop user if exists APPUSER cascade;

  create user APPUSER identified by WElcome12345__;
  grant connect, resource to APPUSER;

  -- DB_DEVELOPER_ROLE Role 부여
  grant DB_DEVELOPER_ROLE to APPUSER ;

  -- Enable REST
  BEGIN
      ORDS_ADMIN.ENABLE_SCHEMA(
          p_enabled => TRUE,
          p_schema => 'APPUSER',
          p_url_mapping_type => 'BASE_PATH',
          p_url_mapping_pattern => 'appuser',
          p_auto_rest_auth=> TRUE
      );

      -- Enable data sharing
      C##ADP$SERVICE.DBMS_SHARE.ENABLE_SCHEMA(
              schema_name => 'APPUSER',
              enabled => TRUE
      );
      commit;
  END;
  /

  -- Set quota
  alter user APPUSER quota unlimited on data;
  ```

- `APPUSER` 로 로그인 후 테스트 테이블 생성

  ```sql
  -- SQL*Plus / Database Action - SQL (ADB) 
  SQL> connect appuser/WElcome12345__@adb23ai_low

  DROP TABLE IF EXISTS DEPARTMENTS CASCADE CONSTRAINTS;

  -- Create departments table
  CREATE TABLE departments (
      department_id INT,
      department_name VARCHAR(50)
  );

  -- Insert data into departments table
  INSERT INTO departments (department_id, department_name)
  VALUES
      (1, 'HR'),
      (2, 'IT'),
      (3, 'Finance');
  
  commit;
  ```

## SQL Firewall 구성

- 다른 세션에서 Firewall Admin (`FWADMIN`) 사용자로 접속하여 SQL Firewall 기능 활성화

  ```sql
  $ sqlplus fwadmin/WElcome12345__@adb23ai_low

  exec dbms_sql_firewall.enable;

  - SQL Firewall 상태 확인

  select *
  from dba_sql_firewall_status;

  STATUS			 STATUS_UPDATED_ON			  EXCLUDE_JOBS
  ------------------------ ---------------------------------------- ------------------------------------
  ENABLED 		 30-JUL-24 05.03.12.039080 AM +00:00	  Y
  ```

- `APPUSER` 사용자의 모든 SQL 문을 추적하기 위해 Capture 작업 시작.  

  ```sql
  -- as FWADMIN user
  begin
    dbms_sql_firewall.create_capture (
      username       => 'APPUSER',
      top_level_only => true,
      start_capture  => true);
  end;
  /
  ```

- `APPUSER` 로 접속하여 SQL 문 수행

  ```sql
  --conn appuser/WElcome12345__@adb23ai_low

  insert into departments values (4, 'Engineering');
  commit;
  select count(*) from departments;
  ```

- Firewall Admin(FWADMIN) 사용자로 캡쳐된 정보 확인.  SQL 문 이외에도 여러 세션 속성 정보가 기록된 것을 확인 

  ```sql
  --conn fwadmin/WElcome12345__@adb23ai_low

  set linesize 120 pagesize 40
  column command_type format a12
  column current_user format a10
  column client_program format a35
  column os_user format a10
  column ip_address format a15
  column sql_text format a30

  select command_type,
        current_user,
        client_program,
        os_user,
        ip_address,
        sql_text
  from   dba_sql_firewall_capture_logs
  where  username = 'APPUSER';

  COMMAND_TYPE CURRENT_US CLIENT_PROGRAM                      OS_USER    IP_ADDRESS      SQL_TEXT
  ------------ ---------- ----------------------------------- ---------- --------------- ---------------------------
  INSERT	     APPUSER	sqlplus@cbaa83547a53 (TNS V1-V3)    youngmin_k 129.213.201.220 INSERT INTO DEPARTMENTS VAL
  SELECT	     APPUSER	sqlplus@cbaa83547a53 (TNS V1-V3)    youngmin_k 129.213.201.220 SELECT COUNT (*) FROM DEPAR
  SELECT	     APPUSER	sqlplus@cbaa83547a53 (TNS V1-V3)    youngmin_k 129.213.201.220 SELECT COUNT (*) FROM DEPAR
  SELECT	     APPUSER	sqlplus@cbaa83547a53 (TNS V1-V3)    youngmin_k 129.213.201.220 SELECT DECODE (USER,:"SYS_B
  EXECUTE      APPUSER	sqlplus@cbaa83547a53 (TNS V1-V3)    youngmin_k 129.213.201.220 BEGIN DBMS_APPLICATION_INFO
  ```

- Capture 프로세스 중지
  ```sql
  -- FWADMIN 
  exec dbms_sql_firewall.stop_capture('APPUSER');
  ```

- Capture 로그를 이용하여 Allow-List 생성 
  ```sql
  -- FWADMIN
  exec dbms_sql_firewall.generate_allow_list ('APPUSER');
  ```

- Allow-List 정보 확인

  ```sql
  -- IP Address
  select *
  from   dba_sql_firewall_allowed_ip_addr
  where  username = 'APPUSER';

  USERNAME	         IP_ADDRESS
  -------------------- ---------------
  APPUSER 	         129.213.201.220
  SQL>


  -- Client OS Program
  select *
  from   dba_sql_firewall_allowed_os_prog
  where  username = 'APPUSER';

  USERNAME	         OS_PROGRAM
  -------------------- ----------------------------------------
  APPUSER 	         sqlplus@cbaa83547a53 (TNS V1-V3)


  -- Client OS Username
  select *
  from   dba_sql_firewall_allowed_os_user
  where  username = 'APPUSER';

  USERNAME	         OS_USER
  -------------------- ----------
  APPUSER 	         youngmin_k


  -- SQL 정보
  select current_user,
        sql_text
  from   dba_sql_firewall_allowed_sql
  where  username = 'APPUSER';

  CURRENT_USER SQL_TEXT
  ------------ ----------------------------------------------------------------------
  APPUSER      INSERT INTO DEPARTMENTS VALUES (:"SYS_B_0",:"SYS_B_1")
  APPUSER      SELECT COUNT (*) FROM DEPARTMENTS
  APPUSER      SELECT DECODE (USER,:"SYS_B_0",XS_SYS_CONTEXT (:"SYS_B_1",:"SYS_B_2"),
  APPUSER      BEGIN DBMS_APPLICATION_INFO.SET_MODULE (:1,NULL); END;
  ```

- `DBMS_SQL_FIREWALL.ENABLE_ALLOW_LIST` 프로시저를 사용하여 Allow-List 활성화
  - `enforce` 파리미터 값을 통해 활성화할 범위 지정 가능
  	- `DBMS_SQL_FIREWALL.ENFORCE_CONTEXT` : Only enforces the context (IP Address, OS User and OS Program) allow-list.
  	- `DBMS_SQL_FIREWALL.ENFORCE_SQL` : Only enforces the SQL allow-list.
  	- `DBMS_SQL_FIREWALL.ENFORCE_ALL` : Enforces the context and SQL allow-lists.

  ```sql
  begin
    dbms_sql_firewall.enable_allow_list (
      username => 'APPUSER',
      enforce  => DBMS_SQL_FIREWALL.ENFORCE_ALL,
      block    => true);
  end;
  /
  ```

- `DBA_SQL_FIREWALL_ALLOW_LISTS` 뷰를 통해 Allow-List 상태 확인 가능
  ```sql
  col username format a8
  col status format a7
  col top_level_only format a15
  col enforce format a12
  col block format a5

  select username,
        status,
        top_level_only,
        enforce,
        block
  from   dba_sql_firewall_allow_lists
  where username='APPUSER';

  USERNAME STATUS  TOP_LEVEL_ONLY  ENFORCE      BLOCK
  -------- ------- --------------- ------------ -----
  APPUSER  ENABLED Y		         ENFORCE_ALL  Y

  ```

## SQL 테스트 

- 다른 세션에서 `APPUSER` 로 접속하여 Allow-List에 포함된 SQL 과 포함되지 않은 SQL 수행 테스트.  Allow-List 에 포함되지 않은 SQL 수행 시 다음의 에러 발생
	- `ORA-47605: SQL Firewall violation error.`

  ```sql
  $ sqlplus appuser/WElcome12345__@adb23ai_low

  -- Known SQL
  select count(*) from departments;

    COUNT(*)
  ----------
          4
          
  insert into departments values (5, 'Cloud');

  commit;

  -- Unknown SQL
  SQL> select * from departments;
  select * from departments
                *
  ERROR at line 1:
  ORA-47605: SQL Firewall violation
  ```

- FWADMIN 사용자로 Violation 정보 확인 

  ```sql
  conn fwadmin/WElcome12345__@adb23ai_low

  column occurred_at format a35

  select sql_text,
        firewall_action,
        ip_address,
        cause,
        occurred_at
  from   dba_sql_firewall_violations
  where  username = 'APPUSER'
  ;

  SQL_TEXT							                                   FIREWALL_ACTION	     IP_ADDRESS
  ---------------------------------------------------------------------- --------------------- ---------------
  CAUSE						                        OCCURRED_AT
  --------------------------------------------------- -----------------------------------
  SELECT * FROM DEPARTMENTS					                           Blocked		         129.213.201.220
  SQL violation					                    30-JUL-24 05.33.13.689655 AM +00:00
  ```

  - 만약, 위에서 위반된 SQL 문을 허용하고 싶으면, 해당 위반 사항을 로그에서 가져와서 Allow_List 에 추가하거나, 혹은 수동으로 추가 가능
  ```sql
  exec dbms_sql_firewall.append_allow_list('APPUSER', dbms_sql_firewall.violation_log);
  ```


- 에러가 발생한 SQL (`select * from departments`) 재수행.  정상 동작함을 확인 가능.  세션 재접속할 필요 없음.
  ```sql 
  SQL> select * from departments;
  select * from departments
                *
  ERROR at line 1:
  ORA-47605: SQL Firewall violation
  -- 이전 결과
  -- ORA-47605: SQL Firewall violation 발생한 세션에서 재접속없이..

  -- 재수행 -> 정상 동작
  SQL> select * from departments;

  DEPARTMENT_ID DEPARTMENT_NAME
  ------------- --------------------------------------------------
              1 HR
              2 IT
              3 Finance
              4 Engineering
              5 Cloud
  ```

## 접속 테스트

- Allowed IP 가 아닌 다른 클라이언트에서 접속
  ```
  $ sqlplus appuser/WElcome12345__@adb23ai_low

  ERROR:
  ORA-47605: SQL Firewall violation
  Help: https://docs.oracle.com/error-help/db/ora-47605/
  ```


- FWADMIN 으로 접속하여 IP 허용
  ```sql
  $ sqlplus fwadmin/WElcome12345__@adb23ai_low

  -- Add IP address.
  begin
    dbms_sql_firewall.add_allowed_context (
      username     => 'APPUSER',
      context_type => dbms_sql_firewall.ip_address,
      value        => '129.154.211.53');
  end;
  /
  ```

- 해당 클라이언트에서 재접속
  ```
  $ sqlplus appuser/WElcome12345__@adb23ai_low

  ERROR:
  ORA-47605: SQL Firewall violation
  Help: https://docs.oracle.com/error-help/db/ora-47605/
  ```

  - 여전히 에러 발생.  OS Username 정보와 OS Program Context 정보가 Allow-list 에 포함되지 않아 발생.
  - Violation Log 정보를 Allow-List 에 등록

  ```sql
  $ sqlplus fwadmin/WElcome12345__@adb23ai_low

  exec dbms_sql_firewall.append_allow_list('APPUSER', dbms_sql_firewall.violation_log);

  -- Allow List 정보 확인
  SQL> select *
      from   dba_sql_firewall_allowed_ip_addr ;

  USERNAME   IP_ADDRESS
  ---------- ---------------
  APPUSER    129.154.211.53
  APPUSER    129.213.201.220


  SQL> select *
      from   dba_sql_firewall_allowed_os_prog ;

  USERNAME   OS_PROGRAM
  ---------- ----------------------------------------
  APPUSER    sqlplus@cbaa83547a53 (TNS V1-V3)
  APPUSER    sqlplus@compute-intance (TNS V1-V3)


  SQL> select *
      from   dba_sql_firewall_allowed_os_user;

  USERNAME   OS_USER
  ---------- ---------------
  APPUSER    opc
  APPUSER    youngmin_k

  ```

- Client 재접속
  ```
  $ sqlplus appuser/WElcome12345__@adb23ai_low

  SQL> 
  --> 정상 접속

  SQL> select * from departments;

  DEPARTMENT_ID DEPARTMENT_NAME
  ------------- ---------------
        1 HR
        2 IT
        3 Finance
        4 Engineering
        5 Cloud
  ```

## Maintenance

`DBMS_SQL_FIREWALL` 패키지에는 SQL Firewall 기능을 관리하기 위한 다양한 함수를 포함하고 있습니다.   그 중 몇가지 기능은 다음과 같습니다.

- `PURGE_LOG` : Capture Log 및  Violation Log  정리
  ```sql
  -- Capture log entire contents.
  exec dbms_sql_firewall.purge_log('APPUSER', null, dbms_sql_firewall.capture_log);

  -- Violation logs older than 30 days.
  exec dbms_sql_firewall.purge_log('APPUSER', systimestamp-30, dbms_sql_firewall.violation_log);

  -- Capture and violation logs older than 30 days.
  exec dbms_sql_firewall.purge_log('APPUSER', systimestamp-30, dbms_sql_firewall.all_logs);
  ```

- `UPDATE_ALLOW_LIST_ENFORCEMENT` : Allow-list 적용 변경
  ```sql
  -- Disable context protection.
  exec dbms_sql_firewall.update_allow_list_enforcement('APPUSER', dbms_sql_firewall.enforce_context, false);

  -- Disable SQL protection.
  exec dbms_sql_firewall.update_allow_list_enforcement('APPUSER', dbms_sql_firewall.enforce_sql, false);

  -- Disable context and SQL protection.
  exec dbms_sql_firewall.update_allow_list_enforcement('APPUSER', dbms_sql_firewall.enforce_all, false);
  ```

- Allow-LIst 전체 disable 
  ```sql
  exec dbms_sql_firewall.disable_allow_list ('APPUSER');
  ```

- SQL Firewall 기능 disable
  ```sql
  exec dbms_sql_firewall.disable;
  ```


> SQL Firewall 기능은 Oracle Data Safe 를 통해서도 구성 가능

