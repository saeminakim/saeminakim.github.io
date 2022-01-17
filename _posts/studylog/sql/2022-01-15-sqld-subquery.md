---
layout: post
title: SQLD 수업 정리_220115
slug: SQLD 수업 정리_220115
categories: studylog
tags: SQL, SQLD
description: >
  SQL 수업 내용 정리
---
# SQLD 수업 정리_220115

### Sub Query

- SQL은 하나의 메인쿼리와 다수개의 서브쿼리로 구성되 수 있음
- 메인쿼리는 SQL문의 최상단에 위치하며 개발자에게 결과를 반환하는 쿼리
- 서브쿼리는 메인쿼리에 종속되는 쿼리로 하나의 레코드를 반환하거나 다수개의 결과를 반환할 수 있는 쿼리문
- 서브쿼리의 사용 위치는 WHERE절, FROM절, SELECT절
    
    ```sql
    -- 선수들의 정보를 출력 (선수ID, 선수명, 소속팀ID, 신장, 몸무게)
    -- 정남일 선수와 동일한 팀에 소속된 선수들만 출력
    SELECT PLAYER_ID, PLAYER_NAME, TEAM_ID, HEIGHT, WEIGHT
    FROM PLAYER
    WHERE TEAM_ID = (SELECT TEAM_ID
                    FROM PLAYER
                    WHERE PLAYER_NAME = '정남일');
    
    -- 조인으로도 할 수 있지만.. 그닥 좋은 방법은 아님  
    SELECT P1.PLAYER_ID, P1.PLAYER_NAME, P1.TEAM_ID, P1.HEIGHT, P1.WEIGHT
    FROM PLAYER P1 INNER JOIN PLAYER P2
    ON P2.PLAYER_NAME = '정남일'
    WHERE P1.TEAM_ID = P2.TEAM_ID;
    
    -- 전체 선수들의 평균 키 이상인 선수들의 목록을 조회
    SELECT *
    FROM PLAYER
    -- 특정 테이블의 집계 결과를 기반으로 검색조건을 지정할 때 
    -- 아래와 같이 서브 쿼리 활용 가능
    WHERE HEIGHT >= (SELECT AVG(HEIGHT)
                    FROM PLAYER);
    
    -- 소속팀 선수들의 평균 키 이상인 선수들의 목록을 조회
    -- 서브쿼리는 메인쿼리의 컬럼에 접근이 가능함
    SELECT *
    FROM PLAYER P1
    -- 아래의 서브쿼리와 같이 서브쿼리에서는 메인 쿼리의 컬럼 정보에 접근할 수 있음
    WHERE HEIGHT >= (SELECT AVG(HEIGHT) 
                    FROM PLAYER P2
                    WHERE P2.TEAM_ID = P1.TEAM_ID);
    
    -- 정현수 선수가 소속된 팀의 연고지와 팀 이름 조회
    -- 서브쿼리를 활용하여 특정 컬럼의 값과 일치하는 레코드만 추출하는 경우
    -- 서브쿼리 결과는 반드시 1개 이하로 조회되어야 함
    -- 아래의 경우 서브쿼리 결과가 2개라 에러 발생
    SELECT TEAM_ID, REGION_NAME, TEAM_NAME
    FROM TEAM
    WHERE TEAM_ID = (SELECT TEAM_ID FROM PLAYER WHERE PLAYER_NAME = '정현수');
    ```
    

### 다중 행 서브쿼리를 활용한 비교 방법

- 서브쿼리의 결과가 2개 이상인 경우 다중 행 서브쿼리의 결과를 비교하기 위한 연산자
1. IN 
    
    : 특정 컬럼의 값이 다중 행 서브쿼리의 결과 중 하나라도 존재하는 경우 참 => 등가조건
    
2. ALL 
    
    : 특정 컬럼의 값이 다중 행 서브쿼리의 결과에 대해 모두 연산자를 만족하는 경우 참
    
    : EX) WEHERE HEIGHT >= ALL (서브쿼리)
    
3. ANY 
    
    : 특정 컬럼의 값이 다중 행 서브쿼리의 결과에 대해 단 하나라도 연산자를 만족하는 경우 참
    
    : EX) WEHERE HEIGHT >= ANY (서브쿼리)
    
