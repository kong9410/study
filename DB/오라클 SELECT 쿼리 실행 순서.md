# 오라클 SELECT 쿼리 실행 순서

> FROM - WHERE - GROUP BY - HAVING - SELECT - ORDER BY 순으로 진행

- FROM은 테이블을 가져옴
- WHERE로 조건에 맞는 결과만 갖도록 데이터 간추림
- GROUP BY로 선택한 칼럼으로 GROUPING 작업
- GROUP BY 이후에 사용되는 조건 절, 그러나 WHERE에 비해 퍼포먼스가 많이 떨어지게 됨
  - HAVING은 GROUP BY이후 각 그룹에 HAVING에 맞는 조건 작업을 함
  - WHERE은 먼저 데이터를 간추린 다음 그룹화함
  - 따라서 일반적인 조건에서는 WHERE절에서 처리하는 방식이 효율적이다
- SELECT, 어떤 열을 출력해줄지 선택한다
- ORDER BY 행의 순서를 정한다

