# Machine Learning on Autonomous Database
- https://apexapps.oracle.com/pls/apex/r/dbpm/livelabs/view-workshop?wid=560&clear=RR,180&session=100695200995843



## 환경 구성

### Task 1: Provision an Oracle Autonomous Database

### Task 2: Create an Oracle Machine Learning user

- Oracle Database -> Autonomous Data Warehouse
- Database actions
- Administration: DATABASE USERS
- Create User
	+ User Name : OMLUSER
	+ Password  : WElcome123___
	+ Graph, OML, Web Access 
	
	```
	-- USER SQL
	CREATE USER OMLUSER IDENTIFIED BY WElcome123___;

	-- ADD ROLES
	GRANT CONNECT TO OMLUSER;
	GRANT CONSOLE_DEVELOPER TO OMLUSER;
	GRANT DWROLE TO OMLUSER;
	GRANT GRAPH_DEVELOPER TO OMLUSER;
	GRANT OML_DEVELOPER TO OMLUSER;
	GRANT RESOURCE TO OMLUSER;
	ALTER USER OMLUSER DEFAULT ROLE CONNECT,CONSOLE_DEVELOPER,DWROLE,GRAPH_DEVELOPER,OML_DEVELOPER,RESOURCE;

	-- ENABLE REST
	BEGIN
	    ORDS.ENABLE_SCHEMA(
	        p_enabled => TRUE,
	        p_schema => 'OMLUSER',
	        p_url_mapping_type => 'BASE_PATH',
	        p_url_mapping_pattern => 'omluser',
	        p_auto_rest_auth=> TRUE
	    );
	    commit;
	END;
	/

	-- ENABLE GRAPH
	ALTER USER OMLUSER GRANT CONNECT THROUGH GRAPH$PROXY_USER;

	-- ENABLE OML
	ALTER USER OMLUSER GRANT CONNECT THROUGH OML$PROXY;
	```


### Task 3: Sign into Oracle Machine Learning

- Database actions -> ORACLE MACHINE LEARNING
	+ ORACLE MACHINE LEARNING 페이지가 로딩되지 않을 때가 있음.  해결 방법은? -> 브라우저 인터넷 사용 기록(캐시) 삭제
- Username / Password : OMLUSER / WElcome123___

### Task 4: Create the CUSTOMERS360 table

you will create the table CUSTOMERS60 by using the Scratchpad. The Scratchpad is available on the Oracle Machine Learning Notebooks home page. The Scratchpad is a one-click access to a notebook for running SQL statements, PL/SQL scripts, and Python scripts. After you run your scripts, the Scratchpad is automatically saved as a notebook by the default name Scratchpad in the Notebooks page. You can access it later in the Notebooks page. You will learn more about notebooks in Lab 1.

- Quick Actions -> Scratchpad
- 테이블 생성 스크립트 입력 후 실행
```
%sql
CREATE TABLE CUSTOMERS360 AS
          (SELECT a.CUST_ID, a.CUST_GENDER, a.CUST_MARITAL_STATUS,
             a.CUST_YEAR_OF_BIRTH, a.CUST_INCOME_LEVEL, a.CUST_CREDIT_LIMIT,
             b.EDUCATION, b.AFFINITY_CARD,
             b.HOUSEHOLD_SIZE, b.OCCUPATION, b.YRS_RESIDENCE, b.Y_BOX_GAMES
       FROM SH.CUSTOMERS a, SH.SUPPLEMENTARY_DEMOGRAPHICS b
       WHERE a.CUST_ID = b.CUST_ID);
```

- 데이터 확인
```
%sql
select * from customers360
fetch first 10 rows only;
```

## Lab 1: Introduction to Oracle Machine Learning Notebooks

This lab walks you through the steps to sign into Oracle Machine Learning, create an Oracle Machine Learning (OML) notebook from scratch, create an OML notebook based on the example template notebooks, and create jobs to schedule notebooks to run at specific day and time.

Estimated Time: 15 minutes

- About Oracle Machine Learning Notebooks  
	Oracle Machine Learning Notebooks is a collaborative user interface supporting data scientists, analysts, developers, and DBAs. You can work with SQL, PL/SQL, Python, and R in the same notebook—using the most appropriate language for the problem at hand. You can also view notebook changes by team members in real time, interactively. Data science team members can explicitly share notebooks and version notebooks as well as schedule notebooks to run at a set time or a repeating schedule. By virtue of being included in Oracle Autonomous Database, machine learning functionality is automatically provisioned and managed. Through Oracle Machine Learning Notebooks, you have access to the in-database algorithms and analytics functions to explore and prepare data, build and evaluate models, score data, and deploy solutions.

