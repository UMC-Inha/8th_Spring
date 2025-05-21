### **Java Exception 종류**

---

- **Checked Exception**
    - 컴파일 시 예외 처리 필수
    - 예: `IOException`, `SQLException`

    ```java
    try {
        FileReader fr = new FileReader("file.txt");
    } catch (IOException e) {
        e.printStackTrace();
    }
    ```

- **Unchecked Exception (RuntimeException)**
    - 컴파일러 예외 처리 강제 없음
    - 예: `NullPointerException`, `IllegalArgumentException`

    ```java
    String s = null;
    s.length(); // NullPointerException 발생 가능
    ```

- **Error**
    - 보통 처리하지 않음
    - 예: `OutOfMemoryError`, `StackOverflowError`

  **OutOfMemoryError**

    - 힙 메모리 부족 시 발생

    ```java
    // 너무 큰 배열을 계속 생성하면 발생 가능
    List<int[]> list = new ArrayList<>();
    while(true) {
        list.add(new int[1_000_000]);
    }
    ```

  **StackOverflowError**

    - 재귀 호출이 너무 깊을 때 발생
    - 코테 문제에서 런타임 에러 시 보통

        ```java
        public void recursive() {
            recursive();
        }
        // 호출하면 StackOverflowError 발생
        recursive();
        
        ```


