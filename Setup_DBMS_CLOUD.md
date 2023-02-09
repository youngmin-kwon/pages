# How To Setup And Use DBMS_CLOUD Package (Doc ID 2748362.1)

- DBMS_CLOUD is pre-installed, configured and maintained in Oracle Autonomous Database
- This package is supported in OracleDatabase 19c beginning with 19.9 and in Oracle Database 21c beginning with 21.3

- Installing DBMS_CLOUD
  - Create a schema owning the DBMS_CLOUD package and install the DBMS_CLOUD code in CDB and all PDBs
  - Create the wallet containing the necessary certificates to accessing HTTP URIs and Object Stores
  - Configure your Oracle environment to use the new SSL wallet
  - Configure your database with ACEs for DBMS_CLOUD
  - Verify the configuration of DBMS_CLOUD

- Configuring users or roles to use DBMS_CLOUD
  - Grant the minimal privileges to a user or role for using DBMS_CLOUD
  - Configure ACEs for a user or role to use DBMS_CLOUD
  - Verify the proper setup of your user or role for using DBMS_CLOUD

## Installing DBMS_CLOUD

DBMS_CLOUD is owned by a separate `C##CLOUD$SERVICE` schema. This step creates the user and grants appropriateprivileges. The user is locked by default so that no connections are directly made as this user.

- When you update to a newer RU and that happens to have a new DBMS_CLOUD deployment, then you have tore-run the installation procedure on top of your existing procedure. The installation is written idempotent, soyou do not have to uninstall and re-install
- Oracle is planning to automatically maintain DBMS_CLOUD in future patch sets so it is mandatory to install DBMS_CLOUD into schema `C##C$CLOUD$SERVICE` to avoid conflicts and problems in the future.
- Currently DBMS_CLOUD is only supported for CDB architectures. While you can install it in non-CDBarchitectures it will not function.
- Installation in non-CDB environments is currently not supported.

### Installation in CDB environments

- 아래의 내용으로 `/home/oracle/dbc/dbms_cloud_install.sql` 파일 생성

```

@$ORACLE_HOME/rdbms/admin/sqlsessstart.sql

set verify off
-- you must not change the owner of the functionality to avoid future issues
define username='C##CLOUD$SERVICE'

create user &username no authentication account lock;

REM Grant Common User Privileges
grant INHERIT PRIVILEGES on user &username to sys;
grant INHERIT PRIVILEGES on user sys to &username;
grant RESOURCE, UNLIMITED TABLESPACE, SELECT_CATALOG_ROLE to &username;
grant CREATE ANY TABLE, DROP ANY TABLE, INSERT ANY TABLE, SELECT ANY TABLE,
CREATE ANY CREDENTIAL, CREATE PUBLIC SYNONYM, CREATE PROCEDURE, ALTER SESSION, CREATE JOB to &username;
grant CREATE SESSION, SET CONTAINER to &username;
grant SELECT on SYS.V_$MYSTAT to &username;
grant SELECT on SYS.SERVICE$ to &username;
grant SELECT on SYS.V_$ENCRYPTION_WALLET to &username;
grant read, write on directory DATA_PUMP_DIR to &username;
grant EXECUTE on SYS.DBMS_PRIV_CAPTURE to &username;
grant EXECUTE on SYS.DBMS_PDB_LIB to &username;
grant EXECUTE on SYS.DBMS_CRYPTO to &username;
grant EXECUTE on SYS.DBMS_SYS_ERROR to &username;
grant EXECUTE ON SYS.DBMS_ISCHED to &username;
grant EXECUTE ON SYS.DBMS_PDB_LIB to &username;
grant EXECUTE on SYS.DBMS_PDB to &username;
grant EXECUTE on SYS.DBMS_SERVICE to &username;
grant EXECUTE on SYS.DBMS_PDB to &username;
grant EXECUTE on SYS.CONFIGURE_DV to &username;
grant EXECUTE on SYS.DBMS_SYS_ERROR to &username;
grant EXECUTE on SYS.DBMS_CREDENTIAL to &username;
grant EXECUTE on SYS.DBMS_RANDOM to &username;
grant EXECUTE on SYS.DBMS_SYS_SQL to &username;
grant EXECUTE on SYS.DBMS_LOCK to &username;
grant EXECUTE on SYS.DBMS_AQADM to &username;
grant EXECUTE on SYS.DBMS_AQ to &username;
grant EXECUTE on SYS.DBMS_SYSTEM to &username;
grant EXECUTE on SYS.SCHED$_LOG_ON_ERRORS_CLASS to &username;
grant SELECT on SYS.DBA_DATA_FILES to &username;
grant SELECT on SYS.DBA_EXTENTS to &username;
grant SELECT on SYS.DBA_CREDENTIALS to &username;
grant SELECT on SYS.AUDIT_UNIFIED_ENABLED_POLICIES to &username;
grant SELECT on SYS.DBA_ROLES to &username;
grant SELECT on SYS.V_$ENCRYPTION_KEYS to &username;
grant SELECT on SYS.DBA_DIRECTORIES to &username;
grant SELECT on SYS.DBA_USERS to &username;
grant SELECT on SYS.DBA_OBJECTS to &username;
grant SELECT on SYS.V_$PDBS to &username;
grant SELECT on SYS.V_$SESSION to &username;
grant SELECT on SYS.GV_$SESSION to &username;
grant SELECT on SYS.DBA_REGISTRY to &username;
grant SELECT on SYS.DBA_DV_STATUS to &username;

alter session set current_schema=&username;
REM Create the Catalog objects
@$ORACLE_HOME/rdbms/admin/dbms_cloud_task_catalog.sql
@$ORACLE_HOME/rdbms/admin/dbms_cloud_task_views.sql
@$ORACLE_HOME/rdbms/admin/dbms_cloud_catalog.sql
@$ORACLE_HOME/rdbms/admin/dbms_cloud_types.sql

REM Create the Package Spec
@$ORACLE_HOME/rdbms/admin/prvt_cloud_core.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_task.plb
@$ORACLE_HOME/rdbms/admin/dbms_cloud_capability.sql
@$ORACLE_HOME/rdbms/admin/prvt_cloud_request.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_internal.plb
@$ORACLE_HOME/rdbms/admin/dbms_cloud.sql
@$ORACLE_HOME/rdbms/admin/prvt_cloud_admin_int.plb

REM Create the Package Body
@$ORACLE_HOME/rdbms/admin/prvt_cloud_core_body.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_task_body.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_capability_body.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_request_body.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_internal_body.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_body.plb
@$ORACLE_HOME/rdbms/admin/prvt_cloud_admin_int_body.plb

-- Create the metadata
@$ORACLE_HOME/rdbms/admin/dbms_cloud_metadata.sql

alter session set current_schema=sys;

@$ORACLE_HOME/rdbms/admin/sqlsessend.sql

```

