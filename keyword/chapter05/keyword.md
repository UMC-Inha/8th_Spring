## - 키워드 정리

#### - Domain

- 소프트웨어로 해결하고자 하는 문제 영역

- 사용자가 이용하는 앱 기능, 회사의 비즈니스 로직을 정의하는 영역

- ex) 온라인서점 사이트를 개발한다면, 온라인 서점은 도메인이 될 수 있다. 온라인 서점 도메인은 다시 주문, 결제, 배송같은 하위 도메인으로도 나뉠 수 있다.

- **Domain vs Entity**

    - Domain은 서비스를 대표하는 개념이고, Entity는 Domain 내에 식별 가능한 객체

    - 도메인 객체가 다시 Entity와 Value Object(VO)로 나뉜다.

    - Entity

        - 식별자를 갖는 객체

    - Value Object

        - 식별자를 가지지 않는 값 그 자체.

#### - 양방향 매핑

- 두 엔티티가 서로를 참조하는 관계

- 데이터베이스 테이블은 외래 키 하나로 양방향을 조회할 수 있지만, **JPA에서는 단방향 연관관계 2개를 묶어 양방향 연관관계를 구현한다*.*

- 따라서 JPA에서는 **두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하고,** 이를 **연관관계의 주인**이라고 한다.

- **연관관계의 주인**

    - **테이블에 외래 키가 있는 곳**으로 정해야 하며, **1:N 관계에서 N 쪽이 연관관계의 주인으로 하는 것이 좋다.**

    - @ManyToOne이 기본

    - 주인이 아님을 표시하는 속성에는 mappedBy 속성을 넣는다.

- **양방향 연관관계 주의점**

    - Member와 Team이 N:1 관계로 있다고 할 때, Member에도 Team을 주입해주고(member.setTeam(team)), Team에도 member를 주입해주어야 한다. (team.getMembers().add(member))

- **연관관계 편의 메서드**

    - 연관관계 설정 시 한 쪽만 설정하는 실수를 할 수도 있으니, 다음과 같이 두 개의 연관관계 설정 코드를 한 코드에 묶은 것을 연관관계 편의 메서드라고 한다.

  ``` java
	 public void setTeam(Team team) {
        this.team = team;
	       team.getMembers().add(this);
	 }
  ```

#### - N + 1 문제

- 특정 객체를 대상으로 수행한 쿼리가 해당 객체가 가지고 있는 연관관계 또한 조회하게 되면서, **1**번의 쿼리로 **N**번의 추가적인 쿼리까지 발생하는 문제를 말한다.

- 예시 코드

  ``` java
  List<Member> members = memberRepository.findAll();
  for (Member member : members) {
      System.out.println(member.getOrders().size()); // 여기서 N + 1 문제 발생
  }
  ```

- 문제점

    - 1번의 쿼리로 여러 번의 요청이 나감 -> DB 부하, 비용 증가

    - 쿼리 수를 예측할 수 없다.

    - 응답 속도가 저하된다.

- 해결책

    - 연관관계의 fetch 전략을 지연로딩(fetchType.LAZY)으로 설정한다.

    - **fetch join** 방식으로 문제를 해결한다.

    - **@EntityGraph** 방식으로 문제를 해결한다.

    - **BatchSize**를 이용한다.

#### - JPA

- Java Persistence API

- 자바 객체를 데이터베이스에 저장(영속)하고, 관리하기 위한 표준 API

- JPA는 자바 객체를 알아서 테이블로 매핑해주는 표준 규약이다.

- JPA의 역할

    - 객체와 태이블 자동 매핑

    - CRUD SQL 자동 생성

    - 트랜잭션 관리

        - 영속성 컨텍스트로 일괄 관리

    - 캐시 관리

        - 1차 캐시를 사용해 중복 조회 최소화

    - 변경 감지(Dirty Checking)

        - 엔티티 값이 바뀌면 자동으로 update 쿼리 수행

- **영속성 컨텍스트**

    - 엔티티를 저장하고 관리하는 메모리 상의 1차 캐시 공간

    - 영속성 컨텍스트의 특징

        - 엔티티 저장

            - EntityManager를 통해 persist() 하면 일단 메모리에 저장한다.

        - 1차 캐시

            - 같은 엔티티를 다시 조회하면 DB에 안 가고 메모리에서 가져옴

        - 변경 감지(Dirty Checking)

            - 엔티티 값이 바뀌면 자동으로 update 쿼리 수행

        - 쓰기 지연

            - 트랜잭션 커밋 시점에 insert/update 쿼리를 한 번에 모아서 보낸다.

        - 엔티티 동일성 보장

    - **엔티티 상태**

        - 비영속(new)

            - 영속성 컨텍스트에 아직 등록되지 않은 상태

        - 영속(managed)

            - 영속성 컨텍스트에 의해 관리되는 상태

        - 준영속(detached)

            - 영속성 컨텍스트에서 분리된 상태

        - 삭제(removed)

            - 삭제 예약 상태(flush 시 delete 쿼리 발생)

