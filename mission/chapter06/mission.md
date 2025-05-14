- 1번
    
    기존
    
    ```sql
    SELECT 
      m.point, 
      m.name, 
      m.description, 
      um.is_completed,
      um.updated_at
    FROM user_mission AS um
    INNER JOIN mission AS m ON um.mission_id = m.id
    WHERE um.user_id = 1
      AND um.updated_at < (
        SELECT updated_at
        FROM user_mission AS um2
        WHERE um2.id = 30 AND user_id = 1
      )
    ORDER BY um.updated_at DESC
    LIMIT 15;
    ```
    
    수정
    
    ```java
    List<Tuple> result = queryFactory
        .select(
            m.point,
            m.name,
            m.description,
            um.isCompleted,
            um.updatedAt
        )
        .from(um)
        .join(um.mission, m)
        .where(
            um.user.id.eq(1L),
            um.updatedAt.lt(
                JPAExpressions
                    .select(um2.updatedAt)
                    .from(um2)
                    .where(um2.id.eq(30L), um2.user.id.eq(1L))
            )
        )
        .orderBy(um.updatedAt.desc())
        .limit(15)
        .fetch();
    ```
    
- 2번
    
    기존
    
    ```sql
    INSERT INTO review (content, star, restaurant_id, user_id)
    VALUES ('음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다', 5, 3, 1);
    ```
    
    수정
    
    ```sql
    
    ```
    
- 3번
    
    **3-1.**
    
    기존
    
    ```sql
    SELECT COUNT(*) AS count_mission
    FROM user_mission AS um
    INNER JOIN mission AS m ON um.mission_id = m.id
    INNER JOIN restaurant AS r ON m.restaurant_id = r.id
    INNER JOIN location AS l ON r.address = l.address
    WHERE um.user_id = 1 AND l.address = '안암동'
    ```
    
    수정
    
    ```java
    Long count = queryFactory
        .select(um.count())
        .from(um)
        .join(um.mission, m)
        .join(m.restaurant, r)
        .join(r.location, l)
        .where(
            um.user.id.eq(1L),
            l.address.eq("안암동")
        )
        .fetchOne();
    		
    				
    ```
    
    **3-2.**
    
    기존
    
    ```sql
    SELECT 
        um.mission_complete,
        m.description AS mission_description,
        m.expired_at,
        r.name AS restaurant_name,
        r.cusine_type,
        l.address AS location_address
    FROM user_mission um
    JOIN mission m ON m.id = um.mission_id
    JOIN restaurant r ON r.id = m.restaurant_id
    JOIN location l ON r.address = l.address
    WHERE um.user_id = ?
      AND l.address = ?
      AND um.id > ?
    ORDER BY um.id ASC
    LIMIT ?
    ```
    
    수정
    
    ```java
    List<Tuple> results = queryFactory
        .select(
            um.missionComplete,
            m.description,
            m.expiredAt,
            r.name,
            r.cusineType,
            l.address
        )
        .from(um)
        .join(um.mission, m)
        .join(m.restaurant, r)
        .join(r.location, l)
        .where(
            um.user.id.eq(userId),
            l.address.eq(address),
            um.id.gt(cursorId)
        )
        .orderBy(um.id.asc())
        .limit(limit)
        .fetch();
    		
    ```
    
- 4번
    
    기존
    
    ```sql
    SELECT 
        u.nickname,
        u.email,
        u.is_phone_verified,
        u.point
    FROM user AS u;
    ```
    
    수정

  ```java
    Tuple result = queryFactory
    .select(u.nickname, u.email)
    .from(u)
    .where(u.id.eq(1L)) 
    .fetchOne();
    		
    ```


  # 시니어 미션

**미션 목표:**

- 기존 본인이 진행했던 프로젝트에서 JPA와 QueryDSL을 활용하면서 발생할 수 있는 **성능 병목을 파악하고 해결책을 제시**하세요.

**미션 상세 내용:**

1️⃣ `spring.jpa.show-sql=true` & `logging.level.org.hibernate.SQL=DEBUG` 활성화하여 실행된 SQL 로그 분석

- 특정 API 호출 시 실행된 **SQL 쿼리 로그를 분석**하고 실행 속도가 느린 쿼리를 찾으세요.
- 불필요한 쿼리가 실행되거나 **JOIN이 과도하게 발생하는 경우** 원인을 분석하세요.

```sql
2025-05-11T23:18:49.118+09:00  INFO 36125 --- [brainboost] [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed initialization in 2 ms
Authorization이 비어있습니다

Hibernate: 
    select
        ge1_0.id,
        ge1_0.created_at,
        ge1_0.description,
        ge1_0.game_type_id,
        ge1_0.img_url,
        ge1_0.name,
        ge1_0.status,
        ge1_0.updated_at,
        ge1_0.version 
    from
        game ge1_0 
    where
        ge1_0.id=?

Hibernate: 
    select
        gre1_0.game_record_id,
        gre1_0.created_at,
        gre1_0.game_id,
        gre1_0.record_key,
        gre1_0.record_value,
        gre1_0.status,
        gre1_0.updated_at,
        gre1_0.user_id,
        gre1_0.user_record_id 
    from
        game_record gre1_0 
    where
        gre1_0.game_record_id=?

```

