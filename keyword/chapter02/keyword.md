## 🎯핵심 키워드

---

## Join 연산

### 1. INNER JOIN
- 두 테이블에서 **공통된 값이 있는 행**만 결합하여 결과로 반환함.
- ON 조건을 만족하는 행만 포함됨.

사용 용도:
- 두 테이블 간에 관계가 있는 데이터만 가져오고, 불필요한 데이터를 제외하고 싶을 때 사용한다.
- 예를 들어, 회원과 주문 테이블을 회원 ID를 기준으로 연결할 때 유용함.

 ```sql
SELECT A.name, B.order_id
FROM customers A
INNER JOIN orders B ON A.customer_id = B.customer_id;
```
⇒ customers 테이블과 orders 테이블에서 customers_id가 일치하는 행만 반환됨.

**장점**

불필요한 데이터 없이 **정확한 관계 데이터를 조회**할 수 있음.

데이터베이스에서 가장 최적화된 형태로 처리되기 때문에 성능이 우수함.

**단점**

join 조건을 만족하지 않는 행은 결과에서 제외되므로, **모든 데이터를 보고 싶을 때는 부적합**함.


### 2. LEFT JOIN (LEFT OUTER JOIN)
- **왼쪽 테이블의 모든 행을 유지**하면서, 오른쪽 테이블에서 일치하는 데이터를 가져옴.
- 오른쪽 테이블에 일치하는 값이 없으면 NULL값이 채워짐.

사용 용도:
- 한쪽 테이블에 반드시 존재하는 데이터를 유지하면서 다른 테이블의 데이터를 추가하고 싶을 때.
- 예를 들어, **모든 고객을 조회하면서, 주문이 없는 고객도 함께 표시하고 싶을 때**.
```sql
SELECT A.name, B.order_id
FROM customers A
LEFT JOIN orders B ON A.customer_id = B.customer_id;
```
=> 아까와 동일한 쿼리에서 INNER JOIN을 LEFT JOIN으로 변경한다면 customers 테이블의 모든 고객이 출력되며, 주문이 없는 고객은 order_id가 NULL로 표시됨.

**장점**

**왼쪽 테이블의 데이터를 보존**하면서, 관련된 데이터를 추가할 수 있음.

두 테이블 간의 관계를 유지하면서 데이터가 없는 경우에도 포함 가능.

**단점**

오른쪽 테이블에서 **일치하는 데이터가 없으면 NULL 값이 포함되므로**, 후처리가 필요할 수 있음.

INNER JOIN보다 성능이 떨어질 수 있음 (특히 대량 데이터에서).

### 3. RIGNT JOIN (RIGHT OUTER JOIN)
- **오른쪽 테이블의 모든 행을 유지**하면서, 왼쪽 테이블에서 일치하는 데이터를 가져옴.
- 왼쪽 테이블에 일치하는 값이 없으면 NULL 값이 채워짐.

**사용 용도**:

- LEFT JOIN과 반대 상황에서 사
- 예를 들어, 모든 주문을 포함하되, 주문한 고객이 없는 경우도 표시하고 싶을 때.
```sql
SELECT A.name, B.order_id
FROM customers A
RIGHT JOIN orders B ON A.customer_id = B.customer_id;
```
=> 여기서는 RIGHT JOIN을 사용했기 때문에 orders 테이블의 모든 주문이 출력되며, 주문한 고객 정보가 없으면 name이 NULL로 표시된다.

**장점**

RIGHT JOIN이 필요한 경우(오른쪽 테이블이 반드시 포함되어야 하는 경우)에 유용함.

**단점**

LEFT JOIN보다 사용 빈도가 낮음. (거의 사용하지 않음)

INNER JOIN보다 성능이 낮아질 수 있다.

### 4. FULL JOIN (FULL OUTER JOIN)
- **두 테이블의 모든 데이터를 포함**하며, 일치하는 데이터는 결합하고, 없는 데이터는 NULL로 채움.
- LEFT JOIN과 RIGHT JOIN을 합친 형태.

**사용 용도:**