### Task 1: Sign into Oracle Machine Learning User Interface

- OCI Console -> Oracle Database -> Autonomous Data Warehouse -> 생성한 ADW 선택
- Database actions -> Oracle Machine Learning ->  Sign in Page : 
	+ Username / Password : OMLUSER / WElcome123___
	


### Task 2: Create a Notebook and define paragraphs using the md, SQL, PL/SQL, Python, and R interpreters

- Create Notebooks : Quick Actions -> Notebooks
- Create
	+ Name : Test Notebook

**Call the Markdown interpreter and generate static html from Markdown plain text**
	
- Markdown paragraph -> Run
```
%md
# Introduction to Oracle Machine Learning Notebooks
1. Task 1: Sign into Oracle Machine Learning User Interface
2. Task 2: Task 2: Create a Notebook and define paragraphs using the md, SQL, PL/SQL, Python, and R interpreters
```

**Call the SQL Interpreter and run SQL Statements**

- SQL statements 테스트
```
%sql
select table_name from user_tables;
```

```
%sql
select count(*) from customers360;
```

**Call the PL/SQL Interpreter and run PL/SQL Scripts**

- PLSQL scripts 수행 테스트
```
%script
CREATE TABLE small_table
(
NAME VARCHAR(200),
ID1 INTEGER,
ID2 VARCHAR(200),
ID3 VARCHAR(200),
ID4 VARCHAR(200),
TEXT VARCHAR(200)
);

BEGIN
FOR i IN 1..100 LOOP
INSERT INTO small_table VALUES ('Name_'||i, i,'ID2_'||i,'ID3_'||i,'ID4_'||i,'TEXT_'||i);
END LOOP;
COMMIT;
END;
```

```
%script
select * from small_table;
```

**Call the Python Interpreter and run Python Statements**

- Python 테스트

```
%python

import pandas as pd
import oml

DATA = oml.sync(table = "SUPPLEMENTARY_DEMOGRAPHICS", schema = "SH")
z.show(DATA.head())

CUST = oml.sync( table = "CUSTOMERS360", schema="OMLUSER" )
z.show( CUST.head() )
```

**Call the R Interpreter and run R Statements**

- R Scripts 테스트

```
%r

ore.is.connected()

```

- Import R libraries
```
library(ORE)
```

- To create a database table from a `data.frame` and get proxy object
```
ore.create( iris, table = "IRIS" )
cat( "Class: ", class(IRIS) )
cat( "\nShape: ", dim(IRIS) )
```

- To list the available data.frame proxy objects, run the following command:
```
ore.ls()
```


### Task 3: Create a Notebook using a Template Example

**Create an OML4Py Notebook using the Classification DT Template Example**

This step demonstrates how to create the OML4Py Classification notebook based on the OML4Py Classification DT (Decision Tree) Example template:

- hamburger icon -> Templates -> Examples
- or Oracle Machine Learning Home page -> Quick Actions -> Examples
- **OML4Py Classification DT** 선택 -> 왼쪽 위의 `Create Notebook` 선택
- `Open Notebook` 클릭

**Create a Time Series Notebook using the OML4SQL Time Series Template Example**

These steps demonstrate how to create the Time Series notebook based on the Example template:

- hamburget -> Templates -> Examples
- **OML4SQL Time Series ESM** 선택 -> 왼쪽 위의 `Create Notebook` 선택
- `Open Notebook` 클릭


### Task 4: Change Interpreter Bindings Order
### Task 5: Create Jobs to Schedule Notebook Run


## Lab 2: Introduction to Oracle Machine Learning for SQL (OML4SQL)

### Introduction

### Task 1: Examine the Data


### Task 2: Prepare the Data

```
%script

CREATE OR REPLACE VIEW ESM_SH_DATA AS
SELECT TIME_ID, AMOUNT_SOLD FROM SH.SALES;
```

```
%script

SELECT count(*) from ESM_SH_DATA;
```

```
%sql
SELECT * from ESM_SH_DATA
WHERE rownum <11;
```


### Task 3: Build Your Model

