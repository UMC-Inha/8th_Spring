# 일반 미션
1. N + 1 문제를 해결할 수 있는 여러 가지 다른 방법들에 대해 조사한 후, [ 핵심 키워드 ] 에 정리
- 정리 완료
  
2. 2주차 미션 때 했던 해당 화면들에 대해 작성햇던 쿼리를 QueryDSL 로 작성하여 리팩토링
   
1번
  
![image](https://github.com/user-attachments/assets/7c24f12e-114a-470c-89fa-6c2f97f0c235)

2번

![image](https://github.com/user-attachments/assets/d367aec0-b13b-4cfc-a0b1-9f4382031a05)

3번

![image](https://github.com/user-attachments/assets/215c2d95-aa21-4c7a-860d-8b9c81044085)

4번

![image](https://github.com/user-attachments/assets/220d755f-9207-42fa-b7be-df089fcb8acd)


# 시니어 미션
- 과거 프로젝트 설정 파일 관련 오류(S3, AWS 등등..) 해결 실패로 예시 코드를 통해 진행하였습니다.

1. spring.jpa.show-sql=true & logging.level.org.hibernate.SQL=DEBUG 활성화하여 실행된 SQL 로그 분석
```yaml
# application‑local.yml
spring:
  jpa:
    show-sql: true         
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    org.hibernate.SQL: debug        
    org.hibernate.type.descriptor.sql.BasicBinder: trace  
```

```html
select order0_.id, order0_.member_id, order0_.created_at
from orders order0_
where order0_.created_at between ? and ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select orderitems0_.order_id, orderitems0_.id, orderitems0_.item_id, ...
from order_item orderitems0_
where orderitems0_.order_id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
select item0_.id, item0_.name, item0_.price
from item item0_
where item0_.id = ?;
```

- SQL 총 1 + 20 (N x 2 | N = 10)
- 실행 약 700ms 이상

2. 성능 병목이 발생한 코드(쿼리)를 QueryDSL 기반으로 최적화
   기존 코드

```java
@Query("select o from Order o " +
       "where o.createdAt between :from and :to")
List<Order> findOrders(LocalDateTime from, LocalDateTime to);
```

QueryDSL 수정 코드

```java
QOrder       o  = QOrder.order;
QMember      m  = QMember.member;
QOrderItem   oi = QOrderItem.orderItem;
QItem        i  = QItem.item;

List<OrderDTO> list = queryFactory
    .select(Projections.constructor(OrderDTO.class,
            o.id,
            m.name,
            o.createdAt,
            o.totalPrice,
            oi.countDistinct()))         
    .from(o)
    .join(o.member, m)
    .leftJoin(o.orderItems, oi).fetchJoin()
    .leftJoin(oi.item, i).fetchJoin()
    .where(o.createdAt.between(from, to))
    .groupBy(o.id, m.name, o.createdAt, o.totalPrice)
    .fetch();
```

- Fetch Join 2번으로 주문-주문상품리스트-상품 한 번에 조회
- DTO 를 통해 필요한 필드만 선택
- SQL 1회, 실행 약 180ms

@Transactional(readOnly = true) 적용

```java
@Transactional(readOnly = true)
public List<OrderDTO> getOrders(LocalDateTime from, LocalDateTime to) {
    return orderQueryRepository.findOrderDtos(from, to);
}
```

- 약 20ms 단축

3. 배치 크기(Batch Fetch Size) 조절을 통한 성능 비교

```java
@Entity
@BatchSize(size = 100)
public class Order {
}

@Entity
@BatchSize(size = 100)
public class OrderItem {
}

@Entity
@BatchSize(size = 100)
public class Item {
}
```

- BatchSize 적용

```java
select order0_.id, ... from orders order0_ where ...
select orderitems0_.order_id, orderitems0_.id, ...
from order_item orderitems0_
where orderitems0_.order_id in (1,2,3,4,5,6,7,8,9,10);
select item0_.id, item0_.name, ...
from item item0_
where item0_.id in (?,?,?,?...);
```

- SQL 3회, 실행 약 250ms
- Fetch Join 보다 느리지만 페이징이 필요할 때 효과적임

4. 최적화 적용 후, 성능 향상 결과 정리

   | 상황 | SQL 횟수 | 평균 응답(ms) | 비고 |
   | --- | --- | --- | --- |
   | 초기(N + 1) | 21 | 700 | 지연 로딩 폭발 |
   | **QueryDSL + fetch join** | **1** | **180** | 가장 빠름, 페이징 X |
   | BatchSize(100) | 3 | 250 | 페이징 가능, 메모리 절약 |
   | + `@Transactional(readOnly=true)` | ‑ | 160 |  |

최적화 전략 정리
- 자주 조회하는 API
  - DTO + Fetch Join + Distint()를 통한 중복 제거
- 대용량, 페이징 화면
  - @BatchSize + page API
  - Fetch Join 은 1:N 컬렉션에 제한이 있기 때문
- 읽기 전용 API
  - @Transactional(readOnly = true) 설정