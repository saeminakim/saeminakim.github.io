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

### PIVOT

- 행으로 존재하는 데이터를 열의 단위로 변경하여 집계할 수 있는 기능
    
    ```sql
    -- JOB / 부서 단위의 집계 결과를 확인
    SELECT JOB, DEPTNO, SUM(SAL)
    FROM EMP
    GROUP BY JOB, DEPTNO;
    
    -- JOB  DEPT_10 DEPT_20 DEPT_30 형식으로 출력
    SELECT JOB,
        SUM(CASE DEPTNO WHEN 10 THEN SAL END) AS "DEPT_10",
        SUM(CASE DEPTNO WHEN 20 THEN SAL END) AS "DEPT_20",
        SUM(CASE DEPTNO WHEN 30 THEN SAL END) AS "DEPT_30"
    FROM EMP
    GROUP BY JOB; 
    ```
    
- PIVOT 절을 활용한 집계 결과 출력 방법
    
    : PIVOT (집계함수 (컬럼명) [ .... ] FOR [열의 생성 기준 컬럼] IN (기준 컬럼 값))
    
    : 주의사항 ⇒  PIVOT 절에서 사용되는 컬럼명은 인라인 뷰를 통해서 처리해야 함
    
    ```sql
    SELECT *
    FROM (SELECT JOB, DEPTNO, SAL FROM EMP)
    PIVOT (SUM(SAL) FOR DEPTNO IN (10, 20, 30));
    ```
    
- PIVOT 절 결과에 대한 컬럼명 별칭 사용 예제
    1. 집계함수의 결과에 별칭을 지정
        
        : FOR에 정의된 컬럼의 값이름_별칭
        
    2. IN에 나열된 각 값에 대해서 별칭을 지정
        
        : IN에 정의된 별칭_집계함수의 별칭
        
        : 집계함수에 별칭이 정의되지 않은 경우 IN에 정의된 별칭만 출력
        
        ```sql
        SELECT *
        FROM (SELECT JOB, DEPTNO, SAL FROM EMP)
        PIVOT (SUM(SAL) AS "SUM" FOR DEPTNO IN (10 AS "D10", 20 AS "D20", 30 AS "D30"));
        
        -- PIVOT절의 집계함수는 다수개를 활용할 수 있음
        -- 주의사항!!
        -- 집계함수에 대해서 별칭을 지정해야만 컬럼명 충돌을 피할 수 있음
        SELECT *
        FROM (SELECT JOB, DEPTNO, SAL FROM EMP)
        PIVOT (SUM(SAL) AS "SUM", COUNT(*) FOR DEPTNO 
        								IN (10 AS "D10", 20 AS "D20", 30 AS "D30"));
        ```
        
    
- UNPIVOT
    - PIVOT된 결과를 해제하는 방법
    - 열을 행으로 변경
    
    ```sql
    -- SELECT 절을 활용한 테이블 생성 예시
    CREATE TABLE TEST1
    AS
    SELECT *
    FROM (SELECT JOB, DEPTNO, SAL FROM EMP)
    PIVOT (SUM(SAL) AS "SUM", COUNT(*) AS "COUNT" FOR DEPTNO IN (10 AS "D10", 20 AS "D20", 30 AS "D30"));
    
    SELECT *
    FROM TEST1;
    
    -- UNPIVOT 절을 활용한 PIVOT 해제 방법
    -- UNPIVOT (컬럼명/출력할 이름 [ .... ] FOR [행의 생성 기준 컬럼] IN (기준 컬럼 이름))
    -- 주의사항
    -- PIVOT 절에서 사용되는 컬럼명은 인라인 뷰를 통해서 처리해야 함
    -- UNPIVOT을 활용하면 NULL 값인 열은 변환되지 않음
    -- 만약 NULL을 포함하여 UNPIVOT을 수행하고자 하는 경우 INCLUDE NULLS 옵션을 사용
    SELECT JOB, DEPTNO, SAL
    FROM TEST1
    UNPIVOT (SAL FOR DEPTNO IN (D10_SUM AS 10, D20_SUM AS 20, D30_SUM AS 30));
    
    -- UNPIVOT을 활용하면 NULL 값인 열은 변환되지 않음
    -- 만약 NULL을 포함하여 UNPIVOT을 수행하고자 하는 경우 INCLUDE NULLS 옵션을 사용
    SELECT JOB, DEPTNO, SAL
    FROM TEST1
    UNPIVOT INCLUDE NULLS (SAL FOR DEPTNO IN (D10_SUM AS 10, D20_SUM AS 20, D30_SUM AS 30));
    
    -- 다수개의 열에 대해서 UNPIVOT을 수행하는 예시
    -- UNPIVOT ( (UNPIVOT 대상 열1, 대상 열2 ...) )
    SELECT JOB, DEPTNO, SAL, COUNT
    FROM TEST1
    UNPIVOT ((SAL, COUNT) FOR DEPTNO IN ((D10_SUM, D10_COUNT) AS 10, 
                                         (D20_SUM, D20_COUNT) AS 20, 
                                         (D30_SUM, D30_COUNT) AS 30));
    
    -- UNPIVOT절에 다수개의 컬럼명을 작성한 후
    -- IN 절의 AS를 사용하여 값을 할당할 수 있음
    SELECT JOB, DEPTNO, DNAME, SAL, COUNT
    FROM TEST1
    UNPIVOT ((SAL, COUNT) FOR (DEPTNO, DNAME) IN ((D10_SUM, D10_COUNT) AS (10, 'ACCOUNTING'), 
                                         (D20_SUM, D20_COUNT) AS (20, 'RESEARCH'), 
                                         (D30_SUM, D30_COUNT) AS (30, 'SALES')));
    ```