#### 1. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

<img src = "https://velog.velcdn.com/images/sm011212/post/1a0a2a60-cf9b-47b8-a5c2-1c11064b1d7a/image.png" width = "300">

#### - 진행 중일 때

``` sql
SELECT
    m.id,
    m.condition,
    m.reward,
    s.name AS store_name,
    um.status AS mission_status
FROM user_mission um
JOIN mission m ON um.mission_id = m.id
JOIN store s ON m.store_id = s.id
WHERE um.user_id = :userId
  AND um.status = '진행중'
ORDER BY m.created_at DESC
LIMIT :pageSize OFFSET :offset;
```

#### - 진행 완료
``` sql
SELECT
    m.id,
    m.condition,
    m.reward,
    s.name AS store_name,
    um.status AS mission_status
FROM user_mission um
JOIN mission m ON um.mission_id = m.id
JOIN store s ON m.store_id = s.id
WHERE um.user_id = :userId
  AND um.status = '진행 완료'
ORDER BY m.created_at DESC
LIMIT :pageSize OFFSET :offset;
```

-> 특정 유저의 미션을 조회해야 하므로 userId가 저장되어 있는 user_mission 테이블에 mission과 store를 join 하고 where 절에 userId로 조건을 적용하는 형식으로 구현했다.

-> status가 어떻게 저장될지 모르니 일단 '진행중', '진행 완료'로 두었다.

-> userId와 pageSize, offset은 미정이기에 위와 같이 작성했다.

#### 2. 리뷰 작성하는 쿼리 (사진은 제외)

<img src = "https://velog.velcdn.com/images/sm011212/post/c9489839-7ef0-40a0-8094-cf4127a1a544/image.png" width = "300">

``` sql
INSERT INTO review (store_id, user_id, rating, content)
VALUES (:storeId, :userId, :rating, :content);
```

-> 현재 리뷰를 작성할 식당, 리뷰를 작성한 유저의 아이디, 별점, 내용만 필요하다고 생각해(사진은 제외) 위와 같은 쿼리를 작성했다.

#### 3. 홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

<img src = "https://velog.velcdn.com/images/sm011212/post/6c5c0b02-0fff-4422-ac6e-e61c8f64f530/image.png" width = "300">

``` sql
SELECT 
	m.id,
	m.condition,
    m.reward,
    datediff(dd, now(), m.end_date) as deadline
	s.name AS store_name,
    s.category AS store_category
    um.id,
    um.status as mission_status
FROM mission m
JOIN store s ON m.store_id = s.id
LEFT JOIN user_mission um
  	ON m.id = um.mission_id AND um.user_id = :userId
WHERE m.region = :region
  	AND m.start_date <= NOW() AND m.end_date >= NOW()
    AND um.id is NULL
ORDER BY m.created_at DESC
LIMIT :pageSize OFFSET :offset;
```

-> 도전이 가능한 미션이므로, 현재 지역에 있는 mission들 중 로그인한 유저의 user_mission 테이블에 등록되지 않은 미션들만 조회한다고 생각했다.

-> 미션 정보는 아이디, 미션 내용, 보상을 조회하도록 작성했다.

-> 특정 유저와 관련된 미션을 가져와야 하므로 특정 유저의 미션 정보를 저장하고 있는 user_mission와 left join했고, 특정 유저의 아이디일 경우에만 조회가 되도록 했다.

-> datediff() 함수를 사용해 남은 날짜를 계산하서 반환하도록 했다.

-> 가게 이름과 가게 분류 정보가 필요하기 때문에 store 테이블과 join 연산을 수행했다.

-> 지역의 경우, 거리 제한이 있는게 아니라 일치로 설정이 되었기 때문에 지역이 일치하는지 where 절로 조건을 설정했다.

-> 페이징은 offset 방식을 적용했다.

#### 4. 마이 페이지 화면 쿼리

<img src = "https://velog.velcdn.com/images/sm011212/post/8cab311e-e55d-4b6b-a6c2-9758e495d991/image.png" width = "300">

``` sql
SELECT 
    u.nickname,
    u.email,
    u.phone_number,
    u.is_phone_verified,
	u.point
FROM user u
```

-> 유저 테이블에 많은 필드가 있을 것이기 때문에 마이 페이지 화면에서 필요한 정보만 조회하도록 설정했다.

#### 시니어 미션. 정렬 기준을 1순위는 포인트로 2순위는 최신순으로 하여 Cursor기반 페이지네이션으로 구현

<img src = "https://velog.velcdn.com/images/sm011212/post/1a0a2a60-cf9b-47b8-a5c2-1c11064b1d7a/image.png" width = "300">

``` sql
SELECT
    m.id,
    m.reward,
    m.condition,
    m.created_at,
    s.name AS store_name,
    um.status AS mission_status
FROM user_mission um
JOIN mission m ON um.mission_id = m.id
JOIN store s ON m.store_id = s.id
WHERE um.user_id = :userId
  AND (
        (m.reward < :lastReward)
        OR (m.reward = :lastReward AND m.created_at < :lastCreatedAt)
      )
ORDER BY m.reward DESC, m.created_at DESC
LIMIT :pageSize;
```

-> 정렬 기준 1순위로 reward를 desc, 2순위로 created_at을 desc 적용해주었다.

-> 커서 기반 페이지네이션 구현을 위해 1.마지막 reward보다 reward값이 작거나, 2. 마지막 reward와 reward 값이 같으면 생성일자를 사용해 비교하도록 구현했다.