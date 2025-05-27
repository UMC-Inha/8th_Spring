- java의 Exception 종류들
    
    java에서의 Exception : 에러(Error), 예외(Exception)로 구분
    
    | Error | JVM 내부에서 발생하는 심각한 오류 | 메모리 부족, StackOverflow… | 개발자가 직접 처리 불가 |
    | --- | --- | --- | --- |
    | Exception | 프로그램 로직상의 오류 | null 접근, 잘못된 형 변환… | 개발자가 try-catch로 처리 가능 |
    - Exception의 종류
        - 일반 예외(Exception) : 컴파일 체크 예외(Checked Exception)
            
            예외 처리 코드가 필수적
            
            컴파일 시 예외 처리 여부를 확인하며, 처리하지 않으면 컴파일 오류가 발생
            
            ex) IOException, SQLException
            
        - 실행 예외(RuntimeException) : 컴파일 시 예외 처리 코드가 필수적인지 검사하지 않는 예외(Unchecked Exception)
            
            예외 처리 코드가 없어도 컴파일은 가능하지만, 실행 중 예외가 발생할 수 있음
            
            ex) NullPointerException, IndexOutOfBoundsException
            
        - 예시
            - NullPointerException (NPE)
                
                ```java
                public class Example {
                    public static void main(String[] args) {
                        String name = null;
                        System.out.println(name.length()); // null 객체에 접근해서 NPE 발생
                    }
                }
                ```
                
            - ArrayIndexOutOfBoundsException
                
                ```java
                int[] arr = new int[3];
                System.out.println(arr[5]); // 배열 인덱스 초과 접근
                ```
                
            - NumberFormatException
                
                ```java
                String input = "abc123";
                int number = Integer.parseInt(input); // 숫자로 변환할 수 없음
                ```
                
            - ClassCastException
                
                ```java
                Animal a = new Cat();
                Dog d = (Dog) a; // Cat을 Dog로 잘못 형변환 → 런타임 오류
                ```
                
        - 예외 처리 방법
            
            ```java
            public class TryCatchExample {
                public static void main(String[] args) {
                    try {
                        String text = null;
                        System.out.println(text.length());
                    } catch (NullPointerException e) {
                        System.out.println("null 값 접근 에러 처리 완료");
                    }
                }
            }
            ```
            
- @Valid
    
    : 요청 데이터(예: JSON 객체)를 자바 객체로 매핑한 후, 객체 내부 필드가 정해진 조건을 만족하는지 자동으로 검사해주는 어노테이션
    
    - Spring에서는 LocalValidatorFactoryBean을 통해 검증을 처리
        
        이를 사용하려면 의존성 추가 후 빈으로 등록해야 함
        
        - Spring에서 동작 과정
            1. 요청 → `DispatcherServlet`으로 진입
            2. `@RequestBody` DTO 객체 생성
            3. `@Valid` → `LocalValidatorFactoryBean`이 유효성 검증 수행
            4. 오류 발생 시 `MethodArgumentNotValidException` 예외 발생
            5. Spring의 `ExceptionHandler`가 예외를 처리 → 400 Bad Request
    - 사용 예시
        - 컨트롤러
            
            ```java
            @PostMapping("/todo")
            public ResponseEntity<?> saveTodo(@RequestBody @Valid TodoRequest request) {
                return ResponseEntity.ok("유효성 통과!");
            }
            ```
            
        - DTO 클래스
            
            ```java
            @Getter
            @Setter
            public class TodoRequest {
                @NotNull(message = "제목은 필수입니다.")
                private String title;
            
                @NotEmpty(message = "내용을 입력해주세요.")
                private String content;
            
                @Min(1)
                @Max(7)
                private int dDay;
            }
            ```
            
    - `@Validated`
        
        : Spring에서 제공하는 어노테이션으로, 메소드 단위, 서비스 계층에서 유효성 검사를 활성화할 수 있음
        
        `@Valid`는 주로 컨트롤러에서만 사용되기 때문에, 서비스나 다른 곳에서 검증을 하고 싶다면 `@Validated`를 사용!
        
        - 동작 원리 : AOP를 사용하여 메소드 호출 시 요청을 가로채고, 유효성 검증을 수행
            
            인터셉터(MethodValidationInterceptor)를 사용하여 메소드의 요청을 처리하고 검증을 진행
            
            ⇒ Controller, Service, Repository 등 모든 계층에서 유효성 검증을 수행 가능
            
        - 예외 처리 : @Valid에서 발생한 예외는 MethodArgumentNotValidException으로, Spring Boot에서 자동으로 처리
            
            @Validated에서는 ConstraintViolationException이 발생할 수 있음
            
        - 사용 예시
            
            ```java
            @Service
            @Validated
            public class TodoService {
                public void saveTodo(@Valid TodoRequest request) {
                    // 유효성 통과된 request만 처리
                }
            }
            ```