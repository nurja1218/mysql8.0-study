- 잠금과 트랜잭션
  - 트랜잭션
    - 하나의 논리적인 작업 단위
    - 하나 이상의 쿼리 또는 명령문의 집합
  - 잠금
    - 데이터베이스에서 동시에 여러 트랜잭션이 동일한 데이터를 변경하지 못하도록 하는 메커니즘
  - 동시성과 정합성
    - 데이터 정합성 → 트랜잭션
      - 데이터베이스의 데이터가 정확하고, 논리적으로 일관된 상태를 유지
    - 동시성 → 잠금
      - 여러 트랜잭션이나 프로세스가 데이터베이스 시스템에서 동시에 실행
  - MyISAM, MEMORY → 트랜잭션 지원 안하는 스토리지 엔진
  - MyISAM과 InnoDB의 처리방식(트랜잭션 유무)
    - MyISAM
      - 부분 업데이트 방지를 위해 → IF-ELSE 덕지덕지
        - 데이터 정합성 보장하기 위한 로직 마다의 조건별 로직이 필요
        ```sql
        INSERT INTO tab_a ...;
        IF(_is_insert1_succeed){
          INSERT INTO tab_b ...;
          IF(_is_insert2_succeed){
            // 처리 완료
          } ELSE {
            DELETE FROM tab_a WHERE ...;
            IF(_is_delete_succeed){
              // 처리 실패 및 tab_a, tab_b 모두 원상 복구 완료
            } ELSE {
              // 해결 불가능한 심각한 상황 발생
              // 이제 어떻게 해야 하나?
              // tab_b에 INSERT는 안되고, 하지만 tab_b에는 INSERT돼버렸는데, 삭제는 안되고...
            }
          }
        }
        ```
    - InnoDB
      - 트랜잭션 사용
        ```sql
        try {
          START TRANSACTION;
          INSERT INTO tab_a ...;
          INSERT INTO tab_b ...;
          CommonUtilHelper;
        } catch (exception) {
          ROLLBACK;
        }
        ```
- 주의사항
  - 꼭 필요한 최소의 코드에만 적용하는 것이 좋다 → 트랜잭션의 범위 최소화
  - ex. 사용자 게시판
    - case 1.
      - create connection
      - 트랜잭션 start
        - 로그인 여부
        - 사용자 글쓰기 내용 오류 여부
        - 업로드 파일 확인 및 저장
        - 사용자 입력 DB저장
        - 첨부 파일 정보 DB저장
        - 저장된 내용 및 기타 정보 DB에서 조회
        - ~~게시물 등록 알림 메일 발송~~
          - 트랜잭션에 넣어선 안되는 로직(통신이 안되면 웹서버+DB서버까지 위험)
        - _알림 메일 이력 DBMS에 저장_
          - 별도 분리
      - 트랜잭션 end
      - close connection
    - case 2.
      - 로그인 여부
      - 사용자 글쓰기 내용 오류 여부
      - 업로드 파일 확인 및 저장
      - create connection
      - 트랜잭션 start
        - 사용자 입력 DB저장
        - 첨부 파일 정보 DB저장
      - 트랜잭션 end
      - close connection
      - 저장된 내용 및 기타 정보 DB에서 조회
      - 게시물 등록 알림 메일 발송
      - create connection
      - 트랜잭션 start
        - 알림 메일 이력 DBMS에 저장
      - 트랜잭션 end
      - close connection
