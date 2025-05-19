# 키워드 미션

## RestContollerAdvice

- Spring MVC의 전역 예외 처리 및 공통 응답 바디 설정을 위한 어노테이션이다.
- 내부적으로는 @ControllerAdvice + @ResponseBody 역할이 결합되어 있다.

### 주요 기능

1. 전역 예외처리
    - `@ExceptionHandler(ExceptionType.class)` 메서드를 통해 특정 예외를 한곳에서 처리
    - 컨트롤러 내 예외가 발생하면 해당 메서드로 포워딩되어 HTTP 상태 코드와 메시지를 직접 반환
2. 공통 응답 바디 설정
    - JSON/XML 형태의 에러 응답 구조를 일관되게 구축
    - 예외 유형별 에러 코드, 사용자 메시지 등을 담아 응답 가능
3. 요청 바인딩 설정
    - @InitBinder 메서드로 모든 컨트롤러에 공통적으로 적용할 바인딩 검증 로직 등록
4. 모델 속성 전역 추가
    - @ModelAttribute 메서드로 모든 뷰또는 API에 공통적으로 필요한 모델 속성 주입

### 사용 예시

```
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 1) 특정 예외 처리
    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(EntityNotFoundException ex) {
        return new ErrorResponse("NOT_FOUND", ex.getMessage());
    }

    // 2) 공통 바인딩 설정
    @InitBinder
    public void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(LocalDate.class, new LocalDateEditor(...));
    }

    // 3) 전역 모델 속성
    @ModelAttribute("apiVersion")
    public String apiVersion() {
        return "v1.0";
    }
}

```

## lombok

- 자바 컴파일 단계에서 Getter/Setter, 생성자, toString, equlas/hashCode, Builder 메서드를 자동 생성해주는 라이브러리

### 주요 어노테이션 및 기능

| 어노테이션 | 역할 |
| --- | --- |
| `@Getter` / `@Setter` | 필드별 getter, setter 메서드 생성 |
| `@NoArgsConstructor`, `@AllArgsConstructor` | 인자 없는 생성자 / 모든 필드를 인자로 받는 생성자 생성 |
| `@RequiredArgsConstructor` | `final` 또는 `@NonNull` 필드만을 인자로 받는 생성자 생성 |
| `@Data` | `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor` 일괄 적용 |
| `@Builder` | 빌더 패턴 지원 (클래스 레벨 또는 생성자 레벨) |
| `@ToString`, `@EqualsAndHashCode` | toString, equals, hashCode 메서드 자동 생성 |
| `@Slf4j` (또는 `@Log4j2`, `@CommonsLog`) | 로깅 프레임워크별 Logger 인스턴스 자동 주입 |

### 코드 예시

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserDto {
    private Long id;
    private String name;
    private String email;
}

@Slf4j
@Service
public class UserService {
    public void register(UserDto dto) {
        log.info("Registering user: {}", dto);
        // ...
    }
}

```

- 위 코드 만으로 UserDto의 getId, setName, toString 등.. 메서드가 모두 생성된다.

### lombok 장점

- 보일러 플레이트 코드가 대폭 축소된다.
- 가독성이 향상되고, 유지보수 시 반복 코드 수정도 방지된다.
- 컴파일 타임에 코드 생성하여 런타임 오버헤드가 없다.