`spring.jpa.show-sql=true` & `logging.level.org.hibernate.SQL=DEBUG`  활성화 모습

- 과거 프로젝트에서는 JSON 처리 및 이미지 처리 등의 기능을 구현하기 위해 복잡한 쿼리는 **JDBC Template 로 처리**했다.
    
    반면 JPA는 간단한 데이터 접근 로직에만 사용되었기 때문에 실행된 JPA 쿼리 중에서는 **불필요한 쿼리 호출이나 JOIN 남발은 발견되지 않았다.**
    

2️⃣ 성능 병목이 발생한 코드(쿼리)를 **QueryDSL 기반으로 최적화**

- `@Query`로 작성된 복잡한 SQL을 QueryDSL로 리팩토링하여 실행 속도를 비교하세요.

### 대상 API

- API 설명: username이 이미 존재하는지 여부를 판단하는 API
- 이유: 테스트를 위해 JMeter로 약 **1만 건 이상의 유저 데이터**를 삽입해두었기 때문에,
    
    쿼리 실행 시간의 변화를 측정하기에 적절한 케이스라고 판단
    

| 방식 | 평균 실행 시간 | 비고 |
| --- | --- | --- |
| JPQL | 약 165ms | `SELECT COUNT(*) FROM User u WHERE u.username = ?` |
| QueryDSL 리팩토링 | 약 162ms | `queryFactory.selectFrom(user).where(user.username.eq(username))` |

→ 실행 시간 차이가 거의 없었다.

### 실행 시간의 차이가 거의 없는 이유

단순 조건 기반의 쿼리일 경우 JPA의 처리 성능이 비슷하며 데이터가 실제 환경에 비해 적은 데이터라고 판단된다**.**



### 언제 성능 차이가 발생할까?

- 조건이 많고, 조건 조합이 동적으로 바뀌는 경우
- 다수의 서브쿼리를 조립해야 할 경우
- 연관 엔티티와의 조인이 동적으로 달라지는 경우

### `@Transactional(readOnly = true)` 적용 여부에 따른 실행 속도 차이도 테스트해보세요.
    - 동일한 username 존재 여부를 판단하는 API에 `@Transactional(readOnly = true)` 추가 후 실행 시간 측정
    - 결과: 약 3~5ms 이내의 미세한 변화만 있었고, 큰 차이는 없었음
    
    → 단순 조회 API에서의 성능 변화는 크지 않다.
    

즉 위의 설정들은 **복잡한 조건 조합 및 동적 쿼리**에서 더 두드러 질 수 있다.

3️⃣ `배치 크기(Batch Fetch Size)` 조절을 통한 성능 비교

- `@BatchSize(size = 100)` 설정 후, **쿼리 실행 횟수 변화 확인을 로그로 남겨보세요.**

```java
@OneToMany(mappedBy = "game", fetch = FetchType.LAZY)
@BatchSize(size = 3)  
private List<GameRecord> gameRecords;

```

→ 유의미한 결과를 판단하기 위해 size를 3으로 줄였으며 (game 관련 데이터가 많이 없었으므로)

```sql
Hibernate: select * from game_record where game_record_id in (?,?,?)
Hibernate: select * from game_record where game_record_id in (?,?,?)
Hibernate: select * from game_record where game_record_id in (?,?,?)

```

→ 위 처럼 sql 문이 3개로 줄어들었다.

- `fetch join`을 사용한 경우와 비교하여 성능 차이를 분석해보세요.

```java
@Query("SELECT g FROM Game g LEFT JOIN FETCH g.gameRecords")
List<Game> findAllRecords();
```

→ 해당 방식으로 fetch join을 사용하면 쿼리문이 한 개만 나가게 된다.

4️⃣ 최적화 적용 후, 성능 향상 결과 정리

- 실행 속도가 개선된 SQL 로그를 비교하고, 어떤 부분이 최적화되었는지 설명하세요.
- **실제 애플리케이션에서 적용할 최적화 전략을 정리**하세요.

이번 실습에서는 JPA 관련 최적화 전략(batch Size, queryDSL 최적화, readOnly = true, fetch join 등)을 적용해보고 SQL 로그 및 실행 속도의 변화를 비교해보았다.

하지만 실제 프로젝트에서는 대부분의 쿼리를 JdbcTemplate **기반의 네이티브 SQL로 처리**했기 때문에,

JPA 최적화 전략을 적극적으로 적용할 기회가 많지 않았던 점은 아쉬움으로 남는다.

그럼에도 불구하고 이번 실습을 통해 다음과 같은 중요한 부분을 공부했다:

- **단순 조회나 소량 데이터 조회의 경우**, @Query 와 QueryDSL 또는 readOnly 적용 등은 **성능 차이가 크지 않다.**
- 반면, **복잡한 조건, 동적 쿼리, 다중 연관 엔티티 조회, 대용량 처리 상황**에서는 JPA 최적화가 성능에 큰 영향을 미칠 수 있다는 것을 알게 되었다.
- 쿼리 최적화 외에도 **데이터 구조 설계(정규화 vs 비정규화)**, **JSON 컬럼 활용**, **필요한 인덱스 구성** 등 다양한 관점에서 성능을 고려해야 함도 함께 공부하게 되었다.

이번 여름에 들어갈 프로젝트에서는 이번에 배운 내용을 사용해서 성능 최적화를 해보고 싶다.