```
%script

BEGIN DBMS_DATA_MINING.DROP_MODEL('ESM_SALES_FORECAST_1');
EXCEPTION WHEN OTHERS THEN NULL; END;
/
DECLARE
      v_setlst DBMS_DATA_MINING.SETTING_LIST;
BEGIN
     
     -- algorithm = exponential smoothing
     v_setlst('ALGO_NAME')            := 'ALGO_EXPONENTIAL_SMOOTHING';

         -- accumulation interval = quarter
         v_setlst('EXSM_INTERVAL')        := 'EXSM_INTERVAL_QTR';

         -- prediction step = 4 quarters
         v_setlst('EXSM_PREDICTION_STEP') := '4';                 

         -- ESM model = Holt-Winters
         v_setlst('EXSM_MODEL')           := 'EXSM_WINTERS';      

         -- seasonal cycle = 4 quarters
         v_setlst('EXSM_SEASONALITY')     := '4';                 

     
     DBMS_DATA_MINING.CREATE_MODEL2(
        MODEL_NAME           => 'ESM_SALES_FORECAST_1',
        MINING_FUNCTION      => 'TIME_SERIES',
        DATA_QUERY           => 'select * from ESM_SH_DATA',
        SET_LIST             => v_setlst,
        CASE_ID_COLUMN_NAME  => 'TIME_ID',
        TARGET_COLUMN_NAME   =>'AMOUNT_SOLD');
END;
```

Examine the script:

- v_setlist is a variable to store SETTING_LIST.
- SETTING_LIST specifies model settings or hyperparameters for the model.
- DBMS_DATA_MINING is the PL/SQL package used for machine learning. These settings are described in DBMS_DATA_MINING — Algorithm Settings: Exponential Smoothing.
- ALGO_NAME specifies the algorithm name. Since you are using Exponential Smoothing as the algorithm, the value of the setting is ALGO_EXPONENTIAL_SMOOTHING.
- EXSM_INTERVAL indicates the interval of the data set or a unit of interval size. For example, day, week, month, and so on. You want to predict for quarterly sales. 
  Hence, the setting is EXSM_INTERVAL_QTR. This setting applies 	- only to the time column with datetime type.
- EXSM_PREDICTION_STEP specifies how many predictions to make. You want to display each value representing a quarter. Hence, a value of 4 gives four values ahead prediction.
- EXSM_MODEL specifies the type of exponential smoothing model to be used. Here the value is EXSM_HW. 
  The Holt-Winters triple exponential smoothing model with additive trend and multiplicative seasonality is applied. 
  This type of model considers various combinations of additive and multiplicative trend, seasonality and error, with and without trend damping. 
  Other options are `EXSM_SIMPLE, EXSM_SIMPLE_MULT, EXSM_HOLT, EXSM_HOLT_DMP, EXSM_MUL_TRND, EXSM_MULTRD_DMP, EXSM_SEAS_ADD, EXSM_SEAS_MUL, EXSM_HW, EXSM_HW_DMP, EXSM_HW_ADDSEA, EXSM_DHW_ADDSEA, EXSM_HWMT, EXSM_HWMT_DMP.`
- EXSM_SEASONALITY indicates how long a season lasts. The parameter specifies a positive integer value as the length of seasonal cycle. The value it takes must be larger than 1. 
  For example, 4 means that every group of four - values forms a seasonal cycle.
- EXSM_SETMISSING specifies how to handle missing values. Time series data can contain missing values. 
  The special value EXSM_MISS_AUTO indicates that, if the series contains missing values it is to be treated as an irregular time series. 
  The Automatic Data Preparation (ADP) setting does not impact this data for time series.

### Task 4: Evaluate Your Model

- 다음 SQL 문을 수행하여 Model Setting 정보 review 가능

```
%sql

SELECT SETTING_NAME, SETTING_VALUE
FROM USER_MINING_MODEL_SETTINGS
WHERE MODEL_NAME = UPPER('ESM_SALES_FORECAST_1')
ORDER BY SETTING_NAME;
```

- To view the model diagnostic view, DM$VG, and evaluate the model, run the following query:

```
%sql

SELECT NAME, round(NUMERIC_VALUE,4), STRING_VALUE
FROM DM$VGESM_SALES_FORECAST_1
ORDER BY NAME;
```

The DM$VG view for time series contains the global information of the model along with the estimated smoothing constants, the estimated initial state, and global diagnostic measures.

