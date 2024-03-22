# Lab 1. Autonomous Database 프로비저닝

- [Lab 1. Autonomous Database 프로비저닝](#lab-1-autonomous-database-프로비저닝)
  - [개요](#개요)
  - [Task 1: Choosing ADW or ATP from the Services Menu](#task-1-choosing-adw-or-atp-from-the-services-menu)
  - [Task 2: Creating the ADB Instance](#task-2-creating-the-adb-instance)

## 개요

이 실습에서는 Oracle Cloud에서 Oracle Autonomous Database (ADW(Autonomous Data Warehouse) 및 ATP(Autonomous Transaction Processing))를 사용하여 시작하는 단계를 안내합니다. 이 실습에서는 새로운 ADW 인스턴스를 프로비저닝합니다.

_참고: 이 실습에서는 ADW를 사용하지만 ATP 데이터베이스 생성 단계는 동일합니다._

예상 소요 시간: 5분

## Task 1: Choosing ADW or ATP from the Services Menu

1. Oracle Cloud에 로그인
    
2. 클라우드 서비스 대시보드로 이동. 최상위 Navigation 메뉴를 표시하려면 왼쪽 상단의 Navigation 메뉴 클릭
    
    **참고:** 대시보드의 **빠른 작업** 섹션에서 Autonomous Data Warehouse 또는 Autonomous Transaction Processing 서비스에 직접 액세스 가능
    
    [![Oracle home page.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/Picture100-36.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/Picture100-36.png)
    
3. **Autonomous Data Warehouse** 클릭
    
    [![Click Autonomous Data Warehouse.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/database-adw.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/database-adw.png)
    
4. 왼쪽 Compartment Drop Down 메뉴에서 Compartment 선택 및 Workload Type 이 Data Warehouse 인지 확인. 오른쪽 상단의 Region 확인
	- **Compartment : apackrsct(root) - oci-bootcamp
	- **Region **: 
		- **oci-bootcamp-user01 ~ oci-bootcamp-user08 : South Korea North(Chuncheon)**
		- **oci-bootcamp-user09 ~ oci-bootcamp-user15 : Japan East(Tokyo)**
    
    [![Autonomous Databases console.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/no-adb-freetier.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/no-adb-freetier.png)
    

## Task 2: Creating the ADB Instance

1. **Create Autonomous Database**를 클릭하여 인스턴스 생성 작업 시작
    
    [![Click Create Autonomous Database.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/Picture100-23.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/Picture100-23.png)
    
2. Autonomous Database 인스턴스 구성 정보 입력 페이지에서 기본 정보 입력
    
    - **Choose a compartment : apackrsct(root) / oci-bootcamp**
    - **Display Name** : **ADW01 ~ ADW15** (ADW + 사용자번호) 
    - **Database Name** : **ADW01 ~ ADW15** (ADW + 사용자번호)
3. 워크로드 유형 : **Data Warehouse** 선택
    
4. Deployment 유형 : **Serverless** 선택
    
    [![Enter the required details.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/task2-4-1.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/task2-4-1.png)
    
5. 데이터베이스 구성 정보:
    
    - **Choose database version** - **19c** 선택
    - **ECPU count** - **2 CPUs** 선택
    - **Storage (TB)** - **1 TB** 선택
    - **Auto Scaling** - **Compute Auto Scaling** 선택
    
    [![Choose the remaining parameters.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/task2-6.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/task2-6.png)
    
6. 관리자 Credentials 설정 - ADMIN 사용자 암호 설정:
    
    - **Password and Confirm Password** - 암호 생성 규칙에 맞는 암호 설정
	    - **WElcome12345__**
    
    [![Enter password and confirm password.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/Picture100-26d.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/Picture100-26d.png)
    
7. 네트워크 액세스 선택 : "Secure access from everywhere." 선택
    
8. 라이센스 유형 선택 : **License Included** 선택
    
9. **Create Autonomous Database** 클릭
    
    [![Click Create Autonomous Database.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/Picture100-27.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/Picture100-27.png)
    
10. 인스턴스 프로비저닝 작업은 몇 분 후 완료됨.  
    프로비저닝 완료 후 이름, 데이터베이스 버전, ECPU 수, 스토리지 크기를 포함한 인스턴스 세부 정보 확인 [![Database instance homepage.](https://github.com/youngmin-kwon/pages/raw/main/adb/datastudio/images/Picture100-32.png)](https://github.com/youngmin-kwon/pages/blob/main/adb/datastudio/images/Picture100-32.png)
    