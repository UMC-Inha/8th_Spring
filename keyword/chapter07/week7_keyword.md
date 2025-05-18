# Keyword

## RestContollerAdvice

  모든 예외처리를 try-catch 로 처리하기엔 불필요한 중복 코드들이 많고 가독성이 하락하여 사용

  ControllerAdvice

- @ExceptionHandler / @ModelAttribute / @InitBinder 가 적용된 메서드들에 AOP 적용하여 Controller 단에 적용하기 위해 고안된 어노테이션
- 클래스에 선언됨
- 모든 @Controller에 대한 전역적으로 발생하는 예외 잡아서 처리

RestControllerAdvice

  - ControllerAdvice + ResponseBody
  - 즉, 예외처리 후 응답으로 객체를 리턴하고 싶을 때 사용

  ExceptionHandler

  - 메서드에 선언
  - 특정 예외 클래스를 지정하면 해당 예외 발생 시 메서드에 정의한 로직으로 처리

## lombok

- 중복되는 코드를 줄여주는 라이브러리
  - 가독성 및 유지보수성 향상
- 즉, 어노테이션 기반으로 코드를 자동완성 해주는 라이브러리
    - 생산성 향상

## @Json 어노테이션들

  **JsonProperty**

- 아래와 같이 사용
- 필드와 Json 키 값을 연결해줌 (지정해준다)

    ```jsx
    @JsonProperty("한국어")
    private String korean;
    ```

**JsonPropertyOrder**

- 아래와 같이 사용
- 순서대로 필드명과 Json 키 값을 지정해준다

  ```jsx
  @JsonPropertyOrder({"isSuccess", "code", "message", "result"})
  public class ApiResponse<T> {
    
      @JsonProperty("isSuccess")
      private final Boolean isSuccess;
      private final String code;
      private final String message;
      @JsonInclude(JsonInclude.Include.NON_NULL)
      private T result;
  ```

**JsonInclude**

  - 객체를 Json 으로 변환할 때 변수에 @JsonIgnore 을 통해 Json 에 포함되지 않도록 할 수 있다.
  - 상황에 따라 달라질 경우 해당 어노테이션의 속성들을 사용할 수 있다.
  - ALWAYS
      - 모든 데이터를 JSON 으로 변환
  - NON_NULL
      - NULL 인 데이터 제외
  - NON_ABSENT
      - NULL 인 데이터 제외
      - 참조 유형의 absent 값은 제외
  - NON_EMPTY
      - NULL, absent, isEmpty()가 true, length가 0 인 데이터 제외
  - NON_DEFAULT
      - empty인 데이터, 기본형 타입이 default 인 데이터, Date의 timestamp 가 0L 인 데이터 제외