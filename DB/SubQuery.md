# SubQuery

하나의 SQL 문장에 nested된 select 문장

```sql
SELECT ...
FROM ...
WHERE ... (SELECT ...
          FROM ...
          WHERE ...)
```

## Single-Row Subquery

서브 쿼리가 하나의 결과만을 반환

```sql
SELECT name, job
FROM employee
WHERE job = (SELECT job
            FROM employee
            WHERE empno = 1234)
```

## Multiple-Row Subquery

하나 이상의 행을 반환하는 Subquery

```SQL
SELECT empno, ename, sal, deptno
FROM emp
WHERE sal IN (SELECT MAX(sal)
             FROM emp
             GROUP BY deptno)
```

## Multiple-Column Subquery

결과 값이 두 개 이상의 컬럼을 반환하는  Subquery

```sql
SELECT empno, sal, deptno
FROM emp
WHERE (sal, deptno) IN (SELECT sal, deptno
                       FROM emp
                       WHERE deptno = 30
                       AND comm IS NOT NULL)
```

## Inline View

From절에 오는 SubQuery

가상의 집합을 만들어 조인을 수행하거나 조회할 때 사용

```sql
SELECT no, name
FROM (SELECT ROWNUM AS rnum, no, name
     FROM employee
     WHERE salary > 10
     AND rnum >= 1)
WHERE ROWNUM <= 20
```



## Scalar Subquery

SELECT 절에서 사용하는 Subquery

한개의 row만 나온다

불필요한 조인을 하지 않기 위해 사용한다.

```sql
SELECT name (SELECT name FROM dept d WHERE d.deptno = e.deptno) deptno
FROM employee e
WHERE job = 'MANAGER'
```

## UNION

테이블을 결합한다

중복되지 않은 값들을 반환

```SQL
SELECT deptno FROM emp
UNION
SELECT detpno FROM dept
```

## UNION ALL

중복되는 값까지 반환

## INTERSECT

공통 행 반환

## MINUS

차집합