- NAME: Indicates the diagnostic attribute name.
- NUMERIC_VALUE: Indicates the calculated statistical value for the model.
- STRING_VALUE: Indicates alphanumeric values for the diagnostic parameter.  
  A few parameters to note for an exponential smoothing algorithm are:
	+ ALPHA: Indicates the smoothing constant.
	+ BETA: Indicates the trend smoothing constant.
	+ GAMMA: Indicates the seasonal smoothing constant.
	+ MAE: Indicates Mean Absolute Error.
	+ MSE: Indicates Mean Square Error.

In exponential smoothing, a series extends infinitely into the past, but that influence of past on future decays smoothly and exponentially fast. 
The smooth rate of decay is expressed by one or more smoothing constants. The smoothing constants are parameters that the model estimates. 
These smoothing constants are represented as α, β, and γ. 
Values of a smoothing constant near one put almost all weight on the most recent observations. 
Values of a smoothing constant near zero allow the distant past observations to have a large influence.

Note that α is associated with the error or noise of the series, β is associated with the trend, and γ is associated with the seasonality factors.

### Task 5: Access Forecasts from Your Model

For a time series model, you use the DM$VP view to retrieve the forecasts for the requested time periods.

- Query the `DM$VP` model detail view to see the forecast (sales for four quarters).  
  The `DM$VP` view for time series contains the result of an ESM model. 
  The output has a set of records such as partition, CASE_ID, value, prediction, lower, upper, and so on and ordered by partition and CASE_ID (time).  
  Run the following statement:

```
%sql

SELECT TO_CHAR(CASE_ID,'YYYY-MON') DATE_ID,
       round(VALUE,2) ACTUAL_SOLD,
       round(PREDICTION,2) FORECAST_SOLD,
       round(LOWER,2) LOWER_BOUND, round(UPPER,2) UPPER_BOUND
FROM DM$VPESM_SALES_FORECAST_1
ORDER BY CASE_ID DESC;
```

  In this step, the forecast shows the amount sold along with the case_id. The forecasts display upper and lower confidence bounds showing that the estimates can vary between those values.

  Examine the statement:

  - TO_CHAR(CASE_ID,'YYYY-MON') DATE_ID: The DATE_ID column has timestamp or case_id extracted in year-month (yyyy-mon) format.
  - round(VALUE,2) ACTUAL_SOLD: Specifies the AMOUNT_SOLD value as ACTUAL_SOLD rounded to two decimal places.
  - round(PREDICTION,2) FORECAST_SOLD: Specifies the predicted value as FORECAST_SOLD rounded to two decimal places.
  - round(LOWER,2) LOWER_BOUND, round(UPPER,2) UPPER_BOUND: Specifies the lower and upper confidence levels rounded to two decimal places.

## Lab 3: Introduction to Oracle Machine Learning for Python (OML4Py)

### Introduction

This lab walks you through the steps to create a database table, create a proxy object, explore and prepare data, build and evaluate models, and use those models to score data using OML4Py. 
This will use a classification example available in OML Notebooks. For illustrative purposes, Task 1 and Task 2 of this lab use iris data set from sklearn datasets to create a database 
table. The rest of the steps take you through the example that uses the SH schema and is available in OML Notebooks. In Oracle Autonomous Database (ADB), the SH schema and associated data 
sets are easily accessible.

**About Oracle Machine Learning for Python(OML4Py)**

Oracle Machine Learning for Python (OML4Py) is a component of Oracle Autonomous (ADB), which includes Oracle Autonomous Data Warehouse (ADW), Oracle Autonomous Transaction Processing (ATP), 
and Oracle Autonomous JSON Database (AJD). OML4Py is also included with on-premise Oracle Database and Database Cloud Service with separate installation. By using OML Notebooks, you can use 
standard Python syntax and overloaded Python functions, use a natural Python API to load in-database machine learning algorithms, call user-defined Python functions in database-spawned and 
controlled Python engines, and leverage automated machine learning (AutoML).


### Task 1: Create a Database Table






### Task 2: Create a Persistent Database Table
### Task 3: Create a Proxy Object for a Database Object
### Task 4: Explore the Data
### Task 5: Prepare the Data
### Task 6: Build Your Model
### Task 7: Evaluate Your Model
### Task 8: Score Data for Deployment Using Your Model
### Task 9: Use the SQL Interface to Score Data and Display Prediction Details
### Task 10: Save and Load Python Objects in a Datastore Instance
### 

## Lab 4: Introduction to Oracle Machine Learning for R (OML4R)


## Lab 5: Introduction to Oracle Machine Learning AutoML UI

### Introduction

