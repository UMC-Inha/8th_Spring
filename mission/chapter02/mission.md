
###  1. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리 (페이징 포함)

```sql
SELECT 
    m.mission_content,
    m.reward_point,
    s.store_name,
    um.status AS mission_status
FROM user_mission um
JOIN user u ON um.user_id = u.id 
JOIN mission m ON um.mission_id = m.mission_id  
JOIN store s ON m.store_id = s.id 
WHERE um.status IN ('in_progress', 'completed')
ORDER BY um.updated_at DESC
LIMIT 15 OFFSET 0;
```

### 2. 리뷰 작성하는 쿼리 (사진 배제)
```sql
INSERT INTO
	review(store_id, user_id, score, review_content)
VALUES
	(1,1,3.5,'맛있어요');
```

### 3. 홈화면 쿼리
### 3-1) 완료된 미션 수 
```sql
SELECT
u.completed_mission
FROM user AS u;
```
### 3-2) 현재 선택된 지역(*안암동)에서 도전 가능한 미션 목록, 페이징 추가
```sql
SELECT 
    m.mission_title,
    m.mission_content,
    m.reward_point,
    m.end_date,
    s.store_name,
    fc.category_name,
    DATEDIFF(m.end_date, CURDATE()) AS remain_day,
FROM 
    mission m
JOIN store s ON m.store_id = s.id 
JOIN region r ON s.region_id = r.region_id 
LEFT JOIN food_category fc ON s.category_id = fc.id  
LEFT JOIN user_mission um ON um.mission_id = m.mission_id AND um.user_id = [사용자ID]
WHERE 
    r.district = '안암동'
    AND um.mission_id IS NULL 
ORDER BY 
    remain_day ASC
```

### 4.마이페이지 화면 쿼리
```sql
SELECT
	u.name,
	u.email,
	u.is_phone_varified,
	u.point
FROM user AS u;
```