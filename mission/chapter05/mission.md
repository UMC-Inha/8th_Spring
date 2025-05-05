## 시니어 미션

### 📌 **미션 상세 내용**

### **1️⃣ 성능을 고려한 연관관계 매핑 & 최적화 적용**

- **`@OneToMany` 컬렉션을 조회할 때 `List<MemberPrefer>`를 `Set<MemberPrefer>`로 변경 후 차이점 분석**
    - 중복된 데이터를 허용하고 순서를 고려해야 할 경우 list 사용을, 중복을 제거해서 유일한 값을 넣어야 할 경우 set이 필요하다는 차이점이 있다.
    - member_prefer 테이블은 list 보다 set으로 구현하는게 적절해 보인다. 그 이유는 “선호도”라는 테이블을 생각해 보면, 컬럼 추가보다는 데이터를 조회할 일이 많기 때문에, 빠른 검색이 가능한 HashSet 등을 사용하는 것이 적합해 보인다. 또한, 애플리케이션 계층에서 중복 선호도를 자동으로 제거할 수 있어 Set이 더 유용하게 활용될 수 있다.
    - 
- **데이터 정합성을 고려하여 `orphanRemoval = true`가 필요한 곳 확인 후 적용**
    - orphanRemoval = true 란, 고아객체가 발생하면 자동으로 DB에서 제거해주는 속성을 의미한다. 고아객체란 부모와 자식이 연관관계가 떨어진 자식을 의미하는데 고아 객체가 되는 방법으로
        - review.setMember(null); → 이렇게 필드에 null을 선언하거나
        - member.getReviews().remove(review); 이렇게 컬렉션에서 제거하는 방법이 있다.
    - **`orphanRemoval = true` 가** 필요한 경우 → 자식 엔티티가 해당 부모 엔티티하고만 부모-자식 관계를 맺고 있을 때!  또한, delete 없이 곧바로 삭제가 가능하므로 간편성 측면에서 유리함
    - 따라서 해당 조건을 만족은 관계를 찾아보면 Restaurant(부모)-RestaurantFood , Restaurant(부모)-RestaurantImage(자식), Review(부모)-ReviewImage(자식) 이렇게 존재해서 각각 @OneToMany 속성에 `orphanRemoval = true` 를 적용했다.
        
        

---

### **2️⃣ 트랜잭션 & 동시성 이슈 처리**

- **하나의 트랜잭션에서 여러 엔티티를 처리하는 비즈니스 로직 작성**
    - 예) `Member`가 탈퇴할 경우 **관련된 모든 데이터를 삭제하는 API** 구현
        
        ```java
        @Transactional
        public void inactivateUser(User user) {
        // 1. 유저 상태 비활성화
        user.setStatus(Status.INACTIVE);
        
        // 2. 관련된 엔티티 수동 삭제
        reviewRepository.deleteAll(user.getReviewList());
        commentRepository.deleteAll(user.getCommentList());
        userMissionRepository.deleteAll(user.getUserMissionList());
        userPreferenceRepository.delete(user.getUserPreference());
        
        // 3. 연관관계 해제 -> 연관관계의 주인이 아니므로, 해제가 필수는 아니지만
        // 1) 객체 상태의 동기화
        // 2) JPA flush 시점까지의 객체 무결성 유지
        // 의 장점이 있기에 자식 쪽의 객체 정보도 지워주는 것이 좋다 
        user.getReviewList().clear();
        user.getCommentList().clear();
        user.getUserMissionList().clear();
        user.setUserPreference(null);
        
        ```
        
        }
        
    - `@Transactional`을 적용하고, `@Modifying`을 활용하여 **Batch Delete 쿼리 최적화**
        - Batch Delete를 하기 위해 하나씩 개념부터 배워보자
        - 우선, JPA는 영속성 컨텍스트의 장점(변경 감지 등)의 장점이 있지만, 이는 조회할 때의 장점인 거고 만약 update를 해야한 다면 컨텍스트가 있든 없든 DB에서의 수정이 필요하므로 JPA의 update 메소드가 존재하지 않는다
            
            따라서, Entity의 상태를 변경하고 여기서 변경 감지를 통해 update를 하는 방식을 사용가능하다.
            
        - 하지만 대량의 데이터가 한 번에 수정이 필요한다면 n 번의 update 반복이 필요하기에 @Modifying 어노테이션으로 Batch Delete라는 대량 업데이트가 가능하다.
        - 하지만 @Modifying을 하면 영속성 컨텍스트를 무시하기에, 업데이트 후 조회 시 실제 DB의 내용과 다를 수가 있다. 따라서 clear() , clearAutomatically = true 등의 방식으로 영속성 컨텍스트 업데이트가 필요하다
        
        ```java
        @Modifying(clearAutomatically = true)
        @Query("DELETE FROM Review r WHERE r.user = :user")
        void deleteByUser(@Param("user") User user);
         
        // 해당 방식으로 comment, userMission, userPreference Repositorty 도 생성하기
        ```
        
        ```java
        @Transactional
        public void inactivateUser(User user) {
            userRepository.inactivateUser(user);
            reviewRepository.deleteByUser(user);
            commentRepository.deleteByUser(user);
            userMissionRepository.deleteByUser(user);
            userPreferenceRepository.deleteByUser(user);
        }
        
        ```
        
- **동시성 문제가 발생할 수 있는 시나리오**를 고민하고 해결책 적용
    - 예) 같은 회원이 동시에 같은 `Store`를 찜하려고 할 때 중복이 발생하지 않도록 `@Lock` 사용
        - 동시에 같은 Store을 찜하는 것은 **동시에 write**을 한다는 의미이며 이게 중복이 발생하면 안 되므로, @Lock(LockModeType.PESSIMISTIC_WRITE) → 을 통해 강력하게 통제하는 것이 좋다.
    - 다양한 락킹 전략에 대해 공부해보고, 이를 정리하기
        - @Lock의 여러 인자
            - Optimistic : 낙관적 락이며, 버전 충돌 시 exception이 발생한다
            - Optimistic_Force_Increment : 낙관적 락이며, 트랜잭션이 끝나면 버전이 증가한다
            - Pessimistic_Read : 비관적락 읽기, 읽기를 제외한 다른 트랜잭션의 접근 차단
            - Pessimistic_Write : 비관적락 쓰기, 모든 트랜잭션의 접근 차단
