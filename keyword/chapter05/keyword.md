# 🎯핵심 키워드

---

## 1. Domain

### **도메인**이란 ⇒ 우리가 **해결하고자 하는 문제 영역 그 자체를 말하기에**

즉, 현실 세계의 개념을 코드로 옮겨 놓은 모델이라고 이해할 수 있다.

이렇게 글로만 말하면 항상 이해가 어렵기에 예시를 들어보면 생각보다 간단하게 개념에 대해 이해할 수 있다.

**예시: 만약에 배달앱**을 만든다고 하면 이때 도메인은?

- 유저(Member)
- 음식(Food)
- 가게(Store)
- 리뷰(Review)
- 주문(Order)
- 쿠폰(Coupon) 등등,,,

이 모든 **현실 세계의 개념**을 코드로 옮긴 것을 **도메인 모델이라고 설명할 수 있다.**

그러면 이것들을 코드로 옮겨야하는데 이번주차때 했던 실습을 떠올려보면 Spring 프로젝트안에서 domain이라는 패키지를 생성하였고, 이 domain이라는 패키지안에 내가 작성한 ERD에 있는 Member, Review, Mission 등등,, 을 집어넣었으니 내가 집어넣었던 유저(Member), 리뷰(Review), 미션(Mission)들이 모두 도메인이라고 칭할 수 있다.

이러한 도메인의 구성요소로는

### 엔티티 (Entity)

- 식별 가능한 객체
- DB 테이블로 매핑됨

### 밸류 오브젝트 (Value Object)

- 식별 필요 없음 (값 자체가 의미)
- 예: 주소(Address), 돈(Money) 등

### 도메인 서비스 (Domain Service)

- 여러 엔티티에 걸친 복잡한 비즈니스 로직
- 예: "회원이 미션을 완료할 수 있는가?"

### 도메인 이벤트 (Event)

- 상태 변화에 따른 반응형 처리

등이 존재한다.

## 2. 양방향 매핑

### 양방향 매핑이란? ⇒  JPA에서 두 엔티티가 서로 참조하며 관계를 맺는 것

즉, **양쪽 모두 연관 객체를 알고 있는 상태**

간단하게 예시를 들어서 이해를 해보자면

```java
public class Review {
    @ManyToOne
    private Member member;
}
```

단방향: 현재 Review는 Member를 알지만 Member는 Review를 모름

<br><br>
```java
public class Member {
    @OneToMany(mappedBy = "member")
    private List<Review> reviews = new ArrayList<>();
}

public class Review {
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
}
```
양방향: 현재 Member도 Review 리스트를 가지고,  Review도 Member를 가짐

예시에서 본것을 한번 풀어보자면

> Review는 "회원이 쓴 리뷰"고,
>
>
> Member는 "내가 쓴 모든 리뷰 리스트"를 알고 있음.
>

이 둘이 서로를 알아야 더 자연스러운 흐름을 가져갈수 있음.

→ "회원의 리뷰들 보기", "이 리뷰 누가 썼는지 보기" 모두 지원해야 하니까!

### 그렇다면 양방향 매핑의 장단점은?

**장점**

| 항목     | 설명                                                                 |
|----------|----------------------------------------------------------------------|
| 직관성   | 객체 그래프 탐색이 자연스럽다 (`member.getReviews()`)              |
| 성능     | `fetch join`, `JPQL` 사용 시 쿼리 효율성이 높다                      |
| 유지보수 | 연관 엔티티 접근이 쉬워지고, 단위 테스트 작성이 용이하다             |

**단점**

| 항목         | 설명                                                                 |
|--------------|----------------------------------------------------------------------|
| 복잡성       | 연관관계 주인 개념 필요 (`mappedBy`)                                 |
| 무한루프     | `@ToString`, `@Json` 사용 시 순환 참조 주의 필요                     |
| 영속성 전이  | `cascade` 설정을 잘못하면 의도치 않은 삭제나 전파가 발생할 수 있음  |

양방향 매핑은 JPA에서 객체 지향의 개념과 쿼리 최적화에도 큰 영향을 주기때문에 핵심 개념임!

## 3. N+1 문제

### N+1 문제란?

⇒ 쿼리가 1번만 나갈 줄 알았는데, N+1번 나가는 문제를 통칭함

List<Member> members = memberRepository.findAll();
for (Member member : members) {
System.out.println(member.getReviews().size());
}

여기서 예상한 쿼리는 SELECT * FROM member 1번이지만  
이게 실제로는 각 멤버마다 `SELECT * FROM review WHERE member_id = ?` → N번 쿼리도 추가돼서 총 1+N 번의 쿼리가 발생한다.

이러한 문제가 생기는 이유:

⇒ **JPA의 기본 fetch 전략이 `@OneToMany` = LAZY** 이기 때문이라고 한다.

쉽게 말한다면 `member.getReviews()`를 호출하는 순간 DB에 `select` 쿼리가 **개별적으로** 날아가고,

그래서 루프 돌면 루프 횟수만큼 추가로 쿼리 발생하여 N+1번 발생

### 그렇다면 이러한 N+1 문제의 해결법은?

1. **Fetch Join** (JPQL 사용)
- 연관된 엔티티까지 **한 번에 조인**해서 가져옴
- SQL도 1번만 나감
- 단점: **페이징 불가능** (`@OneToMany` fetch join에서는)

1. EntityGraph
- 코드 분리도 가능
- 동적 fetch join처럼 동작

1. Batch Size 설정
- `@OneToMany` 시 내부적으로 `IN` 쿼리로 묶어서 한 번에 가져옴
- **중복 쿼리는 줄고, 성능은 올라감**

가장 이해가 편해서 비유를 가져와봤는데,

편의점에 가서 “라면 한 박스 주세요” 했더니, 점원이 “라면 1개요? 또요? 또요?” 하며 1개씩 N번 가져다주는 격이라는 비유로 N+1 문제의 개념을 가장 빠르게 이해했다.

이렇게 비효율적으로 가져오는게 아닌 **박스로 한 번에 가져와야 효율적인 방법인데,**

이것이 바로 **fetch join** 또는 **batch fetch 방식이라고 한다.**