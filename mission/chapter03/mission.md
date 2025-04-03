# API 명세서

### 페이지 종속적 API 설계

- UI가 변경될 때 마다 API 구조가 함께 변경 → 확장성 저하
- 페이지 변경 시 다른 페이지에 미치는 영향을 파악하기 어려움
- 테스트 복잡성 증가
- 서비스 기능 단위로 분리하기 어려워짐

### ⇒ 도메인 중심 설계

---

### 노션으로 제작 후 마크다운 문법으로 변환했습니다.

## 1. 회원

| URI                          | HTTP 메서드 | 응답 코드 | 설명                           | 쿼리 파라미터              |
|------------------------------|-------------|-----------|--------------------------------|----------------------------|
| `/members/join`              | POST        | 201       | 회원가입                       | -                          |
| `/members/login`             | POST        | 200       | 로그인                         | -                          |
| `/members/me`                | GET         | 200       | 회원 정보 조회                 | -                          |
| `/members`                   | PATCH       | 200       | 회원 탈퇴 (소프트 삭제, 백엔드에서 일정 기간 보관) | -                |
| `/members/{memberId}/points` | POST        | 201       | 회원 포인트 지급               | `type={포인트 유형 정보}`  |
| `/members/{memberId}/reviews`| GET         | 200       | 작성 리뷰 조회                 | `page=&size=&sort=`                                      |             |           |                         |                     |

---

## 2. 리뷰

| URI                                 | HTTP 메서드 | 응답 코드 | 설명                           | 쿼리 파라미터              |
|-------------------------------------|-------------|-----------|--------------------------------|----------------------------|
| `/reviews`                          | POST        | 201       | 새 리뷰 작성                   | -                          |
| `/reviews/{reviewId}`               | PATCH       | 200       | 리뷰 수정                      | -                          |
| `/reviews/{reviewId}`               | DELETE      | 204       | 리뷰 삭제                      | -                          |
| `/reviews/{reviewId}/replies`       | GET         | 200       | 리뷰 답변 조회                 | `page=&size=&sort=`        |
| `/reviews/{reviewId}/replies`       | POST        | 201       | 리뷰 답변 작성                 | -                          |
| `/reviews/{reviewId}/replies/{replyId}` | PATCH       | 200       | 리뷰 답변 수정                 | -                          |
| `/reviews/{reviewId}/replies/{replyId}` | DELETE      | 204       | 리뷰 답변 삭제                 | -                          |

---

## 3. 문의

| URI                                 | HTTP 메서드 | 응답 코드 | 설명                           | 쿼리 파라미터              |
|-------------------------------------|-------------|-----------|--------------------------------|----------------------------|
| `/inquiries`                        | POST        | 201       | 문의 작성                      | -                          |
| `/inquiries/{inquiryId}/replies`    | POST        | 201       | 문의 답변 작성                 | -                          |
| `/inquiries/{inquiryId}`            | PATCH       | 200       | 작성한 문의 수정               | -                          |
| `/inquiries`                        | GET         | 200       | 작성한 문의 조회               | `page=&size=&sort=`        |

---

## 4. 미션

| URI                                       | HTTP 메서드 | 응답 코드 | 설명                           | 쿼리 파라미터              |
|-------------------------------------------|-------------|-----------|--------------------------------|----------------------------|
| `/members/{memberId}/missions`            | GET         | 200       | 회원의 모든 미션 조회          | `status={완료 혹은 진행 중 상태}` |
| `/members/{memberId}/missions/{missionId}/complete` | PATCH       | 200       | 회원 미션 완료                 | -                          |
| `/members/{memberId}/missions/{missionId}/start`    | POST        | 201       | 회원 미션 시작                 | -                          |

---

## 5. 가게

| URI                                 | HTTP 메서드 | 응답 코드 | 설명                                  | 쿼리 파라미터              |
|-------------------------------------|-------------|-----------|-------------------------------------|----------------------------|
| `/stores/{storeId}`                 | GET         | 200       | 가게 상세 정보 조회                         | -                          |
| `/stores/{storeId}/reviews`         | GET         | 200       | 가게 리뷰 조회                            | `page=&size=&sort=`        |
| `/stores/{storeId}/missions`        | GET         | 200       | 가게 미션 조회 (회원 기준, 진행 중이거나 미완료 미션 조회) | `status=in_progress&status=pending` |

