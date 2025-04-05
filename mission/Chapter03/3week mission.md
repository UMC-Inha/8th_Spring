## API 명세서

### 홈 화면

| API Endpoint | RequestBody | Request Header | Query String |
| --- | --- | --- | --- |
| GET /api/home |  | Authorization : accessToken (String) | region=? |

---

### 마이 페이지 리뷰 작성

| API Endpoint | RequestBody | Request Header | Query String |
| --- | --- | --- | --- |
| POST /api/reviews | `{`<br>&nbsp;&nbsp;`store_id` : `"?"`,<br>&nbsp;&nbsp;`starRating` : `"?"`,<br>&nbsp;&nbsp;`content` : `"?"`,<br>&nbsp;&nbsp;`imageURL` : `"?"`<br>`}` | Authorization : accessToken (String) |  |

---

### 미션 목록 조회 (진행중, 진행 완료)

| API Endpoint | RequestBody | Request Header | Query String |
| --- | --- | --- | --- |
| GET /api/missions |  | Authorization : accessToken (String) | status=? |

---

### 미션 성공 누르기

| API Endpoint | RequestBody | Request Header | Query String |
| --- | --- | --- | --- |
| PATCH /api/missions | `{`<br>&nbsp;&nbsp;`status` : `"complete"`<br>`}` | Authorization : accessToken (String) | {missionId} |

---

### 회원 가입 하기

| API Endpoint | RequestBody | Request Header | Query String |
| --- | --- | --- | --- |
| POST /api/signup | `{`<br>&nbsp;&nbsp;`name` : `"?"`,<br>&nbsp;&nbsp;`email` : `"?"`,<br>&nbsp;&nbsp;`gender` : `"?"`,<br>&nbsp;&nbsp;`birth` : `"?"`,<br>&nbsp;&nbsp;`phoneNum` : `"?"`,<br>&nbsp;&nbsp;`address` : `"?"`<br>`}` |  |  |