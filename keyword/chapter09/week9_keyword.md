- Spring Data JPA의 Paging
    - Pageable
        - 페이징 인터페이스
        - 요청된 페이지 번호, 페이지 크기, 정렬 방식 등의 정보를 포함
        - 대표 구현체
            - PageRequest.of(page, size, Sort.by(…))

    - Page

      Page<T>

        - 전체 페이지 정보를 포함한 페이징 결과를 제공하는 객체

      특징

        - 전체 페이지 수 ( getTotalPages() )
        - 전체 요소 수 ( getTotalElements() )
        - 현재 페이지에 포함된 데이터 리스트 ( getContent() )
        - 현재 페이지 번호, 사이즈 등의 정보

      언제 사용?

        - 총 데이터 개수까지 꼭 알아야 할 때
        - 페이지 번호를 기반으로 페이지 이동 버튼 만들거나 전체 건수를 표시할 때

      단점

        - count(*) 쿼리가 추가로 실행 → 성능 이슈 가능

      예시 코드

        ```java
        Page<Member> page = memberRepository.findByAge(20, PageRequest.of(0, 10));
        List<Member> members = page.getContent();
        long total = page.getTotalElements();
        ```

    - Slice

      Slice<T>

        - 다음 페이지 존재 여부만 판단하기 위한 최소한의 정보만 제공하는 객체

      특징

        - 총 페이지 수나 총 데이터 수 정보 없음
        - hasNext() 를 통해 다음 페이지 존재 여부만 판단
        - 페이지 번호, 사이즈 등 정보 포함

      언제 사용?

        - 무한 스크롤이나 다음 페이지 존재 여부만 필요한 경우
        - 불필요한 count 쿼리를 줄여 성능 최적화 가능

      예시 코드

        ```java
        Slice<Member> slice = memberRepository.findByAge(20, PageRequest.of(0, 10));
        boolean hasNext = slice.hasNext();
        List<Member> members = slice.getContent();
        ```

- 객체 그래프 탐색

  JPA에서 연관된 엔티티를 탐색하거나 로딩할 때 중요한 개념

  즉시 로딩 (Eager Loading)

    - 연관 엔티티를 즉시 로딩
    - @ManyToOne(fetch = FetchType.EAGER)
    - 단점 : 필요하지 않은 객체도 무조건 join 또는 select 로 로딩 → N+1 문제 유발 가능

  지연 로딩 (Lazy Loading)

    - 연관 엔티티를 실제 사용할 때 로딩
    - @OneToMany(fetch = FethType.Lazy)
    - 하이버네이트 프록시 객체로 감싸고, 접근 시 실제 SQL 실행
    - 성능 유리하지만 영속성 컨텍스트 밖에서 사용 시 LazyInitializationException 발생

  예시

    ```java
    Member member = memberRepository.findById(1L).get();
    String teamName = member.getTeam().getName(); // 객체 그래프 탐색
    ```

    - 위 코드에서 getTeam() 호출 시 Lazy 라면 SQL이 그 시점에 날아감
    - 이 방식으로 연결된 엔티티를 탐색하는 것을 객체 그래프 탐색이라고 함.