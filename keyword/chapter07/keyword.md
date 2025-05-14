에러, 성공 처리 부분의 전체 흐름을 이해하기가 어려워 추가로 정리함

## 워크북 주요 내용

---

API 응답 통일

- API 응답 형식을 일관되게 만들어 클라이언트와 서버 간 소통을 수월하게 하기 위함
- 성공/실패 여부와 상세 정보를 포함해 표준화된 커스텀 응답을 제공하기 위함

에러 핸들러

- 서버에서 발생하는 다양한 예외를 처리해 클라이언트에게 명확한 에러 정보를 전달해줌
- 개발자와 클라이언트 모두 문제를 빠르게 파악할 수 있게 하기 위함

**뒤에서 소개하는 클래스들은 모두 프로젝트에서 사용된 클래스를 포함**

### API 응답 통일 관련 클래스

- 클라이언트가 항상 동일한 형태의 응답 구조를 받도록 구현

---

**ApiResponse<T>**



- **역할**: 모든 API 응답에 공통으로 포함될 필드 정의
- **구성**
    - `isSuccess`(Boolean): 성공 여부 표시함
    - `code`(String): 내부 코드(ex. “COMMON200”)
    - `message`(String): 사용자에게 보여줄 메시지
    - `result`(T): 실제 데이터(payload)

```jsx
// 성공 응답 생성
return ApiResponse.onSuccess(data);
// 실패 응답 생성
return ApiResponse.onFailure(errorStatus);
```

**BaseCode**

- **역할**: Status enum들이 공통으로 구현해야 할 인터페이스
- **추상 메서드**
    - `getReason()` → ReasonDTO 반환
    - `getReasonHttpStatus()` → HTTP 상태 포함 반환
- **효과**: **enum**마다 중복 코드 줄여줌

**SuccessStatus**

- **역할**: 성공 응답에 사용될 상태값 모아둔 enum 클래스
    - 위에 첨부했던 사진 속 응답 필드들을 enum 값으로 관리 (나중에 성공 응답 시에 사용)
    - BaseCode의 구현체
- **구성 예시**

    ```jsx
    _OK(200, "COMMON200", "요청 성공");
    ```

- **필드**
    - `httpStatus`(int): HTTP 상태 코드
    - `code`(String): 내부 코드
    - `message`(String): 설명 메시지


**ReasonDTO**

- **역할**: 응답의 “이유”와 관련된 정보만 골라서 담아 주는 DTO
- **구성**
    - `httpStatus`(int)
    - `isSuccess`(Boolean)
    - `code`(String)
    - `message`(String)
- **사용처**: ApiResponse 생성 시 정보 매핑용
- 현재 프로젝트 코드에서 활용하는 모습이 없지만 다음과 같이 활용 가능

```java
@Getter
@AllArgsConstructor
@JsonPropertyOrder({"isSuccess", "reason", "result"})
public class ApiResponse<T> {
    private final Boolean isSuccess;
    private final ReasonDTO reason;    // DTO 객체를 직접 필드로 추가
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private T result;

    // 성공 응답 생성
    public static <T> ApiResponseV2<T> onSuccess(T result) {
        ReasonDTO reason = SuccessStatus._OK.getReason(); 
        return new ApiResponseV2<>(true, reason, result);
    }

    // 실패 응답 생성
    public static <T> ApiResponseV2<T> onFailure(BaseCode errorCode) {
        ReasonDTO reason = errorCode.getReasonHttpStatus();
        return new ApiResponseV2<>(false, reason, null);
    }
}

```

- `getReason` 메소드를 통해 미리 선언해둔 enum 클래스를 ReasonDTO로 반환받기

### 성공 응답 흐름 정리

---

1. 컨트롤러에서 `ApiResponse.onSuccess(TempConverter.*toTempTestDTO*())` 호출
2. 내부에서 `SuccessStatus` enum(예: `_OK`)객체 필드의 `getter` 메소드를 호출
3. `isSuccess=true`, `code`, `message`과 함께 전달된 `data`를 묶어 `ApiResponse` 인스턴스 생성

    ```java
    public static <T> ApiResponse<T> onSuccess(T result){
            return new ApiResponse<>(
    		        true, 
    		        SuccessStatus._OK.getCode() ,
    		        SuccessStatus._OK.getMessage(), result //result는 TempTestDTO
    		        );
    		        //현재 인자로 들어가있는 값들이 그대로 응답 json을 구성하는 것
    		       
        }
    ```