- `dbms_cloud_install.sql` 파일을 `/home/oracle/dbc` 디렉토리에 생성하였다면, 다음의 명령을 수행한다.

```
$ORACLE_HOME/perl/bin/perl $ORACLE_HOME/rdbms/admin/catcon.pl -u sys/"Welcome3#Welcome3#" --force_pdb_mode 'READ WRITE' -b dbms_cloud_install -d /home/oracle/dbc -l /home/oracle/dbc dbms_cloud_install.sql
```
    수행 후 에러 발생 여부 확인 

- DBMS_CLOUD package 생성 여부 확인
```
REM from within ROOT to see all containers

select con_id, owner, object_name, status, sharing, oracle_maintained 
from cdb_objects 
where object_name = 'DBMS_CLOUD' 
order by con_id;

REM within an individual container only

select owner, object_name, status, sharing, oracle_maintained 
from dba_objects 
where object_name ='DBMS_CLOUD';
```

Note that the installation will force all pluggable database to being opened for installation of DBMS_CLOUD but will retainthe prior state of a PDB after installation, so the queries above will only show/work for open pluggable databases.
If the install logs show any error and/or you have any invalid objects owned by C##CLOUD$SERVICE you need to analyzeand correct those.

### Create SSL Wallet with Certificates

In order to safely access HTTP URIs and Object Stores, a wallet is required with appropriate certificates for the object stores. 

Currently Oracle does not ship the certs as part of RUs

You can download the necessary certificates from https://objectstorage.us-phoenix-1.oraclecloud.com/p/QsLX1mx9A-vnjjohcC7TIK6aTDFXVKr0Uogc2DAN-Rd7j6AagsmMaQ3D3Ti4a9yU/n/adwcdemo/b/CERTS/o/dbc_certs.tar

All Object Storage access is done through https and requires the database to have the certifications of all trusted locations. You need to create a security wallet containing the certificates.

- The wallet has to be created with auto login capabilities.
- On RAC installations, the wallet must either be accessible for all nodes centrally or you have to create the wallet on all nodes for local wallet storage.
- -> RAC 에 설치할 경우, Wallet 은 모든 노드에서 접근가능한 공유 스토리지에 생성하거나, 모든 노드의 로컬 스토리지에 wallet 을 생성하여야 한다.

The wallet can be stored at any location that is accessible by the user you installed the SW for - typically oracle, but it is recommended to manage the security wallet similar to the tde wallet provided by default for Cloud installation in Oracle OCI: by default, TDE wallets in Oracle Database Cloud installations are stored at /opt/oracle/dcs/commonstore/wallets/tde/$ORACLE_UNQNAME.

