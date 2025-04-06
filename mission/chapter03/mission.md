## Ch3 API 설계 미션
### 홈화면

**GET /api/v1/home**

### **Request Header**

| Key | Value | Type |
| --- | --- | --- |
| Authorization | Bearer {access_token} | string |

### 마이페이지 리뷰 작성

**POST /api/v1/reviews**

### **Request Header**

| Key | Value | Type |
| --- | --- | --- |
| Authorization | Bearer {access_token} | string |

### Request Body

```json
{
  "reviewContent": "냠 good 맛있어요",
  "score": 5.0,
  "imageUrl": ""
}
```

### 미션 목록 조회

**GET /api/v1/missions/list**

### **Request Header**

| Key | Value | Type |
| --- | --- | --- |
| Authorization | Bearer {access_token} | string |

### Query Parameters

| Key | Type | 설명 |
| --- | --- | --- |
| status | string | 미션 상태 필터링 값 (`IN_PROGRESS`, `COMPLETED`) |

### 미션 성공 누르기

**POST /api/v1/missions/{missionId}/completed**

### Request Header

| Key | Value | Type |
| --- | --- | --- |
| Authorization | Bearer {access_token} | string |

### Path Parameters

| Key | Type | 설명 |
| --- | --- | --- |
| missionId | integer | 완료 미션 id |

### 회원가입

**POST /api/v1/users/signup**

### Request Header

| Key | Value | Type |
| --- | --- | --- |
| Authorization | Bearer {access_token} | string |

### Request Body

```json
{
  "name": "홍길동",
  "gender": "남자",
  "birth": "2020-12-20",
  "city": "서울특별시",
  "district": "강남구",
  "neighborhood": "역삼동",
  "detail": "123번지",
  "postal_code": "23423"
  "preferFood": ["한식", "중식"]
}
```