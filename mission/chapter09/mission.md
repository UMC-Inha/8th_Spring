- **미션 기록**
    1. 내가 작성한 리뷰 목록
        
        Controller
        
        ```java
        @GetMapping("/{restaurantId}")
            @Operation(summary = "특정 가게의 리뷰 목록 조회 API",description = "특정 가게의 리뷰들의 목록을 조회하는 API이며, 페이징을 포함합니다. query String 으로 page 번호를 주세요")
            @ApiResponses({
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "COMMON200",description = "OK, 성공"),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH003", description = "access 토큰을 주세요!",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH004", description = "acess 토큰 만료",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH006", description = "acess 토큰 모양이 이상함",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
            })
            @Parameters({
                    @Parameter(name = "restaurantId", description = "가게의 아이디, path variable 입니다!")
            })
            public ApiResponse<ReviewResponseDTO.ReviewPreViewListDTO> getReviewList(@ExistRestaurant @PathVariable(name = "restaurantId") Long restaurantId, @CheckPage @RequestParam(name = "page") Integer page){
                Page<Review> reviewList = reviewQueryService.getReviewList(restaurantId,page);
                return ApiResponse.onSuccess(ReviewConverter.toReviewPreViewListDTO(reviewList));
            }
        ```
        
        Service
        
        ```java
        @Override
            public Page<Review> getReviewListByUserId(Long userId, Integer page) {
        
                User user = userRepository.findById(userId).orElseThrow(
                        () -> new GeneralException(ErrorStatus.USER_NOT_FOUND)
                );
        
                Page<Review> reviewPage = reviewRepository.findAllByUser(user, PageRequest.of(page, 10));
                
                // 시니어 미션을 위한 Slice 객체 생성
                Slice<Review> reviewSlice = reviewRepository.findReviewsByUser(user, PageRequest.of(page, 10));
        
                System.out.println(reviewSlice);
                System.out.println(reviewPage);
        
                return reviewPage;
        
            }
        ```
        
        Repository
        
        ```java
        public interface ReviewRepository extends JpaRepository<Review, Long> {
        
            Page<Review> findAllByRestaurant(Restaurant restaurant, Pageable pageable);
        
            Page<Review> findAllByUser(User user, Pageable pageable);
            Slice<Review> findReviewsByUser(User user, Pageable pageable);
        
        }
        
        ```
        
    2. 특정 가게의 미션 목록
        
        Controller
        
        ```java
        @GetMapping("/restaurants/{restaurantId}/missions")
            @Operation(summary = "특정 가게의 미션 목록 조회 API",description = "특정 가게의 미션들의 목록을 조회하는 API이며, 페이징을 포함합니다. query String 으로 page 번호를 주세요")
            @ApiResponses({
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "COMMON200",description = "OK, 성공"),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH003", description = "access 토큰을 주세요!",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH004", description = "acess 토큰 만료",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH006", description = "acess 토큰 모양이 이상함",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
            })
            @Parameters({
                    @Parameter(name = "restaurantId", description = "가게의 아이디, path variable 입니다!")
            })
            public ApiResponse<MissionResponseDTO.MissionListDTO> getMissionListByRestaurantId(
                    @ExistRestaurant @PathVariable(name = "restaurantId") Long restaurantId,
                    @CheckPage @RequestParam(name = "page") Integer page) {
        
                Page<Mission> missionListByRestaurantId = missionQueryService.getMissionListByRestaurantId(restaurantId, page);
                return ApiResponse.onSuccess(MissionConverter.toMissionListDTO(missionListByRestaurantId));
            }
        ```
        
    
    Service
    
    ```java
        @Override
        public Page<Mission> getMissionListByRestaurantId(Long restaurantId, Integer page) {
    
            Restaurant restaurant = restaurantRepository.findById(restaurantId).orElseThrow(
                    () -> new GeneralException(ErrorStatus.RESTAURANT_NOT_FOUND)
            );
    
            Page<Mission> missionPage = missionRepository.findAllByRestaurant(restaurant, PageRequest.of(page, 10));
            return missionPage;
        }
    ```
    
    Repository
    
    ```java
    public interface MissionRepository extends JpaRepository<Mission, Long> {
    
        Page<Mission> findAllByRestaurant(Restaurant restaurant, Pageable pageable);
    
    }
    ```
    
    1. 내가 진행 중인 미션 목록
        
        Controller
        
        ```java
        @GetMapping("/missions")
            @Operation(summary = "내가 진행 중인 미션 목록 조회 API",description = "내가 진행 중인 미션들의 목록을 조회하는 API이며, 페이징을 포함합니다. query String 으로 page 번호를 주세요")
            @ApiResponses({
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "COMMON200",description = "OK, 성공"),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH003", description = "access 토큰을 주세요!",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH004", description = "acess 토큰 만료",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH006", description = "acess 토큰 모양이 이상함",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
            })
            public ApiResponse<MissionResponseDTO.MissionListDTO> getMyMissionList(
                    @CheckPage @RequestParam(name = "page") Integer page) {
        
                Long userId = 1L; // TODO. SecurityUtils 구현 후 토큰에서 userID 가져오기
        
                Page<UserMission> userMissionList = missionQueryService.getMissionListByUserId(userId, page);
                return ApiResponse.onSuccess(MissionConverter.toUserMissionListDTO(userMissionList));
            }
        ```
        
        Service
        
        ```java
         @Override
            public Page<UserMission> getMissionListByUserId(Long userId, Integer page) {
        
                User user = userRepository.findById(userId).orElseThrow(
                        () -> new GeneralException(ErrorStatus.USER_NOT_FOUND)
                );
        
                Page<UserMission> userMissionPage = userMissionRepository.findOngoingByUserId(userId, PageRequest.of(page, 10));
        
                return userMissionPage;
            }
        ```
        
        Repository
        
        ```java
            @Query(
                    value = "SELECT um " +
                            "FROM UserMission um " +
                            "JOIN FETCH um.mission m " +
                            "WHERE um.user.id = :userId " +
                            "  AND um.isCompleted = false",
                    countQuery = "SELECT COUNT(um) " +
                            "FROM UserMission um " +
                            "WHERE um.user.id = :userId " +
                            "  AND um.isCompleted = false"
            )
            Page<UserMission> findOngoingByUserId(
                    @Param("userId") Long userId,
                    Pageable pageable
            );
        ```
        
    2. 진행중인 미션 진행 완료로 바꾸기
        
        Controller
        
        ```java
        @PatchMapping("/users/{userId}/missions/{missionId}")
            @Operation(summary = "진행 중인 미션 진행 완료로 바꾸기 API",description = "해당 mission의 상태를 진행 완료로 변경하는 API이다.")
            @ApiResponses({
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "COMMON200",description = "OK, 성공"),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH003", description = "access 토큰을 주세요!",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH004", description = "acess 토큰 만료",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
                    @io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "AUTH006", description = "acess 토큰 모양이 이상함",content = @Content(schema = @Schema(implementation = ApiResponse.class))),
            })
            public ApiResponse<String> changeMissionStatus(
                    @ExistUser @PathVariable(name = "userId") Long userId,
                    @ExistMission @PathVariable(name = "missionId") Long missionId) {
        
                missionCommandService.changeMissionStatus(userId, missionId);
                return ApiResponse.onSuccess("미션의 상태가 변경되었습니다.");
            }
        ```
        
        Service
        
        ```java
        @Override
            public void changeMissionStatus(Long userId, Long missionId) {
        
                Mission mission = missionRepository.findById(missionId).orElseThrow(
                        () -> new GeneralException(ErrorStatus.MISSION_NOT_FOUND)
                );
        
                User user = userRepository.findById(userId).orElseThrow(
                        () -> new GeneralException(ErrorStatus.USER_NOT_FOUND)
                );
        
                UserMission userMission = userMissionRepository.findByUserAndMission(user, mission).orElseThrow(
                        () -> new GeneralException(ErrorStatus.USERMISSION_NOT_FOUND)
                );
        
                if(userMission.getIsCompleted()){
                    userMission.changeIsCompleted(false);
                }else{
                    userMission.changeIsCompleted(true);
                }
        
                userMissionRepository.save(userMission);
            }
        ```
        
        Repository

  ------
  - **시니어 미션**
 
  # 시니어 미션

