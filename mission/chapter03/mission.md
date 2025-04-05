## API 명세서 작성하기

<br>

### 1. 홈 화면

- API Endpoint
```
GET /api/home

// 홈 화면은 로그인한 사용자의 정보 기반으로 처리되므로 userId를 url에 노출할 필요 없음.
// 만약 url에 포함시키면, 다른 사용자의 userId를 url에 넣어 타인의 홈을 볼 수 있다는 문제가.. 
```

- Request Body
```
x
```

- Request Header
```
Authorization : accessToken (String)
```

- Query String
```
x
```

- Path Variable
```
x
```

<br>

### 2. 마이 페이지 리뷰 작성

- API Endpoint
```
POST /api/my-page/reveiws
```

- Request Body
```
{
    "score" : "별점",
    "reviewBody" : "리뷰내용",
    "imageUrl" : "사진"
}
```

- Request Header
```
Authorization : accessToken (String)
```

- Query String
```
x
```

- Path Variable
```
x
```

### 3. 미션 목록 조회 (진행중, 진행 완료)

- API Endpoint
```
GET /api/missions
```

- Request Body
```
x
```

- Request Header
```
Authorization : accessToken (String)
```

- Query String
```
// 진행중
?status=IN_PROGRESS

// 진행 완료
?status=COMPLETE
```

- Path Variable
```
x
```

<br>

### 4. 미션 성공 누르기

- API Endpoint
```
PATCH /api/missions/{missionId}
```

- Request Body
```
{
    "status" : "COMPLETE"
}
```

- Request Header
```
Authorization : accessToken (String)
```

- Query String
```
x
```

- Path Variable
```
missionId
```

<br>

### 5. 회원가입하기 (소설 로그인 고려 X)

- API Endpoint
```
POST /api/users/sign-up
```

- Request Body
```
{
    "name" : "이름",
    "gender" : "성별",
    "birth" : "생년월일",
    "address" : "주소",
    "preferred-food-category" : ["한식", "양식", "일식", ...],
    "email" : "이메일",
    "password" : "비밀번호"
}
```

- Request Header
```
Content-Type: application/json
```

- Query String
```
x
```

- Path Variable
```
x
```