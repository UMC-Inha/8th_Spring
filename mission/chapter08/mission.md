## - 시니어 미션

#### - @DynamicInsert와 @DynamicUpdate가 어떻게 작동되는지 파악하기

- **기본적인 JPA의 동작 방식**

    - 엔티티의 모든 컬럼을 INSERT, UPDATE SQL에 포함시킨다.

    - 즉, 변경되지 않은 필드라도 모두 set 절에 포함시킨다.

- **@DynamicInsert의 동작 방식**

    - 엔티티 저장 시 null인 필드나 기본값이 지정된 필드를 SQL에서 제외한다.

- **@DynamicUpdate의 동작 방식**

    - 엔티티 수정 시 실제 변경된 필드만 UPDATE SQL에 포함한다.

#### - 기존 방식과 @DynamicInsert, @DynamicUpdate를 적용했을 때 장단점 파악하기

- **@DynamicInsert, @DynamicUpdate를 적용했을 때의 장점**

    - insert의 경우는 바인딩 파라미터 수가 감소하고, update의 경우는 변경 필드만 쿼리에 포함하기 때문에 SQL의 쿼리가 감소한다.

    - @DynamicInsert를 활용해 DB의 기본값을 활용할 수 있다.

    - Update 시 변경되지 않은 컬럼에 대한 불필요한 로깅, 트리거가 방지되어 락 충돌이 최소화된다.

- **@DynamicInsert, @DynamicUpdate를 적용했을 때의 단점**

    - 매번 다른 컬럼의 조합으로 이루어진 SQL이 생성되어 SQL 캐시 히트율이 저하된다.

    - Hibernate가 엔티티 상태를 dirty checking으로 비교해서 SQL을 생성하기 때문에 CPU 사용량이 증가한다.

    - 복잡한 매핑에서는 어떤 컬럼이 빠졌는지 파악이 어려워 디버깅이 번거로워질 수 있다.

#### - @DynamicInsert, @DynamicUpdate를 언제 적용하면 좋을지 파악하기

- **@DynamicInsert**

    - 에니티에 DB 기본값으로 정의된 컬럼이 많을 때

    - 일부 필드만 채우고 나머지는 DB가 자동으로 값을 넣도록 할 때

    - 생성 일시, UUID 자동 생성 등의 기본값 컬럼이 많을 때

- **@DynamicUpdate**

    - 엔티티의 필드가 많고, 일부 필드만 자주 수정될 때

    - 낙관적 락 버전 컬럼을 쓸 경우, 변경 없는 컬럼이 업데이트 되면서 충돌 위험이 높아질 때

- **적용하지 않는게 좋은 상황**

    - 컬럼이 많지 않은 엔티티

    - 수정 시 거의 모든 필드를 업데이트 하는 로직

    - PreparedStatement 캐시 히트율이 중요한 대량 처리

        - PreparedStatement 캐시란, Hibernate 등의 ORM이 이미 준비해 둔 SQL문을 다시 생성하지 않고 재사용하는 것을 말한다.

<br>

#### - Rest Docs가 무엇인지 알아보기

- 스프링 기반 애플리케이션의 REST API 문서를 테스트 코드에서 자동으로 추출해 주는 기능

- 테스트 실행 시 요청/응답 값, 헤더 필드 설명 등을 자동으로 캡쳐해 JSON, Curl 등의 파일을 생성한다.

#### - Swagger와 Rest Docs의 장단점 비교하기

- **Rest Docs의 장점**

    - 실제 테스트를 기반으로 문서를 생성한다.

    - API가 변경되면 테스트가 깨지기 때문에, 자동적으로 문서 갱신 요구가 생기고, 결국 코드와 문서의 일관성이 유지되는 효과가 생긴다.

    - 브라우저가 아닌 정적 호스팅이나 PDF 출력에도 용이하다.

    - 문서 스타일을 자유롭게 커스터마이징 할 수 있다.

