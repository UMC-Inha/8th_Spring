- Fetch Type
    - FetchType이란, JPA가 하나의 Entity를 조회할 때, 연관관계에 있는 객체들을 어떻게 가져올 것이냐를 나타내는 설정값이다.
    - 다른 객체와 연관관계 매핑이 되어있으면 그 객체들까지 조회하게 되는데, 이때 이 객체를 어떻게 불러올 것인지 설정하는 값으로 Eager(즉시로딩)과 Lazy(지연로딩)을 설정할 수 있다
- 즉시로딩
    - 즉시로딩이란 데이터를 조회할 때, 연관된 모든 객체의 데이터까지 한 번에 불러오는 것이다.
    - ex)
        
        ```java
        @Entity
        public class Member {
        
            @Id @GeneratedValue
            private Long id;
            private String username;
        
            @ManyToOne(fetch = FetchType.EAGER) //Team을 조회할 때 즉시로딩을 사용하곘다!
            @JoinColumn(name = "team_id")
            Team team;
        }
        
        @Entity
        public class Team {
        
            @Id @GeneratedValue
            private Long id;
            private String teamname;
        }
        ```
        

- 지연로딩
    - 지연로딩이란, 프록시를 둬서 필요한 시점에 연관된 객체의 데이터를 불러오는 것이다.
    - ex)
        
        ```java
        @Entity
        public class Member {
        
            @Id @GeneratedValue
            private Long id;
            private String username;
        
            @ManyToOne(fetch = FetchType.LAZY) //Team을 조회할 때 지연로딩을 사용하곘다!
            @JoinColumn(name = "team_id")
            Team team;
        }
        
        @Entity
        public class Team {
        
            @Id @GeneratedValue
            private Long id;
            private String teamname;
        }
        ```

        
        
- **Fetch Join**
    
    **Fetch Join의 정의**
    
    : JPQL에서 성능 최적화를 위해 제공하는 조인의 한 종류이다
    
    **JPQL의 정의**
    
    : SQL이 DB에 있는 테이블을 조회하는 쿼리라고 한다면 JPQL은 엔티티 객체를 조회하는 객체지향 쿼리를 의미한다. 따라서 SQL과 다르게  PQL은 **엔티티 직접 조회, 묵시적 조인, 다형성 지원** 등의 기능을 제공하다는 장점을 가지고 있습니다. 또한 JPQL은 특정 데이터베이스에 의존하지 않는다는 특징이 있습니다. (여러 DB에서 동시에 작업 가능)
    
    **Fetch Join에 대해**
    
    JPQL에서 성능 최적화를 위해 제공하는 기능으로 연관된 엔티티나 컬렉션을 한 번에 같이 조회할 수 있는 기능입니다. `JOIN FETCH` 명령어로 사용할 수 있습니다.
    
    1. 엔티티 fetch join
    
    ```java
    String query = "select b from Book b join fetch b.library";
    
    em.flush();
    em.clear();
    
    List<Book> books = em.createQuery(query, Book.class).getResultList();
    
    for (Book book : books) {
        System.out.println(book.getLibrary());
    }
    ```
    
    2. 컬렉션 fetch join
    
    ```java
    String query = "select l from Library l join fetch l.books where l.name = 'libraryA'";
    
    em.flush();
    em.clear();
    
    Library library = em.createQuery(query, Library.class).getSingleResult();
    
    for (Book book : library.getBooks()) {
        System.out.println(book.getName());
    }
    ```

    
    **Fetch join의 한계**
    
    1. 페치 조인 대상에는 별칭을 줄 수 없기에 `SELECT`, `WHERE`, `서브 쿼리`에 페치 조인 대상을 사용할 수 없습니다
    2. 데이터가 증폭되는 문제로 인하여 둘 이상의 컬렉션을 페치할 수 없습니다.
    3. 컬렉션을 페치 조인하면 `페이징 API(setFirstResult, setMaxResults)`를 사용할 수 없습니다.

출처 : https://woo-chang.tistory.com/38




