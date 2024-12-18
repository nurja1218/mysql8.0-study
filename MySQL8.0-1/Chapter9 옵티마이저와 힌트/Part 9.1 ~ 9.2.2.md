- 쿼리 실행 절차
  [https://velog.io/@suker80/MYSql에서의-쿼리-실행-과정](https://velog.io/@suker80/MYSql%EC%97%90%EC%84%9C%EC%9D%98-%EC%BF%BC%EB%A6%AC-%EC%8B%A4%ED%96%89-%EA%B3%BC%EC%A0%95)
  ⇒ 쿼리캐시만 제외시키고 참고
  1. **SQL 파싱**
     - 사용자로부터 온 SQL문을 잘게 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리 (파스트리)
     - Ex
       ```sql
       SELECT name, age FROM users WHERE age > 30;
       ```
       - **토큰화(Tokenization):**
         - 쿼리는 먼저 개별 토큰으로 분해
           - **`SELECT`**, **`name`**, **`,`**, **`age`**, **`FROM`**, **`users`**, **`WHERE`**, **`age`**, **`>`**, **`30`** 등
         - 각 토큰은 특정 유형(예: 키워드, 식별자, 연산자, 리터럴)에 속함
       - **구문 분석(Syntax Parsing):**
         - 분해된 토큰들은 SQL의 문법 규칙에 따라 분석
         - 쿼리가 문법적으로 올바른지 검사
           - **`SELECT`** 뒤에는 컬럼 목록이 나와야 하며, **`FROM`** 뒤에는 테이블 이름이 나와야함
       - **파스트리 생성:**
         - 구문 분석 과정을 통해 파스트리가 생성
         - 파스트리는 쿼리의 구조를 트리 형태로 나타냄
           - 트리의 루트는 **`SELECT`** 문
           - 그 아래에는 **`FROM`** 절과 **`WHERE`** 절이 자식 노드로 연결
       - **의미 분석(Semantic Analysis):**
         - 파스트리를 바탕으로 쿼리의 의미를 분석
         - 참조되는 테이블과 컬럼이 실제로 존재하는지, 데이터 타입이 올바른지 등을 확인
       - **최적화 및 실행 계획 생성:**
         - 파스트리를 사용하여 쿼리의 최적화된 실행 계획을 생성
         - 가장 효율적인 데이터 접근 방법을 결정
           (예: 인덱스 사용 여부, 조인 방식 등)
  2. **최적화 및 실행 계획 수립**
     - 파스트리를 확인하면서, 어떤 테이블로부터 읽고 어떤 인덱스를 이용해 테이블을 읽을 지 선택
       - 불필요한 조건 제거
       - 복잡한 연산 단순화
       - 테이블의 조인이 필요한 경우, 어떤 순서로 읽을지 결정
     - 옵티마이저에서 처리
  3. **실행 계획대로 수행**
     - 스토리지 엔진으로부터 데이터를 가져옴
     - MySQL 엔진과 스토리지 엔진이 함께 참여함
- 옵티마이저의 종류
  - **규칙 기반 최적화**
    - 현재 거의 사용되지 않음
    - 테이블 row는 몇개인지 레코드는 어떤지 읽지 않고 최초의 정해진대로 기준을 삼고서 최적화 → 최적화 별로..
  - **비용 기반 최적화**
    - 쿼리를 처리하기 위한 여러 가지 방법을 만들고, 각 작업의 비용 정보와 대상 테이블의 통계 정보를 이용해서 실행 계획별 비용을 산출하고, `비용이 최소`인 처리 방식을 선택해 쿼리를 실행
    - 현재 MySQL 8.0에서는 비용 기반 최적화를 규칙으로 하여 쿼리 실행 계획을 수립
      - 위 **쿼리 실행 절차의 최적화** 내용을 기반으로 계획 수립
      - +) 비용 추정
        - 각 실행 계획의 리소스 소모를 평가하여, 쿼리 실행에 소요될 "비용"을 예측
        - 기준
          - 디스크 I/O, CPU 사용 시간, 메모리 사용량 등
        - 비용 추정 수치
          - 테이블에 저장된 행의 수와 각 행의 크기
          - 컬럼 내 값의 분포와 고유값의 수
          - 인덱스의 유형(예: B-tree, 해시), 카디널리티, 선택도 등 인덱스 특성
          - 조인 연산에 참여하는 테이블의 크기와 조인 방법
- 풀 테이블 스캔과 풀 인덱스 스캔
  - 풀 테이블 스캔
    - 인덱스를 사용하지 않고, 테이블의 데이터를 처음부터 끝까지 읽어서 요청된 작업을 처리
    - 사용예
      - 레코드 건수가 너무 적어서, 인덱스를 통해 읽는게 더 느린 경우 → 일반적으로 테이블이 페이지 1개
      - SQL문에서 인덱스를 이용할 수 있는 적절한 조건이 없는 경우
      - 조건에 일치하는 레코드 건수가 너무 많은 경우 (20~25% 이상)
  - 풀 인덱스 스캔
    - 인덱스를 처음부터 끝까지 스캔
- 병렬 처리
  - **하나의 쿼리**를 여러 쓰레드가 나누어 동시에 처리하는 것
  - MySQL 8.0부터 도입되었으나, 아직까지는 WHERE 조건 없이 단순히 테이블의 전체 건수를 가져오는 쿼리만 **\*\***병렬로 처리할 수 있음
  - 주로 백그라운드 작업과 일부 쿼리 연산에 대한 병렬 처리를 지원
    ⇒ 서버에 장착된 CPU 코어 개수를 넘어서면 오히려 성능이 떨어질 수 있음
    ```sql
    -- 4개의 스레드를 사용해 쿼리를 병렬 처리
    SET SESSION innodb_parallel_read_threads=4;
    SELECT COUNT(*) FROM salaries;
    ```
  - 그렇다면 그 이전엔?
    - Read Ahead 단일 스레드로 처리
      → InnoDB가 Read Ahead 작업을 병렬로 처리 - Read Ahead: 데이터베이스가 앞으로 필요할 것으로 예상되는 데이터를 미리 읽어 버퍼 풀에 저장하는 과정
    - 읽을 데이터를 예측하고 미리 읽어오는 작업과 동시에 다른 작업을 수행할 수 없음
      (데이터를 읽을 때마다 디스크에서 데이터를 읽어오기 때문에 I/O 대기 시간이 발생)
      → I/O 작업이 진행되는 동안에도 CPU는 다른 쿼리 처리나 작업을 계속 수행
    - I/O 대기 시간 동안에는 다른 작업을 처리할 수 없어 전체적인 성능 저하
      → 병렬 Read Ahead와 효율적인 버퍼 관리를 통해 필요한 데이터가 미리 버퍼 풀에 적재될 수 있음
    - 디스크 액세스의 부하가 증가 데이터를 읽는 소요시간 증가
      → 효율적인 데이터 관리를 통해 디스크 액세스가 최적화