- **Rest Docs의 단점**

    - 각 API 마다 테스트 코드를 추가해야 한다.

    - Swagger처럼 런타임의 UI를 지원하지 않는다.

    - 별도의 빌드 플로우를 통해 문서를 생성한다.

- **Swagger의 장점**

    - 실시간 UI를 제공해주어 브라우저에서 즉시 명세서를 확인할 수 있다.

    - 별도의 도구를 필요로 하지 않는다.

    - 인터페이스를 미리 정의하고 문서화하며 개발할 수 있다.

    - 어노테이션만 잘 설정하면 자동으로 동기화가 된다.

    - 다양한 도구와 통합이 쉽다.

- **Swagger의 단점**

    - 문서와 실제 구현이 일치하지 않을 수 있다는 단점이 있다.

    - 테스트 예시가 포함되지 않아 실제 샘플 데이터 예시를 보여주려면 별도의 작업이 필요하다.

    - 애플리케이션이 실행중일 때만 동작한다.

    - 커스터마이징이 복잡하다.

#### - Swagger와 Rest Docs를 각각 언제 적용하면 좋을지 파악하기

- **Rest Docs를 사용하면 좋을 때**

    - 테스트를 기반으로 문서의 일관성을 철저히 지키고 싶을 때

    - 문서의 신뢰도가 높아야 할 때

    - 정적 문서의 형태로 S3, Github Page 등에 호스팅하거나, 문서의 버전 관리가 필요할 때

    - 복잡한 템플릿 커스터마이징이 필요할 때

    - 테스트 데이터를 예시로 넣고 싶을 때

- **Swagger를 사용하면 좋을 때**

    - 클라이언트, 서버 간의 API를 미리 확정하고 개발을 병렬로 진행할 때

    - 별도 도구 없이도 즉시 공유할 수 있는 문서가 필요할 때

    - 애플리케이션 배포 시 UI가 자동으로 노출되어야 할 때

<br>

## - 실습 미션 기록

#### - 미션 내용

1. 특정 지역에 가게 추가하기 API

2. **가게에 리뷰 추가하기 API**

3. 가게에 미션 추가하기 API

4. **가게의 미션을 도전 중인 미션에 추가(미션 도전하기) API**

#### - 미션 조건

1. github branch를 만들 때 issue를 만들고 branch 생성하여 진행 후 push할 것
2. controller, service, converter, dto, repository를 모두 활용할 것
3. ExceptionAdvice를 적극 활용해야하며 RequestBody에 값이 누락되거나 값이 잘못된 것을 @Valid 어노테이션으로 검증하기
4. **4번 API의 경우는 도전 하려는 미션이 이미 도전 중인지를 검증해야 하며 이를 커스텀 어노테이션을 통해 검증을 해야 함.**
5. **2번 API의 경우도 4번 API처럼 리뷰를 작성하려는 가게가 존재하는지 검증하는 커스텀 어노테이션을 사용할 것.**

#### - 미션 기록

