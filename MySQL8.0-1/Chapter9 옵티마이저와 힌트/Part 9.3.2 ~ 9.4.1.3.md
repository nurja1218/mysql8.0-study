- 힌트
  9.3.2 조인 최적화 알고리즘

  - 실행계획 최적화를 위한 알고리즘 두 개
  - 조인 최적화가 개선됐어도 테이블 개수가 많아지면 실행 계획 수립이 몇 분이 걸릴 수도 있다.

    9.3.2.1 Exhaustive 검색 알고리즘(완전탐색)

  - 사진넣기
  - 이전 버전에서 사용되던 조인 최적화 기법
  - 명시된 모든 테이블의 조합에 대해 비용을 계산해서 최적의 조합 1개를 찾는 방법
  - 테이블 개수 n!

    9.3.2.2 Greedy 검색 알고리즘

  - Exhaustive 알고리즘의 시간 소모 문제점 해결을 위해 5.0에서 도입
  - 절차
    1. 전체 N개의테이블 중에서 optimizer_search_depth 시스템 설정 변수에 정의된 개수의 테이블로 가능한 조인 조합을 생성
    2. 1번에서 생성된 조인 조합 중에서 최소 비용의 실행 계획 하나를 선정
    3. 2번에서 선정된 실행 계획의 첫 테이블을 부분 실행 계획의 첫 번째 테이블로 선정
    4. 전체 N-1개의 테이블 중에서 optimizer_search_depth 시스템 설정 변수에 정의된 개수의 테이블로 가능한 조인 조합을 생성
    5. 4번에서 생성된 조인 조합들을 하나씩 3번에서 생성된 부분 실행 계획에 대입해 실행 비용을 계산
    6. 5번의 비용 계산 결과 최적의 실행 계획에서 두 번째 테이블을 3번에서 새성된 부분 실행 계획의 두 번째 테이블로 선정
    7. 남은 테이블이 모두 없어질 때까지 4~6번까지의 과정을 반복
    8. 최종적으로 부분 실행 계획이 테이블의 조인 순서로 결정됨
  - 각 부분의 최적을 찾고 그걸 조합

  - 최적화를 위한 시스템 변수

    - optimizer_search_depth
      - Greedy 검색 알고리즘과 Exhaustive 검색 알고리즘 중에서 어떤 알고리즘을 사용할지 결정
      - 기본값 62
      - 설정값보다 테이블의 개수가 크면 설정값 만큼은 Exhaustive 사용 나머지는 Greedy사용
      - 0의 경우 옵티마이저가 자동으로 설정. 다만 비추천
      - 4~5가 적당하다..랍니다 — 주간회의시간에 말하기
    - optimizer_prune_level
      - Heuristic 검색이 작동하는 방식 제어
      - Exhaustive나 greedy 모두 상당히 많은 조인경로를 비교한다.
      - 휴리스틱의 포인트는 다양한 조인 순서 비용을 계산하는 도중 이미 계산했던 조인 순서 비용보다 큰경우 중간에 즉시 중지
      - 해당 변수는 on, off 방식 기본값 1 on
    - 휴리스틱의 장점 나열
    - 8.0부터는 optimizer_search_depth 의 값에는 크게 영향받지 않는 것으로 보임
    - optimizer_prune_level 은 0으로 설정 시 depth 변화에 따라 실행 계획 수립에 소요되는 시간이 급증한다.
    - 즉, 0으로 사용할 이유가 없다(8.0에선 휴리스틱 문제점이 대부분 개선됨)

  9.4 쿼리힌트

  - MySql이 많이 발전했지만 여전히 서비스나 비즈니스를 100% 이해하지는 못한다.
  - 그래서 힌트가 제공됨

    - 인덱스 힌트
    - 옵티마이저 힌트

      9.4.1 인덱스 힌트

  - 옵티마이저 힌트가 도입되기 이전에 사용되던 기능들
  - ANSI-SQL 표준 문법을 준수하지 못하게 되는 단점
  - 그래서 비추천
  - SELECT과 UPDATE에서만 사용가능

    9.4.1.1 STRAIGHT_JOIN

  - 옵티마이저 힌트인 동시에 조인 키워드
  - 조인 순서를 고정하는 역할
  - 일반적으로 인덱스 여부로 조인순서 결정, 인덱스에 문제가 없는 경우 레코드가 적은 테이블을 드라이빙으로 선택 (레코드란 where 조건까지 포함해서 그 조건을 만족하는 레코드 건수)
  - SELECT 바로 뒤에 사용되어야 함
  - 다음 기준에 맞게 조인 순서가 결정되지 않는 경우 사용 추천 - 임시테이블과 일반테이블의 조인 - 일반적으로 임시테이블을 드라이빙 테이블로 선정하는 것이 좋음 - 대부분 옵티마이저가 알아서 해줌 - 임시테이블 끼리 조인 - 크기가 작은 테이블을 드라이빙으로 선택해주는 것이 좋다 - 일반테이블끼리 조인 - 양쪽 모두 있거나 없는 경우 레코드 건수가 적은 테이블을 드라이빙으로 선택해주는 것이 좋다 - 그 외의 경우 인덱스가 없는 테이블을 드라이빙으로 설정 - STRAIGHT_JOIN과 비슷한 역할을 하는 옵티마이저 힌트 - JOIN_FIXED_ORDER = STRAIGHT_JOIN - FROM절의 모든 테이블에 대한 조인 순서 결정 - JOIN_ORDER,JOIN_PREFIX,JOIN_SUFFIX - 일부 테이블 조인 순서만 제안

    9.4.1.2 USE INDEX/FORCE INDEX/IGNORE INDEX

  - 가끔 옵티마이저가 실수할때 이용 - USE INDEX - 특정 인덱스를 사용하도록 권장 - FORCE INDEX - 거의 USE INDEX가 사용됨 - 기능은 비슷하지만 강제성이 조금 더 큼 - IGNORE INDEX - 특정 인덱스 사용 못하게 - 풀 테이블 스캔 유도가능 - 용도 명시도 가능 - USE INDEX FOR JOIN - 테이블 간의 조인뿐만이 아닌 레코드를 검색하기 위한 용도까지 포함 - USE INDEX FOR ORDER BY - 명시된 인덱스를 ORDER BY 용도로만 쓰이게 제한 - USE INDEX FOR GROUP BY - 명시된 인덱스를 GROUP BY 용도로만 쓰이게 제한 - 전문 검색 인덱스가 있는 경우 다른 보조 인덱스를 사용할 수 있는 상황이라도 전문 검색 인덱스를 선택하는 경우가 많음 → 옵티마이저가 전문 검색 인덱스나 프라이머리키에 좀더 많은 가중치를 두기 때문 - 최적의 실행 계획은 데이터의 성격에 따라 시시각각 변화 - 옵티마이저가 당시 통계 정보를 가지고 선택하는 것이 가장 좋음 → 가장 좋은 최적화는 그 쿼리를 서비스에서 없애 버리거나 튜닝할 필요가 없게 데이터를 최소화 하는 것 - 모델의 단순화를 통해 쿼리를 간결하게 만들어 힌트를 필요치 않게 하는 것 - 최후순위로 힌트를 선택 → 앞쪽의 작업들은 시간이 많이 들기 때문에 힌트에 의존하는 경우가 많음

    9.4.1.3 SQL_CALC_FOUND_ROWS

  - LIMIT 사용된 쿼리에서 LIMIT 수만큼 찾은 후에도 끝까지 검색을 수행하게 함
    - FOUND_ROWS()를 통해 조건에 만족하는 레코드가 전체 몇건이었는지 알아 낼 수 있음
  - COUNT와 비교
    - SQL_CALC_FOUND_ROWS의 경우
      - 쿼리가 2번 실행됨 (LIMIT 절을 위해 1번, FOUND_ROWS() 함수를 실행하기위해 1번)
      - 힌트 때문에 조건을 만족하는 레코드 전부를 읽어야 함
    - COUNT의 경우
      - 처음 쿼리에서 만족하는 모든 레코드의 건수만 필요하기에 인덱스 만으로(커버링 인덱스) 해결돼서 랜덤 I/O는 발생하지 않음
      - 두번째 쿼리에서는 제한한 개수 만큼만 읽어오게됨
    - SQL_CALC_FOUND_ROWS가 더 많은 작업을 하게되므로 비효율적
      - 성능면에서 나온 기능이 아닌 개발자의 편의를 위해 나온 기능
      - 사용하지 않을 것을 추천
