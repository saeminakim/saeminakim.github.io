---
layout: post
title: SQLD 수업 정리_220108
slug: SQLD 수업 정리_220108
categories: studylog
tags: SQL, SQLD
description: >
  SQL 수업 내용 정리
---
# SQLD 수업 정리_220108

- oracle
    
    ```sql
    -- 현재 접속한 계정의 이름을 확인하는 명령
    show user;
    
    -- mysql show tables와 같은 명령어
    -- 현재 접속한 스키마에 저장된 테이블의 목록을 검색하는 명령
    -- (tab 테이블을 검색하여 테이블의 목록을 조회할 수 있음)
    select * 
    from tab;
    
    select *
    from countries;
    
    -- 특정 테이블의 구조를 검색하는 명령
    desc countries;
    
    select * 
    from departments;
    
    select *
    from employees;
    
    -- 테이블을 조회하는 명령
    -- SELECT 절
    -- SELECT [ALL(기본값) / DISTINCT] 컬럼명
    -- FROM 테이블명;
    
    -- all 옵션이 자동으로 적용된 SQL문
    select department_id
    from employees;
    
    -- all 옵션 : 우측의 컬럼, 컬럼들에 대해 전체를 출력하는 옵션 (중복여부 관계없이 전체 출력)
    select all department_id
    from employees;
    
    -- distinct 옵션 : 중복을 제거한 값을 보여주는 옵션
    select distinct department_id
    from employees;
    
    -- 컬럼명은 별칭을 사용해서 변경할 수 있음
    -- SELECT 컬럼명 AS 별칭
    select *
    from departments;
    
    -- 별칭에 공백이 포함되는 경우 쌍따옴표 사용
    -- (MS-SQL의 경우 '', "", []도 가능)
    select department_id as "부서 아이디", department_name as "부서 명"
    from departments;
    
    -- 사원 테이블을 검색하여 사원의 이름과 월 급여와 연봉 출력
    -- SELECT 컬럼명 -> 연산자(+, -, /, *)를 사용할 수 있음
    select first_name, last_name, salary, salary * 12 as "연봉"
    from employees;
    
    -- 컬럼 데이터의 결합
    -- CONCATENATION 연산
    -- 컬럼과 값 사이를 파이프 연산자(||)를 사용하여 결합할 수 있음
    
    -- 사원 테이블에서 데이터를 조회
    -- ex) Steven King 사원의 급여는 $24000입니다.
    select first_name || ' ' || last_name || ' 사원의 급여는 $' || salary || '입니다.' as msg
    from employees;
    
    -- CONCAT 함수를 사용한 컬럼 및 값의 결합 예시
    -- 주의할 점 
    -- CONCAT 함수는 매개변수를 두 개만 받을 수 있으므로 다수개의 컬럼 및 값을 결합하기 위해서는 CONCAT 함수를 중첩해서 사용해야 함
    select CONCAT (first_name, CONCAT(' ', last_name)) as name
    from employees;
    
    -- 함수
    -- 특정 칼럼의 값을 입력받아 새로운 형태의 값으로 변환하거나 목적에 맞는 연산의 결과를 반환하는 기능
    -- EX) SUM, AVG, COUNT
    
    -- dual 함수의 기능을 테스트하기 위한 dummy 테이블
    SELECT 15.1
    from dual;
    
    -- 수치형 데이터의 자릿수를 제어하는 함수
    -- 부동소수점의 자리 수 제어 - 반올림
    select round(15.127, 2)
    from dual;
    
    -- 수치형 데이터의 자릿수를 제어하는 함수
    -- 부동소수점의 자리 수 제어 - 절삭
    select trunc(15.127, 2)
    from dual;
    
    select floor(15.9357)
    from dual;
    
    select ceil(15.9357)
    from dual;
    
    select length('SQLD ddd ddd')
    from dual;
    
    -- length 함수 예제
    -- 경기장의 지역번호와 전화번호를 합친 번호의 길이를 출력
    select stadium_name, ddd, tel, length(concat(ddd, tel)) as length
    from stadium;
    
    -- ROUND 함수 예제
    -- EMPlOYEES 테이블에서 사원명과 월급을 출력하되 월급은 소수점 둘째자리까지만 출력 (반올림 허용)
    select first_name, last_name, round(salary, 2) as salary
    from employees;
    
    select first_name, last_name, 
        salary / 12, 2,
        round(salary / 12, 2) as round,
        floor(salary / 12) as floor, -- 같거나 작은 수
        ceil(salary / 12) as ceil -- 같거나 큰 수
    from employees;
    
    -- 날짜 관련 함수
    select sysdate
    from dual;
    
    -- 날짜에 대한 연도/월/일을 추출하는 함수
    select extract(year from sysdate)
        , extract(month from sysdate) 
        , extract(day from sysdate)
    from dual;
    
    -- employees 테이블에서 사원명과 입사일자컬럼, 입사년도, 입사월, 입사일자를 출력하는 SQL문 작성
    select first_name || ' ' || last_name as name
        , hire_date
        , extract(year from hire_date) as year
        , extract(month from hire_date) as month 
        , extract(day from hire_date) as day
    from employees;
    ```