- **Annotation과 Validator 생성**
  - 리뷰 등록을 위해서는 Member, Store 정보가 필요하다. Mission 등록을 위해서는 Store 정보가 필요하다. MemberMissoin 등록을 위해서는 Member, Mission 정보가 필요하다. 따라서 Member, Store, Mission이 존재하는지 검사하는 Annotation과 Validator를 먼저 만들어주었다.

      ```java
      @Documented
      @Constraint(validatedBy = MemberExistValidator.class)
      @Target( {
              ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
      @Retention(RetentionPolicy.RUNTIME)
      public @interface ExistMember {
      
          String message() default "해당하는 멤버가 존재하지 않습니다.";
          Class<?>[] groups() default {};
          Class<? extends Payload>[] payload() default {};
      }
      ```

      ```java
      @Documented
      @Constraint(validatedBy = StoreExistValidator.class)
      @Target( {
              ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
      @Retention(RetentionPolicy.RUNTIME)
      public @interface ExistStore {
      
          String message() default "해당하는 가게가 존재하지 않습니다.";
          Class<?>[] groups() default {};
          Class<? extends Payload>[] payload() default {};
      }
      
      ```

      ```java
      @Documented
      @Constraint(validatedBy = MissionExistValidator.class)
      @Target( {
              ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER })
      @Retention(RetentionPolicy.RUNTIME)
      public @interface ExistMission {
      
          String message() default "해당하는 미션이 존재하지 않습니다.";
          Class<?>[] groups() default {};
          Class<? extends Payload>[] payload() default {};
      }
      ```

      ```java
      @Component
      @RequiredArgsConstructor
      public class MemberExistValidator implements ConstraintValidator<ExistMember, Long> {
      
          private final MemberRepository memberRepository;
      
          @Override
          public void initialize(ExistMember constraintAnnotation) {
              ConstraintValidator.super.initialize(constraintAnnotation);
          }
      
          @Override
          public boolean isValid(Long value, ConstraintValidatorContext context) {
              boolean isValid = memberRepository.existsById(value);
      
              if (!isValid) {
                  context.disableDefaultConstraintViolation();
                  context.buildConstraintViolationWithTemplate(ErrorStatus.MEMBER_NOT_FOUND.toString()).addConstraintViolation();
              }
      
              return isValid;
      
          }
      }
      ```

      ```java
      @Component
      @RequiredArgsConstructor
      public class StoreExistValidator implements ConstraintValidator<ExistStore, Long> {
      
          private final StoreRepository storeRepository;
      
          @Override
          public void initialize(ExistStore constraintAnnotation) {
              ConstraintValidator.super.initialize(constraintAnnotation);
          }
      
          @Override
          public boolean isValid(Long value, ConstraintValidatorContext context) {
              boolean isValid = storeRepository.existsById(value);
      
              if (!isValid) {
                  context.disableDefaultConstraintViolation();
                  context.buildConstraintViolationWithTemplate(ErrorStatus.STORE_NOT_FOUND.toString()).addConstraintViolation();
              }
      
              return isValid;
      
          }
      }
      ```

      ```java
      @Component
      @RequiredArgsConstructor
      public class MissionExistValidator implements ConstraintValidator<ExistMission, Long> {
      
          private final MissionRepository missionRepository;
      
          @Override
          public void initialize(ExistMission constraintAnnotation) {
              ConstraintValidator.super.initialize(constraintAnnotation);
          }
      
          @Override
          public boolean isValid(Long value, ConstraintValidatorContext context) {
              boolean isValid = missionRepository.existsById(value);
      
              if (!isValid) {
                  context.disableDefaultConstraintViolation();
                  context.buildConstraintViolationWithTemplate(ErrorStatus.MISSION_NOT_FOUND.toString()).addConstraintViolation();
              }
      
              return isValid;
      
          }
      }
      ```
  - 또한, MemberMission의 경우 이미 도전중인지 검증하는 어노테이션이 필요하기 때문에, memberId와 missionId의 쌍으로 이루어진 값이 있는지 검증하는 validator와 어노테이션을 만들어주었다.

    ```java
    @Documented
    @Constraint(validatedBy = UniqueMemberMissionValidator.class)
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface UniqueMemberMission {
        String message() default "이미 미션에 도전중입니다.";
        Class<?>[] groups() default {};
        Class<? extends Payload>[] payload() default {};
    }
    ```

    ```java
    @Component
    @RequiredArgsConstructor
    public class UniqueMemberMissionValidator
            implements ConstraintValidator<UniqueMemberMission, MemberMissionRequestDto.JoinDto> {
    
        private final MemberMissionRepository memberMissionRepository;
    
        @Override
        public boolean isValid(MemberMissionRequestDto.JoinDto dto,
                               ConstraintValidatorContext context) {
            if (dto.getMemberId() == null || dto.getMissionId() == null) {
                return true;
            }
    
            boolean exists = memberMissionRepository.existsByMemberIdAndMissionId(
                    dto.getMemberId(), dto.getMissionId());
    
            if (exists) {
                context.disableDefaultConstraintViolation();
                context.buildConstraintViolationWithTemplate(
                                "Member "+ dto.getMemberId() +
                                        " 가 이미 미션에 도전중입니다. missionId: "+ dto.getMissionId()
                        )
                        .addPropertyNode("missionId")
                        .addConstraintViolation();
                return false;
            }
    
            return true;
        }
    }
    ```


