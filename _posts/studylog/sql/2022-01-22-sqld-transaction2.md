---
layout: post
title: SQLD 수업 정리_220122
slug: SQLD 수업 정리_220122
categories: studylog
tags: SQL, SQLD
description: >
  SQL 수업 내용 정리
---
# SQLD 수업 정리_220122

### TRANSACTION

- 다수개의 DML 문장들을 하나의 작업으로 묶어서 처리할 수 있는 문법
    
    예) 이체 작업 - 송신인의 계좌에서 잔고를 감소 / 수신인 계좌의 잔고를 증가
    
- COMMIT : 현재까지 수행한 DML 작업의 결과를 데이터베이스에 반영
- ROLLBACK : 현재까지 수행한 DML 작업의 결과를 취소
- SAVEPOINT : 현재 수행중인 지점을 임시 저장할 수 있는 명령
    - ROLLBACK의 기준 위치를 정할 수 있음
- ROLLBACK
    
    ```sql
    
    COMMIT;
    
    SELECT * 
    FROM EMP;
    
    -- 급여 변경
    UPDATE EMP
    SET SAL = SAL * 15;
    
    -- 매니저 변경
    UPDATE EMP
    SET MGR = NULL;
    
    -- UPDAT 내역을 확인할 수 있음
    -- COMMIT된 내용이 아니기 때문에 언제든 ROLLBACK 할 수 있음
    SELECT *
    FROM EMP;
    
    -- UPDATE 실행 내용을 반영하지 않고 취소함
    -- 현재까지 수행한 모든 DML 작업을 취소
    ROLLBACK;
    
    -- 취소 확인
    SELECT *
    FROM EMP;
    
    -- 급여 변경
    UPDATE EMP
    SET SAL = SAL * 1.5;
    
    COMMIT;
    
    SELECT *
    FROM EMP;
    
    -- DML 작업의 결과를 COMMIT한 이후에는 ROLLBACK이 반영되지 않음
    ROLLBACK;
    ```
    

- Commit의 특징
    - 데이터에 대한 변경 사항이 데이터베이스에 반영
    - 이전 데이터는 영원히 소실됨
    - 모든 사용자가 결과를 확인할 수 있음 (중요!!)
    - 커밋되지 않은 DML 작업과 관련하여 각 레코드의 락킹이 해제됨
    - 다른 사용자가 커밋된 데이터에 대해서 수정을 할 수 있음

- SAVEPOINT
    - 일반적으로 ROLLBACK 작업은 현재까지 수행한 모든 DML에 대해서 취소를 처리함
    - SAVEPOINT를 활용하여 부분 ROLLBACK을 구현할 수 있음
    - 특정 작업까지만 취소
    
    ```sql
    
    SAVEPOINT s;
    
    SELECT * 
    FROM EMP;
    
    UPDATE EMP
    SET SAL = SAL * 10;
    
    SAVEPOINT s1;
    
    UPDATE EMP
    SET DEPTNO = 10;
    
    SELECT *
    FROM EMP;
    
    ROLLBACK TO s1;
    
    -- DML 작업의 진행 중 세이브 포인트를 생성할 수 있음
    -- SAVEPOINT 이름;
    -- 생성된 세이브포인트는 ROLLBACK의 기준점이 될 수 있음
    -- ROLLBACK TO 세이브포인트명;
    -- 세이브포인트를 사용하여 ROLLBACK을 수행하는 경우 특정 세이브포인트 생성 이후에 작업된 내용이 취소됨
    ```