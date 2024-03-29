# 데이터 분석 시나리오

- [데이터 분석 시나리오](#데이터-분석-시나리오)
  - [개요](#개요)
  - [Task 1: 데이터 분석 프로젝트 개요](#task-1-데이터-분석-프로젝트-개요)


## 개요

영화 스트리밍 비즈니스 회사의 매출 데이터 분석을 위한 요구사항과 시나리오를 간략히 살펴보겠습니다. 

예상 소요 시간: 5분


## Task 1: 데이터 분석 프로젝트 개요

1. 요구사항 

   - 마케팅 팀에서는 가치가 높은 고객에게 할인 제안 프로모션 수행 예정
   - 고객의 영화 구매 횟수를 기준으로 5개의 고객 등급 분류
   - 다양한 연령대와 결혼 여부 그룹에 따른 영화 장르 선호도 조사 
   - 고객 등급별 영화 장르 선호도 조사 

2. 필요 데이터 정보

   - 기존 데이터베이스에 고객 및 영화 장르 등의 정보를 가진 테이블 존재 
   - 판매 데이터는 매일 Object Storage Bucket 에 로드됨

3. Action Items

   - **관련 데이터 찾기:**  
        이 분석에 필요한 MOVIESALES_CA 및 기타 테이블 확인.  
        데이터 탐색 및 검색은 **Catalog** 도구 사용 가능

   - **데이터 로딩**  
        Object Storage Bucket 의 MOVIESALES_CA 테이블 로딩  
        Local PC 의 Excel 파일 로딩

   - **데이터 변환 및 분석 데이터 준비**  
        영화 판매량을 집계하고 5분위수 값이 포함되도록 데이터 변환  
        데이터 분석을 위해 분석 테이블에 분석에 필요한 비정규화된 컬럼 추가.  분석 테이블에는 분석에 필요한 모든 속성이 있어야 합니다.  
        **Data Transform** 도구를 사용

   - **분석 작업**   
        데이터 분석을 위한 차원 모델 생성.  
        **Data Analysis** 도구를 이용하여 연령대, 장르, 결혼 여부 등 다양한 차원으로 매출 데이터를 분석할 수 있는 분석 뷰를 생성

   - **Data Insights**  
        **Data Insights** 도구를 사용하여 데이터에 숨겨진 다양한 패턴 자동 검색 


**자 그럼 분석 작업을 시작해 보겠습니다.**
