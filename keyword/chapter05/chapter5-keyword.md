# 키워드 미션

## Domain

- 소프트웨어가 다루는 **현실 세계의 문제 영역**.
- 비즈니스의 개념, 규칙, 행동을 **코드로 추상화**한 것.
- 데이터(상태)와 행동(로직)을 함께 가진다.
- **Entity**, **Value Object(VO)**, **Repository** 등이 도메인을 구성하는 대표적인 것들.
- **도메인 주도 설계(DDD)**에서는 도메인을 중심으로 소프트웨어를 설계한다.

### 예시)

- **도서 대여 시스템**
- 회원(Member), 책(Book), 대여 기록(Rental)이 각각 도메인이다.
- 단순히 "데이터"가 아니라, "대여할 수 있는가?" 같은 **비즈니스 로직**도 포함한다.

```java
@Entity
public class Member {
    @Id
    private Long id;
    private String name;
    private boolean availableRental; // 대여 가능 여부

    public boolean canRent() {
        return availableRental;
    }
}
```

- 여기서 `Member`는 **단순 데이터**가 아니라 "대여 가능 여부"라는 **비즈니스 로직**도 갖고 있음.

---

## 양방향 매핑

- **두 객체가 서로를 참조**하는 관계를 설정하는 것.
- 예를 들어, `부모 → 자식`, `자식 → 부모` 모두 접근할 수 있도록 매핑.
- JPA에서는 `@OneToMany`와 `@ManyToOne`을 함께 사용.
- **연관관계 주인(Owner)**를 지정해야 함. (외래키를 실제로 관리하는 쪽)
- **주의**: 무한 순환 참조 (toString(), JSON 변환 시) 가능성 있음.

### 예시)

**부모(Parent)와 자식(Child) 관계**

```java
@Entity
public class Parent {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent") // 연관관계의 주인: Child
    private List<Child> children = new ArrayList<>();
}

@Entity
public class Child {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "parent_id") // 외래키 컬럼
    private Parent parent;
}
```

- `Child`가 **연관관계 주인(Owner)** → 실제 `parent_id` 외래키를 관리함.
- `Parent`는 **읽기 전용(Non-Owner)**.

### ⭐️중요한 점

- 값을 넣을 때 **양쪽 모두 세팅**해줘야 함.

```java

Child child = new Child();
Parent parent = new Parent();

child.setParent(parent);
parent.getChildren().add(child);
```

> 안 하면 둘 간의 객체 그래프가 일치하지 않아서 문제가 생길 수 있음.
>

---

## N + 1 문제

- 1번의 메인 쿼리로 N개의 데이터를 조회한 후,
- 각 데이터마다 1번씩 추가 쿼리를 보내는 현상.
- 총 쿼리 수: **1 + N**

  → 데이터가 많아질수록 성능이 **심각하게 저하**된다.

- 발생 이유: JPA 기본 로딩 전략이 **지연 로딩(LAZY)**이기 때문.

### 예시)

**회원(Member)와 주문(Order) 관계**

```java
@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;

    private String productName;

    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
}
```

**쿼리 발생 흐름**

```java
// 모든 회원 조회
List<Member> members = memberRepository.findAll();

// 회원별 주문 조회
for (Member member : members) {
    List<Order> orders = member.getOrders(); // 여기서 N번 추가 조회 발생!
}
```

- 1번: `SELECT * FROM member`
- N번: 각각 `SELECT * FROM order WHERE member_id = ?`

**→ 총 쿼리 수: 1 + N**

---

### 해결 방법

1. **Fetch Join** 사용
    - 처음부터 연관된 데이터를 함께 가져온다.

    ```java
    @Query("SELECT m FROM Member m JOIN FETCH m.orders")
    List<Member> findAllWithOrders();
    ```

2. **EntityGraph** 사용
    - 특정 필드를 로딩하도록 명시한다.

    ```java
    @EntityGraph(attributePaths = {"orders"})
    List<Member> findAll();
    ```

3. **Batch Size** 조정
    - 다건 조회 시, N개의 쿼리를 IN 절로 묶어서 보낸다.
    - `application.properties`

        ```
        spring.jpa.properties.hibernate.default_batch_fetch_size=100
        ```


---

| 구분 | 설명 | 예시 요약 |
| --- | --- | --- |
| Domain | 현실 세계 문제를 소프트웨어로 추상화. 비즈니스 로직도 포함. | 회원이 책을 대여할 수 있는지 판단하는 메서드 포함 |
| 양방향 매핑 | 객체 간 서로 참조. 연관관계 주인 설정 필수. | Parent-Child 간 @OneToMany / @ManyToOne 사용 |
| N+1 문제 | 1번 조회 후 N번 추가 조회로 성능 저하 발생. | 회원 조회 후, 각 회원 주문을 추가로 N번 조회 |