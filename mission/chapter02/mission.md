1주차 설계 DB
![week1_db-5](https://github.com/user-attachments/assets/8e6c2fa9-3f0c-4e60-8c38-c17b71844c42)

### 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)
```sql
SELECT 
    m.mission_id,
    m.content AS message,
    s.store_name AS store_name,
    mi.mission_status AS is_completed,
    m.point
FROM 
    usermission AS mi
    JOIN mission AS m ON mi.mission_id = m.mission_id
    JOIN store AS s ON s.store_id = m.store_id
WHERE 
    mi.user_id = ?
ORDER BY 
    mi.updated_at DESC
LIMIT 5
OFFSET ?; -- (n-1)*5
```

### 리뷰 작성하는 쿼리
```sql
INSERT INTO 
    review(store_id, user_id, rate, review_content, datetime) 
VALUES 
    ('상점123','닉네임1234', 5.0, '음 너무 맛있어요');
```

### 홈 화면 쿼리 - 성공한 미션 수
```sql
SELECT
    user_id,
    COUNT(*) AS completed_mission
FROM
    usermission
WHERE
    mission_status = 'COMPLETED'
GROUP BY
    user_id;
```

### 홈 화면 쿼리 - 수행 가능한 미션
```sql
SELECT 
    m.mission_id,
    m.content AS message,
    m.point,
    s.store_name,
    fc.food_category AS food_name,
    CASE 
        WHEN um.mission_status = 'COMPLETED' THEN 1 
        ELSE 0 
    END AS is_completed
FROM 
    mission AS m
    JOIN store AS s ON s.store_id = m.store_id
    JOIN food_category AS fc ON fc.food_category_id = s.food_category_id
    LEFT JOIN usermission AS um ON um.mission_id = m.mission_id 
        AND um.user_id = ?
WHERE 
    s.store_address LIKE CONCAT('%', ?, '%')
ORDER BY 
    is_completed ASC,
    m.created_at DESC
LIMIT 5
OFFSET ?;
```

### 마이 페이지 화면 쿼리
```sql
SELECT
    (
        SUM(m.point)
        FROM usermission AS um
        JOIN mission AS m ON um.mission_id = m.mission_id
        WHERE um.user_id = u.user_id
        AND um.mission_status = 'COMPLETED'
    ) AS total_points,
    u.phone_number,
    u.email,
    u.user_name
FROM
    user AS u
WHERE 
    u.user_id = ?;
```