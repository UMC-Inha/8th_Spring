- ReRestful API
    
    Restful API란 REST 규칙을 지킨 웹 API로 이를 이해하기 위해서는 먼저 REST 원칙부터 이해해야 한다.
    
    - REST
        - 정의
            - 웹 서비스가 어떻게 동작해야 하는지에 대한 아키텍처 스타일 혹은 설계원칙을 의미함
        - 세부 원칙
            1. HTTP URL 명시
            2. HTTP Method(POST, GET, PATCH 등)을 통해
            3. 해당 자원(URL)에 대한 CRUD operation 적용
        - 특징
            1. 서버-클라이언트 구조
            2. Stateless
            3. Cacheable
            4. Layered System
            5. Uniform interface
        
        즉, REST란 이론적인 원칙과 가이드라인이며 RESTful API는 그 원칙과 가이드라인을 작 지킨 서비스 API 라고 이해하면 된다.

---------------------------
        
- 홈화면 API 전략
    - BEF
        - 정의
            - 클라이언트의 화면에 맞게 최적화된 API를 제공하는 중간 계층
        
        - 방식
            - BEF는 화면 단위로 구성된 API를 제공한다. 즉 여러 서비스를 조합해서 프론트에 딱 맞는 형태로 조합해준다.
            - ex)
                
                [User API]     
                
                [Mission API]         → [BEF]  → [Web/App]
                
                [Restaurant API]
                
        - 필요성
            - 여러 개의 api의 응답을 조작해야하는 경우
            - 데이터를 전송하는 경우에서 불필요한 데이터를 숨겨야 하는 경우
            - Open API 연동시 제공하는 api 조합으로 특정 기능을 완성해야 하는 경우
            - 프론트엔드에서 많은 양의 연산을 요구하는 경우
        
        - 적용
            - monolithic 구조를 사용하는 크지 않은 프로젝트의 경우에는 굳이 필요성이 없지 않을까 라고 생각합니다. 괜히 도입했다가 코드의 복잡성만 늘어나므로, 백엔드의 규모가 커져 MSA 도입이 되었을 때 BEF를 적용하면 효과적이지 않을까 싶습니다!
