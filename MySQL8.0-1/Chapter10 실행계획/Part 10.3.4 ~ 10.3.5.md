- 기존에는 테이블의 인덱스 된 값의 유니크한 개수만 저장. 이때 균등 분포를 가정(전체 레코드 개수/유니크한 개수)
- 균등분포의 한계를 개선하기 위해 히스토그램 도입. 히스토그램은 여러범위를 나눠서 레코드 건수와 유니크한 값의 개수를 저장
- 히스토그램은 인덱스 되지 않은 칼럼에 대한 데이터를 주로 저장(인덱스 데이터는 b-tree에서 샘플링하는 인덱스 다이브 사용)

Partitions 칼럼

- 파티션은 특정 인덱스를 조건에 따라 그룹을 만들어 놓은 것을 의미. 테이블 생성시 만들 수 있으며 유니크 인덱스로만 파티션 키 생성 가능
- 이때 파티션은 개별 테이블처럼 물리적으로 분리 되어있음. 그래서 실행 계획에는 풀 테이블 스캔으로 표시
- 쿼리 조회 시 필요한 파티션에 바로 접근(파티션 프루닝)
- 실행 계획에서 파티션 칼럼은 쿼리 실행 과정에서 어떤 파티션들에 접근했는지 표시

Type 칼럼(중요)

- 테이블의 레코드에 서버가 접근하는 방식 표시
- system - 레코드 1건 or 0건 테이블 참조
- const - 프라이머리키나 유니크 키 칼럼을 where조건에 사용, 반드시 1건 반환(동등조건?)
- eq_ref - 여러 테이블이 조인할 때, 처음 읽은 테이블의 칼럼 값을 다음 테이블의 프라이머리 키나 유니크 키 칼럼의 검색조건에 활용할 때
- ref - 인덱스 종류와 상관 없이 동등 조건(레코드가 1건이라는 보장 x)
- ref*or* null - nullable 포함
- unique_subquery - WHERE절에서 서브쿼리 사용할 때, 서브쿼리가 유니크한 값을 반환할 때
- index_subquery - 서브쿼리가 유니크하지 않은 인덱스를 사용할 때
- range - 인덱스를 범위로 스캔할 때
- index - 인덱스를 풀 스캔 할 때
- ALL - 풀 테이블 스캔할 때
