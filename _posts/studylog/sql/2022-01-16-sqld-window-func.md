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

### 윈도우함수

- 순위 정보와 같은 정보를 제공하는 함수
- 관계형 데이터베이스는 행과 열이라는 개념으로 테이블 단위의 데이터 저장소를 가짐
- 기본적으로 RDBMS에서 동작하는 SQL문은 하나의 행(레코드)에 대한 컬럼 연산에 최적화되어 있음
    
    EX) 특정 사원에 대한 입사일자, 급여, 관리자 ID를 추출
    
- RDBMS에서 SQL 작성으로 해결하기 어려운 문제는 행(레코드)단위의 연산
    
    : 대표적으로 순위를 구하는 등 
    
- 이러한 행단위의 연산을 손쉽게 처리하기 위해 제공하는 기능이 윈도우 함수 (ANSI 표준에서 정의한 이름)
- 윈도우 함수를 사용하는 방법
    
    ```sql
    -- SELECT 절에서 사용
    SELECT 함수명([인자]) OVER ([PARTITION BY 컬럼명] [ORDER BY 컬럼명] [WINDOWING 절])
    FROM 테이블명;
    ```
    

- RANK, DENSE_RANK, ROW_NUM
    
    ```sql
    -- RANK WINDOW 함수의 사용 예제
    -- PARTITION BY : 순위를 계산할 때 사용할 그룹의 단위 컬럼을 정의
    -- ORDER BY : 순위를 계산할 때 사용할 컬럼의 정렬 방법 정의
    -- EMP 테이블을 대상으로 모든 사원들의 순위를 조회
    -- 순위 조회의 조건은 단순 급여 비교, JOB별로 급여 비교
    SELECT ENAME, JOB, SAL,
        RANK() OVER (ORDER BY SAL DESC) AS "전체급여순위",
        RANK() OVER(PARTITION BY JOB ORDER BY SAL DESC) AS "직무별급여순위"
    FROM EMP;    
    
    -- DENSE_RANK WINDOW 함수
    -- RANK 함수와 마찬가지로 순위를 반환하는 함수
    -- RANK 함수와 차이점은 직전의 순위값이 동일한 데이터가 다수개 있는 경우 
    -- DENSE_RANK 함수는 직전 순위값 + 1을 반환
    -- RANK 함수는 직전 순위값 + 직전 순위에 해당되는 데이터 수
    -- 아래 RANK 함수 순위 4위인 JONES의 순위를 비교해보면 알 수 있음
    SELECT ENAME, JOB, SAL,
        RANK() OVER (ORDER BY SAL DESC) AS "(RANK)",
        DENSE_RANK() OVER(ORDER BY SAL DESC) AS "(DENSE)"
    FROM EMP;    
    
    -- ROW_NUMBER WINDOW 함수
    -- RANK, DENSE_RANK 함수와 같이 순위의 값을 반환하는 함수
    -- RANK, DENSE_RANK 함수와 달리 동등한 값이라도 고유한 순위값을 반환
    -- ROW_NUMBER 함수를 사용할 경우 정렬의 기준을 디테일하게 정의할 필요가 있음
    -- 동일한 값에 대해서 순위의 결정 요소는 오라클의 경우 저장된 순서 등의 값으로 결정됨 
    SELECT ENAME, JOB, SAL,
        RANK() OVER (ORDER BY SAL DESC) AS "(RANK)",
        ROW_NUMBER() OVER(ORDER BY SAL DESC, ENAME DESC) AS "(ROW_NUM)"
    FROM EMP;
    ```
    

