- ExceptionHandler
    
    @ExceptionHandler(xxException.class) : 발생한 xxException에 대해서 처리하는 메소드를 작성한다. 
    
     
    
    ```java
    @Controller
    public class SimpleController {
    
        // ...
    
        @ExceptionHandler
        public ResponseEntity<String> handle(IOException ex) {
            // ...
        }
    }
    ```
    
    이렇게 `@Controller`로 선언된 클래스 안에서 `@ExceptionHandler` 어노테이션으로 메서드 안에서 발생할 수 있는 에러를 처리할 수 있다. 또는 `@ControllerAdvice` 선언된 클래스 안에서도 작동할 수 있다.
    
- RestContollerAdvice
    
    Spring은 전역적으로 예외를 처리할 수 있는 클래스로 @ContorllerAdvice와 @RestControllerAdvice가 존재한다. 이 둘의 차이는 @RestControllerAdvice는 @ControllerAdvice와 달리 @ResponseBody가 붙어 있어 응답을 Json으로 내려준다는 점에서 다르다
    
    **ExceptionHandler와 비교**
    
    1. RestContollerAdvice
        - 이 애노테이션을 단 클래스쪽으로 “컨트롤러”에서 전파된 예외를 한곳으로 모아줍니다.
        - 내부에 정의된 @ExceptionHandler 메서드들이 이 컨트롤러 호출 스택에서 던져진 예외를 받아 처리하죠.
    2. @ExceptionHandler
        - @RestControlAdvice 클래스(또는 개별 @RestControl 클래스) 안의 **메서드**에 붙입니다.
        - 메서드 시그니처로 처리할 예외 타입을 지정하고, 해당 예외가 발생했을 때 이 메서드가 호출됩니다.
        - 예외를 받아 `ResponseEntity<?>` 등을 리턴하면, 그 리턴 값이 최종 HTTP 응답이 됩니다.
- lombok
    
    Lombok이란 어노테이션 기반으로 자동으로 코드를 완성해주는 라이브러리이다.
    
    Lombok을 이용하면 Getter, Setter, Equals 등과 다양한 방면의 코드를 자동완성 시킬 수 있다.
    
- JsonProperty
    
    @JsonProperty 어노테이션은 객체를 JSON 형식으로 변환할 때 객체 이름을 설정할 수 있습니다.
    
    → ApiResponse에서 응답을 받을 때 자주 사용
    
    ```java
    @Getter
    @Setter
    @ToString
    public class UserDTO{
    		private Long userID;
    		private Stirng userName;
    		
    		@JsonProperty("memberPhoneNumber")
    		private String member_phone_number;
    		
    }
    ```
    
- JsonNaming
    
    ```java
    @Getter
    @Setter
    @ToString
    @JsonNaming("value = PropertyNamingStrategies.SnakeCaseCategory.class")
    public class UserDTO{
    		private Long userID;
    		private Stirng userName;
    		private String member_phone_number;
    		
    }
    ```
    
- JsonProperyOrder
    
    ```java
    @JsonPropertyOrder({"field3", "field1", "field2"})
    public class JsonPropertyOrderDummy {
    
        public String field1 = "one";
        public String field2 = "two";
        public String field3 = "three";
    
    }
    ```
    
    Json 결과의 출력 순서를 임의로 정하도록 만들어주는 어노테이션
    
- JsonInclude
    
    ```java
    @JsonInclude(Include.NON_EMPTY)
    @AllArgsConstructor
    public class JsonIncludeDummy {
        public String key1;
        public String key2;
        public String key3;
    }
    
    ```
    
    JsonInclude를 만족하는 상황에 대해서는 JSON 객체를 반환하지 않는다. 예를 들어 key1 = “value1”, key2 = null, key3 = “value3” 라면 결과는
    
    ```java
    {
      "key1": "value1",
      "key3": "value3"
    }
    ```
    
    가 된다.
    
    **역직렬화**
    
    - JSON (또는 바이트 스트림) → 객체
    - “역직렬화”는 JSON 데이터를 자바 객체로 복원하는 과정
    
    **직렬화**
    
    - 객체 → JSON (또는 바이트 스트림)
    - 클라이언트에 JSON을 응답으로 보낼 때, 서버의 도메인 객체나 DTO를 JSON으로 “직렬화” 합니다.

