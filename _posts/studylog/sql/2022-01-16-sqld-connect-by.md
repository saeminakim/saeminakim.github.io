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

### 계층형 질의 CONNECT BY

- 특정 테이블의 데이터 사이에서 자기 참조 형태의 데이터가 저장되어 있는 경우, 가장 상위에 위치한 데이터로부터 가장 하위에 위치한 데이터를 추적하기 위해서는 깊이의 수(DEPTH) 만큼 셀프조인이 필요함
- 자기 참조 형태의 데이터 깊이에 관계없이 손쉽게 처리하기 위한 문법
- 계층형 질의 CONNECT BY로부터 생성되는 정보
    1. LEVEL : 현재 레코드의 깊이 (최상단 1, 깊어질수록 +1)
    2. CONNECT_BY_ISLEAF : 현재 레코드가 마지막 노드인지 확인할 수 있는 값(1이면 마지막 노드, 0이면 마지막 노드 아님)
    

```sql
-- CONNECT BY
SELECT 컬럼명
FROM 테이블명
START WITH 조건식 -- (자기 참조를 시작할 레코드의 검색 조건 - EX)JONES를 검색하는 조건식)
CONNECT BY 조건식 -- (순방향 검색 : 부모 - 자식으로 이동 / 역방향 검색 : 자식 - 부모로 이동)

-- LPAD 함수 : 문자열의 패딩 값을 설정할 수 있는 함수
-- LPAD('패딩값', 패딩값의 중복 횟수)
SELECT LEVEL AS "LV", LPAD(' ', (LEVEL-1) * 2) || ENAME AS "ENAME",
    MGR, CONNECT_BY_ISLEAF AS "ISLEAF"
FROM EMP
-- START WITH 절
-- 계층형 질의 (CONNECT BY)를 시작할 레코드를 검색하는 절
START WITH MGR IS NULL
-- CONNECT BY 절
-- 현재 위치의 레코드에서 다음 계층(위/아래)으로 이동하기 위한 검색 조건을 정의하는 절
-- CONNECT BY 절의 조건은 현재 레코드의 컬럼명 = 이동한 레코드의 컬럼명의 형식으로 작성
-- PRIOR 키워드
-- 현재 레코드의 위치한 컬럼임을 명시적으로 선언하는 키워드
-- 아래의 CONNECT BY 절의 조건은 현재 위치한 레코드의 EMPNO의 값을 사용하여 EMP 테이블에서 
-- 동일한 값을 가지는 MGR 컬럼값의 레코드를 검색하는 조건식
-- PRIOR 키워드를 사용하여 CONNECT BY 조건식을 정의하는 방식에 따라 순방향/역방향 검색을 
-- 정의할 수 있음
CONNECT BY MGR = PRIOR EMPNO;

-- 특정 사원을 기준으로 CONNECT BY를 실행한 예제
-- START WITH 절에 의해 검색된 사원이 레벨 1이 되는 것을 확인할 수 있음
SELECT LEVEL AS "LV", LPAD(' ', (LEVEL-1) * 2) || ENAME AS "ENAME",
    MGR, CONNECT_BY_ISLEAF AS "ISLEAF"
FROM EMP
START WITH ENAME = 'JONES'
CONNECT BY MGR = PRIOR EMPNO;

-- 역방향 검색
SELECT LEVEL AS "LV", LPAD(' ', (LEVEL-1) * 2) || ENAME AS "ENAME",
    MGR, CONNECT_BY_ISLEAF AS "ISLEAF"
FROM EMP
START WITH ENAME = 'SMITH'
CONNECT BY PRIOR MGR = EMPNO;
--CONNECT BY EMPNO = PRIOR MGR;
```