- 두 테이블 간의 모든 데이터를 유지하면서, 공통된 데이터는 연결하고, 없는 데이터는 NULL 처리하고 싶을 때.
- 예를 들어, 고객과 주문 정보를 가져오면서, 주문이 없는 고객과 고객이 없는 주문을 모두 포함하고 싶을 때.
```sql
SELECT A.name, B.order_id
FROM customers A
FULL OUTER JOIN orders B ON A.customer_id = B.customer_id;
```
FULL OUTER JOIN 사용 ⇒
- 주문한 고객은 정상적으로 표시됨.
- 주문하지 않은 고객도 출력됨 (order_id가 NULL).
- 고객이 없는 주문도 출력됨 (name이 NULL).

**장점**

**모든 데이터를 유지하면서 조인할 수 있음**.

INNER JOIN, LEFT JOIN, RIGHT JOIN을 따로 사용할 필요 없이 한 번에 처리 가능.

**단점**

대부분의 DBMS(MySQL 등)에서는 기본 지원하지 않음 (대신 LEFT JOIN과 RIGHT JOIN을 UNION해서 사용하는 방식으로 FULL OUTER JOIN을 대체함).

데이터 양이 많을 경우 **성능이 저하될 가능성이 있음**.

## 페이징
페이징(Pagination)은 **데이터를 일정한 크기로 나누어 한 번에 일부만 조회하는 기술이다.**

데이터의 크기가 커질수록 이러한 데이터를 한 번에 가져오면 **속도가 느려지고, 메모리 사용량이 증가**하기 때문에 **일정한 크기로 잘라서 가져오는 것을 페이징이라고 한다.**

ex) 블로그 게시글 목록 → **한 페이지당 10개씩** 보여줌

이러한 페이징의 장점으로는

1. **불필요한 데이터 조회를 줄여 성능 향상**
2. **서버와 DB의 부하를 줄일 수 있음**
3. **빠른 응답 시간 제공 가능 등이 존재함**

## 페이징 구현 방식
### **(1) OFFSET 기반 페이징 (LIMIT & OFFSET)**

가장 기본적인 방법으로, `LIMIT`과 `OFFSET`을 사용하여 **특정 범위의 데이터만 가져오는 방식**
```sql
SELECT * FROM users
ORDER BY id ASC
LIMIT 10 OFFSET 20;  // 이 부분이 OFFSET 기반 페이징으로 앞의 20개 데이터는 건너뛰고 10개 데이터만 가져온다는 뜻
```
### **장점**

**간단하고 구현이 쉬움**

**MySQL, PostgreSQL 등에서 기본적으로 제공**

### **단점**

OFFSET이 커질수록 속도가 느려짐

LIMIT이전의 데이터를 모두 불러온 후 **버려야 하기 때문에 비효율적**

### **(2) Keyset 기반 페이징 (WHERE 조건 활용)**

OFFSET 기반 페이징의 성능 문제를 해결하기 위해 **마지막 데이터의 키 값을 기반으로 가져오는 방식**
```sql
SELECT * FROM users
WHERE id > 20  // 마지막으로 조회한 id 값
ORDER BY id ASC
LIMIT 10;
```
id > 20: 이전 페이지에서 마지막으로 조회한 데이터 이후의 값만 가져옴 <br>
LIMIT 10 → 10개만 가져옴

### **장점**

OFFSET을 사용하지 않으므로 **대량 데이터에서도 빠름**

**무한 스크롤 방식에 적합**

### **단점**

특정 열을 기준으로 정렬해야 함

**이전 페이지로 돌아가기 어려움** (정확한 페이지 번호를 알기 어려움)

### **(3) Cursor 기반 페이징**

Keyset 기반 페이징과 유사하지만, **정확한 정렬 순서와 커서를 유지하여 데이터를 조회하는 방식**
```sql
SELECT * FROM posts
WHERE created_at < '2000-01-01 12:00:00'
ORDER BY created_at DESC
LIMIT 10;
```
created_at < ‘마지막 데이터의 시간’→ 이전 페이지 마지막의 시간보다 더 오래된 글을 가져옴 <br>
LIMIT 10→ 10개만 가져옴

### **장점**

대량 데이터에서 **고속 처리 가능**

### **단점**

create_at 과 같이 정렬 기준이 명확해야 함

특정 페이지로 이동하기 어려움