## - 시니어 미션

#### - @OneToMany 컬렉션을 조회할 때 List<'MemberPrefer'>를 Set<'MemberPrefer'>로 변경 후 차이점 분석하기

- **List와 Set의 차이**

    - List : 순서 보장, 중복 허용

    - Set : 순서 없음, 중복 불허

- **결론부터 말하면 List를 사용하는 것이 좋다.**

- JPA에서 Set을 사용할 때 문제점

    - JPA는 연관 관계가 Lazy Loading으로 구현되어 있는데, **Set의 경우 기존에 가지고 있던 Entity에 중복된 데이터가 있는지 비교하기 위해 Set에 있는 모든 데이터를 로딩해야 하고 이 때 Proxy가 강제로 초기화된다.**

- ManyToMany 관계에서는 List보다 Set이 더 낫다고 한다. 하지만 ManyToMany 관계는 @ManyToMany 어노테이션 대신 매핑 테이블을 통해 구현하므로, Set을 사용할 일은 없을 것 같다.

-> **연관관계 매핑 시 Set 대신 List를 사용하자.**

#### - 데이터 정합성을 고려하여 orpanRemoval=true가 필요한 곳 확인 후 적용

- **orphanRemoval이란?**

    - 부모 엔티티가 자식 엔티티를 컬렉션에서 제거했을 때, 그 자식 엔티티를 DB에서도 자동으로 삭제해주는 기능

    - 왜 필요할까?

        - @OneToMany와 같은 관계에서 **부모 객체에서 자식을 컬렉션에서 제거해도 DB에서는 삭제가 되지 않고 관계만 끊기기 때문이다. ** orphanRemoval을 쓰면 이걸 자동으로 해결해준다.

    - 즉, 독립적으로 존재가 불가능할 경우에 적용한다.

    - 둘 이상의 부모와 연관관계를 맺을 경우, orphanRemoval 옵션을 false로 하는 것이 좋다.

- **Cascade(영속성 전이)란?**

    - 하나의 엔티티에 수행한 작업을 연관된 다른 엔티티에도 같이 수행하는 기능.

    -  부모가 저장되거나 삭제되면 연관된 자식들도 자동으로 저장되거나 삭제되는 기능

    - CascadeType 종류

        - PERSIST : 부모 저장 시 자식도 저장

        - MERGE : 부모 수정 시 자식도 수정

        - REMOVE : 부모 삭제 시 자식도 삭제

        - REFRESH : 부모를 새로고침하면 자식도 새로고침

        - DETACH : 부모를 영속성 컨텍스트에서 분리하면 자식도 분리

        - ALL : 위의 모든 작업을 전파

- **orphanRemoval과 Cascade의 차이는 무엇일까?**

    - **orphanRemoval은 컬렉션에서 제거되어 관계가 끊겼을 때 삭제를 한다.**

    - **cascade = REMOVE은 부모 엔티티가 삭제될 때 같이 삭제를 한다.**


- **orphanRemove=true가 필요한 곳은 어디가 있을까?**
  
<img src = "https://velog.velcdn.com/images/sm011212/post/ba8ec945-f114-45ac-b217-7c207ab361d3/image.png" width = "600">

- ERD를 보면 N:1 관계가 여러 상황에서 쓰이고 있다.

- member_agree, member_prefer, review, member_mission 엔티티는 두 개 이상의 엔티티와 N:1 관계를 맺고 있기 때문에, orphanRemoval=true 옵션이 좋지 않다고 생각한다.

- review_image의 경우 특정 review 엔티티에 종속적이고, review 엔티티의 컬렉션에서 제거되면 고아 객체가 되기 때문에 이 때는 orphanRemoval=true 옵션을 사용해도 괜찮다고 생각한다.

#### - 하나의 트랜잭션에서 여러 엔티티를 처리하는 비즈니스 로직 작성

- **@Modifying 어노테이션이란?**

    - JPA에서 INSERT, UPDATE, DELETE 처럼 데이터를 변경하는 쿼리를 작성할 때 사용하는 어노테이션

    - JPA의 기본 **@Query는 조회용 쿼리만 지원한다.**

    - 쿼리가 단순 조회가 아니라 변경 작업임을 명시하기 위해 @Modifying을 사용한다.

    - 사용 예시

      ``` java
      @Modifying
      @Query("UPDATE Member m SET m.name = :name WHERE m.id = :id")
      int updateMemberName(@Param("id") Long id, @Param("name") String name);
      ```