Oracle Machine Learning AutoML UI (OML AutoML UI) is a no-code user interface supporting automated machine learning for both data scientist productivity and non-expert user access to powerful in-database algorithms. Like the OML4Py AutoML API, it accelerates machine learning projects by giving quick feedback on data set suitability for producing useful models – alleviating much of the drudgery of the machine learning process. Oracle Machine Learning AutoML UI automates model building with minimal user input – you just have to specify the data and the target in what’s called an experiment and the tool does the rest. However, you can adjust some settings, such as the number of top models to select, the model selection metric, and even specific algorithms. With a few clicks, you can generate editable starter notebooks. These notebooks contain data selection, building the selected model – including the settings used to produce that model – and scoring and evaluation code – all in Python using OML4Py. You can build on this generated notebook to apply your own domain expertise to augment the solution. Similarly, you can deploy models from OML AutoML UI as REST endpoints to Oracle Machine Learning (OML) Services in just a few clicks.

Oracle Machine Learning AutoML UI(OML AutoML UI)는 데이터 과학자 뿐만 아니라 비전문가 사용자가 강력한 데이터베이스 내 알고리즘을 사용하여 자동화된 기계 학습을 지원하는 코드 없는 사용자 인터페이스입니다. 
OML4Py AutoML API와 마찬가지로 유용한 모델을 생성하기 위한 데이터 세트 적합성에 대한 빠른 피드백을 제공하여 기계 학습 프로세스의 고된 작업을 상당 부분 완화함으로써 기계 학습 프로젝트를 빠르게 수행할 수 있습니다.
Oracle Machine Learning AutoML UI는 최소한의 사용자 입력으로 모델 구축을 자동화합니다. 실험(Experiment) 이라는 항목에서 데이터와 대상을 지정하기만 하면 나머지는 도구가 알아서 처리합니다. 
그러나 선택할 상위 모델 수, 모델 선택 메트릭 및 특정 알고리즘과 같은 일부 설정을 조정할 수 있습니다. 
몇 번의 클릭만으로 편집 가능한 스타터 노트북을 생성할 수 있습니다. 이 노트북에는 데이터 선택, 해당 모델을 생성하는 데 사용되는 설정을 포함하여 선택한 모델 빌드, 채점 및 평가 코드가 모두 OML4Py를 사용하여 Python으로 포함되어 있습니다. 
이 생성된 노트북을 기반으로 고유한 도메인 전문 지식을 적용하여 솔루션을 보강할 수 있습니다. 마찬가지로 몇 번의 클릭만으로 OML AutoML UI의 모델을 REST 끝점으로 OML(Oracle Machine Learning) 서비스에 배포할 수 있습니다.

### Task 1: Access Oracle Machine Learning AutoML UI

- Oracle Cloud ADB Detail Page -> Database Actions -> Oracle Machine Learning
- Sign into Oracle Machine Learning user interface.
- Quick Actions -> **AutoML** 선택  
  or Hambuger icon -> AutoML Experiments

### Task 2: Create an Experiment

An Experiment is as a work unit that contains the definition of data source, prediction target, and prediction type along with optional settings. After an Experiment runs successfully, it presents you a list of machine learning models in the leader board. You can select any model for deployment, or use it to create a notebook based on the selected model. When creating an Experiment, you must define the data source and the target of the experiment.

When creating an experiment, you must specify the data source and the target of the experiment. An AutoML experiment can process columns with data type VARCHAR2 greater than 4K , CHAR, CLOB, BLOB, and BFILE as text. Note that columns with data type VARCHAR2 less than 4K are considered as categorical.

Experiments는 선택적 설정과 함께 데이터 소스, 예측 대상 및 예측 유형의 정의를 포함하는 작업 단위입니다. Experiments가 성공적으로 실행되면 리더 보드에 기계 학습 모델 목록이 표시됩니다. 
배포할 모델을 선택하거나 선택한 모델을 기반으로 노트북을 만드는 데 사용할 수 있습니다. 

Experiments를 생성할 때 데이터 소스와 실험 대상을 지정해야 합니다. AutoML Experiments는 데이터 유형이 4K보다 큰 VARCHAR2, CHAR, CLOB, BLOB 및 BFILE이 있는 열 또한 텍스트로 처리할 수 있습니다. 
VARCHAR2 데이터 유형이 4K 미만인 열은 범주형으로 간주됩니다.

- AutoML Experiments 페이지 -> Create
- 정보 입력
  - Name : Customers360
  - Data Source : OMLUSER.CUSTOMERS360 테이블 
