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

- 트랜잭션
    - 트랜잭션(작업의 단위)은 DML(SELECT, INSERT, UPDATE, DELETE)로 구성됨
    - 일반적으로 트랜잭션은 명시적인 COMMIT 또는 ROLLBACK 명령에 의해서 종료됨
    - 하지만 트랜잭션의 수행 중 단 하나의 DDL(CREATE, DROP, ALTER) 명령이 실행되면 현재까지 수행한 트랜잭션의 구성 SQL(DML)은 즉시 COMMIT으로 처리됨
    
    ```sql
    -- 트랜잭션
    -- 다수개의 SQL문이 하나의 작업을 처리하는 경우 모든 SQL문이 성공적으로 수행되었을 때에만 작업한 내용을 반영하기 위한 기법
    -- 트랜잭션을 구성하는 명령어
    -- COMMIT / ROLLBACK
    
    -- mysql의 경우 기본이 자동 커밋(트랜잭션을 수행하지 않고 매번 데이터베이스에 직접 반영)
    -- 자동 커밋을 해제해주는 명령어
    set autocommit = OFF;
    
    -- 테이블의 구조를 확인하는 명령
    -- [describe / desc] 테이블명;
    describe customer;
    
    -- 테이블에 데이터를 입력하는 SQL 문
    insert into customer (customer_id, password, name, regDate, lastUpdate)
    values ('c01', 'pw_c01', '아무개01', now(), now());
    
    select * 
    from customer;
    -- 여기까지 실행하면 위에 입력한 데이터들이 테이블에 들어가 있는걸 확인할 수 있음
    
    -- 기존에 수행한 SQL을 취소하는 명령
    -- rollback
    rollback;
    
    select * 
    from customer;
    -- 다시 조회해보면 입력한 데이터들이 사라져있음
    
    -- commit 명령의 실습
    insert into customer (customer_id, password, name, regDate, lastUpdate)
    values ('c01', 'pw_c01', '아무개01', now(), now());
    
    -- 트랜잭션의 과정 중 입력된 결과를 확인할 수 있음
    -- 실제로 DB에 반영되지 않은 결과
    select * 
    from customer;
    
    -- 트랜잭션을 종료하면서 현재까지 수행한 모든 SQL을 데이터베이스에 반영하는 명령
    commit;
    
    -- 기존에 COMMIT으로 종료된 트랜잭션은 다시 롤백할 수 없음
    rollback;
    
    -- 실제 DB에 반영된 결과
    select * 
    from customer;
    
    -- 트랜잭션의 활용 시 주의할 점
    -- 트랜잭션(작업의 단위)은 DML(SELECT, INSERT, UPDATE, DELETE)로 구성됨
    -- 일반적으로 트랜잭션은 명시적인 COMMIT 또는 ROLLBACK 명령에 의해서 종료됨
    -- 하지만 트랜잭션의 수행 중 단 하나의 DDL(CREATE, DROP, ALTER) 명령이 실행되면 현재까지 수행한 트랜잭션의 구성 SQL(DML)은 즉시 COMMIT으로 처리됨
    
    -- 1. 데이터 입력
    insert into customer (customer_id, password, name, regDate, lastUpdate)
    values ('c02', 'pw_c02', '아무개02', now(), now());
    
    -- 1-1. DDL 구문 실행
    -- 트랜잭션을 수행하는 도중 DDL이 실행되면 현재까지 수행한 SQL문은 즉시 COMMIT됨
    create table dummy (
    	msg varchar(100)
    );
    
    -- 2. 트랜잭션의 취소
    -- (앞서 실행된 DDL문에 의해서 이미 COMMIT된 상태이므로 취소되는 SQL이 없음)
    rollback;
    
    -- 3. 작업 결과를 확인
    select *
    from customer;
    ```