[오류 추가 정리](https://www.notion.so/1fab57f4596b8134a850d7a3b1974426?pvs=21)

### **@Valid**

---

- 객체 필드 유효성 자동 검사
- `@NotNull`, `@Min` 등 제약조건과 함께 사용
- Spring Controller에서 주로 요청 파라미터 검증에 사용
- 검증 실패 시 예외 발생 (`MethodArgumentNotValidException`)

```java
public class User {
    @NotNull
    private String name;

    @Min(18)
    private int age;
}

@RestController
public class UserController {
    @PostMapping("/users")
    public String createUser(@Valid @RequestBody User user) {
        return "User is valid";
    }
}

```

- **내부 동작 흐름**
    1. 요청 body를 객체(`User`)로 역직렬화 (`@RequestBody`)
    2. 해당 객체에 `@Valid`가 붙어 있으면, **Spring Validator**가 자동 호출
    3. 객체의 필드에 붙은 제약 조건(`@NotNull`, `@Min`, `@커스텀`)을 Bean Validation 엔진이 검사
    4. 검증 실패 시 예외 발생 → `@ControllerAdvice` 로 처리 가능
- 더 자세한 흐름
    - `@Valid`가 붙은 객체를 `javax.validation.Validator`가 받음
    - `Validator.validate(object, groups...)` 호출
    - `ValidatorImpl.validate()` 메서드가 실행되고, 다음과 같이 진행됨
        - 객체의 메타데이터(`BeanMetaData`)를 조회
        - 제약조건이 없으면 즉시 종료
        - `validateInContext()` 메서드를 호출해 본격적으로 검사 시작
    - `validateInContext()` 내부에서 각 필드별로 등록된 `ConstraintValidator`들의 `isValid()` 메서드 호출
    - `NotBlankValidator.isValid()`에서 값 검증 실행
    - 검증 실패 시 `ConstraintViolation` 객체 생성
    - 모든 위반 사항을 `Set<ConstraintViolation>`에 모아 반환

- `@NotNull, @NotBlank...` 또한 유사하게 다음과 같이 내부에 **isValid**로 유효성 검사 실행

```java
package org.hibernate.validator.internal.constraintvalidators.bv;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import jakarta.validation.constraints.NotBlank;

public class NotBlankValidator implements ConstraintValidator<NotBlank, CharSequence> {

	@Override
	public boolean isValid(CharSequence charSequence, ConstraintValidatorContext constraintValidatorContext) {
		if ( charSequence == null ) {
			return false;
		}

		return charSequence.toString().trim().length() > 0;
	}
}

```

- 핵심은 isValid
    - 커스텀 애노테이션에서는 ”`*ConstraintValidator`* 의 **isValid 메소드 구현을 통해 @Valid 애노테이션에 의해 검출될 수 있는 것” 이것만 기억하면 된다.

## Validator 정리

---

```java
@Component
@RequiredArgsConstructor
public class CategoriesExistValidator implements ConstraintValidator<ExistCategories, List<Long>> {

    

    @Override
    public void initialize(ExistCategories constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
    }

    @Override
    public boolean isValid(List<Long> values, ConstraintValidatorContext context) {
        boolean isValid = values.stream()
                .allMatch(value -> foodCategoryRepository.existsById(value));

        if (!isValid) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(ErrorStatus.FOOD_CATEGORY_NOT_FOUND.toString()).addConstraintViolation();
        }

        return isValid;

    }
}
```

- 기존 코드는 새로운 도메인 서비스 추가될 시 매번 Validator랑 커스텀 애노테이션 등록해줘야 하는 번거로움이 존재
- 개선책

  ⇒ 통합 Validator로 검증


**ExistsInDbValidator**

```java
package umc.study.validation.validator;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.data.jpa.repository.JpaRepository;
import umc.study.validation.annotation.ExistsInDb;

public class ExistsInDbValidator implements ConstraintValidator<ExistsInDb, Long>, ApplicationContextAware {
    private ApplicationContext ctx;
    private JpaRepository<?, Long> repository;
    private String messageTemplate;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.ctx = applicationContext;
    }

    @Override
    public void initialize(ExistsInDb anno) {
        this.repository = ctx.getBean(anno.repository());
        this.messageTemplate = anno.message();
    }

    @Override
    public boolean isValid(Long value, ConstraintValidatorContext context) {
        if (value == null) return true;
        boolean exists = repository.existsById(value);
        if (!exists) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(messageTemplate)
                    .addConstraintViolation();
        }
        return exists;
    }
}

```

**ExistsInDb**

```java
package umc.study.validation.annotation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;
import org.springframework.data.jpa.repository.JpaRepository;
import umc.study.validation.validator.ExistsInDbValidator;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = ExistsInDbValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface ExistsInDb {
    String message() default "존재하지 않는 데이터입니다.";
    Class<?> [] groups() default {};
    Class<? extends Payload>[] payload() default {};
    Class<? extends JpaRepository<?, Long>> repository();
}
```

- 타입에 따라 일일히 Validator 생성하기 너무 번거로울 것 같아서 통합 validator 선언
- `ApplicationContextAware` : 스프링 컨테이너가 생성한 ApplicationContext 객체를 빈 안으로 주입받기 위해 사용

  (현재 코드에선 런타임에 동적으로 타입을 결정하기 때문에)

- 즉, 런타임에 빈을 주입하기 위해서 사용한 애노테이션
- 값 검증 시 다음과 같이 사용

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class ReviewRequestDTO {

    @ExistsInDb(message = "해당 가게가 존재하지 않습니다.", repository = StoreRepository.class)
    private Long storeId;
}
```

### 여전히 Validator가 repository를 접근하는 문제 발생

---

실제 현업에서는 프로젝트 규모에 따라 다른 방식 사용

1. **중소규모 프로젝트**

```java
public class ExistsInDbValidator implements ConstraintValidator<ExistsInDb, Long> {
    
    @Autowired
    private ApplicationContext applicationContext;
    
    private String serviceBeanName;  // <-- 서비스 빈 이름
    ...
    
    @Override
    public boolean isValid(Long value, ConstraintValidatorContext context) {
        ...
        Object service = applicationContext.getBean(serviceBeanName); // <-- 서비스 빈을 가져옴
        Method method = ReflectionUtils.findMethod(service.getClass(), methodName, Long.class);
        boolean exists = (boolean) ReflectionUtils.invokeMethod(method, service, value);
        ...
    }
}
```

- 이렇게 하면 기존에 Validator에서 리포지토리를 직접 참조하는 것보다 서비스 계층을 통해 참조
- 어노테이션에서 **직접 서비스 빈 이름과 메서드 이름을 지정**
- **런타임에 메서드 호출**
- 중간 레이어 없이 **서비스 직접 호출**
- Spring 컨테이너에 강하게 결합되어 있어 유지보수 안 좋을 수 있음
- Map 접근보다는 **오버헤드가 조금 더 큼**

```java
Validator -> Service -> Repository
```

- 사용 예시

```java
@ExistsInDb(service = "userService", method = "existsById")
private Long userId;
```

1. **대규모 프로젝트**

```java
@Service
public class ValidationService {

    @Autowired
    private Map<String, DomainService> domainServices;
    
    public boolean validateEntityExists(String domain, Long id) {
        DomainService service = domainServices.get(domain + "Service");
        return service.exists(id);
    }
}
```

- **중앙화된 ValidationService** 계층 추가
- service(주입된 서비스 빈)의 exists 메소드가 id 기반으로 엔티티 존재 여부 검증
- 매 검증 요청마다 ApplicationContext에서 빈을 찾을 필요가 없음
- 모든 도메인 서비스는 **공통 인터페이스(DomainService)를 구현**
- Map 자료구조로 O(1) 시간에 서비스 검색 가능

```java
public interface DomainService {
    boolean exists(Long id);
}

@Service
public class UserService implements DomainService {
    private final UserRepository repository;
    
    @Autowired
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    @Override
    public boolean exists(Long id) {
        return repository.existsById(id);
    }
}
```

- 새로운 도메인 서비스 추가되면 Spring이 자동으로 map에 포함시키고 추가된 서비스는 공통 인터페이스의 exists 함수만 구현하면 자동으로 map에 추가

  (map 위에 `@AuthoWired` 로 인해 자동으로 빈을 찾아서 주입해주기 때문에)

```java
public class EntityExistsValidator implements ConstraintValidator<EntityExists, Long> {
    @Autowired
    private ValidationService validationService;
    
    private String domain;
    
    @Override
    public boolean isValid(Long value, ConstraintValidatorContext context) {
        boolean exists = validationService.validateEntityExists(domain, value);
        // ...
    }
}
```

- **표준화된 메서드** 제공

```java
public class EntityExistsValidator implements ConstraintValidator<EntityExists, Long> {
    
    @Autowired
    private ValidationService validationService;
    
    private String domain;
    private String messageTemplate;
    
    @Override
    public void initialize(EntityExists anno) {
        this.domain = anno.domain();
        this.messageTemplate = anno.message();
    }
    
    @Override
    public boolean isValid(Long value, ConstraintValidatorContext context) {
        if (value == null) return true;
        
        boolean exists = validationService.validateEntityExists(domain, value); 
        if (!exists) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(messageTemplate).addConstraintViolation();
        }
        return exists;
    }
}
```

- **validationService**의 `validateEntityExists` 메소드가 엔티티 식별
- 사용 예시

```java
@EntityExists(domain = "user", message = "사용자가 존재하지 않습니다.")
private Long userId;
```

**중소 규모 vs 대규모**

- **일관성 관리에서 차이가 존재**
    - 중소 규모: 각 서비스마다 다른 검증 로직 (유연하지만 일관성 관리 어려움)
    - 대규모: 중앙화된 검증 정책 적용 (규모가 커질수록 유리)

### MemberCommandServiceImpl 예제 코드 의문

---

**기존 로직**

```java
@Override
    @Transactional
    public Member joinMember(MemberRequestDTO.JoinDto request) {

        Member newMember = MemberConverter.toMember(request);
        List<FoodCategory> foodCategoryList = request.getPreferCategory().stream()
                .map(category -> {
                    return foodCategoryRepository.findById(category).orElseThrow(() -> new FoodCategoryHandler(ErrorStatus.FOOD_CATEGORY_NOT_FOUND));
                }).collect(Collectors.toList());

        List<MemberPrefer> memberPreferList = MemberPreferConverter.toMemberPreferList(foodCategoryList);

        memberPreferList.forEach(memberPrefer -> {memberPrefer.setMember(newMember);});

        return memberRepository.save(newMember);
    }
