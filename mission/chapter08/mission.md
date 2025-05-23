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

## 미션 수행 기록

---

**ErrorStatus 추가**

```java
    ...
    // 멤버 관련 에러
    NICKNAME_NOT_EXIST(HttpStatus.BAD_REQUEST, "MEMBER4002", "닉네임은 필수 입니다."),
    MEMBER_NOT_FOUND(HttpStatus.NOT_FOUND, "MEMBER4001", "해당 회원이 존재하지 않습니다."),

    // 카테고리 관련 에러
    FOOD_CATEGORY_NOT_FOUND(HttpStatus.NOT_FOUND, "CATEGORY4001", "카테고리가 없습니다."),

    // 지역 관련 에러
    REGION_NOT_FOUND(HttpStatus.NOT_FOUND, "REGION4001", "해당 지역이 존재하지 않습니다."),

    // 상점 관련 에러
    STORE_NOT_FOUND(HttpStatus.NOT_FOUND, "STORE4001", "해당 가게가 존재하지 않습니다."),

    // 리뷰 관련 에러
    REVIEW_NOT_FOUND(HttpStatus.NOT_FOUND, "REVIEW4001", "해당 리뷰가 존재하지 않습니다."),

    // 미션 관련 에러
    MISSION_NOT_FOUND(HttpStatus.NOT_FOUND, "MISSION4001", "해당 미션이 존재하지 않습니다."),
    ...
```

### 특정 지역에 가게 추가하기

---

**StoreReponseDTO**

```java
public class StoreReponseDTO {

    @Builder
    @Getter
    @NoArgsConstructor
    @AllArgsConstructor
    public static class addResultDto {
        private Long storeId;

        private String storeName;

        private String address;

        private BigDecimal score;
    }
}
```

**StoreRequestDTO**

```java
public class StoreRequestDTO {
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public static class AddDto {
        @ExistsInDb(
                message = "해당 지역이 존재하지 않습니다.",
                repository = RegionRepository.class
        )
        private Long regionId;
				
				@NotBlank
        private String storeName;
				
				@NotBlank
        private String address;

        private BigDecimal score = BigDecimal.valueOf(0.0); 

        @Size(max = 1, message = "이미지는 1장만 등록할 수 있습니다.")
        private List<@NotBlank @URL String> imageList;
    }
}
```

가게 등록할 때는 score 0으로 설정

**StoreConverter**

```java
public class StoreConverter {
    public static StoreReponseDTO.addResultDto toAddResultDTO(Store store) {
        return StoreReponseDTO.addResultDto.builder()
                .storeId(store.getId())
                .storeName(store.getName())
                .address(store.getAddress())
                .score(store.getScore())
                .build();
    }

    public static Store toStore(StoreRequestDTO.AddDto request) {

        return Store.builder()
                .address(request.getAddress())
                .name(request.getStoreName())
                .score(request.getScore())
                .build();

    }
}
```

**StoreCommandServiceImpl**

```java
@Service
@RequiredArgsConstructor
public class StoreCommandServiceImpl implements StoreCommandService{
    private final StoreRepository storeRepository;
    private final RegionRepository regionRepository;
    private final ImageRepository imageRepository;

    @Transactional
    @Override
    public Store addStore(AddDto addDto) {
        Store store = StoreConverter.toStore(addDto);
        Region region = regionRepository.findById(addDto.getRegionId())
                .orElseThrow(() -> new GeneralException(ErrorStatus.REGION_NOT_FOUND));
        store.setRegion(region);
        List<Image> imageList = addDto.getImageList().stream()
                .map(url -> {
                    return Image.builder()
                            .url(url)
                            .store(store)
                            .build();
                }).collect(Collectors.toList());
        imageRepository.saveAll(imageList);
        Store saveStore = storeRepository.save(store);
        return saveStore;
    }
}
```

StoreCommandController

```java
@Controller
@RequiredArgsConstructor
@RequestMapping("/store")
public class StoreCommandController {
    private final StoreCommandServiceImpl storeCommandService;
    @PostMapping("/")
    public ResponseEntity<?> add(@RequestBody @Valid StoreRequestDTO.AddDto request) {
        Store store = storeCommandService.addStore(request);
        return ResponseEntity.ok(StoreConverter.toAddResultDTO(store));
    }
}
```

