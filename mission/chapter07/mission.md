### 실습 
![실습1](https://github.com/user-attachments/assets/4862214e-0c58-4dd3-96e7-f61fd2543a73)

![실습2](https://github.com/user-attachments/assets/002e738b-046a-458a-846c-1cdee3d164d9)

![실습3](https://github.com/user-attachments/assets/d6c1e5ef-3bea-4dd5-9989-5a76555533d9)

- @RestControllerAdvice 사용 시 장점
    - 예외 처리 로직 집중화, 코드 중복 감소, 유지보수 용이
        - 하나의 클래스로 모든 컨트롤러에 예외 처리 가능함
        - 개별의 `try-catch` 블록 필요 없어 코드 중복 감소
    - 일관성 있는 에러 응답 제공
    - 민감한 정보 노출 방지, 보안 강화
    - 유연한 예외 처리
        - 특정 예외 클래스별로 다른 처리 로직을 적용 가능
        - 예외 유형에 따라 HTTP 상태 코드나 응답 바디를 다르게 설정하는 등 유연한 대응이 가능
- 없을 경우 단점
    - 코드 반복 및 복잡성 증가, 유지보수 어려움
        - 모든 컨트롤러나 서비스 레이어에서 발생할 수 있는 예외를 개별적으로 처리해야 하므로, 동일한 예외 처리 코드가 여러 곳에 중복
    - 일관되지 않은 에러 응답
        - 응답 포맷의 일관성이 없어져 클라이언트에게 전달되는 에러 응답 형식이 제각각이 될 수 있음
        - 클라이언트가 서버의 에러 응답을 파싱하고 처리하는 것을 복잡하게 만듬
    - 보안 취약점 발생 가능성
- 미션 목록 조회
    
    
    | API Endpoint | `GET /user-missions` |
    | --- | --- |
    | Request Body | X |
    | Request Header | Authorization: accessToken (String) |
    | Query String | `?status=(enum)` |
    | Path variable | X |
