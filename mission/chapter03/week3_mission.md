# 미션
1. 홈화면

   | METHOD | API Endpoint | Request |
   | --- | --- | --- |
   | GET | /umc/home?memberId= | memberId (Query String) |

2. 마이페이지 리뷰 작성(?) -> 내가 작성한 리뷰 가져오기로 생각

| METHOD | API Endpoint | Request |
| --- | --- | --- |
| GET | /umc/members/{memberId}/reviews | memberId (Path Variable) |

3. 미션 목록 조회 (진행중, 진행 완료)

| METHOD | API Endpoint | Request                                         |
| --- | --- |-------------------------------------------------|
| GET | /umc/members/{memberId}/missions?status= | memberId (Path Variable), status (Query String) |

4. 미션 성공 누르기

| METHOD | API Endpoint | Request |
| --- | --- | --- |
| PATCH | /umc/missions/{missionId} | missionId (Path Variable) |

5. 회원 가입 하기

| METHOD | API Endpoint | Request |
| --- | --- | --- |
| POST | /umc/users/signup | memberName, memberGender, memberBirth, memberAddress, member_prefer_food_category, phoneNum, mail etc.. (Request Body) |


# 시니어 미션
### 1. Soft Delete 란? soft delete 는 HTTP Method 가 어떤게 좋을지?

Soft Delete

- 논리 삭제
- 즉, DB에서 삭제하지 않고 사용자 입장에서 데이터에 접근할 수 없게 하는 방식

왜 사용하는가?

- 서비스 개선을 위해 데이터 분석은 필수
- 혹여나 데이터 복구를 해야하는 상황 대비
- update가 delte 보다 훨씬 빠름
- 데이터는 소중하다.

단점

- 테이블 크기 증가 (inactive 등 컬럼 1 ~ 2 개 늘어남)
- 실제 사용하지 않는 데이터가 존재함

HTTP Method는?

- 실제론 삭제가 아니라 수정이기에 PUT 아니면 PATCH 가 맞는 것 같다고 생각했다.
- 하지만 찾아보았을 때 클라이언트 측에선 Soft/Hard 여부를 알 필요가 없기에 DELETE 가 맞다는 의견들이 많다.

### 2. 컨트롤 URI에 대해 조사, 어떨 때 사용 가능한지 예시 들어 설명

Control URI 는 무엇인가

- 어떤 자원을 제어하거나 조작하기 위한 목적의 엔드포인트
- 일반적인 RESTful API 는 아님

어떠한 상황에서 쓰이는가?

- 특정 자원의 상태 전이
- 일반 CRUD 이외의 특수한 동작을 하는 경우
- 즉, 복잡한 비즈니스 로직이 개입되어 간단한 CRUD 로 표현하기 힘들 때 사용하는 것 같음

예시

- 기기 재부팅 → POST /devices/{deviceId}/reboot
- 보드 고정 → PATCH /boards/{boardId}/fix

위와 같은 느낌..

### 3. 문서 주요 내용 정리

REST 란

- Representational State Transfer 약자
- 하이퍼미디어 기반 분산 시스템을 구축하기 위한 아키텍처 스타일
- 어떤 기본 프로토콜과도 독립적, HTTP에 연결될 필요가 없음
   - 하지만 대부분의 일반적인 REST API 구현은 HTTP를 프로토콜로 사용함 (이 문서도 HTTP용 설계에 중점 둠)

RESTful API 기본 URI 디자인 원칙

- 명사 중심 설계
   - GET /customers → 모든 고객 조회
   - GET /customers/{customerId} → 특정 고객 조회
- 계층 관계 표현
   - GET /customers/{customerId}/orders → 특정 고객의 주문 목록 조회

HTTP Method 로 API 작업 정의

- GET
   - 리소스 조회
- POST
   - 새 리소스 생성
- PUT
   - 기존 리소스 전체 업데이트
- PATCH
   - 기존 리소스 부분적 업데이트
- DELETE
   - 리소스 삭제

쿼리 문자열 사용

- 리소스 컬렉션을 조회할 때 사용
   - 결과 필터링 or 정렬 or 페이징 처리

버전 관리

- API 버전 명시
   - API 변경 사항 파악 및 클라이언트에 영향 미치지 않기 위해 버전 명시
      - GET / v1/customers → URI에 작성
      - Accept: application/vnd.myapi.v1+json → Header에 작성
      - https://adventure-works.com/customers/3?version=2 → 쿼리 String으로 작성

