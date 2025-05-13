### 지연 로딩 vs 즉시 로딩

  **개념 설명**

- **지연 로딩**:
  - 연관 엔티티를 필요할 때 로딩
  - 관련 데이터에 실제로 접근할 때까지 로딩을 지연
- **즉시 로딩**:
  - 엔티티 로딩 시 연관 엔티티도 함께 로딩
  - 단일 쿼리로 필요한 모든 데이터를 검색

  **특징**

   - **지연 로딩**: 초기 로딩 빠르고 메모리 효율적, 하지만 N+1 쿼리 발생 가능
   - **즉시 로딩**: 데이터 즉시 사용 가능, 불필요한 데이터 로딩 위험

  **코드 예시**

   ```java
   @Entity
   public class Post {
       @OneToMany(mappedBy = "post", fetch = FetchType.LAZY)  *// 지연 로딩*
       private List<Comment> comments;
   }
    
   @Entity
   public class Comment {
       @ManyToOne(fetch = FetchType.EAGER)  *// 즉시 로딩*
       private Post post;
   }
   ```

- 컬렉션은 지연 로딩 추천
- 즉시 로딩 사용시 N + 1 위험이 있음
- 연관된 필요없는 엔티티가 대량으로 조회될 수 있음
- 즉시 로딩은 1:1 관계나 필수 데이터에 사용, 대량 데이터에서 사용 시 모든 데이터 조회 시 성능 저하 발생 가능

  **사용 시 주의점**

    - 트랜잭션 범위 밖에서 엔티티 사용할 경우
        - 지연 로딩은 프록시 객체를 사용하다가 실제 접근 시 쿼리를 실행함
        - 따라서, 트랜잭션이 끝난 이후 객체 접근 시 에러 발생
        - 컨트롤러에서 DTO 변환 중에 지연 로딩된 컬렉션 접근 시 자주 발생
        - ex) DTO 안에 엔티티가 가지고 있는 지연로딩된 컬렉션 필드를 컨트롤러에서 접근할 때
        - 해결책 : 서비스 계층에서 DTO로 변환하는 게 좋음 → 지연 로딩된 컬렉션은 트랜잭션 안에서만 안전하게 로딩 가능하기 때문에

    **출처**
    
    https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#fetching-strategies
    
    https://www.geeksforgeeks.org/lazy-loading-vs-eager-loading/
    
    https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy <br><br>

### Fetch Join

  **개념 설명**

- 연관 엔티티를 단일 쿼리로 로딩하여 N+1 문제를 해결
- for 문 이전에 JPQ 에서 fetch join 으로 연관 데이터를 미리 땡겨오면 for 안에서 추가 쿼리가 발생하지 않아 `N+1` 문제 해결 가능

  **특징**

- 쿼리 수 감소로 성능 개선

  **코드 예시**

  ```java
  String jpql = "SELECT p FROM Post p JOIN FETCH p.comments";
  List<Post> posts = entityManager.createQuery(jpql, Post.class).getResultList();
  ```

- 연관 데이터를 즉시 필요로 할 때 사용, 반복 쿼리 방지

  **주의점**

- 조인으로 인해 부모 엔티티가 중복 반환될 수 있으므로 `DISTINCT` 를 추가
- 만약 조인으로 다음과 같은 테이블이 조회되었다고 하면


- 나중에 post_id = 1일 때 comment_id 조회 시 중복 출력될 수 있음
    - 즉, 101과 102만 출력되어야 하는데 101 2번 102 2번 출력될 수 있기에
    - 따라서, `"SELECT DISTINCT p FROM Post p JOIN FETCH p.comments"`  이렇게 사용

  **출처**

  https://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf

  https://nhibernate.info/doc/nhibernate-reference/performance.html

### @EntityGraph

  **개념 설명**

    - JPA에서 fetch 조인을 어노테이션으로 사용할 수 있도록 만들어 준 기능

  **특징**

- 기본 FetchType 무시 가능
- Named 또는 Ad-hoc 그래프 지원

**Named Entity Graph**

```java
  // 1) 엔티티 정의부
  @NamedEntityGraph(
    name = "Member.withMissions",
    attributeNodes = @NamedAttributeNode("missions")
  )
  @Entity
  public class Member {
      @Id
      @GeneratedValue
      private Long id;
    
      private String username;
    
      @OneToMany(mappedBy = "member", fetch = FetchType.LAZY)
      private List<Mission> missions = new ArrayList<>();
  }
    
  // 2) 그래프 적용 조회
  EntityGraph<?> graph = em.getEntityGraph("Member.withMissions");
    
  List<Member> members = em.createQuery(
          "SELECT m FROM Member m WHERE m.active = true", Member.class)
      .setHint("javax.persistence.loadgraph", graph)   // 그래프에 명시된 missions만 즉시 로딩
      // .setHint("javax.persistence.fetchgraph", graph) // missions만 로딩, 나머지는 LAZY 유지
      .getResultList();
    
  for (Member m : members) {
      System.out.println("Member: " + m.getUsername());
      for (Mission mission : m.getMissions()) {
          System.out.println("  - Mission: " + mission.getTitle());
      }
  }
```

- 그래프에 명시된 missions만 즉시 로딩
- `.setHint("javax.persistence.fetchgraph", graph)`  :
    - missions만 로딩, 나머지는 LAZY 유지
