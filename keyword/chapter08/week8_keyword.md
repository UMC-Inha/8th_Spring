**java의 Exception 종류들**

Java Exception Layer

    java.lang.Throwable
     ├─ java.lang.Error           (시스템·VM 오류, 애플리케이션에서 잡지 않음)
     └─ java.lang.Exception       (애플리케이션 레벨)
           ├─ java.lang.RuntimeException   (Unchecked, 런타임 예외)
           └─ [나머지]                       (Checked, 컴파일러가 처리 요구)
    

Error (catch 하지 않음)

- OutOfMemoryError 힙 메모리 고갈
- StackOverflowError 스택 초과 (무한 재귀 등)
- NoClassDefFoundError 클래스 로딩 실패
- InternalError JVM 내부 오류

Checked Exception (컴파일러가 반드시 처리를 요구하는 예외)

- IOException 파일 | 스트림 입출력 실패
- SQLException DB 접근 오류
- ParseException 날짜 | 숫자 파싱 실패
- SAXException | ParserConfigurationException XML 처리 오류
- MalformedURLException | SocketException URL 형식 | 소켓 통신 오류
- 등등

Unchecked Exception (명시적 처리 선택 사항 | 런타임 예외)

- NullPointerException | IndexOutOfBoundsException | NoSuchElementException
    - 널 참조, 인덱스 범위 초과
- ArithmeticException | NumberFormatException 0으로 나눔, 파싱 실패, 잘못된 캐스팅 등
- IllegalArgumentException 메서드에 부적합한 인수 | 상태 등

**@Valid**

빈 검증기 (Bean Validator)를 이용해 객체의 제약 조건을 검증하도록 지시하는 어노테이션

Spring에서는 LocalValidatorFactoryBean이 제약 조건 검증을 처리한다.

위를 이용하려면 LocalValidatorFactoryBean을 빈으로 등록해야하는데 의존성을 추가하면 자동 설정이 된다.

    @PostMapping("/test")
        public void validTest(@RequestBody @Valid DTO.ReqeustDto request) {
            testService.validTestMethod(request);
        }

- 제약 조건
  - @NotNull : null값 불가
  - @NotEmpty : null, “” 불가
  - @NotBlank : null, “”, “ “ 불가
  - @Min : 최소값 설정
  - @Max : 최대값 설정
  - @Pattern : 해당 필드가 특정 형태 가지도록 검증 (정규표현식)
    - ex : “^[0-9-]+$”
    - ^ : 문자열의 시작 의미
    - $ : 문자열의 끝을 의미
    - [ ] : 내부에 패턴 입력

 ![image](https://github.com/user-attachments/assets/dbbd8eeb-d847-468e-a5c2-8d921d7f2a9b)


  - @Email : 이메일 형식
  - @Size : 문자열의 최소, 최대 크기 검증 (@Size(min= , max= ) 이런식)
- 동작 원리
  - 모든 요청은 프론트 컨트롤러인 **디스패처 서블릿**을 통해 컨트롤러로 전달됨
  - 전달 과정에서 컨트롤러 메소드의 객체를 만들어주는 ArgumentResolver 동작 → @Valid도 이 리졸버에 의해 처리됨
    - 대표적으로 @RequestBody는 Json 메세지를 객체로 변환해주는 작업이 위 리졸버의 구현체인 RequestResponseBodyMethodProcessor가 처리 → 이 내부에서 @Valid 로 시작하는 어노테이션 있을 경우 유효성 검사 진행 (커스텀 어노테이션도 포함)
  - @ModelAttribute 사용하면 ModelAttributeMethodProcessor에 의해 @Valid 처리
  - 검증에 오류 발생 시 MethodArgumentNotValidException 예외 발생
    - 디스패처 서블릿에 기본으로 등록된 예외 리졸버인 DefaultHandlerExceptionResolver에 의해 400 에러 발생
- 즉, @Valid는 기본적으로 컨트롤러에서만 동작, 다른 계층에선 검증이 안됨
  - 다른 계층에서 검증하기 위해선 @Validated와 결합되어야 함
