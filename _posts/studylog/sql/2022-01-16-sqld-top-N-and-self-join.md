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

### TOP N

- SELECT 조회 결과 중 상위 N개만 출력하는 쿼리
- 일반적으로 SELECT 쿼리문은 특정 테이블에서 조건에 만족하는 전체 결과를 출력하지만 TOP N 쿼리는 SELECT 절의 출력 결과 중 상위 N번까지만 출력
- 오라클 데이터베이스의 ROWNUM 컬럼 : 실제로 저장된 컬럼이 아님
- SELECT 절을 수행할 때 테이블로부터 데이터를 로딩하는 순서의 값을 임시적으로 제공하는 컬럼
    
    ```sql
    -- ROWNUM 컬럼을 사용하여 가장 처음의 데이터 한 개만 추출하는 방법
    SELECT *
    FROM EMP
    WHERE ROWNUM = 1;
    
    -- ROWNUM 컬럼을 사용하여 상위 N개의 데이터만 추출하는 방법
    SELECT *
    FROM EMP
    WHERE ROWNUM <= 3;
    
    -- ROWNUM 컬럼을 사용할 때 주의사항
    -- WHERE절에서 ROWNUM 컬럼의 값을 비교하는 조건식은 
    -- ROWNUM = N (특정 위치의 레코드만 출력)
    -- ROWNUM < N (특정 위치 이전의 레코드만 출력)
    -- ROWNUM <= N (특정 위치 이전의 레코드만 출력)
    -- 아래와 같이 ROWNUM에 대해 >, >= 조건식은 실행되지 않음
    SELECT *
    FROM EMP
    WHERE ROWNUM > 1;
    
    SELECT *
    FROM EMP  -- ROWNUM이 생성
    WHERE ROWNUM <= 3  -- 상위 3개의 데이터만 이미 선택 완료
    ORDER BY SAL DESC;  -- 이미 선택 완료된 3개의 데이터에 대해 정렬 수행
    
    -- 테이블의 내용을 정렬한 결과에서 ROWNUM 컬럼의 값을 사용하는 예제
    -- 서브쿼리를 사용하여 정렬이 수행된 테이블(인라인 뷰)를 활용하는 방식
    SELECT *
    FROM (SELECT *
            FROM EMP
            ORDER BY SAL DESC) 
    -- 서브 쿼리를 사용하여 FROM 절을 정의하면 정렬이 완료된 상태의 ROWNUM 값을 사용할 수 있음
    WHERE ROWNUM <= 3;
    
    -- MSSQL의 경우 TOP 함수 제공
    SELECT TOP(3)
    FROM EMP;
    
    -- MYSQL의 경우 LIMIT 절 제공
    SELECT *
    FROM EMP
    LIMIT 1, 10;
    
    -- ROW LIMITING 절
    -- MYSQL의 LIMIT 절과 같이 표준화된 페이징 처리 기법을 ANSI 표준으로 제시
    -- ORACLE의 경우 12.1 이후 버전에서 사용 가능, MSSQL 2012 이후 버전에서 사용 가능
    SELECT 컬럼명
    FROM 테이블명
    ORDER BY 컬럼명 정렬조건 [ROW LIMITING 절]
    
    -- [ROW LIMITING 절]
    -- [OFFSET(출력하지 않고 이동할 레코드의 개수를 지정) 5 ROWS] 
    -- [FETCH, [FIRST | NEXT] N ROWS [ONLY | WITH TIE]
    SELECT *
    FROM EMP
    ORDER BY SAL FETCH FIRST 5 ROWS ONLY;
    ```
    

### 셀프 조인

- 자기 자신의 테이블과 조인하는 조인문
- 셀프 조인은 일반적인 조인문과 차이가 없고 동일한 테이블 사이에서 조인이 된다는 점만 다름
- 셀프 조인의 경우 동일한 테이블 사이에서 조인이 일어나기 때문에 동일한 컬럼이 기본적으로 2개씩 존재
- 그렇기 때문에 각 컬럼의 값을 출력하기 위해 반드시 별칭이 필요함
    
    ```sql
    -- 셀프 조인 / 계층형 질의 (CONNECT BY)
    SELECT *
    FROM EMP;
    
    -- 만약 SMITH 사원의 관리자 이름을 조회하고자 하는 경우
    -- EMP 테이블의 경우 MGR의 아이디는 사원 아이디이므로 동일한 EMP 테이블 조인하여 
    -- 특정 사원의 MGR 정보 확인
    SELECT E.EMPNO, E.ENAME, E.MGR, M.ENAME
    FROM EMP E INNER JOIN EMP M
        ON E.MGR = M.EMPNO;
    
    -- JONES 사원이 관리하고 있는 사원의 목록을 조회
    SELECT E.EMPNO, E.ENAME, E.MGR, M.ENAME
    FROM EMP M INNER JOIN EMP E
        ON M.EMPNO = E.MGR
    WHERE M.ENAME = 'JONES';
    
    -- 내가 짠거
    SELECT * 
    FROM EMP
    WHERE MGR = (SELECT EMPNO 
                FROM EMP 
                WHERE ENAME = 'JONES');
    
    -- JONES 사원이 관리하고 있는 사원들이 관리하는 하위 사원의 목록을 조회
    SELECT E.EMPNO, E.ENAME, E.MGR, M2.ENAME
    FROM EMP M1 INNER JOIN EMP M2
        ON M1.EMPNO = M2.MGR
        INNER JOIN EMP E
        ON M2.EMPNO = E.MGR
    WHERE M1.ENAME = 'JONES';
    
    -- 내가 짠거
    SELECT *
    FROM EMP
    WHERE MGR IN (SELECT E.EMPNO
                FROM EMP M INNER JOIN EMP E
                    ON M.EMPNO = E.MGR
                WHERE M.ENAME = 'JONES');
    ```