- 쿼리별 데이터 로딩 요구사항이 다를 때 유용
    - 하나의 엔티티를 조회하더라도 실제로 어떤 연관 데이터를 함께 꺼내야 할 지는 상황에 따라 달라질 수 있기에
    - 위 예시로 볼 때 특정 시점에는 mission과 함께 조회하고 싶을 때  Hint 쪽에 `loadgraph`를 만약, 어떤 시점엔 지연로딩하고 싶을 때는 `fetchgraph`를 사용하면 된다.

  **N + 1 해결**

  ```java
  public interface MemberRepository extends JpaRepository<Member, Long> {
        
      @EntityGraph("Member.withMissions")
      List<Member> findAllByActiveTrue();
  }
  ```

  - 레포지토리의 메소드 위에 해당 애노테이션을 걸어서 실행 전에 미리 데이터를 가져오게끔 설정
  - 미리 데이터를 가져왔기 때문에 메소드 안에서 for문을 통해 `getMissions()`를 하더라도 `N + 1`문제가 발생 안 함

  **출처**

  https://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf

### JPQL

  **개념 설명**

    - JPA에서 제공하는 객체 지향 쿼리 언어

  **기본 구조**

    ```sql
    SELECT ... FROM ...
    [WHERE ...]
    [GROUP BY ... [HAVING ...]]
    [ORDER BY ...]
    ```

  **특징**

- SQL 유사하지만 엔티티를 대상으로 한다는 점에서 차이점이 있음
- 프로바이더가 RDBMS 종류에 따라 해석을 다 달리 해준다.
    - Mysql로 설정하면 Mysql 문법으로, PostgreSql로 설정하면 ~~~…

  **코드 예시**

  ```java
  String jpql = "SELECT p FROM Post p WHERE p.title LIKE :title";
  List<Post> posts = entityManager.createQuery(jpql, Post.class)
      .setParameter("title", "%Java%")
      .getResultList();
  ```

  - 정적(Static) 문자열 기반이라, “조건에 따라 특정 절을 추가”같은 동적 쿼리 생성 기능은 없다.
  - 동적 쿼리를 쓰려면 굉장히 복잡하게 써야 한다..

  ```java
   String jpql = "SELECT p FROM Post p WHERE 1=1";
   if (title != null)   jpql += " AND p.title LIKE :title";
   if (author != null)  jpql += " AND p.author = :author";
   Query q = em.createQuery(jpql, Post.class);
  
   if (title != null)   q.setParameter("title", "%"+title+"%");
   if (author != null)  q.setParameter("author", author);
   List<Post> results = q.getResultList();
  ```

  - 유지보수도 굉장히 떨어질 뿐더러 코드 작성도 어렵다.
  - 또한, 잘못된 엔티티나 속성명을 문자열로 썼을 때 컴파일 단계가 아닌 런타임에 에러가 발생되기에 타입 안전성이 보장되지 않는다.

  **@NamedQuery (명명된 쿼리)**

  - 미리 정의된 변경 불가능한 쿼리 문자열을 갖는 쿼리 정의 가능

  ```java
  @Entity
  @Table
  @NamedQuery(query = "Select e from Employee e where e.eid = :id", name = "find employee by id")
    
  //실제 사용 예시
  Query query = entitymanager.createNamedQuery("find employee by id");
  ```

  **출처**

  https://www.tutorialspoint.com/jpa/jpa_jpql.htm

  https://download.oracle.com/otn-pub/jcp/persistence-2_1-fr-eval-spec/JavaPersistence.pdf

### QueryDSL

  **개념 설명**

- 자바 코드로 SQL/JPQL을 타입 안전하게 생성할 수 있는 도메인 특화 언어(DSL)

  **특징**

- **컴파일 시점 타입 검증**으로 잘못된 필드 접근 방지

  **Q타입 생성**

- `@Entity`가 붙은 클래스에 대해 `QMember`, `QMission` 등 클래스 자동 생성

  **코드 예시**

  ```java
  JPAQueryFactory queryFactory = new JPAQueryFactory(entityManager);
  QPost post = QPost.post;
  List<Post> posts = queryFactory.selectFrom(post)
      .where(post.title.like("%Java%"))
      .fetch();
  ```

- JPQL과 다르게 복잡한 동적 쿼리도 자바 코드와 같이 유연하게 작성 가능

    ```java
    BooleanBuilder builder = new BooleanBuilder();
    if (username != null) {
      builder.and(m.username.eq(username));
    }
    if (minLevel != null) {
      builder.and(m.level.goe(minLevel));
    }
    
    List<Member> results = query
        .selectFrom(m)
        .where(builder)
        .fetch();
    
    ```

  **단점**

    - `UNION`, `FROM` 절 내 서브쿼리, Window Function 등의 고급 SQL 기능은 사용 못함
        - 필요시에 네이티브 쿼리나 Spring Data JDBC 같은 대안을 활용

  **프로젝션 (DTO 조회)**

  `Projections.constructor()` : 쿼리 실행 결과 필드를 DTO 필드에 매핑해서 DTO 생성을 도와줌

    ```java
    List<MemberDTO> dtos = query
        .select(Projections.constructor(
            MemberDTO.class,
            m.id, m.username, m.joinDate))
        .from(m)
        .where(m.active.isTrue())
        .fetch();
    
    ```

  **출처**

  https://coding-business.tistory.com/100

  http://querydsl.com/static/querydsl/3.1.1/reference/html_single/

  http://www.querydsl.com/static/querydsl/4.1.3/reference/html_single/

  https://yongkyu-jang.medium.com/jpa-%EB%8F%84%EC%9E%85-onetoone-%EA%B4%80%EA%B3%84%EC%97%90%EC%84%9C%EC%9D%98-lazyloading-%EC%9D%B4%EC%8A%88-1-6d19edf5f4d3