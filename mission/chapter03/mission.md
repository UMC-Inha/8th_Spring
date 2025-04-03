## - 미션

#### - 홈 화면 엔드포인트 설계

<img src = "https://velog.velcdn.com/images/sm011212/post/02ea3fdb-1197-4d76-9342-89448e3f4b97/image.png" width = "500">

- 조회 요청이기에 **GET** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- userId는 jwt 토큰에 담겨있을 것 같아 엔드포인트에 userId를 명시하지 않았고, 메인 화면의 의미를 담아 /api/main으로 설정해주었다.

- 메인 화면에서 지역 정보를 필요로 하기에 query string으로 region 값을 넣어주었고, 커서 기반 페이징 적용을 위해 cursor 값과 limit 값을 추가로 요청 받도록 설정했다.

- GET 요청이기 때문에 Request Body에는 값을 입력하지 않았다.

#### - 마이 페이지 리뷰 작성

<img src = "https://velog.velcdn.com/images/sm011212/post/3b1bf940-cbd1-4344-aa4a-814c1727e802/image.png" width = "500">

- 등록 요청이기에 **POST** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- 특정 store에 review를 작성하는 것이기 때문에 엔드포인트에 /store/{storeId}를 작성해 storeId를 전달하도록 했고, review를 작성하는 것이기 때문에 /review를 추가로 적어주었다.

- rating 값은 소수값이라고 가정했고, image는 여러 개의 값을 받을 수 있을 것이라 생각해 url string이 담긴 리스트로 설계했다.

#### - 미션 목록 조회

<img src = "https://velog.velcdn.com/images/sm011212/post/7f768827-cd39-4e56-80ec-f99b8c14cdbd/image.png" width = "500">

- 조회 요청이기에 **GET** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- userId는 jwt 토큰에 담겨있을 것 같아 엔드포인트에 userId를 명시하지 않았고, 특정 유저의 미션이므로 user-missions 라는 엔드포인트를 구분했다. (그냥 mission으로 하면 공통 미션인지 아닌지 헷갈리니까)

- 커서 기반 페이징 적용을 위해 query string으로 cursor 값과 limit 값을 추가로 요청 받도록 설정했다.

- GET 요청이기 때문에 Request Body에는 값을 입력하지 않았다.

#### - 미션 성공 누르기

<img src = "https://velog.velcdn.com/images/sm011212/post/70b9dee4-ddf0-40a3-afa3-1a998045d700/image.png" width = "500">

- 특정 작업을 수행하는 요청을 보내기에 **POST** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- userId는 jwt 토큰에 담겨있을 것 같아 엔드포인트에 userId를 명시하지 않았고, 특정 유저의 미션이라는 것을 명시하기 위해 user-missions 엔드포인트로 설정했다. 해당 미션의 정보를 알아야 하기 때문에 {missionId}를 path variable로 받도록 했다.

- 미션 성공을 수행한다는 것을 직관적으로 이해할 수 있도록 컨트롤 URI를 사용해 /success를 명시해주었다.

#### - 회원 가입 하기

<img src = "https://velog.velcdn.com/images/sm011212/post/0bfc757f-dea8-4261-9192-5887ed9a6bc5/image.png" width = "500">

<img src = "https://velog.velcdn.com/images/sm011212/post/d2bbd339-fa6e-45a3-9ab8-edd6fa41de10/image.png" width = "500">

- 새롭게 유저를 등록하므로 **POST** 메서드를 사용했다.

- 아직 유저가 등록되지 않았으므로 access token 정보는 작성하지 않았다.

- 유저 등록에 필요한 정보들을 Request Body로 작성했다.

- 주소 정보, 알림 여부, 권한 정보는 별도의 dto로 받을 수도 있다고 생각해 map 객체로 받도록 설계했다.

#### - 시니어 미션: Soft Delete가 무엇인지 찾아보시고 soft delete에는 어떠한 HTTP Method가 들어가면 좋을지 적어주세요

- Soft Delete란?

    - **데이터를 DB에서 실제로 삭제하지 않고, 삭제된 것처럼 처리하는 방식**

- Soft Delete를 사용하는 이유

    - 삭제한 데이터를 복구할 수 있다.

    - 데이터 간 관계(무결성)을 유지할 수 있다.

        - 실제로 값을 삭제하면 참조 무결성이 깨지거나 오류가 발생할 확률이 높아진다.

    - 데이터 변경 이력을 추적할 수 있다.

- Soft Delete는 어떻게 구현할까?

    - **DB 테이블에 is_deleted, deleted_at과 같은 컬럼을 추가한다.**

    - 삭제시에는 실제로 DELETE를 하는 것이 아닌, 위의 컬럼 값을 변경한다.

