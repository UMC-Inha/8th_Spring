**모든 레포지토리, 서비스 명은 다음 원칙으로 명명**

- **Repository**:
    - **엔티티·필드 중심**: `findBy…`, `save()`, `countBy…`, `deleteBy…`
- **Service**:
    - 해당 비즈니스에 적절한 이름 선택, 행위 중심: `getMemberCompletedMissionCount(), ...`

### 내가 진행 중, 진행 완료한 미션

---

```sql
SELECT
    m.message,
    p.point,
    mi.is_completed,
    s.name
FROM
    member_mission AS mi
        JOIN mission AS m ON m.mission_id = mi.mission_id
        JOIN store AS s ON s.store_id = m.store_id
        JOIN point_mission AS pm ON pm.mission_id = m.mission_id
        JOIN point AS p ON p.point_id = pm.point_id
WHERE
    mi.member_id = [특정_회원_ID]
ORDER BY
    mi.updated_at DESC
LIMIT 7
OFFSET ?; //(N-1)*7

```

**MemberMissionBaseDto 클래스**

```sql
public class MemberMissionBaseDto {
    private String missionSpec;
    private Integer point;
    private String storeName;
}
```

**MemberMissionResponseDto 클래스**

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class MemberMissionIsCompletedResponseDto extends MemberMissionBaseDto {
    private Boolean isCompleted;
}
```

**MemberMissionRepositoryCustom 인터페이스 (레포지토리)**

```java
public interface MemberMissionRepositoryCustom {
    List<MemberMissionIsCompletedResponseDto> findByMemberIdAndMissionStatus(Long memberId, MissionStatus status, int page);
}
```

**MemberMissionRepositoryImpl 구현체 (레포지토리)**

```java
@Repository
@RequiredArgsConstructor
public class MemberMissionRepositoryImpl implements MemberMissionRepositoryCustom {
    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public List<MemberMissionIsCompletedResponseDto> findByMemberIdAndMissionStatus(Long memberId, MissionStatus status, int page) {

        QMemberMission memberMission = QMemberMission.memberMission;
        QMission mission = QMission.mission;
        QStore store = QStore.store;
        QPointMission pointMission = QPointMission.pointMission;
        QPoint point = QPoint.point;
        int pageSize = 7; //가져올 요소 수
        int offset = (page - 1) * pageSize; //건너뛸 페이지 수
        return jpaQueryFactory
                .select(Projections.constructor(MemberMissionIsCompletedResponseDto.class,
                        mission.missionSpec,
                        point.coin,
                        store.name,
                        memberMission.status.eq(MissionStatus.COMPLETED)))
                .from(memberMission)
                .join(memberMission.mission, mission)
                .join(mission.store, store)
                .join(mission.pointMission, pointMission)
                .join(pointMission.point, point)
                .where(
                        memberMission.member.id.eq(memberId),
                        memberMission.status.eq(status)
                )
                .orderBy(memberMission.updatedAt.desc())
                .limit(pageSize)
                .offset(offset)
                .fetch();
    }
}
```

- `memberMission.status.eq(MissionStatus.COMPLETED` :
    - `memberMission.status.eq`를 통해 `COMPLETED` 인 경우만 **TRUE** 전송해서 성공인지 아닌지 구별
- **`memberMission.status.eq(status)` :**
    - status == MissionStatus.IN_PROGRESS → "진행 중인 미션만 조회"
    - status == MissionStatus.COMPLETED → "완료된 미션만 조회"
- `.join(memberMission.mission, mission)` :
    - 두 엔티티가 매핑 관계일 경우 **JOIN** 시 자동으로 외래키 컬럼을 타고 테이블과 **JOIN** 해줌
    - 나중에 외래키 필드명이 바뀌어도 Q타입 객체만 다시 생성하면 자동으로 코드에 반영됨
    - 단일키일 때 → `.join(member.mission, mission)` 자주 씀
    - 복합키일 때 → `.join(mission).on(...)` 명시적으로 사용
- **Projections.constructor :**
    - QueryDSL 결과를 DTO 객체로 매핑할 때 사용
    - 다음과 같이 사용

      Projections.constructor(
      어떤 DTO로 만들지,
      생성자 파라미터 1,
      생성자 파라미터 2,
      , ...)

    - 생성자의 매개변수 순서에 따라 필드를 매핑

**getCompletedAndInProgressMission** 메소드 (서비스)

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberMissionServiceImpl implements MemberMissionService{
    private final MemberMissionRepositoryImpl memberMissionRepository;

    @Override
    public List<MemberMissionIsCompletedResponseDto> getCompletedAndInProgressMission(Long memberId, MissionStatus status, int page) {
        List<MemberMissionIsCompletedResponseDto> memberMissions = memberMissionRepository.findByMemberIdAndMissionStatus(memberId, status, page);
        return memberMissions;
    }
}
```

- 수정이 일어나지 않는 경우 `readOnly = true` 설정하는 게 안전함
- QueryDSL로부터 반환받은 DTO 그대로 반환

### 리뷰 작성

---

```sql
INSERT INTO
    review(store_id, member_id, message, star_rating)
VALUES
    ([회원_ID], [상점_ID], [리뷰 작성 내용], [별점]);

```

**ReviewRequestDto 클래스**

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ReviewSaveRequestDto {
    private Long storeId;
    private Long memberId;
    private String body;
    private float score;
    private List<Image> images;
}
```

**ReviewResponseDto 클래스**

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ReviewSaveResponseDto {
    private String body;
    private float score;
    private String storeName;
    private List<String> imageList;

    public static ReviewSaveResponseDto fromEntity(Review review) {
        return ReviewResponseDto.builder()
                .body(review.getBody())
                .score(review.getScore())
                .storeName(review.getStore().getName())
                .imageList(review.getImageList().stream().map(Image::getUrl).collect(Collectors.toList()))
                .build();
    }
}
```

**ReviewRepositoryCustom 인터페이스 (레포지토리)**

```java
public interface ReviewRepositoryCustom {
    Review saveReviewByMemberIdAndStoreId(ReviewSaveRequestDto requestDto);
}
```

**saveReviewByMemberIdAndStoreId 메소드 (레포지토리)**

```java
@Repository
@RequiredArgsConstructor
public class ReviewRepositoryImpl implements ReviewRepositoryCustom{
    private final JPAQueryFactory queryFactory;
    private EntityManager entityManager;

    @Override
    @Transactional
    public Review saveReviewByMemberIdAndStoreId(ReviewSaveRequestDto requestDto) {
        QStore qStore = QStore.store;
        QMember qMember = QMember.member;

        Store store = queryFactory
                .selectFrom(qStore)
                .where(qStore.id.eq(requestDto.getStoreId()))
                .fetchOne();
        Member member = queryFactory
                .selectFrom(qMember)
                .where(qMember.id.eq(requestDto.getMemberId()))
                .fetchOne();

        Review review = Review.builder()
                .body(requestDto.getBody())
                .score(requestDto.getScore())
                .store(store)
                .imageList(requestDto.getImages())
                .build();

        entityManager.persist(review);
        return review;
    }
}
```

- store, member 조회를 위해 queryDSL 사용
- **requestDto**
    - 잘못된 데이터가 넘어오는 것을 사전에 방지할 수 있음
    - 요청 시 필요한 데이터를 사전에 정의한 필드로 받을 수 있음
- EntityManager
    - Review 저장이므로 `.persist`로 1차 캐시에 엔티티 저장

**saveReview 메소드 (서비스)**

```java
@Service
@RequiredArgsConstructor
@Transactional
public class ReviewQueryServiceImpl implements ReviewQueryService {

    private final ReviewRepositoryImpl reviewRepository;

    @Override
    public ReviewSaveResponseDto saveReview(ReviewSaveRequestDto reviewRequestDto) {
        Review review = reviewRepository.saveReviewByMemberIdAndStoreId(reviewRequestDto);
        return ReviewSaveResponseDto.EntityToDto(review);
    }
}
```

- **responseDto**
    - 엔티티 그대로 넘겨줄 경우 불필요한 필드가 전부 넘어가게 되기 때문에 Dto 생성 후 넘겨주기

### 홈 화면

---

```sql
SELECT
    m.message,
    p.point,
    mm.is_completed,
    s.name AS store_name,
    f.name AS food_name
FROM
    FROM
    member_mission AS mm
        JOIN mission AS m ON m.mission_id = mm.mission_id
        JOIN store AS s ON s.store_id = m.store_id
        JOIN point_mission AS pm ON pm.mission_id = m.mission_id
        JOIN point AS p ON p.point_id = pm.point_id
        LEFT JOIN store_food AS sf ON s.store_id = sf.store_id
        LEFT JOIN food AS f ON f.food_id = sf.food_id
WHERE
    mi.distance_from_member <= [거리_값]
ORDER BY
    m.started_at DESC
LIMIT 7
OFFSET ?; //(N-1)*7

```

### 지역 기반 미션

**MemberMissionByLocationResponseDto 클래스**

```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class MemberMissionByLocationResponseDto extends MemberMissionBaseDto{
    private String categoryName;
    private LocalDateTime deadline;
}
```

**findByMemberIdAndMissionLocation 메소드(레포지토리)**

```java
 @Override
    public List<MemberMissionByLocationResponseDto> findByMemberIdAndMissionLocation(Long memberId, String location, int page) {
        QMission mission = QMission.mission;
        QStore store = QStore.store;
        QPointMission pointMission = QPointMission.pointMission;
        QPoint point = QPoint.point;
        QStoreCategory storeCategory = QStoreCategory.storeCategory;
        QCategory category = QCategory.category;
        QMemberMission memberMission = QMemberMission.memberMission;

        int pageSize = 7;
        int offset = (page - 1) * pageSize;

        return jpaQueryFactory
                .select(Projections.constructor(MemberMissionByLocationResponseDto.class,
                        mission.missionSpec,
                        point.coin,
                        store.name,
                        category.name,
                        mission.deadline))
                .from(memberMission)
                .join(memberMission.mission, mission)
                .join(mission.store, store)
                .join(mission.pointMission, pointMission)
                .join(pointMission.point, point)
                .leftJoin(storeCategory).on(store.id.eq(storeCategory.store.id))
                .leftJoin(category).on(storeCategory.category.id.eq(category.id))
                .where(
                        memberMission.member.id.eq(memberId),
                        memberMission.status.eq(MissionStatus.NOT_STARTED),
                        mission.local.eq(location),
                        mission.deadline.after(LocalDateTime.now())
                )
                .orderBy(mission.deadline.asc())
                .limit(pageSize)
                .offset(offset)
                .fetch();

    }
```

- 특정 지역에 해당하는 미션만 조회 (local 변수 활용)
- `mission.deadline.after(LocalDateTime.now())` : 마감 시간이 현재 시간의 이후 시간인 미션만 필터링하는 조건
    - 아직 마감되지 않은 미션만 조회
    - `.after()` : 인자로 주어진 시간보다 이후인 경우 **true** 반환
- **MissionStatus :** `NOT_STARTED`와 일치하는 미션만 조회
- `deadline` 같이 전송

**getMissionByLocal 메소드(서비스)**

```java
@Override
    public List<MemberMissionByLocationResponseDto> getMissionByLocal(Long memberId, String location, int page) {
        return memberMissionRepository.findByMemberIdAndMissionLocation(memberId, location, page);
    }
```

### 회원 완료한 미션 수

**countByMemberId 메소드(레포지토리)**

```java
@Override
    public Long countByMemberId(Long memberId) {
        QMemberMission memberMission = QMemberMission.memberMission;
        return jpaQueryFactory
                .select(memberMission.count())
                .from(memberMission)
                .where(
                        memberMission.member.id.eq(memberId),
                        memberMission.status.eq(MissionStatus.COMPLETED)
                )
                .fetchOne();
    }
```

**getCompletedMissionCount 메소드 (서비스)**

```java
@Override
    public Long getCompletedMissionCount(Long memberId) {
        return memberMissionRepository.countByMemberId(memberId);
    }
```

### 마이페이지

---

```sql
SELECT
    SUM(p.point) AS total_earned_points,
    m.email,
    m.phone_number
    m.name
FROM
    member AS m
        JOIN point_member AS pm ON pm.member_id = m.member_id
        JOIN point AS p ON pm.point_id = p.point_id
WHERE p.point_type = 'EARNED'
  AND m.member_id = [특정_회원_ID]

```

**PointMemberRepositoryCustom 인터페이스 (레포지토리)**

```java
public interface PointMemberRepositoryCustom {
    MemberInfoResponseDto findByMemberId(Long memberId);
}
```

findByMemberId 메소드 (레포지토리)

```java
@Repository
@RequiredArgsConstructor
public class PointMemberRepositoryImpl implements PointMemberRepositoryCustom{
    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public MemberInfoResponseDto findByMemberId(Long memberId) {
        QMember member = QMember.member;
        QPointMember pointMember = QPointMember.pointMember;
        QPoint point = QPoint.point;

        return jpaQueryFactory
                .select(Projections.constructor(MemberInfoResponseDto.class,
                        point.coin.sum(),
                        member.email,
                        member.phoneNumber,
                        member.name))
                .from(member)
                .leftJoin(pointMember.member, member)
                .leftJoin(point.pointMemberList, pointMember)
                .where(
                        member.id.eq(memberId),
                        pointMember.pointStatus.eq(PointStatus.EARNED)
                )
                .groupBy(member.id, member.email, member.phoneNumber, member.name)
                .fetchOne();
    }
}
```

- point 합을 구하기 위해 point 필드 제외 나머지 `groupBy` 안에 선언

**getMemberInfo 메소드 (서비스)**

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberQueryServiceImpl implements MemberQueryService{
    private final PointMemberRepositoryImpl pointMemberRepository;

    @Override
    public MemberInfoResponseDto getMemberInfo(Long memberId) {
        return pointMemberRepository.findByMemberId(memberId);
    }
}
```