### 가게에 리뷰 추가하기

---

**ReviewRequestDTO**

```java
public class ReviewRequestDTO {

    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public static class addDto {
        @ExistsInDb(
                message = "해당 가게가 존재하지 않습니다.",
                repository = StoreRepository.class
        )
        private Long storeId;

        @NotBlank
        private String comment;

        @NotBlank
        private BigDecimal score;

        @Size(max = 3, message = "이미지는 최대 3장까지 업로드할 수 있습니다.")
        private List<@NotBlank @URL String> imageList; 
    }

}

```

- 이미지는 필수 x
- 넣었을 경우 "" 빈 문자열 방지

**ReviewResponseDTO**

```java
public class ReviewResponseDTO {

    private Long reviewId;

    private String comment;

    private BigDecimal score;

    @Size
    private List<@NotBlank @URL String> imageList;

}

```

**ReviewCommandsServiceImpl**

```java
@Service
@RequiredArgsConstructor
public class ReviewCommandsServiceImpl implements ReviewCommandService{
    private final ReviewRepository reviewRepository;
    private final StoreRepository storeRepository;
    private final ImageRepository imageRepository;

    @Transactional
    @Override
    public Review addReview(AddDto addDto) {
        Review review = ReviewConverter.toReview(addDto);
        Store store = storeRepository.findById(addDto.getStoreId())
                .orElseThrow(() -> new GeneralException(ErrorStatus.STORE_NOT_FOUND));
        review.setStore(store);
        List<Image> imageList = addDto.getImageList().stream()
                .map(url -> {
                    return Image.builder()
                            .url(url)
                            .review(review)
                            .build();
                }).collect(Collectors.toList());
        imageRepository.saveAll(imageList);
        Review saveReview = reviewRepository.save(review);

        return saveReview;
    }
}
```

- 반환 시점에 트랜잭션 커밋 후 DB에 반영된 엔티티 반환
- 이미지 `saveAll` 로 저장

**ReviewCommandController**

```java
@Controller
@RequiredArgsConstructor
@RequestMapping("/reviews")
public class ReviewCommandController {
    private final ReviewCommandsServiceImpl reviewCommandsService;
    @PostMapping("/")
    public ResponseEntity<?> add(@RequestBody @Valid AddDto request) {
        Review review = reviewCommandsService.addReview(request);
        return ResponseEntity.ok(ReviewConverter.toAddResultDTO(review));
    }

}

```

### 가게에 미션 추가하기

---

**MissionRequestDTO**

```java
public class MissionRequestDTO {
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public static class AddMission{
        @ExistsInDb(
                message = "해당 가게가 존재하지 않습니다.",
                repository = StoreRepository.class
        )
        private Long storeId;

        @NotBlank
        private String missionSpec;

        @NotBlank
        private String local;

        @NotBlank
        private LocalDateTime deadline;
    }
}
```

**MissionResponseDTO**

```java
public class MissionResponseDTO {

    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public static class AddResultDto {
        private Long missionId;
        private String missionSpec;
        private LocalDateTime deadline;
    }
}
```

**MissionConverter**

```java
public class MissionConverter {
    public static MissionResponseDTO.AddResultDto toAddResultDTO(Mission mission) {
        return MissionResponseDTO.AddResultDto.builder()
                .missionSpec(mission.getMissionSpec())
                .missionId(mission.getId())
                .deadline(mission.getDeadline())
                .build();
    }

    public static Mission toMission(MissionRequestDTO.AddMission request) {

        return Mission.builder()
                .missionSpec(request.getMissionSpec())
                .local(request.getLocal())
                .deadline(request.getDeadline())
                .build();
    }
}
```

**MissionCommandServiceImpl**