It is recommended to store the SSL wallet in an equivalent location, for example /opt/oracle/dcs/commonstore/wallets/ssl. Assuming you are choosing this location and have unpacked the certs in /home/oracle/dbc, you have to issue the following commands to create the necessary wallet.

If you are already having a wallet for SSL certificates then you do not have to create a new wallet but rather add the required certs to the existing wallet.

```
cd /opt/oracle/dcs/commonstore/wallets/ssl

tar xvf /tmp/dcs_certs.tar

orapki wallet create -wallet . -pwd Oracle123456 -auto_login

#! /bin/bash
for i in `ls <location of cert files>/*cer`
do
orapki wallet add -wallet . -trusted_cert -cert $i -pwd Oracle123456
done

```

정상적인 wallet 생성 여부 확인

```
cd /opt/oracle/dcs/commonstore/wallets/ssl
orapki wallet display -wallet .

# Sample output for a wallet containing only the three certs required for dbms_cloud:
[oracle@hb19cee ssl]$ orapki wallet display -wallet .
Oracle PKI Tool Release 21.0.0.0.0 - Production
Version 21.0.0.0.0
Copyright (c) 2004, 2020, Oracle and/or its affiliates. All rights reserved.

Requested Certificates:
User Certificates:
Trusted Certificates:
Subject: CN=VeriSign Class 3 Public Primary Certification Authority - G5,OU=(c) 2006 VeriSign\, Inc. - For authorized use only,OU=VeriSign Trust Network,O=VeriSign\, Inc.,C=US
Subject: CN=Baltimore CyberTrust Root,OU=CyberTrust,O=Baltimore,C=IE
Subject: CN=DigiCert Global Root CA,OU=www.digicert.com,O=DigiCert Inc,C=US

```

### Configure your Oracle environment to use the new SSL wallet

To have your SSL wallet taken into effect you need to point to the newly created ssl wallet for your Oracle installation by adding it to your sqlnet.ora on the Server side. If you are on a RAC installation then you have to adjust this on all nodes.

- DB 서버의 sqlnet.ora 파일에 wallet 위치 지정
- RAC 환경일 경우 모든 노드에서 수행해야 함

- sqlnet.ora 파일 위치
  - Grid Infra 없이 설치되었다면, $ORACLE_HOME/network/admin 
  - Grid Infra 설치되었다면, $GRID_HOME/network/admin

For cloud installations without Grid infrastructure, the default location of this file is $ORACLE_HOME/network/admin.
For cloud installations with Grid infrastructure, the default location of this file is $GRID_HOME/network/admin.
If you already had a wallet for SSL certificates and added the certificates to the existing one then this step is not necessary.

Assuming you stored your wallet at the suggested location mentioned in the wallet setup section, your entry in sqlnet.ora would look as follows:

sqlnet.ora 파일에 다음 추가

```
# /u01/app/19.0.0.0/grid/network/admin/sqlnet.ora

WALLET_LOCATION=
(SOURCE=
    (METHOD=FILE)
    (METHOD_DATA=(DIRECTORY=/opt/oracle/dcs/commonstore/wallets/ssl))
)

```

### Configure the Database with ACEs for DBMS_CLOUD

Create Access Control Entries (ACEs) to allow communication with Object Stores through https.

By default an Oracle database does not allow any outside communication, so you need to enable the appropriate Access Control Entries. In case your database is behind a firewall, you need to provide the information about your Internet Gateway and configure the Access Control Entries appropriately.

Wrap the following commands into a sql script and execute those in your multi-tenant environment by connecting to CDB$ROOT container as SYS.

You need to execute this only in CDB$ROOT if your installation is in a multitenant environment.
Please ensure to set the variables for your environment appropriately. If you do not set them correctly DBMS_CLOUD will not function properly.

- 아래 내용을 `dbc_aces.sql` 파일로 생성.
- CDB$ROOT container 에 SYS 로 접속 후 sql 스크립트 수행. CDB$ROOT 에서 한번만 수행하면 됨.

```
-- dcs_aces.sql

@$ORACLE_HOME/rdbms/admin/sqlsessstart.sql

-- you must not change the owner of the functionality to avoid future issues
define clouduser=C##CLOUD$SERVICE

-- CUSTOMER SPECIFIC SETUP, NEEDS TO BE PROVIDED BY THE CUSTOMER
-- - SSL Wallet directory
-- define sslwalletdir=<Set SSL Wallet Directory>
define sslwalletdir=/opt/oracle/dcs/commonstore/wallets/ssl

--
-- UNCOMMENT AND SET THE PROXY SETTINGS VARIABLES IF YOUR ENVIRONMENT NEEDS PROXYS
--
-- define proxy_uri=<your proxy URI address>
-- define proxy_host=<your proxy DNS name>
-- define proxy_low_port=<your_proxy_low_port>
-- define proxy_high_port=<your_proxy_high_port>

-- Create New ACL / ACE s
begin
-- Allow all hosts for HTTP/HTTP_PROXY
dbms_network_acl_admin.append_host_ace(
host =>'*',
lower_port => 443,
upper_port => 443,
ace => xs$ace_type(
privilege_list => xs$name_list('http', 'http_proxy'),
principal_name => upper('&clouduser'),
principal_type => xs_acl.ptype_db));
--
-- UNCOMMENT THE PROXY SETTINGS SECTION IF YOUR ENVIRONMENT NEEDS PROXYS
--
-- Allow Proxy for HTTP/HTTP_PROXY
-- dbms_network_acl_admin.append_host_ace(
-- host =>'&proxy_host',
-- lower_port => &proxy_low_port,
-- upper_port => &proxy_high_port,
-- ace => xs$ace_type(
-- privilege_list => xs$name_list('http', 'http_proxy'),
-- principal_name => upper('&clouduser'),
-- principal_type => xs_acl.ptype_db));
--
-- END PROXY SECTION
--

-- Allow wallet access
dbms_network_acl_admin.append_wallet_ace(
wallet_path => 'file:&sslwalletdir',
ace => xs$ace_type(privilege_list =>
xs$name_list('use_client_certificates', 'use_passwords'),
principal_name => upper('&clouduser'),
principal_type => xs_acl.ptype_db));
end;
/

-- Setting SSL_WALLET database property
begin
-- comment out the IF block when installed in non-CDB environments
if sys_context('userenv', 'con_name') = 'CDB$ROOT' then
execute immediate 'alter database property set ssl_wallet=''&sslwalletdir''';
--
-- UNCOMMENT THE FOLLOWING COMMAND IF YOU ARE USING A PROXY
--
-- execute immediate 'alter database property set http_proxy=''&proxy_uri''';
end if;
end;
/

@$ORACLE_HOME/rdbms/admin/sqlsessend.sql

```

```
# Connect to CDB$ROOT
connect sys/<password> as sysdba
@@dbc_aces.sql

```

You can check the proper setup of your database properties.

You should not see any entry for HTTP_PROXY if your environment does not need one.
Property SSL_WALLET should show the directory where your wallet is located.

```
REM from within CDB$ROOT
select * from database_properties where property_name in ('SSL_WALLET','HTTP_PROXY');
```

### Verify Configuration of DBMS_CLOUD

You already verified the proper installation of the DBMS_CLOUD code in the initial installation section. You now want to verify the proper setup of the SSL Wallet and the ACEs.

Wrap the following commands into a sql script and execute it as user SYS either within the CDB or in any PDB:

Please ensure to set the variables for your environment appropriately. If you do not set them correctly then the sample procedure will not work, independent of whether or not you have set up DBMS_CLOUD correctly.

```
-- verify.sql

-- you must not change the owner of the functionality to avoid future issues
define clouduser=C##CLOUD$SERVICE

-- CUSTOMER SPECIFIC SETUP, NEEDS TO BE PROVIDED BY THE CUSTOMER
-- - SSL Wallet directory and password
-- define sslwalletdir=<Set SSL Wallet Directory>
-- define sslwalletpwd=<Set SSL Wallet password>
define sslwalletdir=/opt/oracle/dcs/commonstore/wallets/ssl
define sslwalletpwd=Oracle123456

-- create and run this procedure as owner of the ACLs, which is the future owner
-- of DBMS_CLOUD
CREATE OR REPLACE PROCEDURE &clouduser..GET_PAGE(url IN VARCHAR2) AS
request_context UTL_HTTP.REQUEST_CONTEXT_KEY;
req UTL_HTTP.REQ;
resp UTL_HTTP.RESP;
data VARCHAR2(32767) default null;
err_num NUMBER default 0;
err_msg VARCHAR2(4000) default null;

BEGIN

-- Create a request context with its wallet and cookie table
request_context := UTL_HTTP.CREATE_REQUEST_CONTEXT(
wallet_path => 'file:&sslwalletdir',
wallet_password => '&sslwalletpwd');

-- Make a HTTP request using the private wallet and cookie
-- table in the request context
req := UTL_HTTP.BEGIN_REQUEST(
url => url,
request_context => request_context);

resp := UTL_HTTP.GET_RESPONSE(req);

DBMS_OUTPUT.PUT_LINE('valid response');

EXCEPTION
WHEN OTHERS THEN
err_num := SQLCODE;
err_msg := SUBSTR(SQLERRM, 1, 3800);
DBMS_OUTPUT.PUT_LINE('possibly raised PLSQL/SQL error: ' ||err_num||' - '||err_msg);

UTL_HTTP.END_RESPONSE(resp);
data := UTL_HTTP.GET_DETAILED_SQLERRM ;
IF data IS NOT NULL THEN
DBMS_OUTPUT.PUT_LINE('possibly raised HTML error: ' ||data);
END IF;
END;
/
set serveroutput on
BEGIN
&clouduser..GET_PAGE('https://objectstorage.eu-frankfurt-1.oraclecloud.com');
END;
/

set serveroutput off
drop procedure &clouduser..GET_PAGE;

```

If you have properly configured the SSL wallet and set up your database environment, the script will return "valid response" when you can successfully reach out to Oracle Object Store.

If you receive an error then your installation was not done properly. Please correct any possible errors before continuing. If you cannot successfully access the sample page you will not be able to access any Object Storage either.


## Configuring users or roles to use DBMS_CLOUD

### Grant the minimal privileges to a user or role for using DBMS_CLOUD

The following privileges are needed for a user or role to use DBMS_CLOUD functionality. It is recommended to grant the necessary privileges through a role to make the management of the necessary privileges easier for multiple users.

The example script will use a local role CLOUD_USER and grant privileges to a local user SCOTT. You can modify this script as seen fit for your pluggable database environment and execute it as privileged administrator, e.g. SYS or SYSTEM, within your pluggable database.

SCOTT 사용자 생성
```
conn / as sysdba
alter session set container=pdb1;

create user scott identified by Welcome3#Welcome3#;
grant connect, resource, unlimited tablespace to scott;
```

SCOTT 사용자에게 권한 설정

```
set verify off

-- target sample role
define userrole='CLOUD_USER'

-- target sample user
define username='SCOTT'

create role &userrole;
grant cloud_user to &username;

REM the following are minimal privileges to use DBMS_CLOUD
REM - this script assumes core privileges
REM - CREATE SESSION
REM - Tablespace quote on the default tablespace for a user

REM for creation of external tables, e.g. DBMS_CLOUD.CREATE_EXTERNAL_TABLE()
grant CREATE TABLE to &userrole;

REM for using COPY_DATA()
REM - Any log and bad file information is written into this directory
grant read, write on directory DATA_PUMP_DIR to &userrole;

REM
grant EXECUTE on dbms_cloud to &userrole;
```

Alternatively you can grant the privileges to an individual user. The example script will use a local user SCOTT. You can modify this script as seen fit for your environment.

```
set verify off

-- target sample user
define username='SCOTT'

REM the following are minimal privileges to use DBMS_CLOUD
REM - this script assumes core privileges
REM - CREATE SESSION
REM - Tablespace quote on the default tablespace for a user

REM for creation of external tables, e.g. DBMS_CLOUD.CREATE_EXTERNAL_TABLE()
grant CREATE TABLE to &username;

REM for using COPY_DATA()
REM - Any log and bad file information is written into this directory
grant read, write on directory DATA_PUMP_DIR to &username;

REM
grant EXECUTE on dbms_cloud to &username;

```

### Configure ACEs for a user or role to use DBMS_CLOUD

DBMS_CLOUD is a package with INVOKERs right privilege. To provide all the functionality of DBMS_CLOUD to a user or role you need to enable the appropriate Access Control Entries, similar to the ones of DBMS_CLOUD.

Wrap the following commands into a sql script and execute those in either the CDB or the PDB as SYS where you want to provide DBMS_CLOUD functionality to your user or role:

Please ensure to set the variables for your environment appropriately. If you do not set them correctly then your user or role will not be able to use DBMS_CLOUD.

It is recommended to grant the necessary privileges through a role to make the management of the necessary privileges easier for multiple users.

The example script will use a local role CLOUD_USER and grant privileges to a local user SCOTT. You can modify this script as seen fit for your pluggable database environment and execute it as privileged administrator, e.g. SYS or SYSTEM, within your pluggable database

```
@$ORACLE_HOME/rdbms/admin/sqlsessstart.sql

-- target sample role
define cloudrole=CLOUD_USER

-- CUSTOMER SPECIFIC SETUP, NEEDS TO BE PROVIDED BY THE CUSTOMER
-- - SSL Wallet directory
--define sslwalletdir=<Set SSL Wallet Directory>
define sslwalletdir=/opt/oracle/dcs/commonstore/wallets/ssl

--
-- UNCOMMENT AND SET THE PROXY SETTINGS VARIABLES IF YOUR ENVIRONMENT NEEDS PROXYS
--
-- define proxy_uri=<your proxy URI address>
-- define proxy_host=<your proxy DNS name>
-- define proxy_low_port=<your_proxy_low_port>
-- define proxy_high_port=<your_proxy_high_port>

-- Create New ACL / ACE s
begin
-- Allow all hosts for HTTP/HTTP_PROXY
dbms_network_acl_admin.append_host_ace(
host =>'*',
lower_port => 443,
upper_port => 443,
ace => xs$ace_type(
privilege_list => xs$name_list('http', 'http_proxy'),
principal_name => upper('&cloudrole'),
principal_type => xs_acl.ptype_db));

--
-- UNCOMMENT THE PROXY SETTINGS SECTION IF YOUR ENVIRONMENT NEEDS PROXYS
--
-- Allow Proxy for HTTP/HTTP_PROXY
-- dbms_network_acl_admin.append_host_ace(
-- host =>'&proxy_host',
-- lower_port => &proxy_low_port,
-- upper_port => &proxy_high_port,
-- ace => xs$ace_type(
-- privilege_list => xs$name_list('http', 'http_proxy'),
-- principal_name => upper('&cloudrole'),
-- principal_type => xs_acl.ptype_db));
--
-- END PROXY SECTION
--

-- Allow wallet access
dbms_network_acl_admin.append_wallet_ace(
wallet_path => 'file:&sslwalletdir',
ace => xs$ace_type(privilege_list =>
xs$name_list('use_client_certificates', 'use_passwords'),
principal_name => upper('&cloudrole'),
principal_type => xs_acl.ptype_db));
end;
/

@$ORACLE_HOME/rdbms/admin/sqlsessend.sql
```

The example script will assume a local user SCOTT having been created, as shown in the previous example. You can modify this script as seen fit for your environment.

```
@$ORACLE_HOME/rdbms/admin/sqlsessstart.sql

-- target sample user
define clouduser=SCOTT

-- CUSTOMER SPECIFIC SETUP, NEEDS TO BE PROVIDED BY THE CUSTOMER
-- - SSL Wallet directory
--define sslwalletdir=<Set SSL Wallet Directory>
define sslwalletdir=/opt/oracle/dcs/commonstore/wallets/ssl

-- Proxy definition
--define proxy_uri=<your proxy URI address>
--define proxy_host=<your proxy DNS name>
--define proxy_low_port=<your_proxy_low_port>
--define proxy_high_port=<your_proxy_high_port>

-- Create New ACL / ACE s
begin
-- Allow all hosts for HTTP/HTTP_PROXY
dbms_network_acl_admin.append_host_ace(
host =>'*',
lower_port => 443,
upper_port => 443,
ace => xs$ace_type(
privilege_list => xs$name_list('http', 'http_proxy'),
principal_name => upper('&clouduser'),
principal_type => xs_acl.ptype_db));

--
-- UNCOMMENT THE PROXY SETTINGS SECTION IF YOUR ENVIRONMENT NEEDS PROXYS
--
-- Allow Proxy for HTTP/HTTP_PROXY
-- dbms_network_acl_admin.append_host_ace(
-- host =>'&proxy_host',
-- lower_port => &proxy_low_port,
-- upper_port => &proxy_high_port,
-- ace => xs$ace_type(
-- privilege_list => xs$name_list('http', 'http_proxy'),
-- principal_name => upper('&clouduser'),
-- principal_type => xs_acl.ptype_db));
--
-- END PROXY SECTION
--

-- Allow wallet access
dbms_network_acl_admin.append_wallet_ace(
wallet_path => 'file:&sslwalletdir',
ace => xs$ace_type(privilege_list =>
xs$name_list('use_client_certificates', 'use_passwords'),
principal_name => upper('&clouduser'),
principal_type => xs_acl.ptype_db));
end;
/

@$ORACLE_HOME/rdbms/admin/sqlsessend.sql
```

Your user or role is now properly configured and enabled to using DBMS_CLOUD.

Verify the proper setup of your user or role for using DBMS_CLOUD 

### Create credentials and verify

To access data in Object Store that is not public you need to authenticate with a OCI user in your tenancy that has appropriate privileges to the object storage bucket in the region in question. You need to create either an OCI API signing key or an auth token for a user in your tenancy. See https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/managingcredentials.htm for details.

Assuming you have created an auth token you now need to create a credential object in your database schema for authentication:

```
connect scott/Welcome3#Welcome3#@oradb19_pdb1

BEGIN
DBMS_CLOUD.DROP_CREDENTIAL('OBJ_CRED');
END;
/

BEGIN
DBMS_CLOUD.CREATE_CREDENTIAL(
    credential_name => 'OBJ_CRED',
    username => 'username@mail.com',
    password => 'Auth Token' );
END;
/
```

After creation of your credential object you should now be able to access the bucket in question in your tenancy that the OCI user in your tenancy has privileges for:

```
select object_name from dbms_cloud.list_objects('OBJ_CRED', 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/' );
```

- External Table 생성

```
drop table channels_ext;

BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_EXT',
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel_xt.dmp',
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
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel_dbcs.dmp',
    format => json_object('type' value 'datapump'),
    query => 'SELECT * FROM channels'
);
END;
/

```
drop table channels_xt;

BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_XT',
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channels_xt.dmp',
    format => json_object('type' value 'datapump', 'rejectlimit' value '1'),
    column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/

select * from channels_xt;
```


drop table emp_ext;

begin
  dbms_cloud.create_external_table(
    table_name      => 'emp_ext',
    credential_name => 'OBJ_CRED',
    file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/emp.dat',
    column_list     => 'empno     number(4),
                        ename     varchar2(10),
                        job       varchar2(9),
                        mgr       number(4),
                        hiredate  date,
                        sal       number(7,2),
                        comm      number(7,2),
                        deptno    number(2)',
    format          => json_object('ignoremissingcolumns' value 'true', 'removequotes' value 'true')
 );
end;
/

-- Export Data
begin
  dbms_cloud.export_data (
    credential_name => 'OBJ_CRED',
    file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/emp.dmp',
    query           => 'select * from emp_ext',
    format          => json_object('type' value 'datapump')
  );
end;
/

## Validate the proper user configuration and privilege (accessibility of wallet, privilege to use wallet, database-wide setting of wallet)

If you are encountering problems with DBMS_CLOUD with the user or role you just configured you can always test the proper configuration of your environment without DBMS_CLOUD, using the same sample code that was used for the DBMS_CLOUD setup with the user or role you configured.

Assuming you were setting up a user named SCOTT, wrap the following commands into a sql script and execute it as SYS in the pluggable database you were configuring:

Please ensure to set the variables for your environment appropriately. If you do not set them correctly then the sample procedure will not work, independent of whether or not you have set up your user or role correctly.
The sample code needs additional privileges for your user or role, name EXECUTE on UTL_HTTP. If your user or role does not have this privilege you need to grant it temporarily to successfully run this code.
If you have granted the ACLs through a role then you need to grant those explicitly to user SCOTT for this example to work

```
-- user to trouble shoot
define clouduser=SCOTT

-- CUSTOMER SPECIFIC SETUP, NEEDS TO BE PROVIDED BY THE CUSTOMER
-- - SSL Wallet directory and password
define sslwalletdir=<Set SSL Wallet Directory>
define sslwalletpwd=<Set SSL Wallet password>

-- create and run this procedure as owner of the ACLs, which is the future owner
-- of DBMS_CLOUD
CREATE OR REPLACE PROCEDURE &clouduser..GET_PAGE(url IN VARCHAR2) AS
request_context UTL_HTTP.REQUEST_CONTEXT_KEY;
req UTL_HTTP.REQ;
resp UTL_HTTP.RESP;
data VARCHAR2(32767) default null;
err_num NUMBER default 0;
err_msg VARCHAR2(4000) default null;

BEGIN

-- Create a request context with its wallet and cookie table
request_context := UTL_HTTP.CREATE_REQUEST_CONTEXT(
wallet_path => 'file:&sslwalletdir',
wallet_password => '&sslwalletpwd');

-- Make a HTTP request using the private wallet and cookie
-- table in the request context
req := UTL_HTTP.BEGIN_REQUEST(
url => url,
request_context => request_context);

resp := UTL_HTTP.GET_RESPONSE(req);

DBMS_OUTPUT.PUT_LINE('valid response');

EXCEPTION
WHEN OTHERS THEN
err_num := SQLCODE;
err_msg := SUBSTR(SQLERRM, 1, 3800);
DBMS_OUTPUT.PUT_LINE('possibly raised PLSQL/SQL error: ' ||err_num||' - '||err_msg);

UTL_HTTP.END_RESPONSE(resp);
data := UTL_HTTP.GET_DETAILED_SQLERRM ;
IF data IS NOT NULL THEN
DBMS_OUTPUT.PUT_LINE('possibly raised HTML error: ' ||data);
END IF;
END;
/
set serveroutput on
BEGIN
&clouduser..GET_PAGE('https://objectstorage.eu-frankfurt-1.oraclecloud.com');
END;
/

set serveroutput off
drop procedure &clouduser..GET_PAGE;
```





    CREATE TABLE channels_xt
    ORGANIZATION EXTERNAL
    (
        TYPE ORACLE_DATAPUMP
        DEFAULT DIRECTORY data_pump_dir
        LOCATION ('channels_xt.dmp')
    )
    AS SELECT * FROM channels;



BEGIN  
   DBMS_CLOUD.EXPORT_DATA(
      credential_name => 'OBJ_CRED',
      file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.csv',
      query           => 'SELECT * FROM DEPT',
      format          => JSON_OBJECT('type' value 'csv')
     );
   END;
/  


begin
  dbms_cloud.export_data (
    credential_name => 'OBJ_CRED',
    file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channels_xxx.dmp',
    query           => 'select * from channels',
    format          => json_object('type' value 'datapump')
  );
end;
/


안녕하세요. 권영민입니다.





BEGIN  
   DBMS_CLOUD.EXPORT_DATA(
      credential_name => 'OBJ_CRED',
      file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.csv',
      query           => 'SELECT * FROM CHANNELS',
      format          => JSON_OBJECT('type' value 'csv')
     );
   END;
/  

begin
  dbms_cloud.export_data (
    credential_name => 'OBJ_CRED',
    file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.json',
    query           => 'select * from channels',
    format          => '{"type" : "JSON"}'
  );
end;
/


BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_CSV_EXT',
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel_1_20230127T030644730815Z.csv',
    format          => json_object('type' value 'csv', 'skipheaders' value '1')
    column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/


-- datapump dump 파일 External Table 생성

SQL> BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_DMP_EXT',
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.dmp',
    format => json_object('type' value 'datapump', 'rejectlimit' value '1'),
    column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/

PL/SQL procedure successfully completed.

SQL> select * from channels_dmp_ext;
select * from channels_dmp_ext
*
ERROR at line 1:
ORA-29913: error in executing ODCIEXTTABLEOPEN callout
ORA-29400: data cartridge error
KUP-11010: unable to open at least one dump file for fetch

==> 에러 발생. Datapump Dump 파일 External Table 활용 불가



-- Text 파일(CSV) External Table 생성

SQL> BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_CSV_EXT',
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel.csv',
    format          => json_object('type' value 'csv'),
    column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/

PL/SQL procedure successfully completed.

SQL> select * from channels_csv_ext;

C CHANNEL_DESC	       CHANNEL_CLASS
- -------------------- --------------------
C Catalog	       Indirect
S Direct Sales	       Direct
I Internet	       Indirect
T Tele Sales	       Direct
P Partners	       Others

==> 정상 동작.


-- DBMS_CLOUD.EXPORT_DATA 테스트 : DB 데이터를 Object Storage 로 바로 내리는 방법

--> CSV 포맷으로 내리기

BEGIN  
DBMS_CLOUD.EXPORT_DATA(
  credential_name => 'OBJ_CRED',
  file_uri_list   => 'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel2.csv',
  query           => 'SELECT * FROM CHANNELS',
  format          => JSON_OBJECT('type' value 'csv')
 );
END;
/  

*
ERROR at line 1:
ORA-20000: Export operation is only allowed for Oracle Datapump
ORA-06512: at "C##CLOUD$SERVICE.DBMS_CLOUD", line 1181
ORA-06512: at "C##CLOUD$SERVICE.DBMS_CLOUD", line 3943
ORA-06512: at "C##CLOUD$SERVICE.DBMS_CLOUD", line 3962
ORA-06512: at line 2

==> 에러 발생.

--> Datapump 포맷으로 내리기

BEGIN
DBMS_CLOUD.EXPORT_DATA(
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channel2.dmp',
    format => json_object('type' value 'datapump'),
    query => 'SELECT * FROM channels'
);
END;
/

*
ERROR at line 1:
ORA-20000: ORA-29400: data cartridge error
KUP-06006: The CREDENTIAL access parameter is not supported in populate mode.
ORA-06512: at "C##CLOUD$SERVICE.DBMS_CLOUD", line 1185
ORA-06512: at "C##CLOUD$SERVICE.DBMS_CLOUD", line 3943
ORA-06512: at "C##CLOUD$SERVICE.DBMS_CLOUD", line 3962
ORA-06512: at line 2

==> 에러 발생. 









SQL> BEGIN
DBMS_CLOUD.CREATE_EXTERNAL_TABLE(
    table_name =>'CHANNELS_DMP_EXT',
    credential_name =>'OBJ_CRED',
    file_uri_list =>'https://objectstorage.ap-seoul-1.oraclecloud.com/n/apackrsct01/b/dumpdata/o/channels_xt.dmp',
    format => json_object('type' value 'datapump', 'rejectlimit' value '1'),
    column_list => 'CHANNEL_ID CHAR(1), CHANNEL_DESC VARCHAR2(20), CHANNEL_CLASS VARCHAR2(20)' );
END;
/

