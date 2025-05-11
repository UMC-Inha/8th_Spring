# 💡 6주차 핵심 키워드 💡
<br>

## 1. 지연로딩과 즉시로딩의 차이

### 지연로딩❓

[정의]<br>연관된 엔티티를 실제로 접근할 때 조회.<br>
`@ManyToOne(fetch = FetchType.EAGER)`

[장점]<br>필요할 때만 가져와서 성능이 좋다.

[단점]<br>객체 접근 전에 **프록시**를 주의해야 한다.<br>
`프록시란? 진짜 객체처럼 보이지만 실제로는 지연로딩을 위한 가짜 객체.`

### 즉시로딩❓

[정의]<br>연관된 엔티티를 즉시 같이 조회.
`@ManyToOne(fetch = FetchType.LAZY)`

[장점]<br>한 번에 다 가져온다.

[단점]<br>
- 불필요한 쿼리가 발생한다.
- **N+1 문제** 유발의 원인이 되기 쉽다❗️

### 그래서 권장되는 방식은❓

**모든 연관관계를 기본적으로 LAZY로 두고 필요할 때만 fetch join 등으로 명시적으로 조회하는 방식이 가장 안전하다.**

<br>

## 2. Fetch Join

[정의]<br>연관된 엔티티를 SQL 조인으로 한 번에 조회하도록 만드는 JPA의 기능.<br>
기본적으로 지연로딩은 연관 엔티티를 나중에 별도로 쿼리해서 가져오는데, fetch join을 쓰면 JPA가 미리 한 번의 쿼리로 연관된 모든 데이터를 가져온다.✨

[목적]<br>**N+1 문제**를 해결하기 위해서 사용한다.

[예시]<br>
```java
List<Member> members = em.createQuery(
    "SELECT m FROM Member m JOIN FETCH m.team", Member.class)
    .getResultList();
```
- 이 쿼리는 member와 team을 한 번에 join해서 모두 가져온다.
```sql
SELECT m.*, t.*
FROM member m
JOIN team t ON m.team_id = t.id
```
- 쿼리 딱 1번
- team도 미리 다 가져오기 때문에 N+1 문제 ❌

[장점]<br>
- N+1 문제를 해결할 수 있다.
- 성능이 향상된다.

[주의할점]<br>
- 1:N 조인 시 중복 데이터가 생길 수 있다. -> DISTINCT 또는 DTO로 해결 가능.

<br>

## 3. @EntityGraph

[정의]<br>지연로딩으로 설정된 연관 엔티티들을 미리 함께 가져오도록 JPA에게 알려주는 어노테이션.

[예시]<br>
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();  // ← team도 함께 가져옴!
}
```
이럴 경우 findAll() 호출 시,
```sql
SELECT m.*, t.*
FROM member m
JOIN team t ON m.team_id = t.id
```
- team도 미리 함께 조회
- 추가 쿼리 없이 member.getTeam() 사용 가능

[장점]<br>
1. JPQL 없이 성능 최적화가 가능하다.
2. 코드가 깔끔하다.
3. N+1 문제를 방지할 수 있다.

[단점]<br>복잡한 관계에서는 DTO 쿼리보다 유연성이 낮다.

<br>

## 4. JPQL
**Java Persistence Query Language**

[정의]<br>JPA에서 사용하는 객체 지향 쿼리 언어.<br>
즉, 쉽게 말하자면 : SQL은 테이블을 대상으로 쿼리하고, **JPQL은 엔티티 객체를 대상으로 쿼리**한다.

[예시]<br>
```sql
SELECT * FROM member WHERE name = '져니';
```
- 여기서 member : db 테이블 이름
```java
SELECT m FROM Member m WHERE m.name = '져니'
```
- 여기서 Member : 자바 클래스 이름 (엔티티)
- m.name : 멤버 변수 이름

[특징]<br>
1. 엔티티 대상으로 작성하고, 필드를 기준으로 한다.
2. 테이블, 컬럼 이름을 직접 쓰지 않는다. -> 객체 지향적.
3. db에 독립적이다.

[단점]<br>쿼리가 복잡하면 불편하다. -> querydsl이 보완.

<br>

## 5. QueryDSL

[정의]<br>자바 코드로 SQL처럼 쿼리를 작성할 수 있게 도와주는 도구.<br>
- 문법이 SQL과 비슷하다.
- 엔티티를 대상으로 한다.
- 자바코드로 작성한다.
- 자동완성이 가능하며 컴파일 시점에 오류를 잡아준다.

[예시]<br>
```java
QMember m = QMember.member;

List<Member> result = queryFactory
    .selectFrom(m)
    .where(m.age.gt(20))
    .fetch();
```
- QMember : 자동 생성된 클래스
- 자동완성 가능 + 리팩토링에 강함

### Q클래스란❓

Member 엔티티가 있으면 -> QMember라는 클래스가 자동으로 생성된다.

[장점]<br>
1. 동적 쿼리를 만들기에 매우 편리하다.
2. 서브쿼리, group by, join 등 복잡한 쿼리도 깔끔하게 작성 가능하다.
3. 타입 안전성으로 리팩토링과 협업에 유용하다.

[단점]<br>
1. Q타입 생성을 위한 build.gradle 세팅이 필요한데, 이 설정이 복잡하다.
2. 코드가 늘어날 수 있다.