- Spring Data JPA의 Paging
    
    : **전체 데이터 중 일부만을 잘라내어** 클라이언트에 전달하는 기술
    
    이를 위해 Spring Data JPA에서는 `Pageable`이라는 인터페이스를 기반으로 한 구현체 `PageRequest`를 사용하여 요청을 구성하고, 반환 타입으로 `Page<T>` 또는 `Slice<T>`를 사용
    
    - 구현을 위한 주요 parameter
        - page : 가져올 페이지 번호 (0부터 시작)
        - size : 한 페이지에 포함될 데이터의 개수
        - sort : 데이터 정렬 기준
    
    ```java
    Pageable pageable = PageRequest.of(0, 10, Sort.by("createdAt").descending());
    Page<Post> posts = postRepository.findAll(pageable);
    ```
    
    - Page와 Slice의 공통점
        - 페이징 데이터를 제공: 두 객체 모두 페이징 처리된 데이터를 포함
        - Pageable 지원: PageRequest 같은 Pageable 구현체를 통해 페이징 설정을 적용 가능
        - 메서드 제공: 데이터를 순회하거나 현재 페이지 정보를 확인할 수 있는 다양한 메서드를 지원
    - Page
        
        : 전체 개수까지 계산하는 완전한 페이징
        
        - 요청한 페이지의 데이터뿐 아니라 전체 데이터 개수와 전체 페이지 수까지 포함
            
            두 개의 쿼리가 실행
            
            1. 실제 데이터 조회용 쿼리 `SELECT ... LIMIT`
            2. 전체 개수 조회용 카운트 쿼리 `SELECT COUNT(*)`
        - Slice와 차이점
            - 데이터 조회 쿼리와 함께 전체 데이터 개수를 조회하는 count 쿼리 실행
            - 이를 통해 전체 페이지 수와 전체 데이터 개수를 확인 가능
            - `getTotalPages()` : 전체 페이지 수 반환
            - `getTotalElements()` : 전체 데이터 개수 반환
        
        ```java
        Page<Member> page = memberRepository.findAll(PageRequest.of(0, 5));
        System.out.println(page.getTotalElements()); // 전체 멤버 수
        System.out.println(page.getContent());       // 첫 페이지 멤버 목록
        ```
        
    - Slice
        
        : 전체 개수 정보를 조회하지 않는 단순한 페이징
        
        - **현재 페이지 + 1개 더 조회**해서 다음 페이지 존재 여부(`hasNext()`)만 판단
        - Page와 차이점
            - 데이터 조회 쿼리만 실행
            - 다음 페이지가 존재하는지만 확인하기 위해 limit(size + 1)로 데이터를 가져옴
            - 전체 데이터 개수나 전체 페이지 수는 알 수 X
            - 전체 데이터 개수나 페이지 수를 반환하는 메서드 X
            - 대신 hasNext()를 통해 다음 페이지가 있는지 여부만 확인 가능
        
        ```java
        Slice<Member> slice = memberRepository.findByAgeGreaterThan(20, PageRequest.of(0, 5));
        System.out.println(slice.hasNext());  // 다음 페이지 존재 여부
        System.out.println(slice.getContent()); // 현재 데이터
        ```
        
- 객체 그래프 탐색
    
    : 객체 간의 관계를 따라 **연관된 객체를 탐색하는 과정**
    
    - ex) 회원이 속한 팀 객체를 찾고, 팀의 주소 정보를 거쳐 최종적으로 팀 주소의 도시명 찾기
        
        ```java
        Team team = member.getTeam();
        String city = member.getTeam().getAddress().getCity();
        ```
        
        객체 모델에 기반, 데이터베이스의 테이블 설계와 대응
        
    - JPA가 ORM 방식 사용하여 **자바 객체 수준에서 직관적으로 탐색**할 수 있게 지원
        
        ORM은 데이터베이스의 테이블과 그 관계를 자바 객체와 그 참조로 매핑해주기 때문에, 개발자는 SQL 문을 직접 다루는 대신 객체지향적인 코드로 데이터를 다룰 수 있음
        
        - SQL 기반 접근의 한계
            - SQL을 직접 사용하는 경우, 객체 그래프 탐색의 범위는 작성된 SQL 문에 의해 제한
                
                ex) 특정 SQL이 객체 B와 C만 조회한다면, 객체 D에 접근하려면 추가적인 SQL이 필요..
                
            - 탐색 가능 범위를 확인하거나 확장하려면 DAO(Data Access Object)를 열어서 직접 SQL 문을 분석하고 수정해야 함..
            - 객체 A와 관련된 모든 객체를 한 번에 조회하여 메모리에 로드하는 방식은 비효율적.
                
                결국, 필요한 객체별로 여러 개의 DAO 메서드를 작성해야 함..
                
            
            ⇒ **JPA는 이를 추상화하여 객체 탐색처럼 자연스럽게 처리**
            
    - JPA와 그래프 객체 탐색
        - 연관된 객체를 실제로 사용하는 시점에 필요한 SELECT SQL을 실행
            
            = 지연로딩 → 데이터베이스 조회 시점을 지연시켜 효율적인 리소스 사용이 가능
            
            ```java
            Member member = entityManager.find(Member.class, memberId); // Member만 조회
            System.out.println("회원 이름: " + member.getName());
            
            // 여기서 Team 정보가 필요해짐 -> 이 시점에 Team을 조회하는 SQL 실행!
            Team team = member.getTeam();
            System.out.println("팀 이름: " + team.getName());
            ```
            
        - 객체 A를 사용할 때 항상 객체 B를 함께 사용하는 경우, JPA는 조인을 이용해 A를 조회할 때 B까지 즉시 함께 조회 가능
            
            연관 객체를 즉시 로딩으로 설정하여 구현 가능
            
            ```java
            // @ManyToOne(fetch = FetchType.EAGER)
            // private Member member;
            
            Order order = entityManager.find(Order.class, orderId); // Order와 Member 정보를 JOIN으로 한 번에 조회
            System.out.println("주문 번호: " + order.getId());
            // 추가 SQL 없이 바로 Member 정보 사용 가능
            System.out.println("주문 회원: " + order.getMember().getName());
            ```
            
        - JPA는 연관된 객체를 즉시 로딩할지, 지연 로딩할지 어노테이션으로 정의 가능
            
            이로 인해 객체의 사용 패턴에 따라 최적화된 데이터 조회를 제공!
            
    - 객체를 비교할때!
        - 동등성 (Equality) : 두 객체가 **값(내용)이 같은지**를 비교
            
            자바에서는 `equals()` 메소드를 오버라이드하여 동등성 비교 로직을 정의
            
            ex) `user1.equals(user2)`
            
        - 동일성 (Identity) : 두 객체 참조 변수가 **메모리 상에서 정확히 같은 객체를 가리키는지**를 비교
            
            자바에서는 `==` 연산자를 사용
            
            ex) `user1 == user2`