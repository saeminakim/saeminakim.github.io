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

### 정규식

- 문자열 검색을 위한 패턴 지정

- 연산자
    - . (DOT) : 모든 문자를 의미
    - | (PIPE) : OR 연산자와 동일 → 좌항 또는 우항
    - \ : 특정 연산자를 문자 그대로 비교할 수 있는 연산자
        
           : (\. \| 등과 같이 정규식의 연산자를 문자로 비교할 수 있도록 지원
        
    - ^ (캐럿) : 문자열의 시작 지점
    - $ : 문자열의 종료 지점
    - ? : 0회 또는 1회 일치하는 패턴 검색
    - * : 0회 또는 1회 이상 일치하는 패턴 검색
    - + : 1회 또는 1회 이상 일치하는 패턴 검색
    - {숫자N} : N번 반복되어야 함
    - {숫자N,} : 최소 N번 반복되어야 함
    - {,숫자N} : 최대 N번 반복되어야 함
    - {숫자N1, 숫자N2} : 최소 N1번, 최대 N2번 반복되어야 함
    - () : 소괄호에 정의된 내용을 하나로 처리
    - \숫자N : 기존에 작성된 정규식을 참조하는 연산자
    - [문자1문자2 ...] : [ ] 내부의 문자 중 일치하는 것이 있으면 검색
    - [^문자1문자2 ...] :
    - [0-9] : 문자 0~9까지 중 일치하는 것이 있으면 검색 대상
    - [a-z] : 소문자 a~z까지 중 일치하는 것이 있으면 검색 대상
    - [A-Z] : 대문자 A~Z까지 중 일치하는 것이 있으면 검색 대상
    - [:digit:] : 숫자 문자를 검색[0-9]
    - [:lower:] : 소문자를 검색[a-z]
    - [:upper:] : 대문자를 검색[A-Z]
    - [:alpha:] : 영문자를 검색[A-Za-z]
    - [:alnum:] : 숫자, 영문자를 검색[A-Za-z0-9]
    - [:blank:] : 공백 검색 (띄어쓰기, 탭문자 등)
    - [:space:] : 공백 검색 (띄어쓰기, 탭문자 등)
    - \d : 숫자를 의미 [0-9], [:digit:] 과 동일
    - \D : 숫자가 아닌 문자를 의미 [^0-9] [^[:digit:]] 과 동일
    - \w : 숫자와 영문자를 의미 (언더바가 포함됨) [0-9a-zA-Z_] [[:alnum:]_]
    - \W : 숫자와 영문자 그리고 언더바가 아닌 문자를 의미 [^0-9a-zA-Z_] [^[:alnum:]_]
    - \s : 공백문자 의미 [[:space:]]
    - \S : 공백문자가 아닌 문자를 의미 [^[:space:]]
    
    - ? : 0회 또는 1회 (greedy)
    - ?? : 0회 또는 1회 (non-greedy)
    
    - * : 0회 또는 1회 (greedy)
    - *? : 0회 또는 1회 (non-greedy)
    
    - + : 0회 또는 1회 (greedy)
    - +? : 0회 또는 1회 (non-greedy)
    

```sql
-- 정규식 (REGULAR EXPRESSION)
-- 문자열 검색을 위한 패턴 지정

SELECT REGEXP_SUBSTR('AAB', 'A.B'),
    REGEXP_SUBSTR('ABB', 'A.B'),
    REGEXP_SUBSTR('ACB', 'A.B'),
    REGEXP_SUBSTR('ABC', 'A.B') -- 마지막 글자가 패턴과 일치하지 않음 : NULL 출력
FROM DUAL;

SELECT REGEXP_SUBSTR('A', 'A|B'),
    REGEXP_SUBSTR('B', 'A|B'),
    REGEXP_SUBSTR('C', 'A|B'),
    REGEXP_SUBSTR('AB', 'AB|CD'),
    REGEXP_SUBSTR('CD', 'AB|CD'),
    REGEXP_SUBSTR('BC', 'AB|CD'),
    REGEXP_SUBSTR('AA', 'A|AA'),
    REGEXP_SUBSTR('AA', 'AA|A')
FROM DUAL;

-- 정규식의 연산자 자체를 문자 비교를 하기 위해서 아래와 같이 역슬래쉬를 사용하여 처리할 수 있음
SELECT REGEXP_SUBSTR('A|B', 'A|B'),
    REGEXP_SUBSTR('A|B', 'A\|B')
FROM DUAL;

SELECT REGEXP_SUBSTR('ABCDEFG', '^.'),
    REGEXP_SUBSTR('ABCDEFG', '.$')
FROM DUAL;

SELECT REGEXP_SUBSTR('AC', 'AB?C'), -- AC이거나 ABC
    REGEXP_SUBSTR('ABC', 'AB?C'),
    REGEXP_SUBSTR('ABBC', 'AB?C')
FROM DUAL;

SELECT REGEXP_SUBSTR('AC', 'AB*C'), -- AC, ABC, ABBC, ABBBC ... 
    REGEXP_SUBSTR('ABC', 'AB*C'),
    REGEXP_SUBSTR('ABBBBBC', 'AB*C'),
    REGEXP_SUBSTR('ABBBBBB', 'AB*C')  -- 마지막 문자가 일치하지 않음 NULL
FROM DUAL;

SELECT REGEXP_SUBSTR('AC', 'AB+C'), -- ABC, ABBC, ABBBC ... 
    REGEXP_SUBSTR('ABC', 'AB+C'),
    REGEXP_SUBSTR('ABBBBBC', 'AB+C'),
    REGEXP_SUBSTR('ABBBBBB', 'AB+C') 
FROM DUAL;

SELECT REGEXP_SUBSTR('AB', 'A{2}'),  -- AA
    REGEXP_SUBSTR('AAB', 'A{2}'),
    REGEXP_SUBSTR('AAB', 'A{3,}'),  -- AAA, AAAA, AAAAA, .....
    REGEXP_SUBSTR('AAAAB', 'A{3,}'),
    REGEXP_SUBSTR('AAAAB', 'A{,3}'),
    REGEXP_SUBSTR('AAAB', 'A{3,5}'),  -- AAA, AAAA, AAAAA
    REGEXP_SUBSTR('AAAAAB', 'A{3,5}')
FROM DUAL;

SELECT REGEXP_SUBSTR('ABC', '(AB)+C'), -- ABC, ABABC, ABABABC .....
    REGEXP_SUBSTR('ABABC', '(AB)+C'),
    REGEXP_SUBSTR('ABABBC', '(AB)+C'),
    REGEXP_SUBSTR('ABABC', 'AB+C'),
    
    REGEXP_SUBSTR('ABD', 'AB|CD'), -- AB, CD
    REGEXP_SUBSTR('ABD', 'A(B|C)D')  -- ABD, ACD
FROM DUAL;

SELECT REGEXP_SUBSTR('ABXAB', '(AB|CD)X'), -- ABX, CDX
    REGEXP_SUBSTR('ABXAB', '(AB|CD)X\1'),  -- ABXAB, CDXAB, ABCCD, CDXCD
    
    -- 동일한 문자열이 1번이상 반복되는 패턴을 검색
    REGEXP_SUBSTR('ABABAB', '(.*)\1+'),
    REGEXP_SUBSTR('ABCABCAB', '(.*)\1+'),
    REGEXP_SUBSTR('ABCBCCAB', '(.*)\1+')
FROM DUAL;

-- OR 연산자를 대체할 수 있는 연산자
SELECT REGEXP_SUBSTR('AC', '[AB]C'),  -- AC, BC
    REGEXP_SUBSTR('BC', '[AB]C'),
    REGEXP_SUBSTR('CC', '[AB]C')
FROM DUAL;

SELECT REGEXP_SUBSTR('1a', '[0-9][a-z]'),
    REGEXP_SUBSTR('1D', '[0-9][A-Z]'),
    REGEXP_SUBSTR('Aa', '[a-z][A-Z]'),
    REGEXP_SUBSTR('Aa', '[^a-z][^A-Z]')
FROM DUAL;

SELECT REGEXP_SUBSTR('aA8', '[[:digit:]]'),
    REGEXP_SUBSTR('aA8', '[[:lower:]]'),
    REGEXP_SUBSTR('aA8', '[[:upper:]]'),
    REGEXP_SUBSTR('aA8', '[[:alpha:]]*'),
    REGEXP_SUBSTR('aA8', '[[:alnum:]]'),
    REGEXP_SUBSTR('aA8', '[[:alnum:]]+')
FROM DUAL;

SELECT REGEXP_SUBSTR('(02) 2655-8975', '\(\d{2,3}\)\s+\d{3,4}-\d{4}'),
    REGEXP_SUBSTR('02-2655-8975', '\(\d{2,3}\)\s+\d{3,4}-\d{4}'),
    REGEXP_SUBSTR('b2b', '\w\d\D'),
    REGEXP_SUBSTR('hello123@abc.co.kr', '\w{5,20}@\w+(\.[[:alpha:]]+)+')
FROM DUAL;

SELECT REGEXP_SUBSTR('aaaa', 'a?aa'),  -- GREEDY : 만족하는 최대치를 찾음
    REGEXP_SUBSTR('aaaa', 'a??aa')    -- NON-GREEDY  : a가 없다는 가정 하에(최소한의 조건을 기준으로함) 뒤에 있는 애들을 먼저 찾아봄 -> aa 출력
FROM DUAL;

SELECT REGEXP_SUBSTR('xaxbxc', '\w*?x\w'),
    REGEXP_SUBSTR('xaxbxc', '\w*x\w')   
FROM DUAL;

-- 정규식에 관련된 함수
-- REGEXP_LIKE
-- REGEXP_REPLACE
-- REGEXP_SUBSTR
-- REGEXP_CONCAT

-- REGEXP_LIKE 함수는 원본 문자열이 지정한 패턴에 만족하는 경우 참을 반환하는 함수
SELECT FIRST_NAME, LAST_NAME
FROM EMPLOYEES
WHERE REGEXP_LIKE (FIRST_NAME, '^Ste(v|ph)en$');  -- Steven ot Stephen

-- REGEXP_REPLACE 함수 : 원본 문자열에 대해서 패턴에 일치하는 문자열을 지정된 형식의 문자열로 변환하여 반환하는 함수
-- 만약 지정한 패턴에 일치하지 않는 경우 원본 문자열을 반환
SELECT PHONE_NUMBER,
    REGEXP_REPLACE (PHONE_NUMBER, '(\d{3})\.(\d{3})\.(\d{4})', '(\1) \2-\3') AS "PHONE_NUMBER"
FROM EMPLOYEES;

-- REGEXP_SUBSTR 함수 : 원본 문자열에서 지정한 패턴에 만족하는 부분 문자열을 추출하여 반환하는 함수
-- REGEXP_SUBSTR('원본문자열', '패턴문자열', 시작위치(1), 패턴일치횟수(1), 'c : 대소문자구분 / i : 대소문자구분X', 전체 패턴에 대해서 출력 여부(0)) 
SELECT
    REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'c', 0),
    REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'c', 1),   -- 마지막 1은 첫번째 패턴에 대해서만 가져오겠다는 뜻
    REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'c', 2),
    REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'c', 3),
    REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 1, 1, 'c', 4)
FROM DUAL;    

-- 주의사항
-- REGEXP_SUBSTR 함수의 마지막 인자 (전체 패턴 문자열 / 일부 패턴 문자열 추출여부)는 전체 패턴의 실행 결과에서 부분을 추출하는 역할을 함
SELECT
    -- 시작위치 변경으로 패턴이 일치하지 않기 때문에 부분 패턴 추출 불가
    REGEXP_SUBSTR('1234567890', '(123)(4(56)(78))', 2, 1, 'c', 2)
FROM DUAL;

-- REGEXP_INSTR 함수 : 원본 문자열에서 지정한 패턴이 위치하고 있는 인덱스(위치값)를 반환하는 함수
-- REGEXP_INSTR(원본, 패턴, 시작위치(1), 패턴일치횟수(1), 반환옵션(0 - 시작위치,1 - 패턴종료위치)
SELECT
    REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'c', 0),
    REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 1, 'c', 0)
FROM DUAL;

-- REGEXP_INSTR 함수를 사용하여 부분 패턴의 시작위치를 확인할 수 있음
SELECT
    REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'c', 2),
    REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'c', 3),
    REGEXP_INSTR('1234567890', '(123)(4(56)(78))', 1, 1, 0, 'c', 4)
FROM DUAL;

-- REGEXP_COUNT 함수 : 원본 문자열에서 지정한 패턴이 존재하는 개수를 반환
-- REGEXP_COUNT (원본문자열, 패턴문자열, 시작위치(1))
SELECT
    REGEXP_COUNT('123123123123123', '(123)', 1),
    REGEXP_COUNT('123123123123', '(123)', 4)
FROM DUAL;

-- 사원 테이블에서 FIRST_NAME 컬럼에서 A가 두 개이상 포함된 사원을 추출
SELECT *
FROM EMPLOYEES
WHERE REGEXP_COUNT (FIRST_NAME, 'e') >= 2;
```