- **하나의 트랜잭션에서 여러 엔티티를 처리하는 비즈니스 로직 작성**

    - 먼저 다음과 같이 비즈니스 로직을 작성할 수 있다.

      ``` java
      @Transactional
      public void deleteMember(Long memberId) {
          reviewRepository.deleteByMemberId(memberId);     // 리뷰 삭제
          bookmarkRepository.deleteByMemberId(memberId);   // 찜 삭제
          orderRepository.deleteByMemberId(memberId);       // 주문 삭제
          memberRepository.deleteById(memberId);            // 회원 삭제
      }
      ```
    - 리포지토리에서는 다음과 같이 작성할 수 있다.

      ``` java
      @Modifying
      @Query("DELETE FROM Review r WHERE r.member.id = :memberId")
      void deleteByMemberId(@Param("memberId") Long memberId);
      ```

#### - 동시성 문제가 발생할 수 있는 시나리오를 고민하고 해결책 적용

- **동시성 문제란?**

    - 여러 사용자가 동시에 같은 데이터에 접근하거나 수정할 떄, 데이터 일관성이 깨지거나 얘기치 않은 결과가 발생하는 문제

    - 데이터 무결성과 관련된 중요한 문제이다.

    - 동시성 문제가 발생하는 상황

        - 동시 업데이트, 중복 삽입, 삭제/수정 중 충돌, 낙관적/비관적 락 실패

        - ex) 동일 사용자가 다른 디바이스로 접속 시, 포인트 충전/차감 동시 발생, 여러 번 찜하기, 마지막 남은 재고를 두 명이 동시에 구매

    - 동시성 문제가 발생하면 생기는 결과

        - 데이터 유실, 데이터 중복, 무결성 제약 위반, 프로그램 오류 발생

    - 동시성 문제를 해결하는 방법

        - **낙관적 락**, **비관적 락**, Unique 제약 조건(중복이 안 될 때만) 등

- **낙관적 락(Optimistic Lock)**

    - 락을 미리 걸지 않고, 데이터를 수정할 때 버전을 체크해서 충돌을 감지하는 방식

    - 동작 흐름

        1. 데이터를 조회할 때 버전 값을 함께 읽는다.

        2. 업데이트 할 때 버전 값을 비교한다.

        3. 만약 내가 읽었던 버전과 DB 버전이 다르면 충돌이 감지되어 OptimisticLockException이 발생한다.

    - 코드 예시

      ``` java
      @Entity
      public class Member {
          @Id @GeneratedValue
          private Long id;
  
          @Version
          private Long version;  // @Version을 추가하면 Hibernate가 자동으로 버전 관리를 해준다.
  
          private String name;
      }
      ```

    - 장점

        - Lock이 없어서 대부분의 경우 성능이 좋다.

        - 대량 읽기 시스템에 적합하다.

    - 단점

        - 충돌 발생 시 예외 처리가 필요하다.

        - 충돌 시 재시도 로직이 필요하다.

  -> 트래픽이 많고 읽기 성능이 중요하지만, 데이터 충돌 가능성이 낮거나 충돌 발생 시 처리를 할 수 있을 때 사용하면 좋다.

    - 사용 예시

        - 게시판 게시글 좋아요

        - 상품 조회수 증가

        - 프로필 수정

        - 일반 게시글 수정


- **비관적 락(Pessimistic Lock)**

    - 데이터를 조회할 때부터 다른 트랜잭션이 읽거나 수정을 못하게 락을 잡는 방식

    - 동작 흐름

        1. 데이터를 조회할 때 SELECT FOR UPDATE와 같은 SQL이 실행된다.

        2. 해당 데이터를 다른 트랜잭션이 접근하려고 하면 대기하거나 실패한다.

        3. 내 트랜잭션이 완료되면 락이 풀린다.

    - 코드 예시

  ``` java
  @Lock(LockModeType.PESSIMISTIC_WRITE)
  @Query("SELECT s FROM Store s WHERE s.id = :id")
  Store findByIdForUpdate(@Param("id") Long id);
  ```

    - 장점

        - 충돌을 아예 막을 수 있다. -> 데이터 일관성을 확보할 수 있다.

    - 단점

        - 트래픽이 많으면 락 경쟁이 발생, 데드락 발생 가능성이 있다.

        - 락을 기다리는 동안 대기를 해야하므로 성능 저하가 발생한다.

  -> 트래픽이 많지 않지만 데이터 충돌 가능성이 높고 데이터 일관성이 우선일 때 사용한다.

    - 사용 예시

        - 재고 감소 처리

        - 포인트 차감

        - 좌석 예약 시스템

        - 결제/송금 처리