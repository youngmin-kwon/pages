# ADB 환경에서 Object Storage 에 존재하는 Export Dump 파일을 사용하여 External Table 구성하기 

- Data Pump Dump 파일을 사용하여 External Table 생성 시 반드시 ORACLE_DATAPUMP access driver 를 통해 생성된 파일만 사용 가능. 

    `expdp` 명령을 사용하여 생성된 Dump 파일로 External Table 생성할 수 없음.


## Autonomous Database 환경 테스트

#### Object Storage 에 Dump 파일 생성

- Object Store credential 정보를 `DBMS_CLOUD.CREATE_CREDENTIAL` 프로시저를 사용하여 저장한다.
  
    credential 암호 생성은 [`To create an auth token`](https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcredentials.htm#create_swift_password) 페이지 참조

    Autonomous Database 접속 (sqlplus, database actions, SQL Developer, etc.)

    ```
    BEGIN
    DBMS_CLOUD.DROP_CREDENTIAL('ADWTEST_CRED');
    END;
    /

    BEGIN
    DBMS_CLOUD.CREATE_CREDENTIAL(
        credential_name => 'ADWTEST_CRED',
        username => 'oracleidentitycloudservice/youngmin.kwon@oracle.com',
        password => 'wXULJuXDOZY4-g{MH;;k' );
    END;
    /
    ```

- Move Data to Object Store as Oracle Data Pump Files Using EXPORT_DATA


    ```
    -- Sample Table & Data 

    CREATE TABLE channels
    (
        channel_id 		char(1),
        channel_desc 	varchar2(20),
        channel_class 	varchar2(20)
    );

    insert into channels values ('S','Direct Sales', 'Direct' );
    insert into channels values ('T','Tele Sales', 'Direct' );
    insert into channels values ('C','Catalog', 'Indirect' );
    insert into channels values ('I','Internet', 'Indirect' );
    insert into channels values ('P','Partners', 'Others' );

    commit;

    BEGIN
    DBMS_CLOUD.EXPORT_DATA(
        credential_name =>'ADWTEST_CRED',
        file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.dmp',
        format => json_object('type' value 'datapump'),
        query => 'SELECT * FROM channels'
    );
    END;
    /
    ```

    명령 정상 완료 후 Object Storage 에 해당 파일이 생성되었는지 확인

#### External Table 생성

```
BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_EXT',
    credential_name =>'ADWTEST_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.dmp',
    format => json_object('type' value 'datapump', 'rejectlimit' value '1'),
    column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/

select * from channels_ext;

-- 에러 발생 시 확인
BEGIN
DBMS_CLOUD.VALIDATE_EXTERNAL_TABLE (
    table_name => 'CHANNELS_EXT' );
END;
/

```

## DBCS or On-Premise Database 에서 Export -> ADB 에서 External Table 생성

### DBCS 에서 Data Pump 파일 생성 및 Object Storage 에 복사 

- Data Pump 수행을 위한 구성

    ```
    $ mkdir /home/oracle/data

    # sqlplus 로 DB 접속
    CREATE OR REPLACE DIRECTORY data_dir AS '/home/oracle/data';
    -- 필요 권한 설정
    -- GRANT READ, WRITE ON DIRECTORY test_dir TO scott;
    ```

- Create External Table as Select 구문을 사용하여 Data Pump Dump 파일 생성 

    DBCS 접속 후 다음 수행

    ```
    alter session set container=ORADB19_PDB1;

    -- Sample Table 및 Data 생성
    CREATE TABLE channels
    (
        channel_id 		char(1),
        channel_desc 	varchar2(20),
        channel_class 	varchar2(20)
    );

    insert into channels values ('S','Direct Sales', 'Direct' );
    insert into channels values ('T','Tele Sales', 'Direct' );
    insert into channels values ('C','Catalog', 'Indirect' );
    insert into channels values ('I','Internet', 'Indirect' );
    insert into channels values ('P','Partners', 'Others' );

    commit;

    -- Export Dump 파일 생성
    CREATE TABLE channels_xt
    ORGANIZATION EXTERNAL
    (
        TYPE ORACLE_DATAPUMP
        DEFAULT DIRECTORY data_dir
        LOCATION ('channels_xt.dmp')
    )
    AS SELECT * FROM channels;

    -- 생성된 파일 확인
    !ls -al /home/oracle/data
    total 24
    drwxr-xr-x 2 oracle oinstall  4096 Jan  5 05:25 .
    drwx------ 4 oracle oinstall  4096 Jan  5 05:24 ..
    -rw-r--r-- 1 oracle asmadmin    41 Jan  5 05:25 CHANNELS_XT_79995.log
    -rw-r----- 1 oracle asmadmin 12288 Jan  5 05:25 channels_xt.dmp
    ```

### 생성된 Data Pump Dump 파일을 Object Storage 에 Upload

- OCI CLI 를 사용하여 파일 Upload 수행
- OCI CLI 설치 및 환경 구성은 다음 페이지 참조. [OCI CLI Setup](https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)

    ```
    $ oci os object put -ns apackrsct01 -bn dumpdata --name channels_xt.dmp --file /home/oracle/data/channels_xt.dmp
    ```

### External Table 생성

- ADB 접속 후 External Table 생성 

    `DBMS_CLOUD.CREATE_CREDENTIAL` 명령을 앞 테스트에서 수행하였으면 재수행 필요없음.

    ```
    drop table channels_xt;

    BEGIN
    DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
        table_name =>'CHANNELS_XT',
        credential_name =>'ADWTEST_CRED',
        file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channels_xt.dmp',
        format => json_object('type' value 'datapump', 'rejectlimit' value '1'),
        column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
    END;
    /

    select * from channels_xt;
    ```
