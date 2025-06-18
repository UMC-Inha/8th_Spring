**코드 양이 많아 핵심 코드만 기록했습니다.**

1. **특정 가게 추가하기 API**

url : /restaurants/location/{location_id}

- RestaurantRestController

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/restaurants")
@Validated
public class RestaurantRestController {

    private final RestaurantCommandService restaurantCommandService;

    @PostMapping("/location/{locationId}")
    public ApiResponse<String> createRestaurant(
            @RequestBody @Valid RestaurantRequestDTO.createRestaurantDTO request, @ExistLocation @PathVariable Long locationId) {

        restaurantCommandService.createRestaurant(request, locationId);
        return ApiResponse.onSuccess("식당이 등록되었습니다");

    }

}
```

- RestaurantCommandServiceImpl → 이미지, 메뉴를 받는 로직도 구현했습니다

```java
@Service
@RequiredArgsConstructor
@Transactional
public class RestaurantCommandServiceImpl implements RestaurantCommandService{

    private final RestaurantRepository restaurantRepository;
    private final LocationRepository locationRepository;

    private final RestaurantFoodRepository restaurantFoodRepository;
    private final RestaurantImageRepository restaurantImageRepository;

    @Override
    @Transactional
    public void createRestaurant(RestaurantRequestDTO.createRestaurantDTO request, Long locationId) {

        Location location = locationRepository.findById(locationId).orElseThrow(
                () -> new LocationHandler(ErrorStatus.LOCATION_NOT_FOUND)
        );

        Restaurant restaurant = RestaurantConverter.toRestaurant(request, location);
        restaurantRepository.save(restaurant);

        // foodList → RestaurantFood 리스트 변환
        List<RestaurantFood> foods = request.getFoodList().stream()
                .map(foodName -> RestaurantFood.builder()
                        .restaurant(restaurant)
                        .foodName(foodName)
                        .build()
                )
                .collect(Collectors.toList());

        // 한 번에 저장
        restaurantFoodRepository.saveAll(foods);

        // imageList -> RestaurantImage 리스트 변환
        List<RestaurantImage> images = request.getImageList().stream()
                .map(image -> RestaurantImage.builder()
                        .restaurant(restaurant)
                        .image(image)
                        .build()
                )
                .collect(Collectors.toList());

        // 한 번에 저장
        restaurantImageRepository.saveAll(images);

    }
}
```

- ExistLotation

```java
@Documented
@Constraint(validatedBy = LocationExistValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ExistLocation {

    String message() default "해당하는 위치 정보가 존재하지 않습니다.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

- LocationExistValidator

```java
@Component
@RequiredArgsConstructor
public class LocationExistValidator implements ConstraintValidator<ExistLocation, Long> {

    private final LocationRepository locationRepository;

    @Override
    public void initialize(ExistLocation constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
    }

    @Override
    public boolean isValid(Long aLong, ConstraintValidatorContext context) {

        boolean isValid;

        isValid = locationRepository.existsById(aLong);

        if (!isValid) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(ErrorStatus.LOCATION_NOT_FOUND.toString()).addConstraintViolation();
        }

        return isValid;
    }
}
```

1. **가게에 리뷰 추가하기**
- ReviewRestController

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/reviews")
@Validated
public class ReviewRestController {

    private final ReviewCommandService reviewCommandService;

    @PostMapping("/{restaurantId}/reviews")
    public ApiResponse<String> createReview(
            @RequestBody @Valid ReviewRequestDTO.createReviewDTO request, @ExistLocation @PathVariable Long restaurantId) {

        reviewCommandService.createReview(request, restaurantId);
        return ApiResponse.onSuccess("리뷰가 등록되었습니다");

    }

```

- ReviewCommandSeviceImpl → 이미지를 받는 로직도 구현했습니다

```java
@Service
@RequiredArgsConstructor
@Transactional
public class ReviewCommandServiceImpl implements ReviewCommandService{

    private final ReviewRepository reviewRepository;
    private final ReviewImageRepository reviewImageRepository;

    private final RestaurantRepository restaurantRepository;

    private final UserRepository userRepository;

    @Override
    public void createReview(ReviewRequestDTO.createReviewDTO request, Long restaurantId) {

        // @Validated로 앞에서 미리 유효성 검사 진행
        Restaurant restaurant = restaurantRepository.findById(restaurantId).orElseThrow(
                () -> new GeneralException(ErrorStatus.RESTAURANT_NOT_FOUND)
        );

        // TODO 하드 코딩한 값으로 추후, 값 변경하기
        User user = userRepository.findById(1L).orElseThrow(
                () -> new GeneralException(ErrorStatus.USER_NOT_FOUND)
        );
        Review review = ReviewConverter.toReview(request, restaurant, user);

        reviewRepository.save(review);

        // imageList -> RestaurantImage 리스트 변환
        List<ReviewImage> images = request.getImageList().stream()
                .map(image -> ReviewImage.builder()
                        .review(review)
                        .image(image)
                        .build()
                )
                .collect(Collectors.toList());

        // 한 번에 저장
        reviewImageRepository.saveAll(images);
    }
}
```

- ExistRestaurant

```java
@Documented
@Constraint(validatedBy = RestaurantExistValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ExistRestaurant {

    String message() default "해당하는 식당이 존재하지 않습니다.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

- RestaurantExistValidator

```java
@Component
@RequiredArgsConstructor
public class RestaurantExistValidator implements ConstraintValidator<ExistRestaurant, Long> {

    private final RestaurantRepository restaurantRepository;

    @Override
    public void initialize(ExistRestaurant constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
    }

    @Override
    public boolean isValid(Long aLong, ConstraintValidatorContext context) {

        boolean isValid;

        if(aLong == null) {
            isValid = false;
        }else{
            isValid = restaurantRepository.existsById(aLong);
        }

        if (!isValid) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(ErrorStatus.RESTAURANT_NOT_FOUND.toString()).addConstraintViolation();
        }

        return isValid;
    }
}
```

1. 가게에 미션 추가하기
- MissionRestController

```java
@RestController
@RequiredArgsConstructor
@Validated
public class MissionRestController {

    private final MissionCommandService missionCommandService;

    @PostMapping("/restaurants/{restaurantId}/missions")
    public ApiResponse<String> createMission(
            @RequestBody @Valid MissionRequestDTO.createMissionDTO request, @ExistRestaurant @PathVariable Long restaurantId) {

        missionCommandService.createMission(request, restaurantId);
        return ApiResponse.onSuccess("미션이 등록되었습니다");

    }

}
```

- MissionCommandServiceImpl

```java
@Service
@RequiredArgsConstructor
@Transactional
public class MissionCommandServiceImpl implements MissionCommandService {

    private final MissionRepository missionRepository;
    private final RestaurantRepository restaurantRepository;

    @Override
    public void createMission(MissionRequestDTO.createMissionDTO request, Long restaurantId) {

        // @Validated로 앞에서 미리 유효성 검사 진행
        Restaurant restaurant = restaurantRepository.findById(restaurantId).orElseThrow(
                () -> new GeneralException(ErrorStatus.RESTAURANT_NOT_FOUND)
        );

        Mission mission = MissionConverter.toMission(request, restaurant);

        missionRepository.save(mission);

    }
}

```

1. **가게의 미션을 도전 중인 미션에 추가(미션 도전하기)**

- UserRestController

```java

    // TODO 하드코딩 한 User 값을 받도록 바꿔주기
    @PostMapping("/missions/{missionId}")
    public ApiResponse<String> challengeMission( @ExistMission @PathVariable Long missionId){

        userCommandService.challengeMission(missionId);
        return ApiResponse.onSuccess("해당 미션의 도전을 시작하셨습니다");
    }
```

- UserCommandServiceImpl

```java
    // TODO 하드 코딩한 user 값 변경해주기
    @Override
    public void challengeMission(Long missionId) {

        User user = userRepository.findById(1L).orElseThrow(
                () -> new GeneralException(ErrorStatus.USER_NOT_FOUND)
        );

        Mission mission = missionRepository.findById(missionId).orElseThrow(
                () -> new GeneralException(ErrorStatus.MISSION_NOT_FOUND)
        );

        UserMission userMission = UserMission.builder()
                .isCompleted(Boolean.FALSE)
                .user(user)
                .mission(mission)
                .build();

        userMissionRepository.save(userMission);

    }
```

- ExsitMission

```java
@Documented
@Constraint(validatedBy = MissionExistValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ExistMission {

    String message() default "해당하는 미션이 존재하지 않습니다.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

- MissionExistValidator

```java
@Component
@RequiredArgsConstructor
public class MissionExistValidator implements ConstraintValidator<ExistMission, Long> {

    private final MissionRepository missionRepository;
    private final UserMissionRepository userMissionRepository;
    private final UserRepository userRepository;

    @Override
    public void initialize(ExistMission constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
    }

    @Override
    public boolean isValid(Long aLong, ConstraintValidatorContext context) {

        boolean isValid = false;
        Mission mission = missionRepository.findById(aLong).orElse(null);

        // 미션이 존재하지 않을 때
        if (mission == null) {
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(ErrorStatus.MISSION_NOT_FOUND.toString()).addConstraintViolation();
            return isValid;
        }

        User user = userRepository.findById(1L).orElseThrow(
                () -> new GeneralException(ErrorStatus.USER_NOT_FOUND)
        );
        Boolean existsUserMissionByUserAndMission = userMissionRepository.existsUserMissionByUserAndMission(user, mission);

        // 이미 수행 중인 미션일 때
        if(existsUserMissionByUserAndMission){
            context.disableDefaultConstraintViolation();
            context.buildConstraintViolationWithTemplate(ErrorStatus.MISSION_ALREADY_EXIST.toString()).addConstraintViolation();
            return isValid;
        }

        isValid = true;
        return isValid;
    }
}
```

### 미션 수행 소감

@Valid 애노테이션은 기존에 NotBlank나 Min 같은 표준 검증 애노테이션을 거르는 용도로만 사용했지만, 이번에 커스텀 애노테이션을 도입하면서 중복 코드를 제거하고 유지보수성을 크게 높일 수 있다는 것을 깨달았습니다. validation 패키지를 구성하고 검증 로직이 어떻게 동작하는지 분석하는 데 시간이 걸렸지만, 그 과정을 통해 많은 것을 배우고 메서드 예외 처리와 제약 조건 예외 처리 방식도 다시 한 번 복습할 수 있었습니다.

추가로 리팩터링이 필요한 부분으로는, 현재 사용자 식별자를 하드코딩해 두었다는 점이 있습니다. 이를 해결하기 위해 미리 구현해 둔 ExistUser 애노테이션을 활용해 사용자 식별 값을 외부에서 주입받도록 수정할 계획입니다. 또한 미션 유효성 검사 시 사용자 미션 데이터를 검증할 때는 토큰 기반 인증 방식을 도입하여 관련 유틸리티를 활용하는 방식으로 로직을 보완할 예정입니다.