- **@EntityGraph**
    
    : JPQL로 fetch join을 직접 작성하지 않고 @EntityGraph 어노테이션을 붙임으로서 fetch join을 편리하게 사용할 수 있다.
    
    → N+1 문제를 해결하고, 성능 최적화에 도움을 줄 수 있음
    
    ```java
        public interface MemberRepository extends JpaRepository<Member,Long> ,MemberRepositoryCustom{
        
          @Override //기본 적으로 findAll 을 제공하기 때문에 Override 하여 재정의 후 사용 
          @EntityGraph(attributePaths = {"team"}) // DataJpa 에서 fetch 조인을 하기 위한 설정
          List<Member> findAll();
    
        }
    ```
    
    출처: [https://hstory0208.tistory.com/entry/Spring-Data-JPA-EntityGraph-엔티티-그래프란](https://hstory0208.tistory.com/entry/Spring-Data-JPA-EntityGraph-%EC%97%94%ED%8B%B0%ED%8B%B0-%EA%B7%B8%EB%9E%98%ED%94%84%EB%9E%80)
    
- **JPQL**
    
    Fetch join 키워드 정리할 때, 함께 정리하였습니다.
    
- **QueryDSL**
    
    QueryDSL은 오픈소스 프로젝트로 JPQL을 Java 코드로 작성할 수 있도록 하는 라이브러리이다.
    
    **Q파일**
    
    : QueryDSL이 엔티티 클래스를 기반으로 자동 생성하는 자바 클래스
    
    - 보통 이름이 Q + 엔티티명
    - 내부적으로는 **엔티티의 각 필드 정보를 QueryDSL에서 사용할 수 있도록 정리한 클래스**
    - 타입 안전한 쿼리 클래스
    
    ex)
    
    ```java
    QUser user = QUser.user;
    
    queryFactory
        .selectFrom(user)
        .where(user.name.eq("철수"))
        .fetch();
    
    ```
    
    **장점**
    
    1. **자바 코드로 쿼리를 작성함으로 컴파일 시점에 에러를 잡을 수 있다.**
    2. **복잡한 동적 쿼리를 쉽게 다룰 수 있다.**

       
    
- N+1 문제 해결 방법
    1) **fetch join 사용**
    
    평소에는 지연 로딩 전략을 쓰다가 join이 필요할 때 fetch를 적용함으로 필요할 때 한 번에 데이터를 가져올 수 있게 한다. 즉 join 시 모든 정보가 다 영속성 컨텍스트에 들어가므로 필요할 때마다 값을 불러도 쿼리문이 계속 나가는 문제 해결 가능
    
    **사용 방법**
    
     쿼리 작성 시 명시적으로 fetch join을 지정함으로써 사용할 수 있다
    
    ![image.png](attachment:4451bbc8-c97a-4d12-9148-57a7d4dd544f:image.png)
    
    2) **~~Eager Loading 사용~~**
    
    JPA가 JPQL를 분석해서 SQL을 생성할 때 영속성 컨텍스트에 정보를 가져오는 전략으로 
    
    즉시로딩을 선택하는 방법이 있는데..
    
    즉시 로딩은 해결책이 될 수 없을 뿐더러 더욱 상확이 악화될 수 있다.
    
    그 이유는 Eager은 Hibernate가 판단해서 로딩을 하기 때문이다. 따라서 Hibernate가 자동으로 가져오기 때문에 언제 로딩하는지 알 수 없어 N+1 문제가 발생할 수 있고, 미리 로딩 됐다 하더라도 엔티티 안에 연관관계가 있는 객체들에서 내부적으로 N+1가 **최악의 상황에서는 여러 번 발생할 수 있으므로** 가급적 지연 로딩 전략을 기본으로 세워야 한다.
    
    3) **EntityGraph**
    
    **@EntityGraph는** Data JPA에서 fecth 조인을 어노테이션으로 사용할 수 있도록 만들어 준 기능이다.
    
    → fetch를 어노테이션으로 연결하다는 점에서 편하다고 했지만 관계를 얽히면 오히려 복잡해진다는 말도 많네요 ㅎㅎ
    
    4) **Batch size 조정**
    
    연관된 엔티티를 하나씩 이 아닌 설정한 Batch size 만큼 가져오게 한다.
    
    예를 들어 Batch size를 1000으로 조정했다면 추가로 1000개의 정보를 묶음으로 가져오게 된다.
    
    - **그러면 fetch join이랑 다를게 없지 않을까?**
    
    Batch size의 장점
        1. 영속성 컨텍스트에 다 저장하지 않으므로 메모리 효율성 증가
        2. 페이징이 안 되는 fetch join 만의 문제결
    
    개인적으로 각각의 기술을 잘 쓰면 JPA로도 복잡한 쿼리를 구현할 수 있지만, 하나하나 오류를 파다 보면 끝도 없이 나올 때도 많아서 이걸 다 공부하기는 쉽지 않고,
    
    복잡한 쿼리가 나올 때는 jdbcTemplate를 사용하거나 혹은, 요즘 jdbcClient라는 훨씬 더 간결하고 유연한 쿼리를 작성 가능한 기술이 나와서 해당 기술에 대해 공부하는 것이 더 트렌드에 맞지 않는가 라고 생각합니다