- **Dto와 Converter 생성**
  - 먼저 가게 등록에 필요한 정보는 가게 이름, 지역, 실제 주소라고 생각해 StoreRequestDto를 다음과 같이 작성해주었다.

      ```java
      public class StoreRequestDto {
          @Getter
          public static class JoinDto{
              @NotBlank
              String name;
      
              @Size(min = 5, max = 50)
              String region;
      
              @Size(min = 5, max = 50)
              String address;
          }
      }
      ```

  - 가게의 응답값에는 가게 id, 이름, 지역, 주소 뿐만 아니라 가게의 평점, 리뷰 정보들, 생성 일시까지 필요하다고 생각해 다음과 같이 StoreResponseDto를 작성했다.

      ```java
      public class StoreResponseDto {
          @Builder
          @Getter
          @NoArgsConstructor
          @AllArgsConstructor
          public static class JoinResultDTO{
              Long storeId;
              Float score;
              String region;
              String address;
              List<ReviewResponseDto> reviewResponseDtoList;
              LocalDateTime createdAt;
          }
      }
      ```

  - 리뷰 정보 역시 등록과 응답 형태가 필요했기에 다음과 같이 request, response dto를 만들어주었다. 특히, 리뷰는 memberId와 storeId가 필요하기 때문에 위해서 만든 어노테이션을 적용해주었다.

      ```java
      public class ReviewRequestDto {
          @Getter
          public static class JoinDto {
              @NotBlank
              String title;
      
              Float score;
      
              @ExistMember
              Long memberId;
      
              @ExistStore
              Long storeId;
      
              List<String> imgaeUrlList;
          }
      }
      ```

      ```java
      public class ReviewResponseDto {
          @Builder
          @Getter
          @NoArgsConstructor
          @AllArgsConstructor
          public static class JoinResultDTO{
              Long reviewId;
              String title;
              Float score;
              List<String> imageUrlList;
              LocalDateTime createdAt;
          }
      }
      
      ```

  - 미션 관련 정보는 다음과 같이 request, response dto를 만들어주었다. 미션의 경우는 store 정보가 필요하기 때문에 “@ExistStore” 어노테이션을 적용해주었다.

      ```java
      public class MissionRequestDto {
          @Getter
          public static class JoinDto{
              @NotBlank
              String name;
      
              @NotBlank @Min(0)
              Integer reward;
      
              @NotBlank
              LocalDate deadline;
      
              @NotBlank
              String missionSpec;
      
              @ExistStore
              Long storeId;
          }
      }
      ```

      ```java
      public class MissionResponseDto {
          @Builder
          @Getter
          @NoArgsConstructor
          @AllArgsConstructor
          public static class JoinResultDTO{
              Long missionId;
              Integer reward;
              LocalDate deadline;
              String missionSpec;
              Long storeId;
              String storeName;
              LocalDateTime createdAt;
          }
      }
      ```

  - 멤버 미션의 경우, 등록시에는 member와 missionId만 있으면 된다고 생각해 어노테이션을 적용하여 다음과 같이 작성했다.

      ```java
      public class MemberMissionRequestDto {
      
          @Getter
          public static class JoinDto{
              @ExistMember
              Long memberId;
      
              @ExistMission
              Long missionId;
          }
      }
      ```

  - 응답의 경우 기존 미션 정보 + 미션의 진행상태가 필요하다고 생각해 MemberMissionResponseDto에 missionResponseDto를 담도록 했다.

      ```java
      public class MemberMissionResponseDto {
      
          @Builder
          @Getter
          @NoArgsConstructor
          @AllArgsConstructor
          public static class JoinResultDTO{
              Long memberMissionId;
              String missionStatus;
              MissionResponseDto.JoinResultDTO missionResponseDto;
          }
      }
      ```

  - 생성된 dto와 엔티티를 변환해주기 위해 다음과 같이 컨버터들을 생성해주었다.

      ```java
      public class StoreConverter {
      
          public static StoreResponseDto.JoinResultDTO toJoinResultDTO(Store store){
      
              List<ReviewResponseDto.JoinResultDTO> reviewResponseDtoList = store.getReviewList()
                      .stream().map(ReviewConverter::toJoinResultDTO).toList();
      
              return StoreResponseDto.JoinResultDTO.builder()
                      .storeId(store.getId())
                      .storeName(store.getName())
                      .region(store.getRegion().getName())
                      .score(store.getScore())
                      .reviewResponseDtoList(reviewResponseDtoList)
                      .createdAt(store.getCreatedAt())
                      .build();
          }
      
          public static Store toStore(StoreRequestDto.JoinDto request){
      
              Region region = RegionConverter.toRegion(request.getRegion());
      
              return Store.builder()
                      .name(request.getName())
                      .address(request.getAddress())
                      .region(region)
                      .reviewList(new ArrayList<>())
                      .build();
          }
      }
      ```

      ```java
      public class ReviewConverter {
      
          public static ReviewResponseDto.JoinResultDTO toJoinResultDTO(Review review){
      
              List<String> imageUrlList = review.getReviewImageList().stream().map(
                      ReviewImage::getImageUrl
              ).toList();
      
              return ReviewResponseDto.JoinResultDTO.builder()
                      .reviewId(review.getId())
                      .title(review.getTitle())
                      .score(review.getScore())
                      .imageUrlList(imageUrlList)
                      .createdAt(review.getCreatedAt())
                      .build();
          }
      
          public static Review toReview(ReviewRequestDto.JoinDto request){
              request.getImgaeUrlList().forEach(imageUrl -> {
                  toReviewImage(imageUrl);
              });
      
               return Review.builder()
                      .title(request.getTitle())
                      .score(request.getScore())
                      .reviewImageList(new ArrayList<>())
                      .build();
          }
      
          public static ReviewImage toReviewImage(String imageUrl){
              return ReviewImage.builder()
                      .imageUrl(imageUrl)
                      .build();
          }
      }
      ```

      ```java
      public class RegionConverter {
      
          public static Region toRegion(String regionName){
      
              return Region.builder()
                      .name(regionName)
                      .storeList(new ArrayList<>())
                      .build();
          }
      }
      ```

      ```java
      public class MissionConverter {
          public static MissionResponseDto.JoinResultDTO toJoinResultDTO(Mission mission){
      
              return MissionResponseDto.JoinResultDTO.builder()
                      .missionId(mission.getId())
                      .reward(mission.getReward())
                      .deadline(mission.getDeadline())
                      .missionSpec(mission.getMissionSpec())
                      .storeId(mission.getStore().getId())
                      .createdAt(mission.getCreatedAt())
                      .build();
          }
      
          public static Mission toMission(MissionRequestDto.JoinDto request){
      
              return Mission.builder()
                      .reward(request.getReward())
                      .deadline(request.getDeadline())
                      .missionSpec(request.getMissionSpec())
                      .memberMissionList(new ArrayList<>())
                      .build();
          }
      }
      ```

      ```java
      public class MemberMissionConverter {
          public static MemberMissionResponseDto.JoinResultDTO toJoinResultDTO(MemberMission memberMission){
      
              MissionResponseDto.JoinResultDTO missionResponseDto = MissionConverter.toJoinResultDTO(memberMission.getMission());
      
              return MemberMissionResponseDto.JoinResultDTO.builder()
                      .memberMissionId(memberMission.getId())
                      .missionStatus(memberMission.getStatus().name())
                      .missionResponseDto(missionResponseDto)
                      .build();
          }
      
          public static MemberMission toMission(MemberMissionRequestDto.JoinDto request){
      
              return MemberMission.builder()
                      .status(MissionStatus.CHALLENGING)
                      .build();
          }
      }
      ``` 