- 생성된 `ApiResponse` 객체는 HTTP 응답 본문으로 반환돼, 항상 일관된 구조 유지

    ```json
    json
    복사편집
    {
      "isSuccess": true,
      "code": "COMMON200",
      "message": "요청 성공",
      "result": { /* DTO 객체의 필드가 json으로 변환되어 반환 */ }
    }
    
    ```


### 에러 핸들러 관련 클래스

- 예외 발생 시 일관된 형태의 에러 응답을 반환하도록 처리

---

**ErrorStatus**

- **역할**: 에러 상황별 상태값 모아둔 enum
- **구성 예시**

    ```jsx
    MEMBER_NOT_FOUND(400, "MEMBER4001", "사용자 없음");
    ```

- **필드**
    - `httpStatus`(int)
    - `code`(String)
    - `message`(String)
- **사용처**
    - 에러 상황 발생 지점에서 어떤 종류의 에러인지 식별
    - 그에 맞는 `ErrorReasonDTO` 생성용 정보 제공

**ErrorReasonDTO**

- **역할**: 에러 응답에 포함될 이유 정보 DTO
- **구성**
    - `httpStatus`, `isSuccess(false)`, `code`, `message`
- **비고**: ReasonDTO와 구조 유사하나, 실패 쪽에 최적화됨

**GeneralException**

- **역할**: 커스텀 예외 클래스
    - 코드 전반에서 예외 상황을 throw할 때 쓰는 커스텀 런타임 예외
- **특징**
    - `ErrorStatus` 필드 보유함
    - 생성자에서 ErrorStatus 지정 가능
    - `getReason()` 메서드로 ErrorReasonDTO 반환


**ExceptionAdvice**

- **역할**: `@RestControllerAdvice` 기반 예외 처리기
- **핸들링 대상**
    - `MethodArgumentNotValidException` (유효성 검사 실패)
    - `GeneralException` (커스텀 예외)
    - 기타 `Exception` (기본 예외)
- **처리 흐름**
    1. 예외 유형별 핸들러 메서드 실행
    2. ErrorStatus → ErrorReasonDTO 변환
    3. ApiResponse<ErrorReasonDTO> 형태로 응답 반환


### 에러 처리 흐름 정리

---

- 아래 순서대로 에러 처리 매커니즘이 동작

1. **서비스 레이어에서 예외 상황 발생** (TempQueryServiceImpl)
    - 비즈니스 로직 실행 중 검증 실패 시 도메인별 예외 핸들러 호출

    ```jsx
    @Service
    @RequiredArgsConstructor
    public class TempQueryServiceImpl implements TempQueryService {
        @Override
        public void CheckFlag(Integer flag) {
            // 비즈니스 로직 검증 수행
            if (flag == 0)
                throw new TempHandler(ErrorStatus.TEMP_EXCEPTION);
        }
    }
    ```

2. **도메인별 예외 핸들러 호출** (TempHandler)
    - 특정 도메인(기능 영역)에 대한 예외 처리 담당
    - 인자로 `BaseErrorCode` 의 구현체인 `ErrorStatus.TEMP_EXCEPTION` 파라미터로 전달받음
    - 에러 코드와 함께 부모 클래스인 GeneralException으로 예외 위임

    ```jsx
    public class TempHandler extends GeneralException {
        public TempHandler(BaseErrorCode errorCode) {
            super(errorCode);
        }
    }
    ```

3. **위임받은 공통 예외 처리 클래스 실행** (GeneralException)
    - 마찬가지로, 인자로 `BaseErrorCode` 의 구현체인 `ErrorStatus.TEMP_EXCEPTION` 파라미터로 전달받음
    - 에러 코드, 메시지, HTTP 상태 코드 등의 정보 관리
    - GeneralException에서 생성자로 super() 호출 관례
        - 내부에 담고 있는 `ErrorStatus` 메시지를 Java 예외 시스템에도 동일하게 등록하기 위함
        - 이를 통해 예외 메시지는 스택 트레이스·로그·디버깅 도구 등에 자연스럽게 적용돼서

          로그 쉽게 확인 가능


    ```jsx
    @Getter
    public class GeneralException extends RuntimeException {
        private BaseErrorCode code;
    
        public GeneralException(BaseErrorCode code) {
            super(code.getReason().getMessage());
            this.code = code;
        }
    
        public ErrorReasonDTO getErrorReason() {
            return this.code.getReason();
        }
        
        public ErrorReasonDTO getErrorReasonHttpStatus() {
            return this.code.getReasonHttpStatus();
        }
    }
    ```

