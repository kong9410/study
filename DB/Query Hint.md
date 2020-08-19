# 쿼리 힌트

SQL문 실행시 데이터를 스캐닝 하는 방법을 지시 해주는 구문으로, 오라클 옵티마이저가 아닌 개발자가 직접 최적의 실행 경로를 작성해 주는 것이다.



## 힌트로 할 수 있는 것

액세스 경로, 조인 순서, 병렬 및 직렬 처리, Optimizer의 목표를 변경

데이터값 정렬

드라이빙 테이블 선정



## 기본 사용법

쿼리 서두에 힌트를 명시한다

```sql
SELECT /*+ index_asc(e idx_myemp1_ename) */
		empno, ename, sal
FROM myemp1 e
WHERE ename >= '가'
```



## 옵티마이저 지정 값

| 힌트       | 설명                                                         |
| ---------- | ------------------------------------------------------------ |
| all_rows   | 전체 resource 소비를 최소화<br />Cost-Based 접근 방식으로 all_rows를 full table scan 선호<br />CBO(Cost Based Optimization)는 default로 all_rows를 선택 |
| first_rows | 조건에 맞는 첫번째 row를 리턴하기 위한 resource를 최소화 시키기 위한 힌트<br />Cost-Based 접근방식<br />Index Scan이 가능하면 옵티마이저가 Full Table Scan 대신 Index Scan을 선택<br />Index Scan이 가능하면 옵티마이저가 Sort-Merge보다 Nested Loop를 선택<br />Order By절에 Index Scan이 가능하면, Sort 과정을 피하기 위해 Index Scan을 선택<br />Delete/UpdateBlock에서 무시<br />다음을 포함한 Select 문에서 제외<br />(집합 연산자, Group By For UpDate, Group 함수, Distinct)<br />Full Table Scan 보다는 index scan을 선호하며 Interactive Application인 경우 best response time을 제공<br />sort merge join보다는 nested loop join 선호 |
| rule       | Rule Based 접근 방식을 사용하도록 지정                       |

## Access Methods - 접근 방법

| 힌트                                 | 설명                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| full(table_name)                     | table을 full scan하기 원할 때 사용                           |
| hash(table)                          | hash scan을 선택하도록 지정 (hashkeys parameter로 만들어진 cluster내에 저장된 table에만 적용) |
| cluster(table_name)                  | Cluster Scan을 선택하도록 지정, Clustered Object만 적용      |
| hash_aj                              | not in subquery를 hash anti-join으로 변형                    |
| hash_sj                              | correlated exists subquery를 hash semi-join으로 변형         |
| index(table_name index_name)         | 지정된 index를 강제적으로 쓰게끔 지정<br />in list predicat에 대해서도 가능<br />multi-column inlists는 index를 사용할 수 없다. |
| index_combine(table_name index_name) | Index명이 주어지지 않으면 Optimizer는 해당 테이블의 Best Cost로 선택된  Boolean Bombination Index를 사용, Index 명이 주어지면 특정 Bitmap Index의 Boolean Combination의 사용 |
| index_asc(table_name index_name)     | 지정된 index를 오름차순으로 쓰게끔 지정 (기본 오름차순)      |
| index_desc(table_name index_name)    | 지정된 index를 내림차순으로 쓰게끔 지정                      |
| index_ffs(table index)               | full table scan보다 빠른 full index scan을 유도              |
| rowid(table)                         | rowid로 table scan을 하도록 지정                             |
| merge_aj                             | not in subquery를 merge anti-join으로 변형                   |
| merge_sj                             | correalted exists subquery를 merge semi-join으로 변형        |
| and_equal(table index1, index2)      | single-column index의 merge를 이용한 access path 선택. 적어도 두 개 이상의 index가 지정되어야 함. max로 5개까지 지정 가능 |
| use_concat                           | 조건절의 or를 union all 형식으로 변형한다. 일반적으로 변형은 비용 측면에서 효율적일 때만 일어난다. |

## Join Orders

| 힌트    | 설명                                                         |
| ------- | ------------------------------------------------------------ |
| ordered | from절에 기술된 테이블 순서대로 join이 일어나도록 유도       |
| star    | star query plan이 사용 가능하다면 이를 이용하기 위한 hint. 규모가 가장 큰 테이블이 query에서 join order상 마지막으로 위치하게 하고, nested loop으로 join이 일어나도록 유도<br />적어도 3개 테이블 이상이 조인에 참여<br />large table에 concatenated index는 최소 3컬럼 이상을 index에 포함해야함<br />테이블이 analyze되어 있다면 optimizer가 가장 효율적인 star plan을 선택 |

## Join Operations

| 힌트                      | 설명                                                         |
| ------------------------- | ------------------------------------------------------------ |
| use_nl(table1 table2)     | 테이블 Join시 각 row가 inner 테이블을 nested loop형식으로 join. 지정된 table이 inner table이 된다. |
| use_hash (table_name)     | 각 테이블 간 hash join이 일어나도록 유도                     |
| use_merge (table_name)    | 지정된 테이블들의 조인이 sort_merge 형식으로 일어나도록 유도 |
| driving_site (table_name) | 쿼리 실행이 오라클에 의해 선택된 site가 아닌 다른 site에서 일어나도록 유도 |

## Parallel Execution

| 힌트                         | 설명                                              |
| ---------------------------- | ------------------------------------------------- |
| noparallel (table_name)      | prallel query option을 사용하지 않도록 할 수 있음 |
| parallel(table_name, degree) | 쿼리에 포함된 테이블의 degree를 설정할 수 있다.   |

## Additional Hints

| 힌트           | 설명                                                         |
| -------------- | ------------------------------------------------------------ |
| cache(table)   | full table scan시에 retrieve된 block을 lru list에서 most recently used end에 놓는다 |
| nocache(table) | full table scan시 retrieve된  block을 lru list에서 least recently used end에 놓는다. |
| merge(view)    | complex_view_merging=false로 되어 있을 때 view 또는 subquery의 내용을 merge 가능 |
| nomerge(view)  | complex_view_merging=true로 되어 있을 때 사용, view또는 subquery의 내용을 merge 불가능<br />view또는 subquery자체가 query문에 영향을 많이 받게됨 |
| push_subq      | nomerged subqueries가 execution plan에서 가장 빠른 위치에서 evaluation 되도록 한다. |

