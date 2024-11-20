- 인덱스를 이용한 정렬
  - 이미 B-Tree에 정렬되어 있는 순서대로 조회하기 때문에 별도의 정렬 필요 X
- 조인의 드라이빙 테이블만 정렬
  ```sql
  SELECT *
  FROM employees e, salaries s
  WHERE s.emp_no=e.emp_no AND e.emp_no BETWEEN 10002 AND 10010
  ORDER BY e.last_name // 인덱스 아닌 칼럼으로 정렬
  ```
  드라이빙 테이블인 employees만 정렬하면 되기 때문에 해당 테이블을 우선 정렬후, 정렬된 결과와 salaries 테이블 조인. 이 때 정렬은 소트 버퍼를 통해 실행
- 임시 테이블을 이용한 정렬
  ```sql
  SELECT *
  FROM employees e, salaries s
  WHERE s.emp_no=e.emp_no AND e.emp_no BETWEEN 10002 AND 10010
  ORDER BY s.salary // 드라이빙 테이블이 아닌 테이블의 칼럼 정렬
  ```
  테이블을 조인하고 임시 테이블에 저장 후 정렬
- 스트리밍 & 버퍼링
  - 스트리밍 : Sort나 GroupBy 필요없는 응답이면 스트리밍 방식인 것 같음. 필터링은 스트리밍이 될 수 있을것 같다.
  - 버퍼링 : Sort나 GroupBy 처럼 전체 리스트를 모아서 특정한 처리를 하는 경우
- 정렬 관련 상태 변수
  - Sort_merge_passes : 소트 버퍼를 사용하여 정렬 후 병합을 몇번했는지 횟수(인덱스를 이용한 정렬 이외의 정렬에서 사용)
  - Sort_range : 인덱스 레인지 스캔을 통해 조회한 리스트 정렬 횟수
  - Sort_rows : 정렬한 레코드 건수
  - Sort_scan : 풀 테이블 스캔을 통해 조회한 리스트 정렬 횟수