---

## 6. 포인트

| URI                                 | HTTP 메서드 | 응답 코드 | 설명                           | 쿼리 파라미터              |
|-------------------------------------|-------------|-----------|--------------------------------|----------------------------|
| `/members/{memberId}/points`        | GET         | 200       | 회원 포인트 내역 조회          | `type={포인트유형}&page=&size=` |
| `/members/{memberId}/points`        | POST        | 201       | 회원 포인트 지급               | -                          |

---

### API 명세서에서 엔드포인트를 어떻게 명시하는 것이 좋을지에 대한 고민

- {memberId}, {storeId}와 같은 파라미터를 명시적으로 URL에 포함시킬지, 아니면 토큰 인증을 통해 이러한 정보를 암묵적으로 처리할지?
- 로그인은 회원만 하는건지 가게만 하는 건지?


## 1. 홈 화면

---
### API Endpoint
```
GET /v1/api/home
```
### Query String
- 필요없음, 사용자 식별은 헤더의 인증 토큰으로 처리

### Path Variable
- 고정된 엔드포인트로 접근하기 때문에 경로 변수 x
### Request Header
- 사용자 인증을 위한 토큰이 필요함
```
Authorization: Bearer <token>
Content-Type: application/json
```
### Request Body
- GET 요청이므로 필요하지 않음

### Response Body
```
{
  "status": "success",
  "data": {
    "member": {
      "points": 999999
    },
    "missionStatus": {
      "currentCompletedMissions": 7,
      "totalMissions": 10,
      "missionReward": 1000
    },
    "missions": [
      {
        "id": 1,
        "storeName": "반어학생마라탕",
        "food": "중식당",
        "remainingDays": 7,
        "message": "10,000원 이상 식사",
        "point": 500,
        "status": "inProgress"
      },
      {
        "id": 2,
        "storeName": "반어학생마라탕",
        "food": "중식당",
        "remainingDays": 7,
        "message": "10,000원 이상 식사",
        "point": 500,
        "status": "inProgress"
      }
    ]
  }
}
```

### Status Codes

`200 OK`: 홈 화면 데이터가 조회 성공<br>
`401 Unauthorized`: 인증 토큰이 없거나 유효하지 않은 상태


## 2. 마이 페이지

---
### API Endpoint
```
GET /v1/api/mypage
```
### Query String
```
?page=1&size=10
```
- 페이징을 위해 쿼리 스트링 사용
### Path Variable
- 고정된 엔드포인트로 접근해서 불필요

### Request Header
```
Authorization: Bearer <token>
Content-Type: application/json
```
- 사용자 인증을 위해 토큰이 필요함

### Request Body
- GET 요청이므로 요청 본문x
### Response Body
```
{
  "status": "success",
  "data": {
    "member": {
      "points": 22500
    },
    "pointMember": [
      {
        "date": "2025-06-23",
        "description": "반어학생마라탕",
        "message": "10000원 이상 식사 적립",
        "point": 500,
        "type": "earn"
      },
      {
        "date": "2025-06-23",
        "description": "반어학생마라탕",
        "message": "10000원 이상 식사 적립",
        "point": 500,
        "type": "earn"
      },
      {
        "date": "2025-06-23",
        "description": "포인트 전환설정",
        "message": "포인트 전환설정",
        "amount": -20000,
        "type": "spend"
      },
      ...
    ]
  }
}
```

### Status Codes

---
`200 OK`: 마이 페이지 데이터 조회 성공<br>
`401 Unauthorized`: 인증 토큰이 없거나 유효하지 않은 상태


## 3. 리뷰 작성

---
### API Endpoint
```
POST /v1/api/reviews
```
### Query String
- 데이터를 생성하는 요청이므로 쿼리 스트링 필요없음

### Path Variable
- 고정된 엔드포인트로 접근

