## 🎯핵심 키워드


### Domain

--- 

- 데이터베이스 테이블과 매핑
    ```java
    @Entity
    public class Example {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    // getters and setters
    }
    ```
- `@Entity`: 클래스를 엔티티로 지정.
- `@Id`: 기본 키를 정의.
- `@Column`: 데이터베이스 컬럼과 매핑.
- `@Table`: 테이블 이름 지정 (선택 사항)

<br>
 
- **EntityManager 작동 방식 (JPA 내부 원리)**
    - `EntityManager.persist(entity)`를 호출해 엔티티를 영속성 컨텍스트에 추가
    - 이 상태에서 엔티티는 트랜잭션이 커밋되는 시점에 데이터베이스에 반영
  - 이 상태에서 엔티티는 트랜잭션이 커밋되는 시점에 데이터베이스에 반영
    ```java
    EntityManager em = entityManagerFactory.createEntityManager();
    em.getTransaction().begin();
    Customer customer = new Customer("John Doe");
    em.persist(customer); // 데이터베이스에 INSERT 준비
    em.getTransaction().commit(); // INSERT 실행
    em.close();
    ```

- em.persist(entity)를 통해 엔티티는 영속성 컨텍스트에 들어감
- 이후 엔티티 수정 시 JPA가 변경 감지(Dirty Checking)해 트랜잭션 커밋 시 자동으로 데이터베이스에 반영
- `Spring Data JPA`를 사용할 경우, `EntityManager`를 직접 다루는 대신 `@Repository` 인터페이스를 통해 간접적으로 작업
    1. `@Repository`는 이 인터페이스를 `Spring Bean`으로 등록
    2. `Spring Data JPA`가 이 인터페이스를 감지하고, `JpaRepository`를 구현한 **`프록시 객체`** 를 자동으로 생성
    3. 프록시 객체는 내부적으로 **`EntityManager`** 를 사용해 JPQL `쿼리`를 생성하고 실행

출처:

https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html

https://velog.io/@simhyunmin/%ED%94%84%EB%A1%9D%EC%8B%9C-%EA%B0%9D%EC%B2%B4%EC%99%80-%EC%A7%80%EC%97%B0-%EB%A1%9C%EB%94%A9

### 양방향 매핑

---
```java
public class Member {
    private List<Inquiry> inquiryList = new ArrayList<>();

    public void addInquiry(Inquiry inquiry) {
        inquiryList.add(inquiry);    // Member -> Inquiry
        inquiry.setMember(this);     // Inquiry -> Member
    }
}
```
- 양쪽 엔티티를 수동으로 업데이트해야 하므로 편의 메서드를 사용하는 것이 좋다.
- 주로 관계의 소유자(owning side)나 양방향 매핑에서 사용
- 엔티티를 저장하거나 업데이트할 때 사용
- 특히 양방향 매핑에서 두 엔티티의 관계를 동기화할 때 많이 쓰인다.

<br>

- **테스트 코드 사용 예시**

    ```java
    Member member = new Member();
    Inquiry inquiry = new Inquiry();
    member.addInquiry(inquiry); // 한 번에 양방향 관계 설정
    ```

    - 테스트에서 직접 관계를 설정할 때 편의 메서드를 사용하면 데이터 누락을 방지 가능

출처:

https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html

https://velog.io/@simhyunmin/%ED%94%84%EB%A1%9D%EC%8B%9C-%EA%B0%9D%EC%B2%B4%EC%99%80-%EC%A7%80%EC%97%B0-%EB%A1%9C%EB%94%A9


### N + 1 문제

---
- **지연 로딩(Lazy Loading)** 으로 인해 관련 데이터를 가져올 때마다 쿼리가 추가로 실행되는 문제
    1. 지연로딩은 연관된 엔티티를 처음엔 로드하지 않고, `실제로 접근`할 때 쿼리를 실행해 데이터를 가져옴
    2. 10명의 Member가 작성한 Review를 확인하려고할 때
    3. Member 목록 조회 쿼리 : 1번 → 각 `Member`의 `Review` 조회 : 10개 `추가 쿼리`
    4. 따라서, 총 쿼리 수 : `1 + 10 = 11개` (1 + N 문제라고 하는 게 맞음 어떻게 보면)

    ```java
    //테스트 코드
    //사전 설정 : 10명의 Member 삽입 및 각 Member 마다 Review 1개 추가
    //지연 로딩으로 인해 추가 쿼리 10번 실행
    for (Member member : members) {
                System.out.println(
                "회원: " + member.getName() + 
                ", 리뷰 수: " + member.getReviews().size());
            }
    ```

    - `member.getReviews().size()` 를 호출할 때 마다 `지연로딩`이 적용된 `Reviews` 컬렉션을 가져오기 위해 별도의 쿼리가 실행됨
    - `Member`가 10명이기 때문에 추가 쿼리 10번 실행 
  
    <br>
  
    - **해결 방법**
    - **`EAGER fetch`**: 관련 데이터를 즉시 로드
    - **`JOIN fetch`**: `JOIN FETCH`를 사용해 한 번에 데이터를 조회
