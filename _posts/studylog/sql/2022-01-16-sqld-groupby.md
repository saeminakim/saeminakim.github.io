---
layout: post
title: SQLD 수업 정리_220116
slug: SQLD 수업 정리_220116
categories: studylog
tags: SQL, SQLD
description: >
  SQL 수업 내용 정리
---
# SQLD 수업 정리_220116

### 그룹 함수

- 통계정보를 반환하기 위한 기준 열 설정 방법
- GROUP BY절, ROLLUP, CUBE, GROUPING ...
    
    ```sql
    -- 부서별, JOB별 인원의 수, 급여의 합계를 조회
    -- 일반적인 GROUP BY절을 활용하여 처리할 수 있음
    -- GROUP BY절의 기준열이 다수개인 경우 다수개의 열을 조합한 값을 그룹으로 지정함
    -- 아래의 예에서는 부서명과 직무가 하나의 데이터로 취급되어 
    -- 각 분류별 인원수, 급여 합계가 계산됨
    SELECT DNAME, JOB, COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME, JOB;
    
    -- GROUP BY절의 실행 결과를 정렬할 수 있음
    SELECT DNAME, JOB, COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME, JOB
    ORDER BY DNAME, JOB;
    ```
    

- ROLLUP 함수
    - GROUP BY절에서 사용할 수 있는 함수
    - 기본적인 GROUP BY절의 동작 외에 각 단위별 집계 정보를 추가적으로 제공
- ROLLUP 함수의 결과 분석
    1. GROUP BY 절의 기존 제공 정보
    2. ROLLUP 함수의 1번째 인자인 부서명 기준의 서브 TOTAL정보
    3. 전체 TOTAL 정보
    
    ```sql
    SELECT DNAME, JOB, COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY ROLLUP (DNAME, JOB);
    
    -- 주의!! ROLLUP 함수의 인자의 순서에 따라 서로 다른 값이 나옴
    SELECT JOB, DNAME, COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY ROLLUP (JOB, DNAME);
    
    -- ROLLUP 함수와 ORDER BY 절을 조합하여 사용 가능
    SELECT DNAME, JOB, COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY ROLLUP (DNAME, JOB)
    ORDER BY DNAME, JOB;
    ```
    

- GROUPING 함수
    - GROUP BY ROLLUP에 사용되는 인자의 이름을 사용하여 현재 컬럼의 값이 NULL인지 확인할 수 있는 함수 (0 또는 1을 반환)
    - 현재 컬럼에 대해서 한 단계 위의 집계 정보를 보여주는 위치를 확인하는 용도
    
    ```sql
    SELECT DNAME, GROUPING(DNAME) AS DNAME_GRP,
        JOB, GROUPING(JOB) AS JOB_GRP,
        COUNT(*) AS "인원수", 
        SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY ROLLUP (DNAME, JOB)
    ORDER BY DNAME, JOB;
    
    -- GROUPING 함수와 CASE절을 조합하는 예제
    SELECT 
    CASE GROUPING(DNAME)
        WHEN 1 THEN '전체부서' ELSE DNAME END AS "부서명",
    CASE GROUPING(JOB)
        WHEN 1 THEN '전체직무' ELSE JOB END AS "직무명",
        COUNT(*) AS "인원수", 
        SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY ROLLUP (DNAME, JOB)
    ORDER BY DNAME, JOB;
    
    -- GROUP BY 절의 ROLLUP 함수는 일부의 컬럼만을 대상으로 지정 가능
    -- GROUP BY 절의 모든 인자들을 ROLLUP 함수의 인자로 사용할 필요는 없음
    SELECT DNAME, JOB, COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME, ROLLUP (JOB)
    ORDER BY DNAME, JOB;
    
    -- 부서와 직무, 매니저 별 인원의 수, 급여의 합계 조회
    SELECT DNAME, JOB, MGR, COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME, JOB, MGR
    ORDER BY DNAME, JOB, MGR;
    
    -- ROLLUP 함수의 컬럼 집합을 정의하는 방법
    -- 아래와 같이 ROLLUP 함수의 인자를 작성하면 왼쪽부터 시작하여 
    -- 각 계층별 ROLLUP 함수의 집계 결과가 반환됨
    -- 만약 각 인자별로 계층을 구분하지 않고 
    -- 특정 인자 사이에서는 동등한 레벨로 처리하고자 할 때 컬럼 집합을 선언할 수 있음
    SELECT DNAME, JOB, MGR, COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    -- 아래와 같이 ROLLUP을 정의하면 부서명, 직무명, 매니저 단위 각각의 ROLLUP 집계가 반환됨
    GROUP BY ROLLUP(DNAME, JOB, MGR)
    ORDER BY DNAME, JOB, MGR;
    
    SELECT DNAME, JOB, MGR, COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    -- 아래와 같이 ROLLUP을 정의하면 부서명, 직무명과 매니저 단위 각각의 ROLLUP 집계가 반환됨
    -- 직무명과 매니저가 하나의 컬럼처럼 인식되어 동작함
    GROUP BY ROLLUP(DNAME, (JOB, MGR))
    ORDER BY DNAME, JOB, MGR;
    ```
    

