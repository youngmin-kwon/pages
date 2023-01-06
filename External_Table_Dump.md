# Object Storage 에 존재하는 Export Dump 파일을 사용하여 External Table 구성하기 

## Autonomous Database 환경 테스트

#### 
- Store your object store credentials using the procedure DBMS_CLOUD.CREATE_CREDENTIAL.

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

