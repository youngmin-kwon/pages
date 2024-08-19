# Oracle Database 23ai - Priority Transactions

- [Oracle Database 23ai - Priority Transactions](#oracle-database-23ai---priority-transactions)
	- [Priority Transactions](#priority-transactions)
	- [Priority Transactions 테스트](#priority-transactions-테스트)

## Priority Transactions

Oracle Database 23ai 의 Priority Transactions 기능은 Row Lock 을 보유한 트랜잭션을 언제, 어떤 트랜잭션을 자동으로 Rollback 할 지 제어하는 기능을 제공합니다.  

23ai 이전의 경우, 애플리케이션이 일부 Row를 수정하여 각 Row 에 대해 Row Lock 을 획득하고 오랫동안 Commit 또는 Rollbck 하지 않으면, Row Lock 을 계속 유지하기 때문에 다른 트랜잭션들은 대기할 수 밖에 없습니다.  Row Lock 을 해제하려면 Lock 을 보유한 세션에서 Commit 또는 Rollback 을 수행해야 합니다.  많은 경우엔 데이터베이스 관리자가 트랜잭션을 분석한 후 `ALTER SYSTEM KILL SESSION` 명령으로 세션을 종료하거나 `ALTER SYSTEM CANCEL SQL` 로 SQL 문을 취소하여 blocking 트랜잭션을 수동으로 종료해야 합니다.

Oracle Database 23ai 의 Priority Transactions 기능은 트랜잭션에 우선 순위를 할당하고 각 우선 순위에 대한 timeout을 설정할 수 있습니다.  데이터베이스는 설정된 timeout 을 초과한 트랜잭션이 더 높은 우선 순위의 트랜잭션을 차단하는 경우 자동으로 낮은 우선 순위 트랜잭션을 롤백하고 Row Lock 을 해제하여 더 높은 우선 순위 트랜잭션이 진행할 수 있도록 합니다.

트랜잭션의 우선 순위는 **LOW, MEDIUM, HIGH(Default)** 중에서 선택할 수 있습니다.  Priority Transactions 기능은 사전 정의된 대기 시간 후에 더 높은 우선 순위 트랜잭션이 Row Lock 을 획득할 수 있도록 낮은 우선 순위의 트랜잭션을 자동으로 Rollback 합니다. 

Priority Transactions 기능은 관리 부담을 줄이는 동시에 더 높은 우선순위 트랜잭션의 트랜잭션 대기 시간과 SLA를 유지하는 데 도움이 됩니다.

Priority Transaction 기능을 위해 두가지 init.ora parameter를 설정해야 합니다. 
- `PRIORITY_TXNS_HIGH_WAIT_TARGET / PRIORITY_TXNS_MEDIUM_WAIT_TARGET`
	- 트랜잭션 우선 순위 별 대기 시간 지정
	- 우선 순위가 높은 트랜잭션이 낮은 우선순위 트랜잭션의 Lock 대기 시간.  대기 시간 이후 낮은 우선 순위의 트랜잭션은 자동 Rollback 됨
	- `ALTER SYSTEM` 권한 필요
	- PDB 에서 수정 가능
- `TXN_PRIORITY`
	- 트랜잭션의 우선순위를 설정하는 파라미터
	- LOW, MEDIUM, HIGH(Default)
	- `ALTER SESSION` 명령으로 지정 

## Priority Transactions 테스트

- 테스트 용 테이블을 생성합니다.
	```sql
	create table if not exists mycheck ( value varchar2(20) );

	insert into mycheck values ( 'Not Updated' );
	commit;
	```

- `PRIORITY_TXNS_HIGH_WAIT_TARGET` 파라미터를 변경하여 High Priority 트랜잭션의 Wait Time 을 10으로 설정
	```sql
	-- 기존 Parameter 값 확인
	show parameter priority_txns

	NAME				     TYPE			       VALUE
	------------------------------------ --------------------------------- ------------------------------
	priority_txns_high_wait_target	     integer			               2147483647
	priority_txns_medium_wait_target     integer			               2147483647
	priority_txns_mode		             string			                   ROLLBACK

	-- PRIORITY_TXNS_HIGH_WAIT_TARGET 변경
	alter system set PRIORITY_TXNS_HIGH_WAIT_TARGET = 10;


	-- 변경된 Parameter 값 확인
	show parameter priority_txns

	NAME				     TYPE			       VALUE
	------------------------------------ --------------------------------- ------------------------------
	priority_txns_high_wait_target	     integer			               10
	priority_txns_medium_wait_target     integer			               2147483647
	priority_txns_mode		             string			                   ROLLBACK
	```

- 테스트를 위해 3개의 세션을 접속합니다.  
	- Session 1 : Transaction Priority 를 LOW 로 설정한 후, Update 를 수행합니다.
	- Session 2 : High Priority (Default).  Update 수행 
	- Session 3 : 조회 용 세선

	```sql
	-- Session 1
	-- txn_priority = low 설정
	alter session set txn_priority = low;

	-- SID 정보 확인
	select sys_context('userenv','SID');

	SYS_CONTEXT('USERENV','SID')
	-------------------------------------------------
	3619

	-- Update 수행
	update mycheck set value = 'Session 1';

	1 row updated.   -- 정상 Update 수행
	```

- 두번째 세션에서 동일한 Update 를 수행합니다.  `txn_priority` 파라미터를 설정하지 않으면 default 로 HIGH priority 로 설정됩니다.

	```sql
	-- Session 2
	set time on
	set timing on

	-- SID 정보 확인
	select sys_context('userenv','SID');

	SYS_CONTEXT('USERENV','SID')
	---------------------------------------------------
	8422

	-- Update 수행
	update mycheck set value = 'Session 2';
	-- 대기
	-- 10 초 후
	1 row updated.

	Elapsed: 00:00:09.68

	-- 데이터 조회
	select * from mycheck;

	VALUE
	------------------------------------------------------------
	Session 2
	```

- 두번째 세션에서 수행한 Update 는 Lock Wait 에 의해 대기하게 됩니다. 하지만, Transaction Priority 가 더 높기 때문에 10초 후 자동으로 Update 가 수행됩니다.

- 두 세션에서 Update 수행 후(10초가 지나기 전에) 세번째 세션에서 Row Lock 상태를 조회합니다.

	```sql
	-- Session 3
	col event format a35

	select txn_priority, priority_txns_wait_target from v$transaction; 

	TXN_PRIORITY	      PRIORITY_TXNS_WAIT_TARGET
	--------------------- -------------------------
	LOW					  0


	select sid, event, seconds_in_wait, blocking_session
	from v$session where event like '%enq%';

		SID EVENT			                   SECONDS_IN_WAIT BLOCKING_SESSION
	---------- ----------------------------------- --------------- ----------------
		8422 enq: TX - row lock (HIGH priority)  7		       3619

	SQL> /

		SID EVENT			                   SECONDS_IN_WAIT BLOCKING_SESSION
	---------- ----------------------------------- --------------- ----------------
		8422 enq: TX - row lock (HIGH priority)  9		       3619

	SQL> /

	no rows selected

	-- 10 초 후 Row Lock Wait 가 사라짐.
	```

- 10초 이후 Session 1 에서 테이블을 조회해봅니다.  
	Low Priority 트랜잭션은 자동으로 Rollback 되지만 해당 세션이 종료되지는 않습니다.  트랜잭션이 자동으로 Rollback 되면 세션에서 현재 실행 중인 SQL 또는 다음 SQL 문은 `ORA-63300` 에러가 발생합니다.   이 경우 사용자가 명시적으로 Rollbck 을 수행해야 합니다.
	
	```sql
	-- Session 1
	select * from mycheck;
	*
	ERROR at line 1:
	ORA-63302: Transaction must roll back
	ORA-63300: Transaction is automatically rolled back since it is blocking a higher priority transaction from another session.
	Help: https://docs.oracle.com/error-help/db/ora-63302/

	-- Rollback 수행
	rollback;
	```
