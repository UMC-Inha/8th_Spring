## 🔥미션

---

1주차 때 제공된 IA, WF을 참고하여 (아래 미션 참고 자료를 보셔도 됩니다!) <br>
홈 화면, 마이 페이지 리뷰 작성, 미션 목록 조회(진행중, 진행 완료), 미션 성공 누르기,
회원 가입 하기(소셜 로그인 고려 X) <br>
위의 기능을 구현하는데 필요한 API들을 설계하여 <br>
API Endpoint, Request Body, Request Header, query String, Path variable 이 포함된 명세서 작성

## 1. 회원가입 

### 기본 정보
| 항목        | 내용                  |
|------------|---------------------|
| **URL**    | `/users/signup`     |
| **Method** | `POST`              |
| **인증**   | not login (비로그인 상태) |

---

### Headers
Content-Type: application/json


### Request Body (JSON)
```json
{
  "email": "seungin@example.com",
  "password": "ysi1234567",
  "user_name": "유승인",
  "nickname": "레온",
  "user_age": 28,
  "user_gender": "M",
  "user_phone": "010-1234-5678",
  "user_birth": "1998-07-24",
  "user_address": "서울시 노원구 공릉2동 00아파트",
  "food_categories": "한식"
}
```
### Response Body (JSON) <br>
상태 코드 <br>
201 Created : 회원가입 성공 <br>
```json
{
  "user_id": 1,
  "email": "seungin@example.com",
  "nickname": "하이루",
  "user_birth": "1998-07-24",
  "user_address": "서울시 노원구 공릉2동 00아파트",
  "food_categories": "한식",
  "created_at": "2025-04-02T12:00:00Z"
}
```

## 2. 홈 화면 조회

### 기본 정보
| 항목        | 내용               |
|------------|------------------|
| **URL**    | `/home/{user_id}` |
| **Method** | `GET`            |
| **인증**   | login (로그인 상태)   |

---

### Headers
Authorization: Bearer {jwt_token} (로그인 한 토큰으로 인증)


### Request Body (JSON) <br>
X <br>

### Response Body (JSON) <br>
상태 코드 <br>
200 : 홈 화면 조회 성공 <br>
```json
{
  "user_id": 1,
  "region": "안암동",
  "success_count": 7,
  "mission": [
    {
      "mission_id": 0,
      "mission_restaurant": "반어학생마라탕",
      "content": "10,000원 이상의 식사",
      "completed_points": "500 P",
      "mission_button": "active",
      "deadline": 7
    },
    {
      "mission_id": 1,
      "mission_restaurant": "반어학생마라탕2",
      "content": "20,000원 이상의 식사",
      "completed_points": "1000 P",
      "mission_button": "active",
      "deadline": 9
    },
    {
      "mission_id": 3,
      "mission_restaurant": "반어학생마라탕3",
      "content": "30,000원 이상의 식사",
      "completed_points": "1500 P",
      "mission_button": "active",
      "deadline": 11
    }
  ]
}
```

## 3. 마이 페이지 리뷰 작성 (내가 작성한 리뷰)

### 기본 정보
| 항목        | 내용               |
|------------|------------------|
| **URL**    | `/mypage/reviews` |
| **Method** | `POST`           |
| **인증**   | login (로그인 상태)   |

---

### Headers
Authorization: Bearer {jwt_token} (로그인 한 토큰으로 인증)


### Request Body (JSON)
```json
{
  "store_name": "반이학생마라탕마라반",
  "user_nickname": "닉네임1234",
  "rating": 4.5,
  "content": "음 너무 맛있어요 포인트도 얻고~",
  "created_at": "2022-05-14",
  "review_image": "review.jpg"
}
```
<br>

### Response Body (JSON) <br>
상태 코드 <br>
201 : 마이페이지 리뷰 등록 성공 <br>

```json
{
  "review_id": 1,
  "message": "리뷰가 성공적으로 등록되었습니다."
}
```

## 4. 미션 목록 조회 (진행중)

### 기본 정보
| 항목        | 내용                  |
|------------|---------------------|
| **URL**    | `/missions/ongoing` |
| **Method** | `GET`               |
| **인증**   | login (로그인 상태)      |

---



### Headers
Authorization: Bearer {jwt_token} (로그인 한 토큰으로 인증)

### Request Body
X

### Response Body (JSON) <br>
상태 코드 <br>
200 : 진행중 미션 목록 조회 완료 <br>

```json
{
  "missions": [
    {
      "mission_id": 1,
      "title": "가게이름에서",
      "description": "12,000원 이상의 식사를 하세요!",
      "reward": "성공 시 5% 적립",
      "status": "진행중"
    },
    {
      "mission_id": 2,
      "title": "카페에서",
      "description": "아메리카노 1잔 주문하기",
      "reward": "성공 시 5% 적립",
      "status": "진행중"
    }
  ]
}

```

## 5. 미션 목록 조회 (진행 완료)

### 기본 정보
| 항목        | 내용                    |
|------------|-----------------------|
| **URL**    | `/missions/completed` |
| **Method** | `GET`                 |
| **인증**   | login (로그인 상태)        |

---



### Headers
Authorization: Bearer {jwt_token} (로그인 한 토큰으로 인증)

### Request Body
X

### Response Body (JSON) <br>
상태 코드 <br>
200 : 진행 완료 미션 목록 조회 완료 <br>

```json
{
  "missions": [
    {
      "mission_id": 3,
      "title": "편의점에서",
      "description": "3,000원 이상 구매하기",
      "reward": "성공 시 포인트 500 적립",
      "status": "진행완료",
      "completed_at": "2025-04-03T14:30:00Z"
    },
    {
      "mission_id": 4,
      "title": "패스트푸드점에서",
      "description": "버거 세트 주문",
      "reward": "성공 시 포인트 1000 지급",
      "status": "진행완료",
      "completed_at": "2025-04-01T18:00:00Z"
    }
  ]
}
```
## 6. 미션 성공 누르기 

### 기본 정보
| 항목        | 내용     |
|------------|--------|
| **URL**    | `/missions/{mission_id}/completed` |
| **Method** | `POST` |
| **인증**   | login (로그인 상태) |

---

### Headers
Authorization: Bearer {jwt_token} (로그인 한 토큰으로 인증)

### Request Body
X

### Response Body (JSON)
상태 코드 <br>
201 : 미션 성공 누르기 완료 <br>

```json
{
  "message": "미션을 성공적으로 완료하였습니다."
}
```