- Predict : AFFINITY_CARD - target 컬럼 지정 
- Predict Type : Classification 자동 선택   
    지원하는 Prediction Type:
    + Classification: For non-numeric data type, Classification is selected by default. Classification may be selected for small cardinality numeric data as well.
    + Regression: For numeric data type, Regression is selected by default.
- Case ID : CUST_ID 
- Additional Settings: 추가적인 설정 지정 가능
  - Maximum Top Models :
  - Maximum Run Duration (Hours) :
  - Database Service Level : Low / Medium / High
  - Model Metric : 
- 오른쪽 위쪽 : Start -> Faster Results 

The Leader Board displays the top performing models relative to the model metric selected along with the algorithm and accuracy. Here, you will view the additional metrics Precision, Recall, ROC AUC for the models:

- Metric dialog 에서 Metric 정보 추가 가능
- 원하는 모델 클릭 (row 클릭, Model Name 컬럼 X) - Random Forest -> Rename -> RF_Customers360

### Task 3: Deploy Top Model to Oracle Machine Learning Services

When you deploy a model using the Oracle Machine Learning AutoML UI, you create an Oracle Machine Learning Services endpoint for scoring. Oracle Machine Learning Services extends Oracle Machine Learning functionality to support model deployment and model lifecycle management for in-database OML models through REST APIs.

Oracle Machine Learning AutoML UI를 사용하여 모델을 배포할 때 Oracle Machine Learning Services Endpoint를 생성합니다. 
Oracle Machine Learning Services는 Oracle Machine Learning 기능을 확장하여 REST API를 통해 데이터베이스 내 OML 모델에 대한 모델 배포 및 모델 수명 주기 관리를 지원합니다.

- "Balanced Accuracy" 가장 높은 모델 선택 - Random Forest 모델 선택
- Deploy
- Deploy Model Page
  - Name : RF_CUSTOMERS360
  - URI : rf_customers360
  - Version : 1.0
  - Namespace : DEMO
  - Shared checkbox 선택 





### Task 4: View Oracle Machine Learning Models with Deployed Metadata and REST Endpoint

The deployed models are listed under Deployments on the Models page. 
To view the metadata of the deployed model RF_CUSTOMERS360:

- Hamburger menu -> Models -> Deployments
- RF_CUSTOMERS360 선택하여 Model Metadata 확인 가능
- REST Endpoint의 전체 JSON을 보려면 URI rf_customers360 을 클릭 

### Task 5: Create a Notebook for the Top Model

You can create notebooks based on the top models produced in the experiment. 
This provides the code to build a model with the same settings for the selected model. 
This option is helpful if you want to use the code to re-create a similar machine learning model.  
To create a notebook:

- Hamburger menu -> AutoML Experiments -> Customers360 
- RF_CUSTOMERS360 선택 -> Create Notebook -> 이름 지정 
- Open Notebook 
- OML4Py code 로 Notebook 생성 확인 

### Task 6: View Generated Notebook and Individual Paragraphs

## Lab 6: Introduction to Oracle Machine Learning Services

### Introduction

OML Services extends OML functionality to support model deployment and model lifecycle management for both in-database OML models and third-party Open Neural Networks Exchange (ONNX) machine learning models via REST APIs. These third-party classification, regression or clustering models can be built using tools that support the ONNX format, which includes packages like Scikit-learn and TensorFlow, among several others.

Oracle Machine Learning Services provides REST endpoints through the Oracle Autonomous Database environment. These endpoints enable the storage of machine learning models along with their metadata, the creation of scoring endpoints for the model, and producing scores using these endpoints.

OML 서비스는 OML 기능을 확장하여 REST API를 통해 데이터베이스 내 OML 모델과 타사 ONNX(Open Neural Networks Exchange) 기계 학습 모델 모두에 대한 모델 배포 및 모델 수명 주기 관리를 지원합니다. 
이러한 타사 분류, 회귀 또는 클러스터링 모델은 Scikit-learn 및 TensorFlow와 같은 패키지를 포함하는 ONNX 형식을 지원하는 도구를 사용하여 구축할 수 있습니다.

Oracle Machine Learning Services는 Oracle Autonomous Database 환경을 통해 REST 엔드포인트를 제공합니다. 
이러한 엔드포인트를 사용하면 메타데이터와 함께 기계 학습 모델을 저장하고 모델에 대한 스코어링 엔드포인트를 생성하고 이러한 엔드포인트를 사용하여 스코어를 생성할 수 있습니다.










