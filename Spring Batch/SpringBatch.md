## JobInstance

Job이 실행될 때 하나의 실행 단위

JobExecution과 1:N의 관계를 가짐



## JobExecution

| 이름                 | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| jobParameter         | Job 실행에 필요한 파라미터 정보                              |
| jobInstance          | Job 실행의 단위가 되는 객체                                  |
| stepExecution        | StepExecution을 여러개 가질 수 있는 Collection 타입          |
| status               | Job의 실행 상태<br />(COMPLETED, STARTING, STARTED, STOPPED, FAILED, ABANDONED, UNKOWN (DEFAULT: STARTING)) |
| startTime            | Job이 실행된 시간, null이면 시작하지 않았음                  |
| createTime           | JobExecution이 생성된 시간                                   |
| endTime              | JobExecution이 끝난 시간                                     |
| lastUpdated          | 마지막으로 수정된 시간                                       |
| exitStatus           | Job 실행 결과에 대한 상태<br />(UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED) |
| executionContext     | Job 실행 사이에 유지해야하는 사용자 데이터                   |
| failureException     | Job 실행 중 발생한 예외를 List 타입으로 저장                 |
| jobConfigurationName | Job 설정 이름을 나타냄                                       |



## JobParameters

Job이 실행될 때 필요한 파라미터들을 Map 타입으로 저장하는 객체

JobInstance를 구분하는 기준, Job을 실행할 때 JobParameter로 date 정보를 넘기면 하나의 JobInstance가 생성, 그러나 같은 파라미터를 넘기면 생성되지 않고, 실행되지 않을 것.

JobParameters - JobInstance = 1:1



`@StepScope`, `@JobScope` Bean을 생성할때만 Job Parameters가 생성됨



## Step

Step은 배치처리를 정의하고 제어하는 데 필요한 모든 정보가 들어있는 도메인 객체

Job을 처리하는 실질적인 단위로 사용됨



## StepExecution

Step의 실행 정보를 담는 객체

| 이름             | 설명                                                         |
| ---------------- | ------------------------------------------------------------ |
| JobExecution     | 현재의 JobExecution 정보를 담고 있음                         |
| stepName         | Step의 이름                                                  |
| status           | Step의 실행 상태<br />(COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN (DEFAULT: STARTING)) |
| readCount        | 성공적으로 읽은 레코드 수                                    |
| writeCount       | 성공적으로 쓴 레코드 수                                      |
| commitCount      | Step의 실행에 대해 커밋된 트랜잭션 수                        |
| rollbackCount    | Step의 실행에 대해 롤백된 트랜잭션 수                        |
| processSkipCount | 쓰기에 실패해 건너뛴 레코드 수                               |
| writeSkipCount   | 쓰기에 실패해 건너뛴 레코드 수                               |
| startTime        | Step이 실행된 시간, null이면 시작하지 않았음                 |
| endTime          | Step의 실행 성공 여부와 관련 없이 Step이 끝난 시간           |
| lastUpdated      | 마지막으로 수정된 시간                                       |
| executionContext | Step 실행 사이에 유지해야하는 사용자 데이터가 들어 있음      |
| exitStatus       | Step 실행 결과에 대한 상태를 나타냄<br />(UNKNOWN, EXECUTING, COMPLETED, NOOP, FAILED, STOPPED (DEFAULT: UNKNOWN)) |
| terminateOnly    | Job 실행 중지 여부                                           |
| filterCount      | 실행에서 필터링된 레코드 수                                  |
| failureExecution | Step 실행 중 발생한 예외를 List에 저장                       |



## Batch Status

Job, Step의 실행 결과를 Spring에서 기록할 때 사용하는 Enum

## ExitStatus

Step의 실행 후 상태

`on("FAILED").to(stepB())` = exitCode가 FAILED로 끝나면 stepB로 가라



## Scope

### @JobScope

Step 선언문에 사용

Job 실행시에 Bean 생성, 종료시에 Bean 삭제

### @StepScope

Tasklet이나 ItemReader, ItemProcessor, ItemWriter에 사용

Step 실행시에 Bean 생성, 종료시에 Bean 삭제

### 장점

- JobParameter의 Late Binding
- StepContext, JobExecutionContext 레벨에서 Job Parameter 할당
- 비지니스 로직 처리단계에서 Job Parameter를 할당
- 동일 컴포넌트 병렬, 동시 사용에 유용
- @StepScope가 있다면, 각 Step의 별도의 Tasklet을 생성 관리하기 때문에, 서로의 상태 침법 불가



## Chunk

데이터 덩어리로 작업할 때 각 커밋 사이에 처리되는 row 수

Chunk 지향이란 한번에 하나씩 데이터를 읽어 Chunk라는 덩어리를 만든 뒤, Chunk 단위로 트랜잭션을 다루는 것을 의미

실패할 경우 Chunk 만큼 롤백