```java
@Service
@RequiredArgsConstructor
public class MissionCommandServiceImpl implements MissionCommandService{
    private final MissionRepository missionRepository;
    private final StoreRepository storeRepository;

    @Transactional
    @Override
    public Mission addMissionToStore(AddMission request) {
        Mission mission = MissionConverter.toMission(request);
        Store store = storeRepository.findById(request.getStoreId())
                .orElseThrow(() -> new GeneralException(ErrorStatus.STORE_NOT_FOUND));
        mission.setStore(store);
        Mission saveMission = missionRepository.save(mission);
        return saveMission;
    }
    }
}
```

**MissionCommandController**

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/missions")
public class MissionCommandController {
    private final MissionCommandService missionCommandService;

    @PostMapping("/")
    public ResponseEntity<?> join(@RequestBody @Valid MissionRequestDTO.AddMission request) {
        return ResponseEntity.ok(MissionConverter.toAddResultDTO(missionCommandService.addMissionToStore(request)));
    }
}
```

### **가게의 미션을 회원의 도전 중인 미션에 추가 (전부? 하나만?)**

---

- **하나만 추가하는 경우**

**MissionRequestDTO.AddMemberMission**

```java
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public static class AddMemberMission{
        @ExistsInDb(
                message = "해당 미션이 존재하지 않습니다.",
                repository = MissionRepository.class
        )
        private Long missionId;

        @ExistsInDb(
                message = "해당 회원이 존재하지 않습니다.",
                repository = MemberJpaRepository.class
        )
        private Long memberId;

        private MissionStatus missionStatus = MissionStatus.IN_PROGRESS;
    }
```

**MissionConverter.toMemberMission**

```java
public static MemberMission toMemberMission(MissionRequestDTO.AddMemberMission request) {
        return MemberMission.builder()
                .status(request.getMissionStatus())
                .build();
    }
```

**MissionCommandController.addMemberMission**

```java
// 가게의 미션을 회원의 도전 중인 미션에 추가 (하나만)
    @PostMapping("/members")
    public ResponseEntity<?> addMemberMission(@RequestBody @Valid AddMemberMission request) {
        return ResponseEntity.ok(MissionConverter.toAddResultDTO(missionCommandService.addMemberMission(request)));
    }
```

**MissionCommandServiceImpl.addMemberMission**

```java
    @Transactional
    @Override
    public Mission addMemberMission(AddMemberMission request) {
        MemberMission memberMission = MissionConverter.toMemberMission(request);
        Mission mission = missionRepository.findById(request.getMissionId()).orElseThrow(() -> new GeneralException(ErrorStatus.MISSION_NOT_FOUND));
        Member member = memberJpaRepository.findById(request.getMemberId()).orElseThrow(() -> new GeneralException(ErrorStatus.MEMBER_NOT_FOUND));
        memberMission.setMemberMission(member, mission);
        memberMissionRepository.save(memberMission);
        return mission;

    }
```

- 추가 후 미션 상세 정보 반환이 더 좋을 것 같아서 Mission 반환
- mission_id, member_id로 조회 후 memberMission에 추가
- **추가로, 가게 미션 전부 추가할 경우**
    - store_id로부터 mission 가져오고(List) member_id를 통해 회원 조회 후 memberMission에 추가

### GeneralException 관련 내용

---

- 현재 GeneralException 하나만 두고 이를 상속받아 TempHandler를 정의하거나 다른 MemberHandler 정의했었음
- 도메인의 예외 상황별로 세밀하게 분기하거나 특별한 처리를 하기가 어려움
- `GeneralException`을 상속받는 도메인별 예외를 만들고
- `ExceptionAdvice`에서 예외별로 `@ExceptionHandler`를 두는 방식으로 세밀한 조정이 가능하게 바꿀 수 있음

```java
public class RegionNotFoundException extends GeneralException {
    public RegionNotFoundException() {
        super(ErrorStatus.REGION_NOT_FOUND);
    }
}
```

- 실제 사용 예시

```java
.orElseThrow(() -> new RegionNotFoundException());
```

- 예외가 커지거나 복잡해질 때는 나중에라도 메인 별 커스텀 예외 클래스로 분리하는 게 관리에 도움이 된다.
- 현재 예제는 다양한 에러 처리가 필요할 것 같지 않아 GeneralException으로 일관된 형식만 유지