- CUBE 함수의 사용
    - CUBE 함수는 사용 가능한 모든 데이터 집합에 대해서 집계결과를 제공하는 함수
    - ROLLUP 함수에 비해 다른 각도의 데이터 집계를 추가적으로 확인할 수 있음
    - ROLLUP 함수에 대비 계산량이 증가하기 때문에 속도는 느림
    - CUBE 함수는 인자의 순서가 변경되어도 출력 순서는 바뀔 수 있지만 전체 결과가 바뀌진 않음
    
    ```sql
    SELECT 
        CASE GROUPING(DNAME) WHEN 1 THEN '전체부서' ELSE DNAME END AS "부서", 
        CASE GROUPING(JOB) WHEN 1 THEN '전체직무' ELSE JOB END AS "직무",
        COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    GROUP BY ROLLUP(DNAME, JOB)
    ORDER BY DNAME, JOB;
    
    -- CUBE 함수 사용 예제
    -- ROLLUP 함수를 CUBE로 변경하여 간단히 확인할 수 있음
    SELECT 
        CASE GROUPING(DNAME) WHEN 1 THEN '전체부서' ELSE DNAME END AS "부서", 
        CASE GROUPING(JOB) WHEN 1 THEN '전체직무' ELSE JOB END AS "직무",
        COUNT(*), SUM(SAL)
    FROM EMP NATURAL JOIN DEPT
    GROUP BY CUBE(DNAME, JOB)
    ORDER BY DNAME, JOB;
    
    -- (문제)위의 CUBE를 사용하여 집계를 확인하는 예제와 동일한 결과를 반환할 수 있는 
    -- 쿼리문 작성
    -- UNION ALL 연산자 사용하여 처리
    SELECT DNAME, JOB, 
        COUNT(*) AS "사원수", 
        SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME, JOB
    UNION
    SELECT DNAME, '전체직무' AS "JOB", 
        COUNT(*) AS "사원수", 
        SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME
    UNION
    SELECT '전체부서' AS "DNAME", '전체직무' AS "JOB", 
        COUNT(*) AS "사원수", 
        SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    -- 이 부분까지가 ROLLUP 함수의 영역
    UNION
    SELECT '전체부서' AS "DNAME", JOB, 
        COUNT(*) AS "사원수", 
        SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY JOB;
    
    -- 위의 쿼리문의 실행 결과에서 확인할 수 있듯이 ROLLUP, CUBE 함수의 실행결과는 
    -- 기존의 SELECT 쿼리의 집합연산을 사용해도 동일한 결과를 얻을 수 있음
    -- 하지만 ROLLUP, CUBE 함수를 사용할 때는 데이터의 조회 연산(FROM)이 한 번만 일어나는 반면,
    -- 집합연산을 사용하게 되면 다수번의 데이터 접근 및 적재가 일어나기 때문에
    -- ROLLUP, CUBE 함수를 사용하는 것이 성능 향상 부분에 많은 개선을 보임
    
    -- (문제) 일반 그룹함수와 집합 연산을 사용하여 부서별, 직무별 인원수와 급여의 합계를 조회
    SELECT DNAME, '전체직무' AS "JOB", COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY DNAME
    UNION ALL
    SELECT '전체부서' AS "DNAME", JOB, COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY JOB;
    
    -- GROUPING SETS 함수를 사용하여 위의 예제와 동일한 결과를 생성하는 방법
    -- GROUPING SETS 함수는 GROUP BY 절의 기준 컬럼을 동등한 관계로 다수개 선언할 수 있는 함수
    -- GROUPING SETS 함수 인자의 순서가 변경되어도 출력의 순서는 바뀔 수 있지만 
    -- 결과의 순서가 바뀌진 않음
    -- GROUP BY GROUPING SETS (DNAME, JOB)
    -- GROUP BY DNAME + GROUP BY JOB
    SELECT 
        CASE GROUPING(DNAME) WHEN 1 THEN '전체부서' ELSE DNAME END AS "부서명",
        CASE GROUPING(JOB) WHEN 1 THEN '전체직무' ELSE DNAME END AS "직무명",
        COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY GROUPING SETS (DNAME, JOB)
    ORDER BY DNAME, JOB;
    
    -- 부서/직무/MGR, 부서/직무, 직무/MGR 별 집계를 조회하는 쿼리문을 작성
    -- GROUPING SETS 함수의 인자값으로 다수개의 컬럼이 조합된 컬럼 집합을 정의할 수 있고 
    -- 아래와 같이 다수개의 기준을 인자로 전달하여 통합된 결과를 얻을 수 있음
    SELECT DNAME, JOB, MGR,
        COUNT(*) AS "인원수", SUM(SAL) AS "급여합계"
    FROM EMP NATURAL JOIN DEPT
    GROUP BY GROUPING SETS ((DNAME, JOB, MGR), (DNAME, JOB), (JOB, MGR));
    ```