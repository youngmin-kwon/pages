# Oracle Database 23ai - `DB_DEVELOPER_ROLE` Role 

Oracle Database 23ai 에서는 데이터베이스 개발자에게 필요한 Role 및 Privileges 를 제공하는 `DB_DEVELOPER_ROLE` 이 추가되었습니다.

`DB_DEVELOPER_ROLE` 을 사용하면 Oracle Database 용 애플리케이션 설계, 빌드, 배포에 필요한 모든 권한을 빠르게 할당할 수 있습니다. (데이터 모델을 빌드하는 데 필요한 시스템 권한과 애플리케이션을 모니터링하고 디버깅하는 데 필요한 개체 권한 포함)

데이터베이스 관리자는 애플리케이션 개발자에게 필요한 권한을 개별적으로 부여하거나 DBA 역할을 부여하는 대신 `DB_DEVELOPER_ROLE` 을 부여할 것을 권장합니다. `DB_DEVELOPER_ROLE` 은 최소 권한 원칙을 준수하고 개발 환경에 대한 보안을 강화합니다.

- `DB_DEVELOPER_ROLE` grant / revoke 
    ```sql
    -- As Admin User
    grant db_developer_role to testuser1;

    revoke db_developer_role from testuser1;
    ```

- `DB_DEVELOPER_ROLE` 에 할당되어 있는 Privileges 정보 확인
    ```sql
    -- For Sys Privileges
    SELECT privilege 
    FROM role_sys_privs 
    WHERE role='DB_DEVELOPER_ROLE'
    ORDER BY 1;

    PRIVILEGE
    ----------------------------------------
    CREATE CUBE
    CREATE CUBE BUILD PROCESS
    CREATE CUBE DIMENSION
    CREATE DIMENSION
    CREATE DOMAIN
    CREATE JOB
    CREATE MINING MODEL
    CREATE MLE
    CREATE SESSION
    DEBUG CONNECT SESSION
    EXECUTE DYNAMIC MLE
    FORCE TRANSACTION
    ON COMMIT REFRESH

    13 rows selected.


    -- For Object Privileges
    SELECT table_name, privilege 
    FROM role_tab_privs 
    WHERE role = 'DB_DEVELOPER_ROLE'
    ORDER BY 1;

    TABLE_NAME                     PRIVILEGE
    ------------------------------ ----------
    DBA_PENDING_TRANSACTIONS       SELECT
    DBMS_REDACT                    EXECUTE
    DBMS_RLS                       EXECUTE
    DBMS_TSDP_MANAGE               EXECUTE
    DBMS_TSDP_PROTECT              EXECUTE
    JAVASCRIPT                     EXECUTE
    V_$PARAMETER                   READ
    V_$STATNAME                    READ

    8 rows selected.


    -- Role Privileges
    SELECT granted_role 
    FROM role_role_privs 
    WHERE role ='DB_DEVELOPER_ROLE';

    GRANTED_ROLE
    --------------------
    RESOURCE
    CTXAPP
    ```

