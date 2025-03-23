### 내가 진행 중, 진행 완료한 미션(페이징 포함)

---
```sql
SELECT 
    m.message,
    p.point,
    mi.is_completed,
    s.name
FROM 
    member_mission AS mi
        JOIN mission AS m ON m.mission_id = mi.mission_id
        JOIN store AS s ON s.store_id = m.store_id
        JOIN point_mission AS pm ON pm.mission_id = m.mission_id
        JOIN point AS p ON p.point_id = pm.point_id
WHERE 
    mi.member_id = [특정_회원_ID]
ORDER BY 
    mi.updated_at DESC
LIMIT 7
OFFSET ?; --(N-1)*7
```

<br>
<br>

### 리뷰 작성, (사진의 경우 배제)

---
```sql
INSERT INTO 
    review(store_id, member_id, message, star_rating)
VALUES
    ([회원_ID], [상점_ID], [리뷰 작성 내용], [별점]);
```

<br>
<br>

### 홈 화면 (현제 선택 된 지역에서 도전 가능한 미션 목록, 페이징 포함)

---
- 미션 가져오는 쿼리

```sql
SELECT
    m.message,
    p.point,
    mm.is_completed,
    s.name AS store_name,
    f.name AS food_name
FROM
    FROM
    member_mission AS mm
        JOIN mission AS m ON m.mission_id = mm.mission_id
        JOIN store AS s ON s.store_id = m.store_id
        JOIN point_mission AS pm ON pm.mission_id = m.mission_id
        JOIN point AS p ON p.point_id = pm.point_id
        LEFT JOIN store_food AS sf ON s.store_id = sf.store_id
        LEFT JOIN food AS f ON f.food_id = sf.food_id
WHERE 
    mi.distance_from_member <= [거리_값]
ORDER BY 
    m.started_at DESC
LIMIT 7
OFFSET ?; --(N-1)*7
```
- 상점 중 `food` 등록 안 한 상점 존재할 수 있으니 `LEFT JOIN`
- `point`, `point_mission`는 모든 미션에 포인트 부여 필수라고 생각해서 `INNER JOIN`

<br>

- 회원 완료한 미션 수 가져오기

```sql
SELECT 
    member_id,
    COUNT(*) AS completed_mission
FROM
    member_mission
WHERE
    is_completed = TRUE
GROUP BY
    member_id;
```

<br>
<br>

### 마이 페이지 화면

---
```sql
SELECT
    SUM(p.point) AS total_earned_points,
    m.email,
    m.phone_number
    m.name
FROM
    member AS m
        JOIN point_member AS pm ON pm.member_id = m.member_id
        JOIN point AS p ON pm.point_id = p.point_id
WHERE p.point_type = 'EARNED' --포인트 상태(얻은) 필터링 조건
  AND m.member_id = [특정_회원_ID]
```
- 휴대폰 번호, 서비스 로직에서 `NULL`값 처리