- Soft Delete에는 어떤 HTTP Method를 사용하는 것이 좋을까?

    - PATCH : 상태값 변경처럼 처리할 수 있다.

    - DELETE : 클라이언트 입장에서는 삭제 요청의 의미로 보이면서, 서버에서 세부적으로 작업을 구현할 수 있다.

  -> **DELETE가 조금 더 명시적이고**, PATCH는 다른 작업(정보 수정 등)에 사용하는 것이 좋다고 생각한다.

#### - 시니어 미션: 컨트롤 URI에 대해 조사해주시고 어떠할 때 사용이 가능한 지 예시를 들어 설명해주세요.

- 컨트롤 URI란?

    - 리소스의 상태나 동작을 제어하기 위한 URI

    - CRUD 외에 특별한 동작이나 상태 변경을 요청할 때 사용하는 URI 설계 패턴

- 컨트롤 URI 사용 예시

    - 게시글 좋아요의 엔드포인트를 /posts/{postId}/like와 같이 사용한다.

    - 미션 완료 처리를 /missions/{missionId}/complete와 같이 사용한다.

- 컨트롤 URI를 사용하는 이유

    - 의도를 명확하게 표현할 수 있다.

        - /missions/{missionId}/complete라는 엔드포인트를 봤을 때 미션 완료 요청이라는 것이 직관적으로 해석 가능하다.

    - API 문서화와 Swagger 설명이 더욱 직관적이다.

    - 응용 동작 추가가 편하다. 위에서 complete 외에도 reject와 같이 다른 동작도 추가할 수 있다. (이를 PATCH에서 관리하면 복잡해진다.)

    - 권한 체크, 로그 추적에도 유리하다. 해당 액션에만 특정 로그를 남길 수 있기 때문이다.

#### - 시니어 미션: Azure-Arthitecture Center 문서를 읽고 주요 내용을 간단히 정리해주세요.

- **REST란?**

    - 하이퍼미디어 기반 분산 시스템을 구축하기 위한 아키텍쳐 스타일. 개방형 표준을 사용한다.

- **RESTful API 디자인 원칙**

    - 리소스를 중심으로 디자인되며, 클라이언트에서 액세스할 수 있는 모든 종류의 개체, 데이터 또는 서비스가 리소스에 포함된다.

    - 리소스마다 해당 리소스를 고유하게 식별하는 URI인 **식별자**가 있다.

    - 클라이언트가 리소스의 표현을 교환해 서비스와 상호 작용한다. 교환 형식으로는 JSON을 사용한다.

    - **비저장 요청 모델(stateless)을 사용한다.** 각 요청은 독립적이어야 한다.

- **리소스를 중심으로 API 디자인을 구성한다.**

    - **리소스 URI는 동사(작업)가 아닌 명사(리소스)를 기반으로 해야 한다.**

    - 컬렉션 및 항목에 대한 URI를 계층 구조로 구성하는 것이 좋다.

    - **URI를 복잡하게 하는 것은 좋지 않다.**

        - /customers/1/orders/99/products 이런 요청은 유지 보수가 어렵고 유지보수성이 떨어진다.

        - /customers/1/orders로 바꿔서 모든 주문을 찾고, /orders/99/products로 주문의 제품을 찾을 수도 있다.

        - 리소스 URI를 **컬렉션/항목/컬렉션보다 더 복잡하게 요구하지 않는 것이 좋다.**

- **HTTP 메서드 측면에서 API 작업 정의**

    - RESTful API에서 사용하는 일반적인 HTTP 메서드는 GET, POST, PUT, PATCH, DELETE가 있다.

    - **GET**

        - 리소스의 표현을 검색

        - 성공시 HTTP 상태 코드 200을, 찾을 수 없으면 404를 반환한다.

    - **POST**

        - 새로운 리소스를 만든다.

        - 리소스가 만들어지면 201을, 반환한다.

        - 일부 처리를 수행하지만 새로운 리소스를 만들지 않으면 201을, 반환할 결과가 없으면 204를 반환하기도 한다. 잘못된 요청일 경우에는 400을 반환한다.

    - **PUT**

        - 리소스를 만들거나 대체(전체 수정)

        - **PUT 요청을 여러 번 제출해도 그 결과는 항상 같아야 한다.**(idempotent)

        - 새로운 리소스를 만들면 201, 업데이트 시에는 200 혹은 204를, 업데이트 할 수 없으면 409를 반환한다.

    - **PATCH**

        - 리소스의 부분  수정


- **DELETE**

    - 리소스 제거

- **데이터 필터링 및 페이지 매기기**

    - 값을 필터링 할 때는 **쿼리 스트링**을 통해 필터를 전달할 수 있다.

        - ex) /orders?limit=25&offset=50

- **HATEOAS를 사용하여 관련 리소스 탐색**

    - 각 HTTP GET 요청은 응답에 포함된 하이퍼링크를 통해 요청된 개체와 직접 관련된 리소스를 찾는 데 필요한 정보를 반환해야 하며, 이러한 각 리소스에 대해 사용할 수 있는 작업을 설명하는 정보도 함께 제공되어야 한다.