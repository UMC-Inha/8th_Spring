- **지연로딩과 즉시로딩의 차이**
    - 지연 로딩 (Lazy Loading)
        
        : 연관 엔티티를 처음에는 조회하지 않고, 엔티티가 실제로 필요할 때 불러오는 전략
        
        - 처음에 엔티티가 로드될 때는 프록시 객체로 로딩, 사용 시점에 초기화
        
        ```java
        @Entity
        public class Member {
            @ManyToOne(fetch = FetchType.LAZY)
            private Team team;
        }
        ```
        
        ```java
        Member member = memberRepository.findById(1L).get();
        // 이 시점에서는 team은 아직 로딩되지 않음
        String teamName = member.getTeam().getName(); // 여기서 DB 쿼리가 실행됨
        ```
        
        - 장점
            - 필요할 때만 쿼리가 실행되어 불필요한 데이터 로딩 줄임
            - 대량 데이터 처리 시 유리
        - 단점
            - 연관 데이터가 많을 경우 N+1 문제 발생
                
                (5주차에 정리) 처음 조회한 엔티티와 연관된 엔티티들이 하나씩 로딩되면서 추가 쿼리가 발생하는 현상
                
            
            ⇒ 해결하기 위해, JPA는 Fetch Join과 같은 전략 제공
            
    - 즉시 로딩 (Eager Loading)
        
        : 엔티티를 조회할 때, 연관된 엔티티도 함께 조회하는 방식
        
        - JOIN 쿼리를 자동으로 만들어 실행
        
        ```java
        @Entity
        public class Member {
            @ManyToOne(fetch = FetchType.EAGER)
            private Team team;
        }
        ```
        
        ```java
        Member member = memberRepository.findById(1L).get();
        // member와 team을 함께 조회
        ```
        
        - 장점
            - 데이터가 필요할 때마다 추가적인 쿼리를 실행할 필요가 없어 성능이 향상 가능
                
                특히 @ManyToOne, @OneToOne과 같은 연관 관계에서 많이 사용
                
        - 단점
            - 불필요한 데이터까지 로딩될 수 있음
            - JOIN 많아지면 쿼리 복잡도 증가 + 성능 저하
    
    일반적으로 지연 로딩을 기본으로 사용하고, 성능 최적화가 필요한 특정 상황에서 Fetch Join 등을 통해 필요한 데이터만 즉시 가져오는 전략이 권장
    
- **Fetch Join**
    
    : JPA에서 제공하는 최적화 기법 중 하나로, 지연 로딩으로 설정된 연관 데이터를 한 번의 쿼리로 함께 가져오고자 할 때 사용
    
    ```java
    // 일반 쿼리
    SELECT m FROM Member m;
    
    // Fetch Join 사용
    SELECT m FROM Member m JOIN FETCH m.team;
    ```
    
    - `SELECT 절`에 명시한 엔티티와 함께 연관 엔티티가 '영속성 컨텍스트'에 로딩
        
        하나 이상의 컬렉션 (`@OneToMany`)을 동시에 Fetch Join 할 경우, 데이터 중복이 발생하여 예상치 못한 결과를 얻거나 메모리 문제
        
        ⇒ `DISTINCT` 키워드를 사용하여 중복을 제거
        
    - 컬렉션 Fetch Join 사용 시 페이징(Paging) 처리에 주의
        
        Fetch Join으로 인해 결과 집합의 크기가 예상보다 커질 수 있으며, JPA 구현체에 따라 메모리에서 페이징 처리가 되어 성능 저하가 발생
        
    - 성능 최적화에 강력한 수단이지만 주의가 필요
        - JOIN 테이블 많아지면 메모리 과부화
        
        ⇒ 단순한 연관 데이터 조회에서는 매우 유용하지만, 대량의 데이터를 동반한 복잡한 연관 관계에서는 주의 깊게 사용
        
- **@EntityGraph**
    
    : JPA에서 fetch join을 어노테이션으로 사용할 수 있도록 만들어 준 기능
    
    - 메서드 선언부에 `@EntityGraph` 어노테이션을 붙여 어떤 연관 엔티티를 함께 로딩할지 명시
    - Repository 인터페이스 메서드에 적용하여 사용하기 편리
    - JPQL과 함께 사용하여 복잡한 쿼리 없이 필요한 연관 데이터를 로딩 가능
    
    ```java
    public interface MemberRepository extends JpaRepository<Member, Long> {
    
        @EntityGraph(attributePaths = {"team", "address"}) // team과 address를 함께 로딩
        
        List<Member> findAll(); // Member와 연관된 team, address를 함께 조회
    }
    
    ```
    
    - **J**PQL 없이 간단하게 Fetch Join 효과
    - 기본적으로 `LEFT OUTER JOIN`으로 연관 엔티티를 가져옴
        
        `INNER JOIN`이 필요 시, JPQL Fetch Join을 사용하거나 `@EntityGraph`의 `type` 속성을 변경
        
    - 엔티티 조회 시 추가적인 조인 쿼리를 필요로 하지 않아, 성능 최적화에 적합
