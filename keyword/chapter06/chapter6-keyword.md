# **키워드 미션**

## **지연로딩 vs 즉시로딩**

- JPA에서 연관관계를 맺는 엔티티를 어떻게 가져올지 결정하는 전략이다.

### **즉시로딩 (FetchType.EAGER)**

- 즉시로딩은 실제 DB에서 즉시 한 번에 조회하여, 하나의 쿼리로 가져온다.
- 만약 MemberPrefer를 조회할 때 Member, FoodCategory도 한번에 쿼리를 날리는 방법이다.
- 따라서 엔티티 조회하는 순간 연관된 모든 객체가 한꺼번에 로딩된다.
- 단순한 코드에서는 편하나, N+1 문제가 발생할 가능성이 높다.

### **지연로딩**

- 연관된 엔티티를 처음에는 프록시 객체로만 들고 있괴, 실제 사용하는 순간에 쿼리를 날려서 데이터를 가져온다.
- 만약 MemberPrefer를 조회할 때 Member, FoodCategory이 각각 분리되어 MemberPrefer은 DB를, Member 및 FoodCategory는 프록시를 조회한다.

### **지연로딩 장점**

- 필요한 순간만 쿼리를 실행하여 성능 최적화할 수 있다.
- 따라서 가급적 지연 로딩을 사용하는 것을 권장한다.

## N+1문제 최종 정리

### **즉시 로딩 (EAGER) — N+1 문제 발생**

``` java
@OneToMany(fetch = FetchType.EAGER)
private List<MemberPrefer> memberPreferList;
```

### JPA 동작 방식

1. 먼저 모든 `Member` 조회 (`SELECT * FROM member`)
2. 그리고 **각 Member마다** 추가로 메뉴 목록 조회

``` sql
SELECT * FROM member;
SELECT * FROM member_prefer WHERE member_id = 1;
SELECT * FROM member_prefer WHERE member_id = 2;
...
SELECT * FROM member_prefer WHERE member_id = 100;
```

즉시 로딩이면 = 한 명 불러올 때마다 연관된 걸 **바로 같이** 가져오려 함

**결과: 1 + N번의 쿼리** = **총 101번 → 성능 저하**

---

### **지연 로딩 (LAZY) — 성능 최적화 가능**

``` java
@OneToMany(fetch = FetchType.LAZY)
private List<MemberPrefer> memberPreferList;
```

### JPA 동작 방식

1. `Member`만 조회

```sql
SELECT * FROM member;
```

1. `memberPreferList`는 실제로 접근할 때까지 **쿼리 안 날림**
2. `member.getMemberPreferList()` 호출할 때 **그때서야** 쿼리 실행됨

즉, 필요한 순간까지 미뤄두기 = Lazy

### 해결방법

1. **지연 로딩(LAZY)** 사용 — 기본 전략이 LAZY임
2. 그리고 꼭 필요할 땐 한 번에 가져오자

``` java
@Query("SELECT m FROM Member m JOIN FETCH m.memberPreferList")
List<Member> findAllWithPrefer();
```

`JOIN FETCH` 쓰면 N+1 방지하면서 연관된 데이터 한 방에 가져온다.

## **Fetch Join**

- JPA에서 연관된 엔티티를 하나의 SQL 쿼리로 가져오기 위한 JPQL 구문이다.
- 이는 N+1문제를 해결하는 가장 전통적인 방법이다.

``` java
@Query("SELECT m FROM Member m JOIN FETCH m.orders")
List<Member> findAllWithOrders();
```

## **@EntityGraph**

- Fetch Join을 어노테이션으로 깔끔하게 표현할 수 있게 만든 JPA 기능이다.
- JPQL 없이 Fetch 전략을 제어할 수 있다.

### 사용 예시

``` java
@EntityGraph(attributePaths = "orders")
@Query("SELECT m FROM Member m")
List<Member> findAllWithOrders();

```

혹은 Spring Data JPA에서 더 간단하게

``` java
@EntityGraph(attributePaths = "orders")
List<Member> findAll();
```

내부적으로 Fetch Join 쿼리가 실행된다.

**기존 방식과 비교**

``` java
@Query("SELECT m FROM Member m JOIN FETCH m.orders")
List<Member> findAllWithOrders();
```

``` java
@EntityGraph(attributePaths = "orders")
@Query("SELECT m FROM Member m")
List<Member> findAllWithOrders();

```

### **장점**

- JPQL 없이 Fetch Join 가능하다
- 조건이 복잡하지 않다면 깔끔하고, 유지보수에 편리하다.

### **단점**

- 조건을 세밀하게 줄 수 없으며, 페이징 + 컬렉션 Fetch에서 사용을 주의해야한다.

## **JPQL (Java Persistence Query Language)**

- JPA의 쿼리언어로, SQL과 비슷하지만 테이블이 아니라 엔티티 기준으로 작성된다.

### 예시

``` java
@Query("SELECT m FROM Member m WHERE m.name = :name")
List<Member> findByName(@Param("name") String name);
```

여기서 `m.name`은 **DB 컬럼명**이 아니라 **Member 엔티티의 필드명을 뜻한다.**

### **특징**

- 객체 지향적이다. : Member 같은 클래스 이름과 필드 이름으로 쿼리를 작성한다.
- JOIN, WHERE, ORDER BY 등 SQL 기능 대부분 지원한다.
- 하지만 정적 쿼리 기반이라 조건이 많아지면 불편하고 지저분해진다.

## **QueryDSL**

- 타입 안전한 JPQL 빌더로, 동적 조건 결합이 가능한 JPA 쿼리도 이다.

### 예시

``` java
QMember m = QMember.member;

List<Member> members = queryFactory
    .selectFrom(m)
    .where(m.name.eq("홍길동"))
    .fetch();
```

---

### 동적 조건 처리 예제

``` java
BooleanBuilder builder = new BooleanBuilder();

if (searchName != null) {
    builder.and(m.name.eq(searchName));
}

if (ageGt != null) {
    builder.and(m.age.gt(ageGt));
}

List<Member> result = queryFactory
    .selectFrom(m)
    .where(builder)
    .fetch();

```

조건 있는 것만 `.where()`에 포함한다.