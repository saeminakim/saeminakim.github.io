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

### CONNECT BY

- 계층 질의를 위한 CONNECT BY절
- 계층 질의 : 하나의 테이블에서 특정 컬럼의 값이 다른 레코드의 컬럼에 의해서 참조되는 경우 서로 다른 레코드의 값을 상호 참조하여 질의하는 방법
    
    예) 사이트를 가입할 추천인을 등록하는 경우 -- 기존에 존재하는 회원의 아이디가 새롭게 가입된 회원에 의해서 참조
         특정 부서에 근무하고 있는 직원과 해당 직원을 관리하는 매니저의 경우 -- 기존에 존재하는 사원의 아이디(팀장/매니저)가 다른 사원에 의해서 참조
    
- 계층의 깊이가 깊지 않은 경우는 셀프 조인으로 해결할 수 있음 (자기 참조 조인문)
- 기본 문법
    
    ```sql
    SELECT 컬럼명
    FROM 테이블명
    START WITH [조건식 - 계층을 추적할 시작 레코드를 지정]
    CONNECT BY [조건식 - 다음 계층으로 이동하기 위한 조건식 지정] 
    							-> 다음으론 내가 어디로 가면되는지에 대한 조건
    ```
    
    ```sql
    SELECT *
    FROM EMP
    START WITH ENAME = 'KING'
    CONNECT BY MGR = PRIOR EMPNO;
    
    -- 계층적 질의를 수행하는 경우 위의 SQL문 실행 결과와 같이 
    -- 계층 구조를 확인하기 어려운 문제가 있음
    ```
    

- 계층적 질의의 결과 이해를 이한 추가 제공 컬럼
    1. LEVEL 
        
        : 계층적 질의 결과를 이해하기 위한 컬럼으로 현재 레코드의 깊이(DEPTH) 정보를 제공하는 컬럼
        
        : 가장 상위(START WITH 절의 조건에 맞는 레코드)의 깊이 값은 1부터 시작
        
        ```sql
        SELECT LEVEL AS "LEVEL", EMPNO, ENAME, MGR
        FROM EMP
        START WITH ENAME = 'KING'
        CONNECT BY MGR = PRIOR EMPNO;
        ```
        
    2. CONNECT_BY_ISLEAF 
        
        : 계층적 질의 결과를 이해하기 위한 컬럼으로 현재 레코드가 더이상 내려갈 수 없는 레코드인지에 대한 여부 값을 반환
        
        : 가장 하위(CONNECT BY절의 조건에 FALSE가 되는 레코드)의 값은 1, 그 외에는 0
        
        ```sql
        SELECT LEVEL AS "LEVEL", 
            CONNECT_BY_ISLEAF AS "ISLEAF", 
            EMPNO, ENAME, MGR
        FROM EMP
        START WITH ENAME = 'KING'
        CONNECT BY MGR = PRIOR EMPNO;
        ```
        
    3. CONNECT_BY_ISCYCLE 
        
        : 계층적 질의 결과를 이해하기 위한 컬럼으로 순환참조 레코드 여부를 판단하는 값을 반환
        
        : 일반적으로 계층적 구조를 생성할 때 단방향인 경우가 대다수임(위에서 아래로 일방향 검색)
        
        : 하지만 순환 구조의 계층적 구조를 가지는 경우 상위의 레코드가 하위의 레코드를 참조하는 경우가 발생
        
        : 순환 참조가 발생하는 레코드의 값은 1, 그 외에는 0
        -- 주의사항 
        -- CONNECT_BY_ISCYCLE 컬럼을 사용하기 위해서는 아래와 같이 CONNECT BY절에 NOCYCLE 옵션을 지정해야만 가능
        
        ```sql
        SELECT LEVEL AS "LEVEL", 
            CONNECT_BY_ISLEAF AS "ISLEAF", 
            CONNECT_BY_ISCYCLE AS "ISCYCLE", 
            EMPNO, ENAME, MGR
        FROM EMP
        START WITH ENAME = 'KING'
        CONNECT BY NOCYCLE MGR = PRIOR EMPNO;
        
        -- 순환 참조 구조를 생성하기 위해서 가장 상위의 KING을 하위 레코드에 연결
        UPDATE EMP
        SET MGR = 7654
        WHERE ENAME = 'KING';
        
        SELECT *
        FROM EMP;
        
        UPDATE EMP
        SET MGR = NULL
        WHERE ENAME = 'KING';
        ```
        
    4. CONNECT_BY_ROOT 
        
        : 계층적 질의 결과를 이해하기 위한 컬럼으로 현재 레코드의 최상위 루트 레코드 정보를 출력하기 위한 컬럼
        
        : 최상위 루트 레코드 정보 중 출력할 컬럼의 값을 인자로 전달
        
    
    5. SYS_CONNECT_BY_PATH 
    
    : 계층적 질의 결과를 이해하기 위한 컬럼으로 현재 레코드를 검색하기 위해서 진행된 최상위 루트부터의 경로를 반환
    
    : 경로 정보를 출력하기 위한 컬럼의 이름와 구분 문자값을 인자로 전달
    
    ```sql
    SELECT LEVEL AS "LEVEL", 
        CONNECT_BY_ROOT (EMPNO) AS "ROOT",
        SYS_CONNECT_BY_PATH (EMPNO, ', ') AS "PATH",
        EMPNO, ENAME, MGR
    FROM EMP
    START WITH ENAME = 'KING'
    CONNECT BY MGR = PRIOR EMPNO;
    ```
    

- 계층적 질의 검색 방법
    1. 순방향 검색 
        - 위에서 아래로 전체 구조를 따라 내려가는 방식
        - 가장 상위의 한 점으로부터 하위로 뻗어나가는 방식
    2. 역방향 검색
        - 아래에서 위로 추적하는 방식
        - 가장 하위의 한 점에서 가장 상위로 이동하는 하나의 PATH
    
    ```sql
    -- 일반적인 순방향 검색의 경우 CONNECT BY 조건식의 PRIOR 키워드의 위치를 변경하여 
    -- 역방향 검색을 수행할 수 있음
    SELECT LEVEL AS "LEVEL", 
        CONNECT_BY_ISLEAF AS "ISLEAF", 
        EMPNO, ENAME, MGR
    FROM EMP
    START WITH ENAME = 'MARTIN'
    CONNECT BY PRIOR MGR = EMPNO;
    ```