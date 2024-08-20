# Oracle Database 23ai : Aggregation over INTERVAL Data Types

Oracle Database 23ai 에서는 INTERVAL 데이터타입에 대해 SUM, AVG 함수를 사용할 수 있습니다. 23ai 이전 버전에서는 MIN, MAX 함수만 사용할 수 있었습니다.

INTERVAL 데이터 타입은 다음 두가지가 있습니다:
- INTERVAL YEAR TO MONTH — 년/월 internval 저장
- INTERVAL DAY TO SECOND — 일/시간/분/초(소수점포함) interval 저장

간단한 테스트를 통해 기능을 확인해 보겠습니다.

- 테스트 테이블 생성 및 데이터 입력
    ```sql
    drop table if exists interval_test purge;

    create table interval_test (
    id          number,
    start_time  timestamp,
    end_time    timestamp,
    duration    interval day to second generated always as (end_time - start_time) virtual
    );

    insert into interval_test (id, start_time, end_time) values    
        (1, timestamp '2024-04-10 08:45:00.0', timestamp '2024-04-10 18:01:00.0'),
        (2, timestamp '2024-04-11 09:00:00.0', timestamp '2024-04-11 17:00:00.0'),
        (3, timestamp '2024-04-12 08:00:00.0', timestamp '2024-04-12 17:45:00.0'),
        (4, timestamp '2024-04-13 07:00:00.0', timestamp '2024-04-13 16:00:00.0');
    commit;

    alter session set nls_timestamp_format = 'yyyy-mm-dd hh24:mi:ss';

    select * from interval_test;
            ID START_TIME           END_TIME             DURATION
    ---------- -------------------- -------------------- --------------------
            1 2024-04-10 08:45:00  2024-04-10 18:01:00  +00 09:16:00.000000
            2 2024-04-11 09:00:00  2024-04-11 17:00:00  +00 08:00:00.000000
            3 2024-04-12 08:00:00  2024-04-12 17:45:00  +00 09:45:00.000000
            4 2024-04-13 07:00:00  2024-04-13 16:00:00  +00 09:00:00.000000
    ```

- INTERVAL 데이터 타입 컬럼에 대해 Aggregation 함수 테스트
    ```sql
    select min(duration),
           max(duration),
           sum(duration),
           avg(duration)
    from interval_test;

    MIN(DURATION)        MAX(DURATION)        SUM(DURATION)                  AVG(DURATION)
    -------------------- -------------------- ------------------------------ ------------------------------
    +00 08:00:00.000000  +00 09:45:00.000000  +000000001 12:01:00.000000000  +000000000 09:00:15.000000000
    ```

- 23ai 이전 버전에서는 SUM, AVG 함수 사용 시 다음과 같은 에러가 발생합니다.
    ```sql
    select sum(duration) from interval_test;
           *
    ERROR at line 1:
    ORA-00932: inconsistent datatypes: expected NUMBER got INTERVAL DAY TO SECOND


    select avg(duration) from interval_test;
            *
    ERROR at line 1:
    ORA-00932: inconsistent datatypes: expected NUMBER got INTERVAL DAY TO SECOND
    ```

- INTERVAL 데이터 타입 컬럼에 대한 Analytic 함수 테스트
    ```sql
    set linesize 120
    column sum_duration format a30
    column avg_duration format a30
    select id,
        duration,
        sum(duration) over () as sum_duration,
        avg(duration) over () as avg_duration
    from interval_test;

            ID DURATION             SUM_DURATION                   AVG_DURATION
    ---------- -------------------- ------------------------------ ------------------------------
            1 +00 09:16:00.000000  +000000001 12:01:00.000000000  +000000000 09:00:15.000000000
            2 +00 08:00:00.000000  +000000001 12:01:00.000000000  +000000000 09:00:15.000000000
            3 +00 09:45:00.000000  +000000001 12:01:00.000000000  +000000000 09:00:15.000000000
            4 +00 09:00:00.000000  +000000001 12:01:00.000000000  +000000000 09:00:15.000000000
    ```

Oracle Database 23ai 이전 버전에서는 INTERVAL 데이터 타입에 대한 SUM, AVG 계산을 위해 interval 정보를 초(Seconds)로 변환한 후 연산 작업을 수행해야 하는 복잡한 과정이 있었지만, 23ai 에서는 SUM, AVG 함수를 바로 적용 가능하여 훨씬 간편하게 연산을 수행할 수 있습니다.
