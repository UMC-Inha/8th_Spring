### 일반 미션

- 기준 ERD

![image](https://github.com/user-attachments/assets/55282218-46a8-4b4a-a602-67877db104c9)

- 미션 인증

![image](https://github.com/user-attachments/assets/6b5a0b4b-f4ea-4014-9149-c12ca7a3709b)

- 미션 레포 주소
  -  https://github.com/iseevict/8th_spring_prac/tree/mission5

## 시니어 미션

### 1. @OneToMany 컬렉션을 조회할 때 List\<MemberPrefer>를 Set\<MemberPrefer>로 변경 후 차이점 분석
- **`@OneToMany` 컬렉션을 조회할 때 `List<MemberPrefer>`를 `Set<MemberPrefer>`로 변경 후 차이점 분석**
    - 모두 단방향으로 구현하여 @OneToMany 가 없어 변경은 못하였습니다.
    - 차이점
        - List는 중복을 허용하지만 Set 은 중복을 허용하지 않음
        - List는 순서를 보장, Set은 순서를 보장하지 않음
    - 성능 차이
        - List는 순차적으로 요소에 접근하는 데 효율적
            - 중복 검사에 O(n) 만큼 시간 복잡도를 가짐
        - Set은 중복 검사가 내장되어 있음
            - 중복 검사에 O(1) 만큼 시간 복잡도를 가짐
    - 메모리 차이
        - List는 순차적으로 데이터를 저장하므로 배열 기반 → 메모리 오버헤드가 상대적으로 적음
            - but, 중복 데이터로 오히려 더 많은 메모리를 사용할 수 있음
        - Set은 중복을 허용하지 않지만, 내부적으로 해시 테이블이나 트리 구조와 같은 자료 구조를 사용 → 메모리 오버헤드가 더 클 수 있음
    - 결론
        - List는 중복 허용, 순서 보장 이 필요한 경우 적합
        - Set은 중복을 허용하지 않으며 순서가 중요하지 않은 경우, 빠른 중복 검사 및 성능 최적화가 필요한 경우 적합

### 2. 데이터 정합성을 고려하여 orphanRemoval = true 가 필요한 곳 확인 후 적용
- 데이터 정합성?
    - 데이터가 정확하고, 오류 없이 일관성을 유지하는 것
- orphanRemoval 란?
    - JPA 에서 부모 엔티티에서 자식 엔티티가 고아 상태 (orphaned) 로 남아 있을 때 이를 자동으로 삭제하도록 설정하는 옵션
- 필요한 곳 확인 후 적용
    - 일대다 단방향/양방향 의 경우 사용하는 것으로 알고 있습니다.
    - 제가 구현한 코드는 현재 다대일 단방향만 존재하기에 사용할 곳이 없습니다.

### 3. 하나의 트랜잭션에서 여러 엔티티를 처리하는 비즈니스 로직 구현

#### 3-1. Member 가 탈퇴할 경우 관련된 모든 데이터를 삭제하는 API 구현
- https://github.com/iseevict/8th_spring_prac/tree/mission5

#### 3-2. @Transactional 을 적용하고, @Modifying 을 활용하여 Batch Delete 쿼리 최적화
- 최적화 전
  -  https://github.com/iseevict/8th_spring_prac/commit/d62d87fe63f07d1987df2c8d73f8238277d7b3ed

- 최적화 후
  - https://github.com/iseevict/8th_spring_prac/commit/c8b131ac649090fe90a0295b62c6760bdca38ca7

-> for-each 로 삭제하던 것들 @Modifying + @Query 를 사용하여 최적화 완료

### 4. 동시성 문제가 발생할 수 있는 시나리오를 고민하고 해결책 적용

#### 4-1. 같은 회원이 동시에 같은 Store 를 찜하려고 할 때 중복이 발생하지 않도록 @Lock 사용
- 같은 회원이 동시에 같은 Restaurant에 리뷰를 하려고 할 때 중복이 발생하지 않도록 @Lock 사용? → 잘 모르겠음.. 거래나 티켓팅 관련 서비스의 경우 중요하게 쓰일 것 같음
#### 4-2. 다양한 락킹 전략에 대해 공부해보고, 이를 정리하기
- 크게 2가지
    - 비관적 락 / 낙관적 락
- 비관적 락 (Pessimistic Locking)
    - 자원에 대한 잠금을 설정 → 다른 트랜잭션이 자원에 접근하지 못하도록 막는 방식
    - 주로 쓰기 잠금 사용, 동시성 제어 가능
    - **`LockModeType.PESSIMISTIC_WRITE`**
        - 데이터 **쓰기 잠금**. 다른 트랜잭션이 해당 데이터를 수정하지 못하게 함
    - **`LockModeType.PESSIMISTIC_READ`**
        - 데이터 **읽기 잠금**. 다른 트랜잭션이 해당 데이터를 수정할 수 없지만 읽을 수는 있음
    - 장점
        - 데이터 일관성 보장 가능
        - 동시에 여러 트랜잭션이 동일한 데이터 수정하려는 경우 경쟁 조건 방지 가능
    - 단점
        - 성능 저하 가능
        - 트랜잭션이 끝날 때까지 자원을 잠금 상태로 유지 → 대기시간 증가
- 낙관적 락 (Optimistic Locking)
    - 엔티티에 버전 관리를 추가하여 사용
    - @Version 어노테이션을 통해 버전 정보 관리, OptimisticLockException 을 처리하여 충돌 감지

    ```java
    @Version  // 버전 관리 필드
        private Long version;
    ```

    - 장점
        - 성능 뛰어남
        - 데이터 잠기지 않음 → 읽기 성능 향상, 충돌일어나지 않으면 잠금 비용 발생X
    - 단점
        - 충돌 일어날 수 있음
        - 데이터 수정 후 다른 트랜잭션에서 업데이트 하려고 할 때 OptimisticLockException 발생 → 따로 처리 로직 추가 필요
- 추가적으로 트랜잭션 격리 수준을 통해 관리 가능