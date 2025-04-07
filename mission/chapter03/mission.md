### 1. 홈 화면

| 항목            | 내용                             |
|-----------------|----------------------------------|
| API Endpoint    | `/home`                          |
| Method          | `GET`                            |
| Request Body    | x                                |
| Request Header  | `Authorization = "Access Token"` |
| query String    | x                                |
| Path variable   | x                                |


### 2. 마이 페이지

| 항목            | 내용                                 |
|-----------------|--------------------------------------|
| API Endpoint    | `/user/{nickname}`                   |
| Method          | `GET`                                |
| Request Body    | x                                    |
| Request Header  | `Authorization = "Access Token"`     |
| query String    | x                                    |
| Path variable   | `nickname(varchar(20))`              |


### 3. 리뷰 작성

| 항목            | 내용                                               |
|-----------------|----------------------------------------------------|
| API Endpoint    | `/restaurant/{restaurant_id}/review/`             |
| Method          | `POST`                                            |
| Request Body    | `{ "star": Integer, "content": "text", "img_url_list": string[] }` |
| Request Header  | `Authorization = "Access Token"`                  |
| query String    | x                                                 |
| Path variable   | `restaurant_id(Long)`                             |



### 4. 미션 목록 조회

1) 첫 조회 시

| 항목            | 내용                                               |
|-----------------|----------------------------------------------------|
| API Endpoint    | `/user/{user_id}/mission`                          |
| Method          | `GET`                                              |
| Request Body    | x                                                  |
| Request Header  | `Authorization = "Access Token"`                   |
| query String    | ?completed = enum(’true’, ‘false’)
| Path variable   | `user_id(Long)`                                    |

2) 두 번째 이상 조회 시
   
| 항목            | 내용                                               |
|-----------------|----------------------------------------------------|
| API Endpoint    | `/user/{user_id}/mission`                          |
| Method          | `GET`                                              |
| Request Body    | x                                                  |
| Request Header  | `Authorization = "Access Token"`                   |
| query String    | ?completed = enum(’true’, ‘false’)&cursor-updated-at = 2024-02-28 |
| Path variable   | `user_id(Long)`                                    |

### 5. 미션 성공 누르기

| 항목            | 내용                                               |
|-----------------|----------------------------------------------------|
| API Endpoint    | `/user/{user_id}/mission/{mission_id}`            |
| Method          | `PATCH`                                            |
| Request Body    | x                                                  |
| Request Header  | `Authorization = "Access Token"`                   |
| query String    | x                                                  |
| Path variable   | `user_id(Long)`, `mission_id(Long)`               |


### 6. 회원가입 하기

| 항목            | 내용                                                    |
|-----------------|---------------------------------------------------------|
| API Endpoint    | `/user/join`                                            |
| Method          | `POST`                                                  |
| Request Body    | `{                                                     |
|                 |    "username": "varchar(20)",                          |
|                 |    "password": "text",                                 |
|                 |    "nickname": "varchar(20)",                          |
|                 |    "phone_number": "varchar",                          |
|                 |    "birth_date": "datetime(6)",                        |
|                 |    "gender": "enum('MAN', 'WOMAN')",                   |
|                 |    "email": "text"                                     |
|                 | }                                                      |
| Request Header  | x                                                       |
| query String    | x                                                       |
| Path variable   | x                                                       |

-----------------------------------------------------------------

## 시니어 미션

- [ ]  Soft Delete가 무엇인지 찾아보시고 soft delete에는 어떠한 HTTP Method가 들어가면 좋을지 적어주세요
    - 내용들을 간단하게 정리하여 주세요
        
        데이터를 삭제하는 방법에는 Soft Delete와 Hard Deleter가 존재한다.
        
        Soft Delete는 데이터베이스에서 데이터를 삭제하지 않고, 사용자 입장에서는 데이터를 접근할 수 없게 하는 방식을 의미한다. 예시로는 ERD 모델링에서 나온 user의 status 칼럼을 활용한다고 생각하면 된다.
        
        Hard Delete는 우리가 흔히 아는 데이터베이스에서 데이터를 직접 삭제하는 방식을 의미한다.
        
        각 상황에 맞게 Delete 방식을 선택해야 하며 즉시 삭제가 아닌 선택을 번복할 수 있는 상황에서 soft delete를 선택하면 된다.
        
        mysql의 경우는 아직 내부적으로 스케쥴링 하는 기능이 없어서 spring에서 스케쥴러를 사용해 정기적으로 검사하는 방식을 이용하면 된다.
        
        Soft Delete는 상태를 바꾸는 것이므로 PATCH 방식, Hard Delete는 데이터를 삭제하는 것이므로 DELETE 방식이 적합해 보인다. 
        
    

- [ ]  컨트롤 URI에 대해 조사해주시고 어떠할 때 사용이 가능한 지 예시를 들어 설명해주세요.
    - 내용들을 간단하게 정리하여 주세요
        
        컨트롤 URI(Control URI)는 리소스에 대한 CRUD(Create, Read, Update, Delete) 외의 특정한 행위를 수행할 때 사용하는 URL 패턴이다.
        
        RESTful API에서는 **리소스는 URL로 표현하고, 행위는 HTTP Method로 구분**해야 하지만,
        
        경우에 따라 HTTP Method만으로 **특정 행위**를 나타내기 어려울 때 컨트롤 URI를 사용
        
        할 수 있습니다.
        
        예를 들어 무언가를 만드는 행위라던가, 프로세스를 고정하는 행위를 나타내야 할 때, /items/{id}/**create**, /vi/home/**fix →** 이렇게 추가적인 정보로 특정 행위를 표현할 수 있다.
        

- [ ]  https://learn.microsoft.com/ko-kr/azure/architecture/best-practices/api-design - 문서를 읽고 주요 내용을 간단히 정리해주세요.
    - 내용들을 간단하게 정리하여 주세요
        
        RESTful 웹 API 디자인
        
        - 리소스를 중심으로 API 구성
            - 리소스 → URL
            - 컬렉션을 참조 → 복수명사 사용 ex) /customers/5
            - 엔티티 간의 관계 표시 → ex) /customers/5/orders (고객이 한 주문들)
            - 다수의 작은 리소스를 가져오는 번잡한 URL 피하기
        - HTTP 의미 체계 준수
            - 미디어 유형
                - content-type 헤더에 표현 형식을 지정
            - GET
                - 정상적인 경우 → 200
                - 요청이 처리되었지만 응답 내용이 없을 경우 204
            - POST
                - 새 리소스를 만드는 경우 → 201
                - 반환 결과가 없을 경우 → 204
                - 클라이언트가 잘못된 요청을 할 경우 → 400
            - 비동기 작업
                - POST, PUT 등의 Method를 수행할 때, 작업이 완료될 때까지 시간이 걸릴 수 있다
                    
                    이 때 요청이 수락되었지만 아직 완료되지 않았음을 나타내는 경우 → 202
                    
        - HATEOAS
            - 정의
                - 응답에 포함된 하이퍼링크를 통해 요청된 개체와 직접 관련된 리소스를 찾는데 필요한 정보를 반환
            - 장점
                - API 문서에 덜 의존적
                - 더 나은 확장
