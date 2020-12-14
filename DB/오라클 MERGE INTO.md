# 오라클 MERGE INTO

table에 존재하는 데이터는 그대로 변경만 하고, 없는 데이터는 삽입하여 적절하게 통합하기 위한 예약어. 해당 key에 맞는 데이터가 이미 존재하면 UPDATE, 존재하지 않으면 INSERT를 하여 row에 충돌나지 않게 update, insert등의 작업을 한꺼번에 한다.



## 문법

```sql
MERGE INTO /* 테이블명 - Update 또는 Insert되는 테이블, 뷰 */
USING /* 조회 서브 쿼리, into와 동일하면 dual 사용 */
ON /* 1과 2의 조인 조건, key와 일치 여부 */
WHEN MATCHED THEN /* 일치되는 경우 update */
UPDATE SET
[칼럼] = [값]
, [칼럼] = [값]
...
WHEN NOT MATCHED THEN /* 일치 안되는 경우 insert */
INSERT (칼럼1, 칼럼2, ...)
VALUES (값1, 값2 ...)
```

> 조건절에 사용한 칼럼은 update 불가



삭제도 가능하다

```sql
MERGE INTO 테이블
USING 조회쿼리
ON 조인조건
WHEN MATCHED THEN 일치되는 경우 DELETE
DELETE 테이블 ...
WHERE 칼럼=값 ...
WHEN NOT MATCHED THEN 일치하지 않는 경우 INSERT
INSERT 칼럼 ...
VALUES 값 ...
```



## 예시

```SQL
MERGE INTO CUSTOMER C
USING
(
      SELECT USERNO
           , USERNAME
           , ADDRESS
           , PHONE
       FROM NEW_JOIN
      WHERE INPUT_DATE = '20170724'
) N
ON ( C.USERNO = N.USERNO)
WHEN MATCHED THEN
UPDATE
SET C.USERNAME = N.USERNAME
  , C.ADDRESS  = N.ADDRESS
  , C.PHONE    = N.PHONE
WHEN NOT MATCHED THEN
INSERT ( USERNO
       , USERNAME
       , ADDRESS
       , PHONE
       )
 VALUES (
         N.USERNO
       , N.USERNAME
       , N.ADDRESS
       , N.PHONE
 )
```

