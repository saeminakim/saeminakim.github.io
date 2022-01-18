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

### 집합 연산자

- 다수개의 SQL 실행 결과를 취합하는 연산자 => 집합
- 합집합, 교집합, 차집합
- UNION
    
    : 서로 다른 SQL 실행 결과를 합집합으로 변환하는 연산자. 중복 제거
    
- UNION ALL
    
    : 서로 다른 SQL 실행 결과를 합집합으로 변환하는 연산자. 중복 제거 안됨
    
- INTERSECT
    
    : 서로 다른 SQL 실행 결과를 교집합으로 반환하는 연산자. 중복 제거
    
- EXCEPT
    
    : 서로 다른 SQL 실행 결과를 차집합으로 반환하는 연산자. 중복 제거
    
    ```sql
    -- K리그 소속 선수들 중 소속이 삼성인 선수들과 소속이 전남드래곤즈인 선수들에 대한 
    -- 정보를 조회 (팀ID, 팀이름, 선수ID, 선수이름, 백넘버)
    SELECT TEAM_ID, TEAM_NAME, PLAYER_NAME, BACK_NO
    FROM TEAM INNER JOIN PLAYER USING (TEAM_ID)
    WHERE TEAM_NAME IN ('삼성블루윙즈' , '드래곤즈');
    
    -- 위의 예제를 집합연산으로 처리하는 방법
    SELECT TEAM_ID, TEAM_NAME, PLAYER_NAME, BACK_NO
    FROM TEAM INNER JOIN PLAYER USING (TEAM_ID)
    WHERE TEAM_NAME = '삼성블루윙즈'
    UNION
    SELECT TEAM_ID, TEAM_NAME, PLAYER_NAME, BACK_NO
    FROM TEAM INNER JOIN PLAYER USING (TEAM_ID)
    WHERE TEAM_NAME = '드래곤즈';
    
    -- K리그 소속 선수들 중 소속이 삼성블루윙즈인 선수들과 포지션이 GK인 선수들의 목록을 조회
    SELECT *
    FROM PLAYER 
    WHERE POSITION = 'GK' OR TEAM_ID = (SELECT TEAM_ID 
                                        FROM TEAM 
                                        WHERE TEAM_NAME = '삼성블루윙즈');
    
    -- 집합연산을 사용하여 위의 쿼리문을 수정한 예제
    -- UNION 실행결과 88개
    SELECT *
    FROM PLAYER 
    WHERE POSITION = 'GK'
    UNION
    SELECT *
    FROM PLAYER 
    WHERE TEAM_ID = (SELECT TEAM_ID 
                    FROM TEAM 
                    WHERE TEAM_NAME = '삼성블루윙즈');
    
    -- 집합연산을 사용하여 위의 쿼리문을 수정한 예제
    -- UNION ALL 실행결과 92개(중복 제거되지 않음)
    SELECT *
    FROM PLAYER 
    WHERE POSITION = 'GK'
    UNION ALL
    SELECT *
    FROM PLAYER 
    WHERE TEAM_ID = (SELECT TEAM_ID 
                    FROM TEAM 
                    WHERE TEAM_NAME = '삼성블루윙즈');
    
    -- K리그 소속 선수들을 대상으로 포지션 별 키의 평균 값과 팀별 키의 평균 값을 조회
    -- 주의사항!!
    -- 집합연산으로 합집합되는 두 개의 쿼리문의 컬럼명은 상위의 쿼리문 컬럼명을 따라감
    -- 밑의 예에서 포지션 컬럼과 팀 컬럼 확인
    SELECT 'POSITION' AS "TAG", POSITION AS "POSITION", ROUND(AVG(HEIGHT), 2) "평균키"
    FROM PLAYER
    WHERE POSITION IS NOT NULL
    GROUP BY POSITION
    UNION
    SELECT 'TEAM' AS "TAG", TEAM_ID AS "TEAM", ROUND(AVG(HEIGHT), 2) "평균키"
    FROM PLAYER
    GROUP BY TEAM_ID;
    
    -- K리그 소속 선수들을 대상으로 삼성블루윙즈 소속이면서 포지션이 미드필더가 아닌 
    -- 선수들을 조회
    SELECT * 
    FROM PLAYER
    WHERE POSITION != 'MF' AND TEAM_ID = (SELECT TEAM_ID 
                                        FROM TEAM 
                                        WHERE TEAM_NAME = '삼성블루윙즈');
    
    -- 집합연산을 사용하여 처리하는 예제
    -- 차집합(오라클 : MINUS / MSSQL : EXCEPT)
    SELECT *
    FROM PLAYER
    WHERE TEAM_ID = (SELECT TEAM_ID 
                    FROM TEAM 
                    WHERE TEAM_NAME = '삼성블루윙즈')
    MINUS                
    SELECT * 
    FROM PLAYER
    WHERE POSITION = 'MF';
    
    -- K리그 소속 선수들 대상으로 삼성블루윙즈 소속이면서 포지션이 GK인 선수 조회
    SELECT *
    FROM PLAYER
    WHERE POSITION = 'GK' AND TEAM_ID = (SELECT TEAM_ID 
                                        FROM TEAM 
                                        WHERE TEAM_NAME = '삼성블루윙즈');
    
    -- 서브쿼리와 IN 연산자를 활용하는 예제                         
    SELECT *
    FROM PLAYER
    WHERE TEAM_ID = (SELECT TEAM_ID 
                    FROM TEAM 
                    WHERE TEAM_NAME = '삼성블루윙즈')
        AND PLAYER_ID IN (SELECT PLAYER_ID
                        FROM PLAYER
                        WHERE POSITION = 'GK');
                        
    -- 서브쿼리와 EXISTS 연산자를 활용하는 예제                         
    SELECT *
    FROM PLAYER P
    WHERE TEAM_ID = (SELECT TEAM_ID 
                    FROM TEAM 
                    WHERE TEAM_NAME = '삼성블루윙즈')
        AND EXISTS (SELECT PLAYER_ID
                    FROM PLAYER
                    WHERE P.PLAYER_ID = PLAYER_ID AND POSITION = 'GK');
    
    -- 집합연산을 사용하여 처리하는 예제
    -- INTERSECT 연산자 (교집합)
    SELECT *
    FROM PLAYER
    WHERE TEAM_ID = (SELECT TEAM_ID 
                    FROM TEAM 
                    WHERE TEAM_NAME = '삼성블루윙즈')
    INTERSECT
    SELECT *
    FROM PLAYER
    WHERE POSITION = 'GK';
    ```