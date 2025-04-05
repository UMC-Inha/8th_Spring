## 3주차 미션
1. 홈화면
| API Endpoint | `GET /home` |
| --- | --- |
| Request Body | X |
| Request Header | Authorization: accessToken (String) |
| Query String | `user_id` |
| Path variable | X |
 - 홈화면 조회이므로 GET
- 헤더에 첨부된 토큰을 통해 사용자를 구분하므로 api에서 {user_id} 생략

2. 마이페이지 리뷰 작성
| API Endpoint | `POST /store/{store_id}/review` |
| --- | --- |
| Request Body | {
    "food_category_id": "bigint",
    "rate": "float",
    "review_content": "string",
    "review_image_url": "string"
    } |
| Request Header | Authorization: accessToken (String) |
| Query String | X |
| Path variable | `store_id` |
- 리뷰 등록이므로 POST
- 특정 가게에 리뷰를  작성해야하므로 path variable로 store_id 가져옴

3. 미션 목록 조회
| API Endpoint | `GET /user-missions` |
| --- | --- |
| Request Body | X |
| Request Header | Authorization: accessToken (String) |
| Query String | `?status=(enum)` |
| Path variable | X |
- 목록 조회이므로 GET

4. 미션 성공 누르기
| API Endpoint | `PATCH /user-missions/{mission_id}` |
| --- | --- |
| Request Body | {
    "mission_status": "completed"
    } |
| Request Header | Authorization: accessToken (String) |
| Query String | X |
| Path variable | `mission_id` |
- 미션 상태 변경이므로 PATCH
 - 특정 미션의 정보를 지목해야하므로 path variable로 mission_id 가져옴

5. 화원가입하기
| API Endpoint | `POST /user/register` |
| --- | --- |
| Request Body | {
    "user_name": "string",
    "gender": "enum",
    "birth": "date",
    "address”: "string”
    } |
| Request Header | X |
| Query String | X |
| Path variable | X |
- 회원가입은 인증이 필요하지 않으므로 `Authorization` 헤더가 없음