- **1번 미션**
  - 1번 미션은 단순 등록이기 떄문에 다음과 같이 엔드포인트는 “/”, 등록 메서드는 POST로 컨트롤러에 작성해주었다.

      ```java
      @RestController
      @RequiredArgsConstructor
      @Validated
      @RequestMapping("/stores")
      public class StoreRestController {
          private final StoreQueryService storeQueryService;
      
          @PostMapping("/")
          public ApiResponse<StoreResponseDto.JoinResultDTO> join(@RequestBody @Valid StoreRequestDto.JoinDto request){
              Store store = storeQueryService.registerStore(request);
              return ApiResponse.onSuccess(StoreConverter.toJoinResultDTO(store));
          }
      }
      ```

  - 단순 등록 미션이므로 요청 Dto를 컨버터로 엔티티로 변환하면 될 거라고 생각해 서비스 로직을 다음과 같이 작성했다.

      ```java
      @Override
          public Store registerStore(StoreRequestDto.JoinDto request){
      
              Store store = StoreConverter.toStore(request);
      
              return storeRepository.save(store);
          }
      ```
- **2번 미션**
  - 먼저 리뷰 관련 요청을 담당하는 ReviewRestController를 만들어주었고, 리뷰를 등록할 수 있는 메서드도 만들어주었다.

      ```java
      @RestController
      @RequiredArgsConstructor
      @Validated
      @RequestMapping("/reviews")
      public class ReviewRestController {
          private final ReviewQueryService reviewQueryService;
      
          @PostMapping("")
          public ApiResponse<ReviewResponseDto.JoinResultDTO> join(@RequestBody @Valid ReviewRequestDto.JoinDto request){
              Review review = reviewQueryService.registerReview(request);
              return ApiResponse.onSuccess(ReviewConverter.toJoinResultDTO(review));
          }
      }
      ```

  - 다음으로 리뷰 관련 서비스 로직을 담당하는 ReviewQuerySerice를 생성해 리뷰를 등록할 수 있는 서비스 로직을 만들어주었다.

      ```java
      @Service
      @RequiredArgsConstructor
      @Transactional
      public class ReviewQueryServiceImpl implements ReviewQueryService {
      
          private final ReviewRepository reviewRepository;
      
          private final StoreRepository storeRepository;
      
          private final MemberRepository memberRepository;
      
          @Override
          public Review registerReview(ReviewRequestDto.JoinDto request) {
              Review review = ReviewConverter.toReview(request);
      
              Member member = memberRepository.findById(request.getMemberId()).orElseThrow(() -> new GeneralException(ErrorStatus.MEMBER_NOT_FOUND));
              Store store = storeRepository.findById(request.getStoreId()).orElseThrow(() -> new GeneralException(ErrorStatus.STORE_NOT_FOUND));;
      
              review.setMember(member);
              review.setStore(store);
      
              return reviewRepository.save(review);
          }
      }
      ```

    - Store나 Member가 존재하지 않을 수 있다고 생각해 다음과 같이 ErrorStatus에 관련 예외를 정의해두었다.

        ```java
        MEMBER_NOT_FOUND(HttpStatus.BAD_REQUEST, "MEMBER4001", "사용자가 없습니다."),
        STORE_NOT_FOUND(HttpStatus.BAD_REQUEST, "STORE4001", "가게가 존재하지 않습니다.");
        ```

    - Review가 Member, Store와 연관관계를 맺고 있기 떄문에, 다음과 같이 Review 엔티티 내부에 연관관계 편의 메서드를 작성해 연관관계를 맺도록 했다.

        ```java
        public void setMember(Member member) {
                this.member = member;
                member.getReviewList().add(this);
            }
        
            public void setStore(Store store) {
                this.store = store;
                store.getReviewList().add(this);
            }
        ```

    - 2번 미션은 등록에 필요한 정보만 입력받아 엔티티를 저장하면 된다고 생각해 저장한 review 엔티티를 다시 responseDto로 변환해 반환하는 형식으로 메서드를 작성했다.