4. **에러 상태 및 코드 정의** (ErrorStatus의 getReason 메소드 호출)
    - 각 에러 상황에 대한 코드, 메시지, HTTP 상태 코드 정의
    - 위에서 살펴보았던 성공 응답과 마찬가지로 에러 응답을 일관된 형태로 만들기 위함
    - getReason() 및 getReasonHttpStatus() 메소드로 에러 정보 DTO 생성할 수 있음

    ```jsx
    @Getter
    @RequiredArgsConstructor
    public enum ErrorStatus implements BaseErrorCode {
        // 일반 에러
        _INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "COMMON500", "서버 에러, 관리자에게 문의 바랍니다."),
        _BAD_REQUEST(HttpStatus.BAD_REQUEST, "COMMON400", "잘못된 요청입니다."),
        
        // 특정 도메인 에러 (예: TEMP)
        TEMP_EXCEPTION(HttpStatus.BAD_REQUEST, "TEMP4001", "이러이러한 이유로 에러가 발생했습니다."),
        
        // 다른 도메인 에러들...
        ;
    
        private final HttpStatus httpStatus;
        private final String code;
        private final String message;
    
        @Override
        public ErrorReasonDTO getReason() {
            return ErrorReasonDTO.builder()
                    .message(message)
                    .code(code)
                    .isSuccess(false)
                    .build();
        }
    
        @Override
        public ErrorReasonDTO getReasonHttpStatus() {
            return ErrorReasonDTO.builder()
                    .message(message)
                    .code(code)
                    .isSuccess(false)
                    .httpStatus(httpStatus)
                    .build();
        }
    }
    ```


1. **여기서 예외를 감지 → ExceptionAdvice 이동 (`@RestControllerAdvice` 선언된)**
    - 결과적으로 `GeneralException` 이 해당 선언된 메소드 인자로 들어가게 된다.

    ```java
    @ExceptionHandler(value = GeneralException.class)
    //GeneralException.class 관련 예외 처리 메소드임을 나타냄
    public ResponseEntity onThrowException(GeneralException generalException,
                                           HttpServletRequest request) 
    {
        ErrorReasonDTO errorReasonHttpStatus = generalException
    																				   .getErrorReasonHttpStatus();
        return handleExceptionInternal(
            generalException,
            errorReasonHttpStatus,
            null,
            request
        );
    }
    ```

    - **내부에서 ErrorReasonDTO 추출**

        ```java
        
        ErrorReasonDTO reason = ex.getErrorReasonHttpStatus();
        return handleExceptionInternal(ex, reason, ...);
        
        ```


    → 일관된 에러 응답(JSON)으로 클라이언트에 최종 에러 응답 전송
    
    ```json
    {
      "isSuccess": false,
      "code": "TEMP4001",
      "message": "이러이러한 이유로 에러가 발생했습니다.",
      "result": null
    }
    
    ```


쉽게 비유를 들자면 다음과 같은 흐름으로 이해할 수 있다.

1. **ExceptionAdvice (대법원)**
    - 모든 예외의 최종 처리자
    - 표준화된 응답 형식으로 클라이언트에게 전달
    - 애플리케이션 전체에 단 하나만 존재

2. **GeneralException (법무부)**
    - 모든 예외 유형의 기본 프레임워크
    - 에러 코드와 메시지를 체계적으로 관리
    - 모든 도메인별 예외의 상위 클래스

3. **TempHandler 등 도메인(Member, Mission)별 핸들러 (전문 부처)**
    - 특정 비즈니스 영역에 특화된 예외 처리
    - 해당 도메인의 예외 상황 정의 및 처리

**⇒** 만약 대법원, 범무부, 전문 부처가 없다면 민원이나 사건 발생 시 **‘누구한테 처리를 맡겨야 할 지’**

**‘어떤 방식으로 처리해야 할 지‘** 등과 같은 어려움이 존재하기 때문에 필수적이라고 할 수 있다.

⇒ 또한, 에러 발생 시 해당 메서드 내에서 잡지 않고 상위로 예외를 전파하는 것은

    사건 발생 후 그 사건을 상급 법원인 **대법원**으로 이송해서 처리하는 것과 똑같다.

### 에러 vs 성공 응답 처리 흐름 차이 요약

---

### 성공 흐름

- **정상적인 코드 실행 경로**로 진행
- 컨트롤러 → 서비스 → 리포지토리 → 서비스 → 컨트롤러로 순차적 실행
- 마지막에 컨트롤러가 결과를 래핑하여 반환

### 에러 흐름

