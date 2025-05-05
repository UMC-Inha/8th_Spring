- Domain
    
    : 소프트웨어가 해결하고자 하는 문제 영역 or 특정 비즈니스 로직
    
     ⇒ 애플리케이션이 집중해야 할 비즈니스 자체
    
    - 도메인 객체는 데이터와 그 데이터를 처리하는 비즈니스 로직을 캡슐화함
        
        ex) 쇼핑몰 애플리케이션
        
        - Member과 Product은 각각 도메인에 해당하며, 각각의 고유한 특성과 역할을 가짐
        - ‘VIP 고객에겐 10% 할인’ 같은 정책은 도메인 로직
    - 단순히 데이터를 담는 DTO와 달리, 도메인 객체는 규칙과 책임을 가짐
    - 도메인 객체는 데이터베이스의 테이블과 직접적으로 연결되기도 하지만, 데이터의 저장소와는 독립적으로 애플리케이션의 실제 로직을 담당하는 주체이다.
    
    도메인은 기획의 요구사항을 구현하고, 문제를 풀기 위해 설계된 소프트웨어 프로그램의 기능성을 정의
    
- 양방향 매핑
    
    : 두 엔티티가 서로를 참조할 수 있도록 설정하는 방법
    
    - 양방향 매핑을 통해 양쪽 엔티티에서 상대방 엔티티에 접근할 수 있게 됨으로써 데이터 접근성과 로직 작성의 유연성이 높아짐
        
        ex) Member와 Review 엔티티 간의 관계
        
        - Member는 여러 개의 Review를 가지고 있을 수 있으며, 반대로 각 Review는 특정 Member에 속하게 됨
        
        이때, 양방향 매핑을 적용하면 
        
        - Member 엔티티에서 Review를 참조 (`List<Review>`)
        - Review 엔티티에서 Member를 참조 (`Member member`)
    
    ⇒ 단방향 매핑에서 각각의 관계를 개별적으로 가져와서 다시 매칭해야 하는 번거로움 줄여줌
    
    - 양방향 매핑의 단점
        - 순환 참조에 의해 발생하는 무한 루프와 엔티티 사이의 관계 설정이 꼬일 때 발생하는 데이터 불일치 문제
            
            ex) Member 엔티티의 Review 리스트에 리뷰를 추가하면, 동시에 해당 Review 엔티티의 Member 필드도 적절히 설정해 주어야 데이터의 일관성이 유지된다.
            
    
    ⇒ 이러한 문제를 방지하기 위해 양방향 매핑에서는 보통 하나의 엔티티를 "주인"으로 지정하고, 관계의 변경을 주인 엔티티에서만 관리하도록 함
    
    ```java
    class Review {
      @ManyToOne
      @JoinColumn(name = "member_id")
      private Member member;  // ★ 관계의 주인
    }
    
    class Member {
      @OneToMany(mappedBy = "member")
      private List<Review> reviewList;
    }
    ```
    
    - JPA에서는 mappedBy 속성을 통해 관계의 주인을 지정하여, 반대쪽 엔티티의 데이터 업데이트에 의한 데이터 일관성 문제를 방지
    
- N + 1 문제
    
    : JPA에서 연관된 데이터를 조회할 때 JPA가 **불필요하게 많은 쿼리**를 실행해서 **성능 저하**가 발생하는 현상
    
    - 기본적으로 JPA는 지연 로딩을 통해 주 엔티티를 조회하고, 필요한 시점에 연관 엔티티를 추가로 로딩한다.
    - 그러나 다수의 연관 엔티티를 가진 경우, N+1 문제가 발생하여 성능에 악영향을 미칠 수 있다.
        
        ex) 회원 목록을 조회하면서 각 회원이 작성한 리뷰도 같이 보고 싶을 때
        
        - 처음 회원 목록을 가져오는 쿼리 1개가 실행
        - 그러나 이후 각 회원마다 리뷰를 가져오기 위해 N개의 쿼리가 추가로 실행
            
            → 총 N+1개의 쿼리가 발생
            
    
    ⇒ 이 문제를 해결하기 위해서는 Fetch Join이나Batch Loading을 사용
    
    - Fetch Join : JOIN을 통해 주 엔티티와 연관 엔티티를 한 번에 가져오는 방법
        
        ```java
        @Query("SELECT m FROM Member m JOIN FETCH m.reviews")
        List<Member> findAllWithReviews();
        ```
        
    - Batch Loading : JPA에서 지정한 수만큼 데이터를 한 번에 로딩하여 쿼리 호출을 줄이는 방식으로, 연관 엔티티를 한 번에 묶어 로딩
        
        ```java
        # application.yml
        jpa:
          properties:
            hibernate:
              default_batch_fetch_size: 100
        ```