- RestContollerAdvice
    
    : Spring에서 REST API 예외를 전역적으로 처리할 수 있도록 도와주는 어노테이션
    
    API 개발 시 모든 컨트롤러에서 발생할 수 있는 예외를 한 곳에서 처리할 수 있기 때문에 유지보수가 편하고, 에러 응답 형식도 일관되게 제공 가능
    
    - **전역 예외 처리** : 특정 컨트롤러에 종속되지 않고, 특정 타입의 예외가 발생했을 때 어떤 메서드를 실행하여 처리할지 정의
    - 일관된 JSON 응답 : 응답 형식이 JSON으로 고정되므로 RESTful 서비스에 적합하며, API 사용자에게 일관된 에러 형식을 제공 가능
    - 핸들러 메서드: @ExceptionHandler를 통해 예외 유형별로 메서드를 정의하여, 상황에 맞는 에러 메시지와 HTTP 상태 코드를 설정 가능
    
    ```java
    @RestControllerAdvice
    public class GlobalExceptionHandler {
    
        // 사용자 정의 예외 처리
        @ExceptionHandler(CustomException.class)
        public ResponseEntity<ErrorResponse> handleCustomException(CustomException ex) {
    		    // CustomException에서 정의된 에러 코드를 사용하여 응답 생성
            return buildErrorResponse(ex.getErrorCode());
        }
    
        // 그 외 모든 예외 처리
        @ExceptionHandler(Exception.class)
        public ResponseEntity<ErrorResponse> handleGeneralException(Exception ex) {
            return buildErrorResponse(CommonErrorCode.INTERNAL_SERVER_ERROR);
        }
    
        // 공통 응답 생성 메서드
        private ResponseEntity<ErrorResponse> buildErrorResponse(ErrorCode errorCode) {
            return ResponseEntity.status(errorCode.getHttpStatus())
                    .body(new ErrorResponse(errorCode.name(), errorCode.getMessage()));
        }
    }
    ```
    
    - 주의할 점
        - 예외가 복잡해지면 여러 @RestControllerAdvice 클래스로 예외 처리를 모듈화하여 관리하는 것이 좋음
        - 예상하지 못한 예외에 대비해 최상위 예외 핸들러를 정의하여, 모든 에러가 적절한 에러 메시지로 응답될 수 있도록 하기
        - 클라이언트가 이해할 수 있도록 **에러 메시지 통일** 필요
- lombok
    
    : Java에서 자주 사용하는 코드를 자동으로 만들어주는 **라이브러리**
    
    클래스를 짧고 간결하게 유지할 수 있어서 생산성과 가독성이 높아짐
    
    - 주로 사용하는 어노테이션
        - `@Getter, @Setter` : 클래스나 필드에 적용하여 getter, setter 메서드를 자동 생성
        - `@AllArgsConstructor, @NoArgsConstructor` : 모든 필드를 매개변수로 받는 생성자나 기본 생성자를 자동 생성
        - `@Builder` : 빌더 패턴으로 객체 생성
        - `@ToString` : toString() 메서드를 자동 생성하여 객체의 정보를 문자열로 반환
            
            객체를 호출할 때 나올 메세지를 설정 가능
            
        
        ```java
        @Getter
        @Setter
        @AllArgsConstructor
        @NoArgsConstructor
        @ToString
        @Builder
        public class User {
            private Long id;
            private String name;
            private String email;
            ...
            @Builder.Default
            @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
            private List<UserMission> useerMissionList = new ArrayList<>();
            ...
        }
        ```
        
        ```json
        // NoArgsConstructor
            public User() {}
        
            // AllArgsConstructor
            public User(Long id, String name, String email) {
                this.id = id;
                this.name = name;
                this.email = email;
            }
        
            // Getter for id
            public Long getId() {
                return id;
            }
            ...
        ```
        
    - 주의할 점
        - 디버깅의 어려움 : 자동 생성된 코드가 보이지 않기 때문에 디버깅 시 생성된 메서드의 호출 흐름을 추적하기 어려움
        - 의존성 : Lombok은 컴파일 타임에 동작하므로, 프로젝트의 빌드 도구와의 호환성 문제에 주의가 필요

- Response Entity
    
    : Spring에서 HTTP 응답 전체를 직접 제어할 수 있게 해주는 객체
    
    - HttpEntity : HTTP 요청 또는 응답에 해당하는 HttpHeader와 HttpBody를 포함하는 클래스
        
        ⇒ ResponseEntity는 사용자의 HttpRequest에 대한 응답 데이터를 포함하는 클래 
        
    - 요소
        - **HTTP 상태 코드** (ex. 200 OK, 404 Not Found)
        - **HTTP 헤더** (선택)
        - **응답 바디** (body, 실제 클라이언트에게 보낼 데이터)
    - 사용 예시
        
        ```java
        @Getter
        @AllArgsConstructor
        @JsonPropertyOrder({"isSuccess", "code", "message", "result"})
        public class ApiResponse<T> {
        
            @JsonProperty("isSuccess")
            private final Boolean isSuccess;
            private final String code;
            private final String message;
            @JsonInclude(JsonInclude.Include.NON_NULL)
            private T result;
        }
        ```
        
        ```java
        @GetMapping("/user/{id}")
        public ResponseEntity<ApiResponse<UserDto>> getUser(@PathVariable Long id) {
            UserDto user = userService.findById(id);
            ApiResponse<UserDto> response = new ApiResponse<>(true, "조회 성공", user);
            return ResponseEntity.ok(response);
        }
        ```
        
    - 특징
        - 응답 형식을 일관되게 유지 가능
        - 상태 코드 제어가 자유로움
        - 에러 핸들링에 유연하게 사용 가능
            
            `@RestControllerAdvice`와 함께 활용