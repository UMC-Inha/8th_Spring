# 키워드 미션

## Java의 Exception 종류들

### Exception이란

- 프로그램 실행 도중 발생할 수 있는 예외상황을 처리하기 위한 구조이다.
- 모든 예외는 Throwable을 상속하며, 크게 두가지로 나뉜다.
- Checked Exception (컴파일러 체크), Unchecked Exception ( = Runtime Exception, 실행 중 오류)

### Checked Exception (컴파일러가 체크)

- **`IOException`  : 입출력 작업 중 오류가 발생했을 때 (파일, 네트워크 등)**
- **`SQLException` : 데이터베이스 접근 중 오류가 발생했을 때**
- **`ParseException` : 문자열을 날짜/숫자 등으로 변환할 때 실패했을 때**
- try-catch 또는 throws로 반드시 처리가 필요하다.
- 외부 리소스 관련 예외에서 자주 사용된다.

### Unchecked Exception (컴파일러가 강제하지 않음)

- **`NullPointerException` : 객체가 null인데 메서드나 필드를 참조할 때**
- **`IllegalArgumentException` : 메서드에 유효하지 않은 인자가 전달됐을 때**
- **`IndexOutOfBoundsException` : 배열, 리스트 등에서 잘못된 인덱스를 참조할 때**
- 대부분의 비즈니즈 로직 오류는 이쪽에서 발생한다.
- 런타임 중에 발생할 수 있다.

### Throwable내에 Exception말고 상속받는 Error

- 사용자가 직접 처리하면 안되고, JVM 이나 시스템 수준의 오류이다.
- 직접 처리하지 않고 로그만 남기고 종료하는 것이 일반적이다.
- OutOfMemoryError, StackOverflowError

! 7주차 내용을 복습하자면, RestControllerAdvice를 통해서, 커스텀 예외를 만들고, 예외를 처리할 수 있도록 한다.

## @Valid - 입력값 검증

- Spring에서 요청 객체의 유효성 검사를 수행하기 위한 어노테이션이다.
- @Valid가 적용된 파라미터를 검증할 때 해당 dto에 선언한 조건과 맞지 않을 경우 에러가 발생한다.

### 유효성 어노테이션 종류들

| 애너테이션 | 설명 |
| --- | --- |
| `@NotNull` | null 이면 안 됨 |
| `@NotBlank` | 공백 + null 허용 안 함 (문자열 전용) |
| `@NotEmpty` | null 또는 빈 컬렉션/문자열 허용 안 함 |
| `@Email` | 이메일 형식인지 검사 |
| `@Min(value)` | 최소값 제한 |
| `@Max(value)` | 최대값 제한 |
| `@Size(min, max)` | 문자열/컬렉션 길이 제한 |
| `@Pattern(regexp=...)` | 정규식 매칭 |

### 사용예시

**`UserDto.java`**

```
@Getter
@Setter
public class UserDto {
    @NotBlank
    private String name;

    @Email
    @NotBlank
    private String email;
}

```

**`UserController.java`**

```
@PostMapping("/users")
public ResponseEntity<?> createUser(@RequestBody @Valid UserDto dto) {
    // 유효성 검사 통과된 경우만 여기까지 옴
    return ResponseEntity.ok(userService.createUser(dto));
}
```

→ dto 내에 notblank 와 같은 조건이 맞지 않을 경우에는 Spring이 자동으로 **`MethodArgumentNotValidException`**을 발생시킨다.

그리고 이를 처리하기 위해서는 **`@RestControllerAdvice`**를 활용한다.

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<String> handleValidationException(MethodArgumentNotValidException ex) {
        String error = ex.getBindingResult().getFieldError().getDefaultMessage();
        return ResponseEntity.badRequest().body("입력 오류: " + error);
    }
}
```

## 추가 - @Validated vs @Valid

**`@Valid`**는 **`javax.validation`** 기반이고,

**`@Validated`**는 Spring의 **`org.springframework.validation.annotation.Validated`** 로, 그룹 검증 단일 패러미터 검증 등 더 고급 기능을 지원한다.

# 피드백 반영 학습

## 예외처리 흐름 정리

`ExceptionAdvice.java`

```
    @ExceptionHandler(value = GeneralException.class) // 커스텀 예외는 이 친구를 상속 또는 구현해서 -> 예외에 걸리도록 함.
    public ResponseEntity onThrowException(GeneralException generalException, HttpServletRequest request) {
        ErrorReasonDto errorReasonHttpStatus = generalException.getErrorReasonHttpStatus();
        return handleExceptionInternal(generalException,errorReasonHttpStatus,null,request);
    }
```

`@ExceptionHandler(value = GeneralException.class)` 의미

- GeneralException 또는 그 자식 클래스가 던져졌을 때 해당 메서드가 호출된다는 뜻이다.
- 즉 GeneralException을 상속받는 모든 커스텀 예외들까지 포괄적으로 처리할 수 있다.

### Spring 작동 방식

- 컨트롤러나 서비스에서 예외 발생 → @RestControllerAdvice 클래스 검색 →
- @ExceptipnHandler또는, ResponseEntityExceptionHandler에서 적절한 메서드 실행 ->
- 공통 포맷으로 ResponseEntity 리턴

```
컨트롤러 예외 (GeneralException) 발생 → ExceptionAdvice 호출 & onThrowException 메서드 실행 → 에러 응답 객체 생성 및 반환
```

## @ExceptionHandler vs @RestControllerAdvice

### @ExceptionHandler (in Controller)

- 적용범위 : 해당 컨트롤러 클래스 안에서만 적용되고, 사용된다.
- 특정 컨트롤러 적용 예외처리를 위해 사용된다.
- 특수한 예외에만 사용한다.

### 예시

```
@RestController
@RequestMapping("/api/user")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        throw new UserNotFoundException("사용자를 찾을 수 없습니다");
    }

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

### @RestControllerAdvice

- 적용범위 : 전체 컨트롤러에 적용된다. (글로벌)
- 전 어플리케이션 공통 처리의 목적으로 사용된다.
- 별도의 클래스에 해당 어노테이션을 선언한다.
- 공통 응답 포맷용으로 많이 사용된다.

### 보통 많이 사용하는 전략

- 전역 Advice + 공통 ErrorResponse + ErrorCode Enum 구조를 사용한다.
- 특수 경우에만 개별 컨트롤러에 @ExceptionHandler를 추가해서 override해주는 전략

## Json 관련 어노테이션

Jackson은 JSON 직렬화를 담당하는 라이브러리로 해당 어노테이션들이 핵심으로 사용된다.

| 어노테이션 | 설명 |
| --- | --- |
| `@JsonProperty("name")` | JSON 필드 이름을 지정 |
| `@JsonInclude(Include.NON_NULL)` | null인 필드는 제외하고 응답 |
| `@JsonIgnore` | 해당 필드를 응답에서 제외 |

```
@Getter
@AllArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL) -> null인 필드 제외하고 응답
public class ErrorResponse {

    @JsonProperty("error_code") -> json 필드 이름을 error_code로 설정 
    private String code;

    private String message;

    @JsonIgnore
    private String internalLog;
}
```