**미션 목표:**

- **Slice**와 **Page**의 구조적 차이와 적용 시점 파악하기
- **for**와 **stream**의 **성능, 가독성, 유지보수성**을 직접 비교

**미션 상세 내용:**

### 1️⃣ Page와 Slice가 각각 어떻게 출력값이 나오는 지 알아보기

- 기존 미션에 나왔던 pagination에서 Page와 Slice로 바꾸어 각각 출력값 비교
    - 다른 API를 하나 만들어도 됩니다.
    
    ![image.png](attachment:71a7fc10-eb51-4c81-879e-b095c8cd9ce7:image.png)
    

### 2️⃣ Page, Slice 각각 적용 시 장단점 파악하기

- 찾아본 원리를 토대로 서로의 장단점 적어보기

| 구분 | Page | Slice |
| --- | --- | --- |
| **전체 정보** | 전체 요소 수(totalElements), 전체 페이지 수(totalPages) 제공 | 전체 정보 제공 안 함 |
| **추가 쿼리** | 전체 개수(count) 조회 위한 추가 `COUNT` 쿼리 실행 | `COUNT` 쿼리 없이 size + 1만 조회해 “다음 페이지 존재 여부” 판단 |
| **성능** | 대용량 테이블에서 `COUNT` 쿼리로 인한 부하 발생 가능 | `COUNT` 쿼리 생략으로 상대적으로 빠름 |
| **필요 정보** | 사용자에게 “총 몇 건, 총 몇 페이지”를 보여줄 때 | 전체 개수는 필요 없고 “더 가져올 데이터가 있는지만” 알면 될 때 |