### Request Header
```
Authorization: Bearer <token>
Content-Type: application/json
```
- 사용자 인증을 위해 토큰이 필요함
### Request Body
- 입력한 리뷰 데이터를 포함한다.
```
{
  "storeName":"반어학생마라탕",
  "starRating": 5,
  "message": "음식이 정말 맛있었어요! 서비스도 훌륭했습니다.",
  "images": [
    "https://example.com/images/review1.jpg",
    ...
  ]
  ...
}
```
### Response Body
- 리뷰 작성 결과
```
{
  "status": "success",
  "data": {
    "reviewId": 1
  }
}
```

### Status Codes

---
`201 Created`: 리뷰 작성 성공<br>
`401 Unauthorized`: 인증 토큰이 없거나 유효하지 않은 상태


## 4. 미션 목록 조회(진행중, 진행 완료)

---
### API Endpoint
```
GET /v1/api/missions
```

### Query String
```
?page=1&size=10
```
- 페이징을 위해 쿼리 스트링 사용

### Path Variable
- 고정된 엔드 포인트를 사용하므로 경로변수 x
### Request Header
```
Authorization: Bearer <token>
Content-Type: application/json
```
- 사용자 인증을 위해 토큰이 필요함
### Request Body
```
{
  "status": ["inProgress", "completed"]
}
```
- 진행 중, 진행 완료 미션을 가져오기 위해 request body에 추가
### Response Body
```
//진행 중 미션 json 데이터 예시

{
  "status": "success",
  "data": {
    "missions": [
      {
        "id": 1,
        "storeName": "가게이름a",
        "message": "12000원 이상의 식사를 하세요!",
        "point": 500,
        "status": "inProgress"
      },
      {
        "id": 2,
        "storeName": "가게이름b",
        "message": "12000원 이상의 식사를 하세요!",
        "point": 500,
        "status": "completed"
      }
    ]
  }
}
```

### Status Codes

---
`200 OK`: 미션 목록 데이터 조회 성공<br>
`401 Unauthorized`: 인증 토큰이 없거나 유효하지 않은 상태

## 5. 미션 성공 누르기

---
### API Endpoint
```
PATCH /v1/api/missions/{missionId}/complete
```
### Query String
### Path Variable
```
missionId
```
- missionId는 현재 진행 중 미션의 id를 나타낸 것
### Request Header
```
Authorization: Bearer <token>
Content-Type: application/json
```
- 사용자 인증을 위해 토큰이 필요
### Request Body
- 미션 성공 버튼을 눌러 상태를 변경하는 요청이므로 필요 없음
### Response Body
```
{
  "status": "success",
  "data": {
    "missionId": 1,
    "storeId": 920394810,
    "status": "completed",
    "point": 500
  }
}
```
- 구분 번호 표시 및 기타 정보 json 데이터에 담아서 전송

### Status Codes

---
`200 OK`: 미션 상태 `completed`로 변경<br>
`401 Unauthorized`: 인증 토큰이 없거나 유효하지 않은 상태
`400 Bad Request` : 미션이 이미 완료된 상태

## 6. 회원 가입 하기(소셜 로그인 고려 X)

---
### API Endpoint
```
POST /v1/api/members/join
```
### Query String
- 모든 데이터는 request body를 통해 전달
### Path Variable
- 고정된 엔드 포인트 사용
### Request Header
```
Content-Type: application/json
```
- 회원 가입은 인증이 필요 없는 api이므로 `Authorization` 헤더 불필요
### Request Body
```
{
  "email": "example@example.com",
  "password": "password123!",
  "name": "홍길동",
  "birthDate": "2002-11-25",
  "AgreedToTerms": true,
  "food": ["중식", "한식", "양식"],
  "address": "연남동",
  "marketingAccepted": false,
  "locationAceepted": false
}
```
- 사용자가 입력한 회원 가입 정보 포함
### Response Body
```
//성공 예시
{
  "status": "success",
  "data": {
    "userId": 1,
    "email": "example@example.com",
    "name": "홍길동",
    "createdAt": "2025-04-01T09:41:00Z"
  }
}

//실패 예시

{
  "status": "error",
  "message": "Email already exists."
}
```

### Status Codes

---
`201 Created`: 회원 가입 성공 <br>
`401 Unauthorized`: 인증 토큰이 없거나 유효하지 않은 상태
`400 Bad Request` : 이메일이 이미 존재하는 경우