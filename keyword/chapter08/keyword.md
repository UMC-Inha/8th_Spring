- java의 Exception 종류들
    
    ### Error와 Exception 정리
    
    ---
    
    1. **Error**
    - **JVM 내부 오류**를 나타냄
        - 메모리 부족
        - 스택 오버플로우 등
    - **회복 불가능**한 심각한 문제
    - 애플리케이션 코드에서 **처리하거나 잡지 않음**
    
    1. **Exception**
    - java.lang.Exception를 상속
    - 두 가지 종류로 나뉨:
        
        ### 2.1. Checked Exception
        
        - RuntimeException 을 제외한 Exception
        - **컴파일 타임**에 처리(try–catch) 또는 선언(throws) 필수
        - 외부 I/O, 네트워크, 파일 입출력 등 **회복 가능**한 예외 상황에 사용
        - 예시: `IOException`, `ClassNotFoundException`
        
        ### 2.2. Unchecked Exception (RuntimeException)
        
        - RuntimeException을 상속
        - **처리나 선언 강제 없음**
        - 프로그래밍 버그(널 참조, 잘못된 타입 변환 등) 나타냄
        - 스프링에서는 **기본 롤백 대상**
        - 예시: `NullPointerException`, `IllegalArgumentException`
        
    
    ---
    
    **정리**
    
    - **Error**: JVM 수준의 심각한 오류, 애플리케이션 코드에서 건드리지 않음
    - **Checked Exception**: 반드시 처리·선언, 회복 가능한 상황
    - **Unchecked Exception**: 자유롭게 던지고, 스프링 트랜잭션·글로벌 핸들러에서 처리
- Custom 어노테이션(ExistCategory)
    
    ### 과정
    
    1. @ExistCategory 가 붙어 있으면, 클라이언트 요청이 컨트롤러에 도달하자마자 **Bean Validation** 단계에서 걸리고,
    2. 이때 발생하는 **MethodArgumentNotValidException**은 프로젝트의 @ControllAdvice 가 공통 400 에러로 매핑하고
    
    ![image.png](attachment:03dfd103-058c-4522-9e75-63a989d44e89:image.png)
    
    1. 실패한 필드의 메시지 텔플릿(FOOD_CATEGORY_NOT_FOUND)을 result 객체 안에 담아 응답한다.
    
    ![image.png](attachment:43dc953d-e146-4ec3-b7ba-9b0f869a2e00:image.png)
    
    ![image.png](attachment:924c1195-90e3-4597-a566-f356190e5157:image.png)
    
    ### Custom 어노테이션을 사용하는 이유
    
    1. **중복 로직 제거**
        
        서비스나 컨트롤러에서 매번 try-catch 등의 에러 처리를 사용하는 대신 어노테이션으로 한 번에 처리하도록 해준다.
        
    2. **표현의 일관성**
    3. **유지보수 편의성**
        
        검증 로직을 한 곳에 모아 두면, 나중에 검사 기준이 바뀌어도 validator 클래스에서 로직을 수정하면 된다
        
- @Valid
    - **@Valid**
        - **용도**: @RequestBody의 DTO 바인딩 직후 Bean Validation을 실행
        - **사용처**: 파라미터 선언부(@RequestBody MyDTO myDTO) 또는 Custom 어노테이션
        - **예외 타입**: 검증 실패 시 → **MethodArgumentNotValidException**
    - **@Validated**
        - **용도**: 메서드 진입 전 AOP 방식으로 파라미터(경로·쿼리·폼 등) 제약조건을 검사
            - PathVariable, RequestParam 조사
        - **사용처:** Custom 어노테이션
        - **어디에 붙이나**: 컨트롤러(@RestController)나 서비스 클래스(@Service) 위
        - **예외 타입**: 검증 실패 시 → **ConstraintViolationException**
        
        ### 정리
        
        | 대상 파라미터 | 바인딩 검증 어노테이션 | 필요 설정 | 예외 타입 |
        | --- | --- | --- | --- |
        | @RequestBody | @Valid | (컨트롤러에 @Valid만) | MethodArgumentNotValidException |
        |  @PathVariable, @RequestParam  | (표준·커스텀) | 컨트롤러 또는 메서드에 @Validated | ConstraintViolationException |
        
    
    ### 에러 비교
    
    ![image.png](attachment:763e408a-9390-4468-b48c-5cd24fc8da37:image.png)
    
    ![image.png](attachment:6070a608-da52-4639-843a-5fe222cd0638:image.png)
    
    → 실제로 각각 ConstraintViolationException, MethodArgumentNotValidException 에러 부분이고 에러가 달라서 이와 같이 요청 결과도 다르게 처리된다.

    **notion에** 각 image 원본이 저장되어있습니다!
