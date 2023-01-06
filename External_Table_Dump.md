# ADB 환경에서 Object Storage 에 존재하는 Export Dump 파일을 사용하여 External Table 구성하기 

- Data Pump Dump 파일을 사용하여 External Table 생성 시 반드시 ORACLE_DATAPUMP access driver 를 통해 생성된 파일만 사용 가능. 
    `expdp` 명령을 사용하여 생성된 Dump 파일로 External Table 생성할 수 없음.


## Autonomous Database 환경 테스트

### Object Storage 에 Dump 파일 생성

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

- External Table 생성

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
    ```

## DBCS or On-Premise Database 에서 Export -> ADB 에서 External Table 생성

