# chapter 3 키워드
## Restful API란

### 1. REST란
Representational State Transferf의 약자로 웹의 장점을 최대한 활용할 수 있도록 설계된 소프트웨어 아키텍처 스타일이다.  
REST가 갖는 특징은 아래와 같다.
- 클라이언트-서버 구조
- 상태 비저장(Stateless)
- 캐시 가능(Cacheable)
- 계층화된 시스템(Layered System)
- 코드 온 디맨드(Code on Demand, 선택적)
- 인터페이스 일관성(Uniform Interface)


### 2. RESTful API란?
REST의 원칙을 따르는 API를 Restful API라고 한다.
HTTP 프로토콜을 활용하여 자원을 CRUD 방식으로 처리하는 API를 의미한다.

### 3. RESTful API 특징
- 리소스 기반 : URL은 동사(행위)가 아니라 명사를 사용해야 한다.
- HTTP 메서드 활용 :  GET, POST, PUT, DELETE의 HTTP 메서드를 사용하여 자원을 조작한다.
- 상태 비저장 : 요청 간 클라이언트의 상태를 서버가 저장하지 않는다.
- 계층적 구조 : 클라이언트는 API 서버만 알면 되고, 내부 서비스 구조는 숨길 수 있다.
- 일관된 인터페이스 : 요청 형식과 응답 형식이 일관되게 유지된다.

### 4. RESTful API의 HTTP 메서드
1. GET : 조회
2. POST : 생성
3. PUT : 갱신(전체)
4. PATCH : 갱신(일부)
5. DELETE : 삭제

### 5. 설계 규칙
- 명사 기반의 URL 사용 (/getUserInfo(x) -> /users/{id}(o))
- http 메서드 활용
- 응답 데이터 Json 활용
- 에러 코드 사용 