- **3번 미션**
  - 다음과 같이 요청을 바탕으로 Mission을 생성하고, 연관관계 편의 메서드를 사용해 연관관계를 맺는 등록 메서드를 서비스 로직에 작성해주었다.

      ```java
      Service
      @RequiredArgsConstructor
      public class MissionCommandServiceImpl implements MissionCommandService {
      
          private final MissionRepository missionRepository;
      
          private final StoreRepository storeRepository;
      
          @Override
          public Mission registerMission(MissionRequestDto.JoinDto request) {
              Mission mission = MissionConverter.toMission(request);
      
              Store store = storeRepository.findById(request.getStoreId()).orElseThrow(() -> new GeneralException(ErrorStatus.STORE_NOT_FOUND));;
      
              mission.setStore(store);
      
              return missionRepository.save(mission);
          }
      }
      ```

  - 다음으로 유저가 요청을 전송해서 서비스 로직을 호출할 수 있도록 MissionController를 작성해주었다.

      ```java
      @RestController
      @RequiredArgsConstructor
      @Validated
      @RequestMapping("/missions")
      public class MissionRestController {
      
          private final MissionCommandService missionCommandService;
      
          @PostMapping("")
          public ApiResponse<MissionResponseDto.JoinResultDTO> join(@RequestBody @Valid MissionRequestDto.JoinDto request){
              Mission mission = missionCommandService.registerMission(request);
              return ApiResponse.onSuccess(MissionConverter.toJoinResultDTO(mission));
          }
      }
      - **4번 미션**
  - 다음과 같이 요청을 바탕으로 MemberMission을 생성하고, 연관관계 편의 메서드를 사용해 연관관계를 맺는 등록 메서드를 서비스 로직에 작성해주었다.

      ```java
      @Service
      @RequiredArgsConstructor
      public class MemberMissionCommandServiceImpl implements MemberMissionCommandService {
      
          private final MemberRepository memberRepository;
      
          private final MissionRepository missionRepository;
      
          private final MemberMissionRepository memberMissionRepository;
      
          @Override
          public MemberMission joinMemberMission(MemberMissionRequestDto.@Valid JoinDto request) {
      
              Member member = memberRepository.findById(request.getMemberId()).orElseThrow(() -> new GeneralException(ErrorStatus.MEMBER_NOT_FOUND));
              Mission mission = missionRepository.findById(request.getMissionId()).orElseThrow(() -> new GeneralException(ErrorStatus.MISSION_NOT_FOUND));
      
              MemberMission memberMission = MemberMissionConverter.toMission(request);
              memberMission.setMember(member);
              memberMission.setMission(mission);
      
              return memberMissionRepository.save(memberMission);
          }
      }
      ```

  - 다음으로 유저가 요청을 전송해서 서비스 로직을 호출할 수 있도록 MemberMissionController를 작성해주었다.

      ```java
      @RestController
      @RequiredArgsConstructor
      @Validated
      @RequestMapping("/memberMissions")
      public class MemberMissionRestController {
      
          private final MemberMissionCommandService memberMissionCommandService;
      
          @PostMapping("")
          public ApiResponse<MemberMissionResponseDto.JoinResultDTO> joinMemberMission(@RequestBody @Valid MemberMissionRequestDto.JoinDto request){
              MemberMission memberMission = memberMissionCommandService.joinMemberMission(request);
              return ApiResponse.onSuccess(MemberMissionConverter.toJoinResultDTO(memberMission));
          }
      }
      ```