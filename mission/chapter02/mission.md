- **미션 기록**
 
    **1. 나의 미션**
    
    ```sql
    SELECT 
      m.point, 
      m.name, 
      m.description, 
      um.is_completed
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
    
    현재 user의 id = 1
    
    한 페이지 당 가져올 미션 개수 : 15개

    **2. 리뷰 페이지**

    2-1. 리뷰 작성
    
    ```sql
    INSERT INTO review (content, star, restaurant_id, user_id)
    VALUES ('음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다', 5, 3, 1);
    ```
    
    2-2. 리뷰 조회
    
    ```sql
    SELECT 
        r.id AS review_id,
        r.user_nickname,
        r.content,
        r.updated_at,
        JSON_ARRAYAGG(ri.image_url) AS image_urls
    FROM review r
    LEFT JOIN review_image ri ON r.id = ri.review_id
    WHERE r.updated_at < (
    		SELECT updated_at
    		FROM review AS r2
    		WHERE r2.id = 15
    	)
    GROUP BY r.id
    ORDER BY r.updated_at DESC
    LIMIT 15;
    ```
    
    mysql JSON 관련 함수 모음: https://adjh54.tistory.com/519
    
    → JSON 함수를 사용해서 리뷰 이미지도 한 번의 쿼리로 리스트 형태로 담을 수 있게 해봤습니다
    
    하나의 쿼리로 만들긴 했지만 최적화 부분에서 두 개의 쿼리로 나눠서(리뷰 내용, 리뷰 사진들) 보낼지 아니면 한 번에 보낼지는 고민해야할 부분 같습니다
    
    **3. 마이페이지**
    
    3-1. 미션 개수
    
    ```sql
    SELECT COUNT(*) AS count_mission
    FROM user_mission AS um
    INNER JOIN mission AS m ON um.mission_id = m.id
    INNER JOIN restaurant AS r ON m.restaurant_id = r.id
    INNER JOIN location AS l ON r.address = l.address
    WHERE um.user_id = 1 AND l.address = '안암동'
    ```
    
    3-2 my mission(특정 지역에 대한)
    
    ```sql
    SELECT 
      r.name, r.cusine_type, m.description, m.expired_at
    FROM user_mission AS um
    INNER JOIN mission AS m ON um.mission_id = m.id
    INNER JOIN restaurant AS r ON m.restaurant_id = r.id
    INNER JOIN location AS l ON r.address = l.address
    WHERE um.user_id = 1
      AND l.address = '안암동'
      AND um.updated_at < (
        SELECT um2.updated_at
        FROM user_mission AS um2
        INNER JOIN mission AS m2 ON um2.mission_id = m2.id
        INNER JOIN restaurant AS r2 ON m2.restaurant_id = r2.id
        INNER JOIN location AS l2 ON r2.address = l2.address
        WHERE um2.id = 15 AND um2.user_id = 1 AND l2.address = '안암동'
      )
    ORDER BY um.updated_at DESC
    LIMIT 4;
    ```
    
    **4. 마이페이지**
    
    ```sql
    SELECT 
        u.nickname,
        u.email,
        u.is_phone_verified,
        u.point
    FROM user AS u;
    ```
