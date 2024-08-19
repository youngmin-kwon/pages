# Oracle Database 23ai - Schema Privileges

- [Oracle Database 23ai - Schema Privileges](#oracle-database-23ai---schema-privileges)
  - [Schema Privileges to Simplify Access Control](#schema-privileges-to-simplify-access-control)
  - [Schema Privileges 테스트](#schema-privileges-테스트)

## Schema Privileges to Simplify Access Control

많은 애플리케이션에서 데이터를 소유한 스키마와 해당 데이터를 액세스하여 사용하는 애플리케이션 운영 계정을 분리하여 운영합니다.  이를 통해, 업무 분리와 최소 권한 모델을 제공하여 보안 위험을 낮출 수 있습니다.   

하지만, 스키마 객체가 추가, 변경 되었을 때 관리에 불편한 점이 많이 있었습니다.  이전에는 개발자가 권한 관리를 위해 다음의 방식 중 하나를 사용하였습니다.
- 애플리케이션 스키마의 개별 테이블과 뷰마다 권한 부여
- `ANY` 권한 부여 - `SELECT ANY TABLE, UPDATE ANY TABLE`, ...

첫번째 방식은 모든 테이블이나 뷰에 대해 애플리케이션 사용자에게 권한을 개별적으로 부여해야 합니다.  새로운 테이블이나 뷰가 추가되었을 때 스크립트 등을 개발하여 추가적으로 권한 부여 작업을 수행해야 합니다. 

두번째 방식인 `ANY` 권한을 부여하는 것은 편리하지만, 해당 사용자에게 데이터베이스의 모든 테이블을 액세스할 수 있는 권한을 부여하기 떄문에 보안 측면에서는 위험할 수 있습니다.

이 문제를 해결하기 위해 Oracle Database 23ai 는 새로운 스키마-레벨 권한 기능을 제공합니다.  예를 들어, BOB 이라는 사용자에게 HR 스키마의 모든 테이블을 조회할 수 있도록 다음과 같이 권한을 부여할 수 있습니다.
- `GRANT SELECT ANY TABLE ON SCHEMA HR TO BOB`

위와 같이 스키마-레벨 권한을 부여하면 사용자 BOB 은 HR 스키마의 모든 테이블에 대해 조회 가능하고, HR 스키마의 테이블만 조회할 수 있습니다.

사용자는 특별한 권한이 없어도 자신의 스키마에 스키마-레벨 권한을 부여할 수 있습니다.   다른 사람의 스키마에 스키마-수준 권한을 부여하려면 `GRANT ANY SCHAMA` 또는 `GRANT ANY PRIVILEGE` 시스템 권한이 필요합니다.

## Schema Privileges 테스트

- 테스트 사용자 및 테스트 테이블 생성

    ```sql
    -- connect as ADMIN user
    -- create user and tables

    CREATE USER APP_SCHEMA identified by WElcome12345__ ;
    CREATE USER APP_USER identified by WElcome12345__ ;

    grant create session to APP_USER;
    grant create session, create table, unlimited tablespace to APP_SCHEMA;

    CREATE TABLE if not exists APP_SCHEMA.DATA1 (name VARCHAR2(255));
    INSERT INTO APP_SCHEMA.DATA1 VALUES ('Bob'), ('Jane');

    COMMIT;
    ```

- `APP_USER` 로 접속하여 `APP_SCHEMA` 의 테이블을 조회해 봅니다.  현재 조회 권한이 없어 에러가 발생합니다.
    ```sql
    connect APP_USER/WElcome12345__@adb23ai_low

    select * from APP_SCHEMA.DATA1;
                            *
    ERROR at line 1:
    ORA-00942: table or view "APP_SCHEMA"."DATA1" does not exist
    ```

-  Schema Privileges 를 `APP_USER` 에게 부여합니다.  다시 `APP_SCHEMA` 사용자로 접속하여 `APP_USER` 에게 스키마-레벨 테이블 조회 권한을 부여합니다.
    ```sql
    connect APP_SCHEMA/WElcome12345__@adb23ai_low

    grant select any table on schema app_schema to app_user;
    ```

- `APP_USER` 사용자로 접속하여 `APP_SCHEMA` 의 테이블을 정상적으로 조회할 수 있습니다.
    ```sql
    connect APP_USER/WElcome12345__@adb23ai_low

    select * from APP_SCHEMA.DATA1;

    NAME
    --------------------------------------------------------------------------------
    Bob
    Jane
    ```

- 현재 세션의 Privilege 정보를 `SESSION_SCHEMA_PRIVS` 뷰를 통해 확인할 수 있습니다.
    ```sql
    select * from session_schema_privs;

    PRIVILEGE                 SCHEMA
    ------------------------- --------------------
    SELECT ANY TABLE          APP_SCHEMA
    ```

- APP_SCHEMA 에 새로운 테이블을 생성한 후 APP_USER 에서 바로 조회가능한 것을 환입니다.
    ```sql
    -- APP_SCHAMA 로 접속 후 새로운 테이블 생성
    connect APP_SCHEMA/WElcome12345__@adb23ai_low

    CREATE TABLE if not exists DATA2 (country    VARCHAR2(255));
    INSERT INTO DATA2 VALUES ('United Kingdom'), ('South Korea');

    COMMIT;
        
    -- APP_USER 로 접속 후 조회
    connect APP_SCHEMA/WElcome12345__@adb23ai_low

    select * from APP_SCHEMA.DATA2;

    COUNTRY
    --------------------------------------------------------------------------------
    United Kingdom
    South Korea
    ```

위에서 살펴본 바와 같이 스키마-레벨 권한 기능을 사용하면 해당 스키마의 모든 객체에 대한 권한을 특정 사용자에게 손쉽게 부여할 수 있습니다.  또한, 새로운 객체가 생성 되었을 때도 추가 권한 부여없이 바로 사용할 수 있습니다.