### 3️⃣ 언제 적용하면 좋을 지 파악하기

- 장단점을 토대로 언제 Page, Slice를 언제 적용하면 좋을 지 적기
    - **page을 사용하면 좋은 경우**
        - 화면이나 API 응답에 **전체 데이터 건수와 전체 페이지 수**를 반드시 표시해야 할 때
        - “첫 페이지 / 마지막 페이지” 네비게이션, 특정 페이지로 바로 이동 같은 **정교한 페이징 UI**를 구현할 때
        - 검색 결과 전체 개수를 활용해 **통계나 보고서**를 작성해야 할 때
    - **Slice를 사용하면 좋은 경우**
        - **전체 개수 불필요**, 단순히 “다음 데이터가 더 있는지만” 판단하면 될 때
        - **쿼리 성능**이 중요하고 불필요한 count 쿼리를 줄여야 할 때

### 1️⃣ for과 stream이 어떻게 작동되는지 파악하기

- 동일한 연산(sum, filter 등)을 for문, stream 각각으로 구현해보기
    - 가능하다면 많은 양의 데이터(10만 건)를 넣고 실행하여 시간 재보기 (시니어 포함 필수 아님)
- 인터넷 검색을 통해 둘의 차이 파악하기
    
    **for**
    
    for문은 개발자가 직접 인덱스나 이터레이터를 제어하면서 루프를 수행하는 **외부 반복(external iteration)** 방식
    

**stream**

Stream은 데이터 소스(Collection 등)에서 **“무엇을 할 것인가(what)”**만 선언하면, 내부에서 반복과 처리를 해 주는 **내부 반복(internal iteration)** 방식

https://nicednjsdud.github.io/java/java-java-language-stream-vs-for/?utm_source=chatgpt.com

### 2️⃣ for, stream 각각 적용 시 장단점 파악하기

- 찾아본 원리를 토대로 서로의 장단점 적어보기
    
    속도의 경우에서는 for 문이 stream 보다 대체적으로 빠른 경우가 많다.
    
    한편 stream의 장점은 병렬 처리이다.
    
    데이터를 **병렬 처리**해야 할 경우, for문은 직접 스레드를 생성·관리해야 하지만, Stream API는 parallesStream 등의 호출만으로 내부에서 스레드를 분할·관리해 준다.
    
    따라서 대규모 데이터셋을 여러 코어로 분할해 연산할 때는 stream이 효율적이다.
    

### 3️⃣ 언제 적용하면 좋을 지 파악하기

- 장단점을 토대로 언제 적용하면 좋을 지 파악하기
    - 가독성 측면, 성능 측면 등
