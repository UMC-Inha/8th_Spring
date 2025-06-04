## - 키워드 정리

#### - Spring Data JPA의 Paging

- **주요 인터페이스와 클래스**

    - **Pageable**

        - 페이지 번호와 페이지 크기, 정렬 정보를 담는 인터페이스

        - PageRequest를 구현체로 사용한다.

        - 주요 메서드, 필드

            - content: 현재 페이지에 포함된 엔티티 리스트

            - pageable: page, size, sort 정보를 포함한 객체

            - total: 전체 요소 수

            - getContent(): 조회된 엔티티 리스트 반환

            - getTotalElements(): total값 반환

            - getTotalPages(): totalElements/pageSize 반환

            - hasNext() ,hasPrevious(): 현재 페이지와 전체 페이지 수를 비교해 판단

    - **Page**

        - 조회된 데이터 리스트와, 전체 페이지 수, 전체 요소 수, 현재 페이지 등의 데이터를 제공한다.

    - **slice**

        - Page에서 몇몇 요소를 제거했다. 다음 페이지가 있는지 여부만 제공한다.

- **사용 방법**

    - **api 요청 예시**

        - 다음과 같이 status가 ACTIVE, username이 kim인 유저를 페이징으로 조회하는 요청을 보낸다고 할 때

      ``` java
      GET /api/users?status=ACTIVE&usernameLike=kim&page=1&size=15&
      ```

    - **요청 dto**

        - 검색 조건 dto는 다음과 같이 정의할 수 있다.

      ``` java
      @Getter @Setter
       public class UserSearchDto {
         private String status;  
         private String usernameLike;
       }
  
      ```

    - **컨트롤러 로직**

        - @PageableDefault : Pageable 객체의 기본값을 설정

      ``` java
       @GetMapping
      public Page<UserDto> searchUsers(
          UserSearchDto condition,                      
          @PageableDefault(size = 20, sort = "createdAt", direction = DESC)
          Pageable pageable
      ) {
          Page<User> page = userService.searchUsers(condition, pageable);
          return page.map(UserDto::fromEntity);
      }
      ```

    - **서비스 로직**

      ``` java
      public Page<User> searchUsers(UserSearchDto cond, Pageable pageable) {
          return userRepository.searchByCondition(cond, pageable);
      }
      ```

    - **Repository(QueryDSL)**

  ``` java
   @Override
    public Page<User> searchByCondition(UserSearchDto cond, Pageable pageable) {
        QUser u = QUser.user;

        // 1) 컨텐츠 조회
        List<User> content = queryFactory
            .selectFrom(u)
            .where(
                cond.getStatus() != null ? u.status.eq(cond.getStatus()) : null,
                cond.getUsernameLike() != null ? u.username.contains(cond.getUsernameLike()) : null,
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .orderBy(QuerydslUtils.getOrderSpecifier(u, pageable.getSort()))
            .fetch();

        // 2) 전체 카운트 조회
        long total = queryFactory
            .select(u.id)
            .from(u)
            .where(
                cond.getStatus() != null ? u.status.eq(cond.getStatus()) : null,
                cond.getUsernameLike() != null ? u.username.contains(cond.getUsernameLike()) : null,
            )
            .fetchCount();

        return new PageImpl<>(content, pageable, total);
  }
  ```

    - **응답 JSON 형태**

  ``` java
  {
    "content":[
      { "id":12, "username":"kim1", "email":"...", "status":"ACTIVE"},
      …
    ],
    "pageable":{ … },
    "totalElements":123,
    "totalPages":9,
    "last":false,
    "number":1,
    "size":15,
    "sort":{…},
    "first":false,
    "numberOfElements":15
  }
  ```

#### - 객체 그래프 탐색

- **객체 그래프**

    - 엔티티나 도메인 객체들이 서로 참조, 연결되어 이루는 네트워크

    - 현실 세계의 관계를 객체 간 참조로 자연스럽게 표현할 수 있다.

    - 하나의 객체에서 다른 객체로 자유롭게 이동할 수 있어 필요한 데이터를 조회하기 쉽다.

- **FetchType.EAGER vs FetchType.LAZY**

    - **FetchType.EAGER**

        - 엔티티 조회 시점에 연관 데이터를 함께 즉시 fetch join하여 한 번에 로딩한다.

        - 연관 데이터를 곧바로 사용할 때 쿼리 없이 사용할 수 있다.

        - 불필요한 데이터까지 항상 조인하기 때문에 **대량 조인 발생 시 성능 저하가 발생할 수 있다.**

    - **FetchType.LAZY**

        - 연관된 데이터를 실제로 사용할 때 데이터베이스에서 조회한다.

        - 불필요한 조인이나 조회를 피할 수 있기 때문에 성능이 향상될 수 있다.

        - 접근 시점에 추가 쿼리가 발생하기 때문에 **N+1문제가 발생할 수 있다.**

  -> **무조건 LAZY를 적용해 N+1을 예방하고, 필요한 곳에 Fetch Join을 적용해 탐색해야 한다.**

- **객체 그래프 탐색 방법**

    - 객체 그래프 탐색 시에는 N+1문제가 발생하지 않도록 주의해야 한다.

    - **JPQL로 Fetch Join**

    - **JPA의 Entity Graph**

    - **@EntityGraph**

    - **QueryDSL Fetch Join**