출처 [https://velog.io/@juhyeon1114/Java-유용한-Jackson-어노테이션-정리#4-jsoninclude](https://velog.io/@juhyeon1114/Java-%EC%9C%A0%EC%9A%A9%ED%95%9C-Jackson-%EC%96%B4%EB%85%B8%ED%85%8C%EC%9D%B4%EC%85%98-%EC%A0%95%EB%A6%AC#4-jsoninclude)

- HttpStatus
    
    스프링이 제공하는 **HTTP 상태 코드** 열거형(`org.springframework.http.HttpStatus`)
    
    | 분류 | 상태 코드 예시 | 의미 |
    | --- | --- | --- |
    | 1xx (정보) | 100 Continue | 요청을 받았으며 작업을 계속하라는 응답 |
    | 2xx (성공) | 200 OK / 201 Created | 성공적으로 요청이 처리됨 |
    | 3xx (리다이렉션) | 301 Moved Permanently | 리소스가 다른 URI로 이동됨 |
    | 4xx (클라이언트 오류) | 400 Bad Request / 404 Not Found | 잘못된 요청, 인증·인가 실패, 리소스 없음 등 |
    | 5xx (서버 오류) | 500 Internal Server Error / 503 Service Unavailable | 서버 내부 오류, 일시적 과부하 등 |
- ResponseEntity
    - **`org.springframework.http.ResponseEntity<T>`**
    - HTTP 응답 전체(상태 코드, 헤더, 바디)를 캡슐화하는 컨테이너
    - 주요 생성 방식
        
        ```java
        
        ResponseEntity.ok(body);                        // 200 OK + body
        ResponseEntity.status(HttpStatus.CREATED)
                      .header("Location", "/users/123")
                      .body(createdUserDto);            // 201 Created + 헤더 + body
        ```
        
    - **구성 요소**
        1. **상태 코드** (`HttpStatus`)
        2. **HTTP 헤더** (`HttpHeaders`)
            
            → 앞에서 소개한 HttpStatus를 바로 사용가능하다
            
        3. **응답 바디** (`T`)
- ResponseEnttiyHandler
    
    ResponseEnttiyHandler는 스프링이 제공하는 예외처리 추상 클래스
    
    ![image.png](attachment:ce7bb121-a579-4ed1-a47b-17ae8de1d8d5:image.png)
    
    1. **필요한 이유**
    - 스프링 MVC에서 HTTP 요청 바인딩이나 검증 과정 중에 발생하는 예외가 꽤 많습니다.
        - @Valid 검증 실패 → `MethodArgumentNotValidException`
        - 잘못된 HTTP 메시지 포맷 → `HttpMessageNotReadableException`
        - 파라미터 형 변환 실패 → `TypeMismatchException`
        - …외 다양한 예외들
    - 이 예외들을 **하나하나 @ExceptionHandler로 다 구현**하려면 반복 코드가 엄청 늘어난다.
    2. **기능**
    - 위에서 열거한 스프링 표준 예외들을 **미리 다루는(@ExceptionHandler 메서드가 구현된)** 클래스입니다.
    - **상속시,**
        1. **기본 핸들러**들이 이미 준비된 상태라서 바로 쓸 수 있고
        2. 내가 필요한 부분만 오버라이드해서(재정의) 메시지나 응답 구조를 바꿀 수 있습니다.
    
    ```java
    @RestControllerAdvice
    public class GlobalExceptionHandler 
         extends ResponseEntityExceptionHandler {
    
      // 예를 들어, @Valid 검증 실패 시
      @Override
      protected ResponseEntity<Object> 
          handleMethodArgumentNotValid(
             MethodArgumentNotValidException ex,
             HttpHeaders headers,
             HttpStatusCode status,
             WebRequest request) {
        // 기본 구현 대신 내가 원하는 형태로 리턴
      }
    
      // 나머지 예외는 모두 상위 클래스 기본 로직이 처리
    }
    ```