- 일반 집계 함수
    
    ```sql
    -- SUM, COUNT, AVG, MAX, MIN .... 
    
    -- 사원의 이름과 관리자 아이디, 급여, 동일한 관리자 아이디를 가지는 사원의 
    -- 급여 총합을 조회하는 예제 
    SELECT ENAME, MGR, SAL,
        SUM(SAL) OVER (PARTITION BY MGR) AS "MGR_SUM"
    FROM EMP;    
    
    -- 윈도우 함수의 집계 함수를 사용하여 누적값을 계산
    -- 집계함수() OVER ([PARTITION BY 컬럼명] [ORDER BY 컬럼명] [RANGE UNBOUNDED PRECEDING])
    -- RANGE UNBOUNDED PRECEDING : PARTITION으로 구분된 데이터의 영역에서 첫번째 데이터부터 
    -- 현재 데이터까지의 범위를 의미
    -- 주의!!! RANK가 동일한 경우 순차적인 누적값이 아님
    -- 아래의 실행 결과에서 SCOTT과 FORD, WARD와 MARTIN 참고
    SELECT ENAME, MGR, SAL,
        SUM(SAL) OVER (PARTITION BY MGR ORDER BY SAL ASC RANGE UNBOUNDED PRECEDING) AS "MGR_SUM"
    FROM EMP; 
    
    -- 기본적인 집계함수는 윈도우 함수로 사용 가능
    -- SUM, COUNT, AVG, MAX, MIN .... 
    SELECT ENAME, MGR, SAL,
        MAX(SAL) OVER (PARTITION BY MGR ORDER BY SAL ASC) AS "MAX(SAL)"
    FROM EMP; 
    
    -- 매니저가 같은 사원 중 가장 높은 급여를 받는 사원만 조회
    -- 윈도우 함수의 결과를 서브쿼리로 활용하여 아래와 같이 적용할 수 있음
    SELECT ENAME, MGR, SAL
    FROM (SELECT ENAME, MGR, SAL,
            MAX(SAL) OVER (PARTITION BY MGR ORDER BY SAL ASC) AS "MAX_SAL"
          FROM EMP)
    WHERE SAL = MAX_SAL;
    
    -- 파티션 별 최대값을 가지는 데이터를 추출하는 경우 아래와 같이 RANK 함수를 사용하는 것이 
    -- 성능상 유리함
    SELECT ENAME, MGR, SAL
    FROM (SELECT ENAME, MGR, SAL,
            RANK() OVER (PARTITION BY MGR ORDER BY SAL ASC) AS "SAL_RANK"
          FROM EMP)
    WHERE SAL_RANK = 1;
    
    -- 집계함수를 사용하여 윈도우 함수를 적용하는 경우 ORDER BY절의 기준 컬럼에 따라 
    -- 동일한 파티션이라도 서로 다른 값이 반환될 수 있음
    SELECT ENAME, MGR, HIREDATE, SAL, 
        MIN(SAL) OVER (PARTITION BY MGR ORDER BY HIREDATE) AS "MIN(SAL)"
    FROM EMP; 
    
    -- ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    -- 현재 행을 기준으로 직전 행 1개, 이후 행 1개의 데이터를 포함하여 함수를 적용하는 방법
    -- ROWS : 윈도우 함수에 적용할 데이터의 건수를 의미
    
    -- RANGE UNBOUNDED PRECEDING
    -- 현재 파티션 내에서 현재 행을 기준으로 파티션에 1번째 행까지를 포함하는 범위 지정
    SELECT ENAME, MGR, HIREDATE, SAL, 
        ROUND(AVG(SAL) OVER (PARTITION BY MGR ORDER BY HIREDATE ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING), 2)AS "AVG(SAL)"
    FROM EMP; 
    
    -- COUNT 함수(개수를 반환)
    SELECT ENAME, MGR, SAL, 
        COUNT(*) OVER (PARTITION BY MGR) AS "COUNT(*)"
    FROM EMP; 
    
    -- 윈도우 함수에서 COUNT 함수를 사용할 때 RANGE를 사용하여 조건을 지정할 수 있음
    -- ORDER BY SAL ASC RANGE BETWEEN 50 PRECEDING AND 150 FOLLOWING
    -- 현재 행을 기준으로 SAL 컬럼의 값에서 50 작은 값부터 150을 더한 값 사이에 있는 
    -- SAL 데이터의 개수를 조회하는 방법
    SELECT ENAME, MGR, SAL, 
        COUNT(*) OVER (ORDER BY SAL ASC RANGE BETWEEN 50 PRECEDING AND 150 FOLLOWING) AS "COUNT(*)"
    FROM EMP; 
    
    -- 부서별 직원들을 연봉이 높은 순서부터 정렬하고 파티션 내에서 가장 먼저 나온 사원의 정보를 
    -- 출력하는 예시
    -- FIRST_VALUE 함수
    -- 현재 윈도우(파티셔닝) 내에서 가장 첫번째에 위치한 값을 추출하는 함수
    -- 전체 파티션에 대한 첫번째 값인 경우가 대부분이지만 아래의 예와 같이 범위를 지정하여 
    -- 파티션 내부의 특정 범위내의 첫번째 값을 추출할 수 있음
    SELECT ENAME, DEPTNO, SAL,
        FIRST_VALUE (ENAME) OVER (PARTITION BY DEPTNO ORDER BY SAL DESC ROWS UNBOUNDED PRECEDING) AS ENAME_FV
    FROM EMP;
    
    -- FIRST_VALUE 함수
    -- 현재 윈도우(파티셔닝) 내에서 가장 마지막에 위치한 값을 추출하는 함수
    -- 전체 파티션에 대한 마지막 값인 경우가 대부분이지만 아래의 예와 같이 범위를 지정하여 
    -- 파티션 내부의 특정 범위내의 마지막 값을 추출할 수 있음
    SELECT ENAME, DEPTNO, SAL,
        LAST_VALUE (ENAME) OVER (PARTITION BY DEPTNO ORDER BY SAL DESC ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS ENAME_LV
    FROM EMP;
    
    -- 사원명과 JOB, SAL, 해당 사원의 급여와 JOB별 평균 급여의 차이를 출력하는 쿼리문
    -- 본인의 급여 - JOB별 평균 급여의 값이 + 라면 평균이상, -라면 평균 미만, 0이면 평균으로 
    -- 출력
    SELECT ENAME, JOB, SAL,
        ROUND(AVG(SAL) OVER (PARTITION BY JOB) ,2) AS "평균급여",
        SAL - ROUND(AVG(SAL) OVER (PARTITION BY JOB) ,2) AS "차이",
        CASE 
            WHEN (SAL - AVG(SAL) OVER (PARTITION BY JOB)) > 0 THEN '평균이상'
            WHEN (SAL - AVG(SAL) OVER (PARTITION BY JOB)) < 0 THEN '평균미만'
            ELSE '평균' END AS "급여수준"
    FROM EMP;
    
    SELECT ENAME, JOB, SAL, "평균급여", "차이",
        CASE 
            WHEN "차이" > 0 THEN '평균이상'
            WHEN "차이" < 0 THEN '평균미만'
            WHEN "차이" = 0 THEN '평균'
        END AS "급여수준"    
    FROM (SELECT ENAME, JOB, SAL,
            ROUND(AVG(SAL) OVER (PARTITION BY JOB) ,2) AS "평균급여",
            SAL - ROUND(AVG(SAL) OVER (PARTITION BY JOB) ,2) AS "차이" 
          FROM EMP);
    
    -- LAG 함수
    -- 윈도우/파티션 내의 현재 위치를 기준으로 직전 N번재 위치의 값을 추출하는 함수
    -- LAG (컬럼명, 위치값의 정수 - 현재 레코드의 행 위치를 기준으로 직전 N번째)
    SELECT ENAME, HIREDATE, SAL,
        LAG (ENAME, 1) OVER (ORDER BY HIREDATE ASC) AS "직전에 입사한 사원명"
    FROM EMP;    
    
    -- LAG 함수의 NULL 값 처리 방법
    -- LAG(컬럼명, 위치값, NULL일때 사용 컬럼/값)
    SELECT ENAME, HIREDATE, SAL,
        LAG (ENAME, 1, 'NO_VALUE') OVER (ORDER BY HIREDATE ASC) AS "직전에 입사한 사원명"
    FROM EMP; 
    
    -- LEAD 함수
    -- 윈도우 파티션 내의 현재 위치를 기준으로 다음 N번째 위치의 값을 추출하는 함수
    -- LEAD (컬럼명, 위치값의 정수 - 현재 레코드의 행 위치를 기준으로 직전 N번째)
    SELECT ENAME, HIREDATE, SAL,
        LEAD (ENAME, 1) OVER (ORDER BY HIREDATE ASC) AS "직후에 입사한 사원명"
    FROM EMP; 
    
    -- LEAD 함수의 NULL 값 처리 방법
    -- LEAD(컬럼명, 위치값, NULL일때 사용 컬럼/값)
    SELECT ENAME, HIREDATE, SAL,
        LEAD (ENAME, 1, '마지막') OVER (ORDER BY HIREDATE ASC) AS "직후에 입사한 사원명"
    FROM EMP; 
    
    SELECT ENAME, HIREDATE, SAL,
        LEAD (ENAME, 1, '마지막') OVER (PARTITION BY DEPTNO ORDER BY HIREDATE ASC) AS "직후에 입사한 사원명"
    FROM EMP; 
    
    -- 비율에 관련된 함수
    -- RATIO_TO_REPORT
    -- 특정 컬럼의 값이 현재 파티션/윈도우 내의 총합을 기준으로 차지하고 있는 비율의 값을 
    -- 반환하는 함수
    SELECT ENAME, DNAME, SAL,
        SUM(SAL) OVER (PARTITION BY DNAME) AS "SAL_SUM_DNAME",
        ROUND(RATIO_TO_REPORT(SAL) OVER(), 2) AS "SAL_RTR"
    FROM EMP NATURAL JOIN DEPT
    WHERE DNAME = 'SALES';
    
    -- 순위출력 함수
    -- RANK, DENSE_RANK, ROW_NUMBER (순위값을 점수로 반환)
    -- PERCENT_RANK (순위의 비율값을 반환)
    -- 가장 높은 순위는 값이 0, 가장 낮은 순위의 값은 1
    SELECT ENAME, DNAME, SAL,
        RANK() OVER (PARTITION BY DNAME ORDER BY SAL DESC) AS "RANK_SAL",
        PERCENT_RANK() OVER (PARTITION BY DNAME ORDER BY SAL DESC) AS "PRANK_SAL"
    FROM EMP NATURAL JOIN DEPT;
    
    -- 누적 순서 비율을 반환하는 함수
    -- 현재 레코드가 소속되어 있는 윈도우/파티션에서의 순서 위치를 의미하는 실수의 값을 반환
    -- CUME_DIST
    SELECT ENAME, DNAME, SAL,
        ROUND(CUME_DIST() OVER (PARTITION BY DNAME ORDER BY SAL DESC), 2) AS "CUME_DIST"
    FROM EMP NATURAL JOIN DEPT;
    
    -- 데이터를 등급으로 구분하여 반환하는 NTILE 함수
    -- 데이터를 함수의 인자의 개수로 분할한 결과값을 반환하는 함수
    SELECT ENAME, DNAME, SAL,
        RANK() OVER (ORDER BY SAL DESC) AS "RANK_SAL",
        NTILE(5) OVER (ORDER BY SAL DESC) AS "NTILE_SAL"
    FROM EMP NATURAL JOIN DEPT;
    ```