- **미션 기록**
    - **내가 진행중, 진행 완료한 미션**
        
        `UserMissionRepositoryImpl`
        
        ```java
        @Repository
        @RequiredArgsConstructor
        public class UserMissionRepositoryImpl implements UserMissionRepositoryCustom {
            private final JPAQueryFactory queryFactory;
        
            private final QUserMission userMission = QUserMission.userMission;
            private final QStore store = QStore.store;
            private final QMission mission = QMission.mission;
        
            @Override
            public List<UserMission> findChallengingMissionByUser(Long userId, Pageable pageable) {
                return queryFactory
                        .select(userMission)
                        .join(userMission.mission, mission).fetchJoin()
                        .join(mission.store, store).fetchJoin()
                        .where(userMission.user.id.eq(userId)
                                .and(userMission.status.eq(MissionStatus.CHALLENGING)))
                        .offset(pageable.getOffset())
                        .limit(pageable.getPageSize())
                        .fetch();
            }
        
            @Override
            public List<UserMission> findCompletedMissionByUser(Long userId, Pageable pageable) {
                return queryFactory
                        .selectFrom(userMission)
                        .join(userMission.mission, mission).fetchJoin()
                        .join(mission.store, store).fetchJoin()
                        .where(userMission.user.id.eq(userId)
                                .and(userMission.status.eq(MissionStatus.COMPLETED)))
                        .offset(pageable.getOffset())
                        .limit(pageable.getPageSize())
                        .fetch();
            }
        
            @Override
            public Long countCompletedMissionByUser(Long userId, String regionName) {
                return queryFactory
                        .select(userMission.count())
                        .from(userMission)
                        .join(userMission.mission, mission)
                        .join(mission.store, store)
                        .where(userMission.user.id.eq(userId)
                                .and(userMission.status.eq(MissionStatus.COMPLETED))
                                .and(store.region.name.eq(regionName)))
                        .fetchOne();
            }
        }
        ```
        
        - 특정 `userId`의 미션 중 **진행 중 또는 완료된 미션을** 가져옴
        - 페이징 처리 적용
        - `fetchJoin()`으로 `mission`, `store`까지 연관 객체를 **한번에 조회**
        
        `*UserMissionRepositoryCustom*`
        
        ```java
        @Repository
        public interface UserMissionRepositoryCustom {
            //List<UserMission> findByMemberId(Long memberId);
            List<UserMission> findChallengingMissionByUser(Long UserId, Pageable pageable);
            List<UserMission> findCompletedMissionByUser(Long UserId, Pageable pageable);
            Long countCompletedMissionByUser(Long UserId, String regionName);
        }
        ```
        
        `*UserMissionRepository*`
        
        ```java
        public interface UserMissionRepository extends JpaRepository<UserMission, Long>, UserMissionRepositoryCustom {
            // 특정 회원이 특정 미션에 도전 중인지 확인
            boolean existsByUserIdAndMissionIdAndStatus(Long userId, Long missionId, MissionStatus status);
            boolean existsByMissionIdAndStatus(Long missionId, MissionStatus missionStatus);
        
            // 회원-미션 관계 조회
            Optional<UserMission> findByUserIdAndMissionId(Long userId, Long missionId);
            // 특정 회원의 진행 중인 미션 목록 조회
            Page<UserMission> findByUserIdAndStatus(Long userId, MissionStatus status, Pageable pageable);
        }
        ```
        
    - 리뷰 작성
        
        `*ReviewRepositoryCustom*`
        
        ```java
        public interface ReviewRepositoryCustom {
            public Long save(User user, Store store, String content, BigDecimal rating);
        }
        ```
        
        `ReviewRepositoryImpl`
        
        ```java
        @Repository
        @RequiredArgsConstructor
        public class ReviewRepositoryImpl implements ReviewRepositoryCustom {
            private final JPAQueryFactory queryFactory;
            private final QReview review = QReview.review;
        
            //리뷰 작성 및 저장
            @Override
            public Long save(User user, Store store, String content, BigDecimal rating) {
                return queryFactory
                        .insert(review)
                        .columns(review.user, review.store, review.content, review.rating)
                        .values(user, store, content, rating)
                        .execute();
            }
        }
        ```
        
        - 회원이 특정 가게에 대해 리뷰를 작성할 때 `Review` 테이블에 삽입
        - `User`, `Store`, `Content`, `Rating` 필드를 저장
        
        `*ReviewRepository*`
        
        ```java
        @Repository
        public interface ReviewRepository extends JpaRepository<Review, Long>, ReviewRepositoryCustom{
            @Modifying
            @Transactional
            @Query(value = "INSERT INTO Review (user_id, store_id, content, rating) VALUES (:userId, :storeId, :content, :rating)", nativeQuery = true)
            void insertReview(@Param("memberId") Long userId,
                              @Param("storeId") Long storeId,
                              @Param("content") String content,
                              @Param("rating") BigDecimal rating
            );
        
            Page<Review> findByUserId(long id, Pageable unpaged);
        }
        ```
        
    - 홈화면
        
        `MissionRepositoryImpl`
        
        ```java
        @Repository
        @RequiredArgsConstructor
        public class MissionRepositoryImpl implements MissionRepositoryCustom{
            private final JPAQueryFactory queryFactory;
            private final QMission mission = QMission.mission;
            private final QStore store = QStore.store;
            private final QUserMission userMission = QUserMission.userMission;
        
            // 홈화면 : 지역 기반 미션 조회
            @Override
            public Page<MissionDto> findMissionByRegion(long regionId, Pageable pageable) {
                List<MissionDto> content = queryFactory
                        .select(new QMissionDto(
                                mission.id,
                                mission.missionSpec,
                                store.name,
                                mission.point,
                                mission.status
                        ))
                        .from(mission)
                        .join(mission.store, store)
                        .where(store.region.id.eq(regionId))
                        .offset(pageable.getOffset())
                        .limit(pageable.getPageSize())
                        .fetch();
        
                long total = queryFactory
                        .select(mission.count())
                        .from(mission)
                        .join(mission.store, store)
                        .where(store.region.id.eq(regionId))
                        .fetchOne();
        
                return new PageImpl<>(content, pageable, total);
            }
        
            public List<MissionDto> findAvailableMissionsByUserAndRegion(Long userId, Long regionId, Pageable pageable) {
                return queryFactory
                        .select(new QMissionDto(
                                mission.id,
                                mission.missionSpec,
                                store.name,
                                mission.point,
                                mission.status
                        ))
                        .from(mission)
                        .join(mission.store, store)
                        .where(
                                store.region.id.eq(regionId),
                                mission.status.eq(MissionStatus.OPEN),
                                mission.id.notIn(
                                        JPAExpressions
                                                .select(userMission.mission.id)
                                                .from(userMission)
                                                .where(userMission.user.id.eq(userId))
                                )
                        )
                        .offset(pageable.getOffset())
                        .limit(pageable.getPageSize())
                        .fetch();
            }
        
        }
        ```
        
        - **성공 미션**: 특정 유저가 특정 지역에서 완료한 미션 수를 세는 쿼리
        - **도전 가능 미션**: 해당 유저가 아직 수행하지 않은 미션들 중, 선택한 지역에 있는 OPEN 상태 미션 목록 조회
        
        `*MissionRepository*`
        
        ```java
        public interface MissionRepository extends JpaRepository<Mission, Long>, MissionRepositoryCustom {
            // 미션이 이미 도전 중인지 확인
            boolean existsByStatusAndId(MissionStatus status, Long id);
            // 특정 가게의 미션 목록 조회
            Page<Mission> findByStoreId(Long storeId, Pageable pageable);
        }
        ```
        
    - 마이페이지
        
        `UserRepositoryImpl`
        
        ```java
        @Repository
        @RequiredArgsConstructor
        public class UserRepositoryImpl implements UserRepositoryCustom {
            private final JPAQueryFactory queryFactory;
            private final QUser user = QUser.user;
        
            @Override
            public UserInfoDto findUserInfoById(Long userId) {
                return queryFactory
                        .select(Projections.constructor(UserInfoDto.class,
                                user.name,
                                user.email,
                                user.phoneNumber,
                                user.point))
                        .from(user)
                        .where(user.id.eq(userId))
                        .fetchOne();
            }
        }
        ```
        
        - 마이페이지에 표시할 **회원의 기본 정보** 조회
        - `userId`로 회원 조회하고 필요한 정보만 추려서 DTO로 반환