```

- 로직 요약

```java
// 1. Member entity 변환
// 2. Category ID들에 대해 findById().orElseThrow() 반복
// 3. Category 리스트 → MemberPrefer 리스트 변환
// 4. Member와 연관 설정
// 5. Member 저장
```

- 개별 조회 + 개별 검증
- `preferCategory.size()`만큼 `findById()` 호출 → **N번 쿼리..**
- **`findAllById` 사용해서 개선 가능 →** 한 번에 전체 조회 → **1번 쿼리**
- MemberPrefer 테이블에 save하는 로직도 없음

**코드**

```java
@Transactional
    @Override
    public Member joinMember(JoinDto request) {
        Member member = MemberConverter.toMember(request);
        List<Category> foodCateogoryList = categoryRepository.findAllById(request.getPreferCategory());
        if (foodCateogoryList.size() != request.getPreferCategory().size()) {
            throw new GeneralException(ErrorStatus.FOOD_CATEGORY_NOT_FOUND);
        }
        Member saveMember = memberJpaRepository.save(member);
        List<MemberCategory> memberCategoryList = MemberPreferConverter.toMemberPreferList(member, foodCateogoryList);
        memberJpaCategory.saveAll(memberCategoryList);
        return saveMember;
    }
```

- saveAll로 MemberCategory DB 반영
- findAllById로 List<Category> 조회
- 로직 요약

```java
// 1. Member entity 변환
// 2. findAllById()로 전체 Category 조회 (없는 ID는 무시됨)
// 3. Member 저장
// 4. MemberCategory 리스트 생성 및 저장
// 5. 컨트롤러 반환

```

출처

https://www.geeksforgeeks.org/exception-handling-in-spring-boot/?utm_source=chatgpt.com

https://openapi-generator.tech/docs/generators/spring/?utm_source=chatgpt.com

https://www.baeldung.com/spring-boot-bean-validation?utm_source=chatgpt.com

https://medium.com/%40AlexanderObregon/how-spring-boot-handles-validation-annotations-33b987c1a5cb

https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-validation.html?utm_source=chatgpt.com

https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ExceptionHandler.html?utm_source=chatgpt.com

https://hibernate.org/validator/documentation/?utm_source=chatgpt.com