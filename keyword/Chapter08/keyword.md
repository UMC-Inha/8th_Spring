## - 키워드 미션

#### - java의 Exception 종류들

- **RuntimeException**

    - NullPointerException, IllegalArgumentException 등

- **Exception**(Checked 예외)

    - I/O Exception, SQLException, ClassNotFoundException 등

- **Spring 프레임워크 관련 예외**

    - BeansException: 스프링 빈 처리의 슈퍼클래스

    - ApplicationContextException: 애플리케이션 콘텍스트 초기화 실패 예외

- **MVC/Web 관련 예외**

    - HttpMessageNotReadableException: 요청 JSON/XML 파싱 실패

    - HttpMessageNotWritableException: 응답 직렬화 실패

    - MethodArgumentNotValidException: @Valid 검증 실패시 발생

- **Data/DAO 관련 예외**

    - DuplicateKeyException: 고유 키 제약 위반 예외

    - DataIntegrityViolationException: FK 체크 제약 조건 위반 예외

    - TransactionException: 트랜잭션 시작, 커밋 오류 예외

- **Spring Security 예외**

    - AuthenticationException: 인증 실패 예외

    - AccessDeniedException: 인가 실패 예외

#### - @Valid

- 입력값 검증을 자동으로 해주는 어노테이션

- **동작 원리**

    - @Valid가 붙은 객체나 파라미터를 처리할 때, DTO나 엔티티에 선언된 @NotNull, @Size와 같은 어노테이션을 검사한다.

    - 하나라도 위반되면 **MethodArgumentNotValidException** 혹은 **COnstraintViolationException** 예외를 발생시킨다.

    - @RequestBody 이후에 자동으로 검증되어 HTTP 400 응답으로 처리해준다.

- **사용 예시**

  ``` java
  public ApiResponse<MemberResponseDTO.JoinResultDTO> join(@RequestBody @Valid MemberRequestDTO.JoinDto request)
  ```

- @ControllerAdvice를 사용해 에러 메시지 형식을 통일할 수 있다.

- **@Validated**와 함께 검증 그룹을 지정해서 상황별로 다른 검증 규칙을 적용할 수 있다.

- **@Valid vs @Validated**

    - @Valid

        - DTO 필드의 제약 조건을 검사한다.

        - 검증 그룹을 지원하지 않는다.

    - @Validated

        - 스프링 전용 어노테이션이다.

        - 클래스나 메서드에 붙여서 메서드 인자 전체를 검증하는 **AOP 기반 방식**이다.

        - 검증 그룹 지정 기능을 지원한다.

        - 컨트롤러 뿐만 아니라 **서비스 빈에도 적용이 가능하다.**

<br>