4. EXISTS 
    
    : 다중행 서브쿼리의 결과가 단 하나라도 있으면 참
    
    : EX)WHERE EXISTS (서브쿼리)
    
    ```sql
    SELECT TEAM_ID, REGION_NAME, TEAM_NAME
    FROM TEAM
    WHERE TEAM_ID IN (SELECT TEAM_ID FROM PLAYER WHERE PLAYER_NAME = '정현수');
    
    -- 포지션이 MF, FW이고 소속팀이 K03인 선수들의 키보다 더 큰 선수들을 조회
    SELECT *
    FROM PLAYER
    WHERE HEIGHT > ANY(SELECT HEIGHT
                    FROM PLAYER
                    WHERE POSITION IN ('MF', 'FW')
                        AND TEAM_ID = 'K03');
    
    -- 소속팀별로 신장이 가장 작은 선수들을 조회
    -- 다중 컬럼 서브쿼리를 활용하는 예제
    SELECT *
    FROM PLAYER
    WHERE (TEAM_ID, HEIGHT) IN (SELECT TEAM_ID, MIN(HEIGHT) 
                                FROM PLAYER 
                                GROUP BY TEAM_ID);
    ```
    

### 연관 서브쿼리

- 일반적으로 대다수의 서브쿼리는 메인쿼리와 독립적으로 작성하는 경우가 많음
- 서브쿼리만 독립적으로 실행해도 결과가 반환된다는 뜻
- 반면 연관 서브쿼리는 메인쿼리의 특정 컬럼의 값을 참조하여 서브쿼리의 실행에 참조하는 서브쿼리를 의미
- 연관 서브쿼리를 사용할 때 메인쿼리의 테이블은 반드시 별칭이 선언되어야 함
- 별칭을 가지고 연관 서브쿼리에서 메인쿼리의 테이블 정보를 참조할 수 있음
    
    ```sql
    -- EMPLOYEES 테이블에서 각 부서별 평균 급여 이상을 수령하는 사원을 조회
    SELECT *
    FROM EMPLOYEES E1
    -- 아래의 서브쿼리는 연관 서브쿼리로 외부의 메인쿼리 EMPLOYEES 테이블의 
    -- DEPARTMENT_ID 컬럼을 참조
    -- 이를 위해 메인쿼리의 FROM절에서 반드시 별칭을 선언해야 함
    WHERE SALARY >= (SELECT AVG(SALARY)
                    FROM EMPLOYEES E2
                    WHERE E1.DEPARTMENT_ID = E2.DEPARTMENT_ID);
    ```
    

### 서브쿼리가 활용되는 기타의 경우

1. SELECT 절에서 사용되는 경우
    
    ```sql
    -- (PLAYER 테이블에서 선수아이디, 선수명, 신장, 몸무게, 
    -- 해당 소속팀의 평균 키, 몸무게를 조회)
    SELECT P1.PLAYER_ID, P1.PLAYER_NAME, P1.HEIGHT, P1.WEIGHT, P1.TEAM_ID,
        (SELECT ROUND(AVG(HEIGHT), 2) 
        FROM PLAYER P2 
        WHERE P1.TEAM_ID = P2.TEAM_ID) AS "평균키",
        (SELECT ROUND(AVG(WEIGHT), 2) 
        FROM PLAYER P2 
        WHERE P1.TEAM_ID = P2.TEAM_ID) AS "평균몸무게"
    FROM PLAYER P1;
    ```
    
2. FROM절에서 사용되는 경우(인라인 서브쿼리, 인라인 뷰)
    
    ```sql
    -- (PLAYER 테이블에서 포지션이 MF인 선수들을 대상으로 소속팀 ID, 
    -- 소속팀 이름, 선수명, 포지션, 백넘버 조회
    SELECT TEAM_ID, TEAM_NAME, PLAYER_NAME, POSITION, BACK_NO
    FROM TEAM INNER JOIN (SELECT TEAM_ID, PLAYER_NAME, POSITION, BACK_NO 
                            FROM PLAYER
                            WHERE POSITION = 'MF') USING(TEAM_ID);
    ```
    
3. HAVING절에서 사용되는 경우
    
    ```sql
    -- (각 소속팀 ID, 소속팀명, 소속팀 선수들의 평균키 조회. 
    -- 단, 평균키가 삼성블루윙즈 팀의 평균키보다 작은 팀만 대상으로 출력)
    -- 서브쿼리는 내부에 서브 쿼리를 가질 수 있음 -> 서브쿼리 중첩 가능
    SELECT TEAM_ID, ROUND(AVG(HEIGHT), 2)
    FROM TEAM T INNER JOIN PLAYER P USING (TEAM_ID)
    GROUP BY TEAM_ID
    HAVING AVG(HEIGHT) < (SELECT AVG(HEIGHT) 
                            FROM PLAYER
                            WHERE TEAM_ID = (SELECT TEAM_ID
                                            FROM TEAM
                                            WHERE TEAM_NAME = '삼성블루윙즈'))
    ORDER BY TEAM_ID;
    ```