- **예외 발생 시 정상 경로가 중단**되고 예외 경로로 전환
- 발생한 예외는 호출 스택을 거슬러 올라감
- 일반 코드 흐름이 아닌 **예외 처리 경로**로 전달
- `@ExceptionHandler`가 전역적으로 예외를 잡아서 처리

⇒ 이렇게 다른 흐름이 있기에 TempRestController에서 `/temp/exception` 에 대해 성공, 실패 응답 구조가 다르게 나올 수 있는 것

```java
@GetMapping("/exception")
    public ApiResponse<TempExceptionDTO> exceptionAPI(@RequestParam Integer flag) {
        tempQueryService.CheckFlag(flag);
        return ApiResponse.onSuccess(TempConverter.toTempExceptionDTO(flag));
    }
```

- 위 코드에서 `CheckFlag(flag)` 가 에러를 반환할 시
    - 프로그램이 기존 코드 흐름에서 벗어나 에러 처리 매커니즘을 따라 실행된다.
    - `ExceptionAdvice` 에서 ResponseEntity로 래핑하여 반환

    ```java
    @RestControllerAdvice
    public class ExceptionAdvice {
        @ExceptionHandler(GeneralException.class)
        public ResponseEntity<ApiResponse> handleException(GeneralException e) {
            // 예외 정보 추출
            ApiResponse errorResponse = ApiResponse.onFailure(
                e.getErrorReason().getCode(),
                e.getErrorReason().getMessage(),
                null
            );
            
            // 여기서 ResponseEntity로 래핑하여 반환
            return new ResponseEntity<>(errorResponse, e.getErrorReason().getHttpStatus());
        }
    }
    
    ```

- 성공할 시
    - 직접 `ApiResponse.onSuccess(result)` 호출

- **성공: 컨트롤러 → ApiResponse → 클라이언트**
- **에러: 예외 발생 → ExceptionAdvice → ApiResponse → ResponseEntity → 클라이언트**

### ExceptionAdvice 실질적인 가치

- 애플리케이션에서 발생하는 모든 예외를 일관된 방식으로 처리하고 클라이언트에게 통일된 형태의 응답을 제공

---

**ExceptionAdvice 유무로 달라지는  코드 (Controller 로직)**

```java
// ExceptionAdvice 없을 때
// 모든 에러 발생 경우에 대해 try-catch 구문 사용해야 함
   public ResponseEntity<ApiResponse<UserDto>> getUser(Long id) {
       try {
           User user = userService.findById(id);
           return ResponseEntity.ok(ApiResponse.onSuccess(userMapper.toDto(user)));
       } catch (UserNotFoundException e) {
           return ResponseEntity.status(HttpStatus.NOT_FOUND)
               .body(ApiResponse.onFailure("USER_001", "사용자를 찾을 수 없습니다", null));
       } catch (Exception e) {
           return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
               .body(ApiResponse.onFailure("SYS_001", "서버 오류가 발생했습니다", null));
       }
   }
   
   
 
// ExceptionAdvice 있을 때
   public ResponseEntity<ApiResponse<UserDto>> getUser(Long id) {
       User user = userService.findById(id); 
       // 예외 발생 시 자동으로 ExceptionAdvice가 처리해주기에
       return ResponseEntity.ok(ApiResponse.onSuccess(userMapper.toDto(user)));
   }
```

- Advice가 없을 때도 일관된 응답 형식을 반환할 수 있었지만, 발생할 수 있는 에러를 예측하고 모두 try-catch로 묶어야 했었음
- Advice가 있음으로써 발생된 예외에 대해서 중앙 처리를 해주는 기관이 생긴 셈

**작동 방식**

1. `@RestControllerAdvice` 어노테이션으로 전역 예외 처리기 선언
2. `@ExceptionHandler` 어노테이션으로 처리할 예외 유형 지정
3. 예외 발생 시 해당 타입의 핸들러 메소드가 자동으로 호출됨
4. 예외 정보를 API 응답 형식으로 가공
5. 적절한 HTTP 상태 코드와 함께 응답 반환

### 참고

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestControllerAdvice.html?utm_source=chatgpt.com

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html?utm_source=chatgpt.com

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/ResponseEntityExceptionHandler.html?utm_source=chatgpt.com

https://docs.oracle.com/javase/8/docs/api/java/lang/RuntimeException.html?utm_source=chatgpt.com

https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html?utm_source=chatgpt.com

https://www.geeksforgeeks.org/exception-handling-in-spring-boot/?utm_source=chatgpt.com