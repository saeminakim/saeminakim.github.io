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

- 프로시저 (Stored Procedure)
    - 함수와 유사하지만 함수는 하나의 SQL문을 실행하는 반면 프로시저는 다수개의 SQL문을 실행할 수 있음
    - 기본 문법
        
        ```sql
        create procedure 프로시져명(매개변수) 
        begin
        --      프로시저가 호출될 때 실행할 SQL문
        end
        ```
        
    
    ```sql
    -- DELIMITER 명령어
    -- 기본적으로 SQL문은 마무리를 ;로 처리
    -- 프로시저를 생성할 때 다수개의 SQL문을 작성하여 하나의 프로시저를 구성하는데
    -- 이 때 ;이 잘못 인식되어 프로시저가 생성되지 않고 즉시 SQL문이 실행되는 경우가 발생할 수 있음
    -- DELIMITER 명령어를 통해서 임시적으로 SQL의 종료 기호를 변경할 수 있음
    DELIMITER $$
    create procedure getCustomer() 
    begin
    	select customer_id, name, age, regDate
        from customer;
    end $$
    -- 프로시저의 작성이 끝난 후 원래 종료 기호인 ;으로 변경
    DELIMITER ;
    
    -- 프로시저 실행 방법
    -- CALL 프로시저명(매개변수);
    call getCustomer();
    
    -- 매개변수를 사용하는 프로시저의 작성
    -- (매개변수 : 프로시저의 실행에 필요한 정보를 저장하는 변수)
    DELIMITER $$
    create procedure setCustomer(
    	c_id varchar(30), 
        c_pw char(30), 
        c_nm char(15),
        c_age int,
        c_addr varchar(300)) 
    begin
    	-- 프로시저의 매개변수를 활용한 INSERT 실행
        insert into customer
    	values (c_id, c_pw, c_nm, c_age, c_addr, now(), now());
    end $$
    DELIMITER ;
    
    drop procedure setCustomer;
    
    -- 매개변수를 사용한 프로시저 호출
    call setCustomer('c04', 'c04_pw', '아무개04', 35, '서울시 동작구');
    call setCustomer('c05', 'c05_pw', '아무개05', 25, '서울시 노원구');
    select *
    from customer;
    
    -- c01 회원의 주소 값을 경기로 김포시로 변경하고, lastupdate 컬럼의 값을 현재 시간으로 변경
    update customer
    set address = '경기도 김포시'
    	, lastUpdate = now()
    where customer_id = 'c01';
    
    DELIMITER $$
    create procedure setCustomerAddress(
    	c_id varchar(30), 
        c_addr varchar(300)) 
    begin
    	-- 프로시저의 매개변수를 활용한 주소 변경 실행
        update customer
        set address = c_addr
        where customer_id = c_id;
        -- 프로시저의 매개변수를 활용한 lastUpdate 변경 실행
        update customer
        set lastUpdate = now()
        where customer_id = c_id;
    end $$
    DELIMITER ;
    
    call setCustomerAddress('c05', '서울시 강서구');
    
    -- 프로시저 테스트를 위한 테이블 생성
    -- 게시판 테이블 board
    create table board (
    	board_id int auto_increment primary key,
        writer varchar(30) not null,
        title varchar(200) not null,
        contents text not null,
        readCount int default 0,
        regDate datetime default now(),
        lastUpdate datetime default now(),
        foreign key (writer) references customer (customer_id)
    );
    
    select * 
    from board;
    
    -- board 테이블에 게시글의 정보를 입력할 수 있는 setBoard 프로시저를 작성한 후, 테스트를 수행
    DELIMITER $$
    create procedure setBoard(
    	writer varchar(30),
        title varchar(200),
        contents text
    ) 
    begin
        insert into board (writer, title, contents)
    	values (writer, title, contents);
    end $$
    DELIMITER ;
    
    call setBoard('c01', 'Happy 떡볶이 day', '으으으으으음~! 맛있는 떡볶이~!');
    
    -- board 테이블 게시글의 정보를 검색할 수 있는 getBoard 프로시저를 작성한 후 테스트를 수행
    -- getBoard 프로시저는 게시글의 ID값을 매개변수로 받음
    DELIMITER $$
    create procedure getBoard(id int) 
    begin
    	-- 특정 게시글의 조회수를 증가하는 SQL 실행
        -- 1. 특정 게시글의 조회수를 저장하기 위한 변수의 선언
        declare rc int;
        
        -- 2. 생성된 변수에 특정 게시글의 조회수를 저장
        select readCount + 1 into rc
        from board
        where board_id = id;
        
        -- 3. 조회수를 저장하고 있는 변수를 사용하여 조회수를 증가
        update board 
        set readCount = rc
        where board_id = id;
    
        select * 
        from board
        where board_id = id;
    end $$
    DELIMITER ;
    
    call getBoard(1);
    ```