- **JPQL** (Java Persistence Query Language)
    
    : JPA에서 제공하는 표준 쿼리 언어로, 엔티티 객체를 기준으로 동작
    
    - 일반 SQL과 유사한 문법을 사용하지만, 데이터베이스 테이블이 아닌 JPA 엔티티 기반이라 엔티티 간의 연관 관계를 따라 탐색할 수 있음
    
    ```java
    SELECT r FROM Review r WHERE r.content = :content // Review 엔티티에서 
    ```
    
    - 장점
        - SQL에 비해 객체 지향적 접근을 지원 → 연관관계 활용
        - 엔티티 간의 연관성을 기반으로 질의할 수 있어, 개발자에게 보다 직관적인 쿼리 작성 환경을 제공
    - 단점
        - 동적 쿼리 작성이 어려움
        - 쿼리 오류를 컴파일 시점에 감지할 수 없음
    
    주로 간단한 조회나 정적인 쿼리에 많이 사용되며, 복잡한 조건을 갖춘 동적 쿼리에는 적합X
    
- **QueryDSL**
    
    : 동적 쿼리를 보다 안전하고 효율적으로 작성할 수 있도록 지원하는 라이브러리
    
    - java 기반으로 쿼리를 작성하므로, 컴파일 시점에 오류를 확인할 수 있으며, 타입 안정성을 제공
    
    ```java
    QReview review = QReview.review;
    queryFactory.selectFrom(review)
                .where(review.content.eq("JPA")) // 자바 코드로 조건 작성
                .fetch(); // 쿼리 실행
    ```
    
    - 장점
        - JPQL의 단점을 보완
        - java 코드로 간단하게 표현, 가독성 향상
    
    ⇒ Q 클래스를 통해 필드에 접근하여 조건을 설정할 수 있으므로, 복잡한 조건과 필터링이 필요한 경우 유용
    

- **N+1 문제 해결 방법**
    1. Fetch Join 사용 (JPQL)
        
        : JPQL 쿼리에서 `JOIN FETCH`를 사용하여 연관 엔티티를 처음 조회할 때 함께 로딩
        
        ex) 회원 목록을 조회하며 각 회원의 팀 정보를 함께 가져오는 경우
        
        ```java
        // Member와 Team은 LAZY 로딩으로 설정되어 있다고 가정
        List<Member> members = entityManager.createQuery(
            "SELECT m FROM Member m JOIN FETCH m.team", Member.class)
            .getResultList();
        // SQL 쿼리: SELECT m.*, t.* FROM Member m JOIN Team t ON m.team_id = t.id
        // 결과: Member 목록을 조회하는 1번의 쿼리만 실행
        ```
        
        - 특정 조회 로직에서만 N+1 문제를 해결하고 싶을 때 유용
    2. @EntityGraph 사용 (Spring Data JPA Repository)
        
        : Spring Data JPA Repository 인터페이스 메서드에 `@EntityGraph` 어노테이션을 사용하여 연관 엔티티를 함께 로딩하도록 설정
        
        ex) 모든 회원을 조회하며 팀 정보를 함께 가져오는 경우
        
        ```java
        public interface MemberRepository extends JpaRepository<Member, Long> {
            @EntityGraph(attributePaths = {"team"}) // "team" 연관 엔티티를 함께 로딩
            List<Member> findAllWithTeam(); // 이 메서드 호출 시 @EntityGraph 설정이 적용됩니다.
        }
        
        // 사용 코드
        List<Member> members = memberRepository.findAllWithTeam();
        // SQL 쿼리: SELECT m.*, t.* FROM Member m LEFT OUTER JOIN Team t ON m.team_id = t.id
        // 결과: Member 목록을 조회하는 1번의 쿼리만 실행됩니다.
        ```
        
        Repository 메서드를 통해 쉽게 N+1 문제를 해결할 수 있습니다. 특정 조회 메서드에만 적용하기 편리
        
    3. @BatchSize 설정
        
        : hibernate가 제공하는 기능으로, 지연 로딩된 엔티티를 조회할 때 설정된 `size`만큼 `IN` 절을 사용하여 한 번에 가져오는 방식
        
        N개의 개별 쿼리를 Batch Size로 묶어 실행
        
        ex) 회원 목록 조회 후 각 회원의 팀 정보를 사용하려 할 때
        
        ```java
        @Entity
        public class Member {
            @Id @GeneratedValue
            private Long id;
            private String username;
        
            @ManyToOne(fetch = FetchType.LAZY)
            @BatchSize(size = 10) // BatchSize 설정
            private Team team;
        
            //...
        }
        ```
        
        ```java
        // Member와 Team은 LAZY 로딩으로 설정되어 있다고 가정
        // Member 엔티티를 100건 조회
        List<Member> members = memberRepository.findAll();
        
        // 각 Member의 team 이름을 사용하는 시점
        for (Member member : members) {
            System.out.println(member.getTeam().getName());
        }
        
        // SQL 쿼리 발생 패턴:
        // 1. Member 100건 조회 쿼리 (1번)
        // 2. 각 Member의 team 접근 시, @BatchSize(10)에 의해 10개씩 묶어서 Team 조회 쿼리 (10번)
        // 총 11번의 쿼리 발생 (1 + 10)
        ```
        
        - 지연 로딩을 유지하면서 N+1 문제를 완화하고 싶을 때 사용