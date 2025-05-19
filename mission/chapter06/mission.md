# 🔥미션

---

## 2주차 미션 때 했던 해당 화면들에 대해 작성했던 쿼리를 QueryDSL로 작성하여 리팩토링하기

---

### 1번: 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)
```sql
SELECT 
    mr.mission_id, 
    m.title AS mission_name, 
    m.status AS mission_status, 
    m.reward_points AS points_earned,
    mr.completed_date
FROM 
    mission_record AS mr --미션 기록 테이블 별칭 지정
JOIN 
    mission AS m ON mr.mission_id = m.id --미션 테이블 별칭 지정 및 id를 통하여 조인
WHERE 
    mr.user_id = 1 --유저 아이디 입력 (현재는 예시로 1로 두고 진행)
ORDER BY 
    mr.completed_date DESC, mr.mission_id ASC --가장 최근에 완료된 미션이 먼저 표시되도록 내림차순 정렬, mission_id는 오름차순으로 정렬
LIMIT 15 OFFSET 0; --페이징 처리하기 (한번에 최대 15개)
```

QueryDSL로 수정
```java
@Repository
@RequiredArgsConstructor
public class MemberMissionRepositoryImpl implements MemberMissionRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    @Override
    public List<MemberMissionResponse> findMissionsByMemberId(Long memberId, int offset, int limit) {
        QMemberMission memberMission = QMemberMission.memberMission;
        QMission mission = QMission.mission;

        return queryFactory
                .select(Projections.constructor(MemberMissionResponse.class,
                        mission.id,
                        mission.title,
                        mission.status.stringValue(),
                        mission.rewardPoints,
                        memberMission.completedDate
                ))
                .from(memberMission)
                .join(memberMission.mission, mission)
                .where(memberMission.member.id.eq(memberId))
                .orderBy(
                        memberMission.completedDate.desc().nullsLast(),
                        mission.id.asc()
                )
                .offset(offset)
                .limit(limit)
                .fetch();
    }
}
```

### 2번: 리뷰 작성하는 쿼리
```sql
INSERT INTO review ( 
    user_id,   
    store_id, 
    review_text, 
    rating, 
    created_at
) VALUES (
    1, --예시로 사용자 아이디 입력
    5, --가게 ID 입력 
    '내부가 깔끔하고 음식이 맛있어요!.', --리뷰 내용 입력
    4.5, --평점 입력 (0.0 ~ 5.0)
    NOW() --현재 시간 자동 입력
);
```

QueryDSL로 수정
```java
@Service
@RequiredArgsConstructor
public class ReviewService {

    private final MemberRepository memberRepository;
    private final StoreRepository storeRepository;
    private final ReviewRepository reviewRepository;

    public Review createReview(CreateReviewRequest request) {
        Member member = memberRepository.findById(request.getMemberId())
                .orElseThrow(() -> new RuntimeException("존재하지 않는 사용자"));
        Store store = storeRepository.findById(request.getStoreId())
                .orElseThrow(() -> new RuntimeException("존재하지 않는 가게"));

        Review review = Review.builder()
                .member(member)
                .store(store)
                .title(request.getTitle())
                .reviewText(request.getReviewText())
                .rating(request.getRating())
                .build();

        return reviewRepository.save(review);
    }
}
```

### 3번: 홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)
```sql
SELECT 
    m.id AS mission_id,
    m.title AS mission_name,
    m.reward_points,
    m.deadline AS mission_deadline,
    s.store_name AS store_name,
    s.address AS store_address,
    u.points AS user_points, --user 테이블의 사용자 포인트도 가져와야함
    r.name AS region_name --맨 위 화면에 나타나는 지역을 표시하기 위해 가져옴 
FROM 
    mission AS m
JOIN 
    store AS s ON m.store_id = s.id --미션과 가게 정보 연결
JOIN 
    region AS r ON s.id = r.store_id --가게와 지역 연결
JOIN 
    user AS u ON u.id = 1 --특정 user ID를 통해서 조인
WHERE 
    r.name = '안암동' --선택한 지역 (예시로 사진처럼 안암동)
AND 
    m.status = 'active' --활성화된 미션만 조회
ORDER BY 
    m.deadline ASC --마감일 기준으로 오름차순 정렬 (가까운 순서)
LIMIT 15 OFFSET 0; --페이징 처리 (최대 15개)
```

QueryDSL로 수정
```java
@Repository
@RequiredArgsConstructor
public class MissionRepositoryImpl implements MissionRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    @Override
    public List<HomeMissionResponse> findHomeMissionsByRegionAndMember(String regionName, Long memberId, int offset, int limit) {
        QMission mission = QMission.mission;
        QStore store = QStore.store;
        QRegion region = QRegion.region;
        QMember member = QMember.member;

        return queryFactory
                .select(Projections.constructor(HomeMissionResponse.class, // DTO를 통해서 받을 필드 지정하기
                        mission.id,
                        mission.title,
                        mission.rewardPoints,
                        mission.deadline,
                        store.name,
                        store.address,
                        member.points,
                        region.name
                ))
                .from(mission) // 미션을 기준으로
                .join(mission.store, store) // 조인할 것들 조인
                .join(store.region, region)
                .join(member)
                .where(
                        region.name.eq(regionName),
                        mission.status.eq(MissionStatus.CHALLENGING), // 진행중인 미션만
                        member.id.eq(memberId)
                )
                .orderBy(mission.deadline.asc()) // 마감일순으로 정렬
                .offset(offset) // 페이징 처리
                .limit(limit)
                .fetch();
    }
}
```

### 4번: 마이 페이지 하면 쿼리
```sql
--사용자의 정보 및 포인트, 리뷰에 관한 쿼리
SELECT 
    u.id AS user_id,
    u.user_name AS user_nickname,
    u.email AS user_email,
    u.phone_number AS user_phone_number,
    u.phone_verity AS user_phone_verity,
    u.points AS user_points, --현재 유저의 포인트 
    SUM(m.reward_points) AS total_points_earned, --미션 완료 후 지급되는 포인트의 총 합계 
    COUNT(r.id) AS total_review_write --지금까지 작성한 리뷰수를 알기위해 리뷰 식별자를 count
FROM 
    user AS u
LEFT JOIN 
    mission_record AS mr ON u.id = mr.user_id AND mr.completed_date IS NOT NULL --미션을 완료한 경우한 포함시킴 
LEFT JOIN 
    mission AS m ON mr.mission_id = m.id
LEFT JOIN 
    review AS r ON u.id = r.user_id
WHERE 
    u.id = 1 --유저 id 입력
GROUP BY 
    u.id;

--1:1 문의 목록 조회에 관한 쿼리 
SELECT 
    inq.id AS inquiry_id,
    inq.notification_id,
    inq.title AS inquiry_title,
    inq.content AS inquiry_content,
    inq.created_at AS inquiry_created_at
FROM 
    inquiry_notifications AS inq
WHERE 
    inq.user_id = 1 --유저 ID 입력
ORDER BY 
    inq.created_at DESC; --생성일자 기준으로 최신순 정렬 

--알림 설정에 관한 쿼리
SELECT 
    n.id AS notification_id,
    n.message AS notification_message,
    n.status AS notification_status,
    n.created_at AS notification_created_at
    n.updated_at AS notification_updated_at
FROM 
    notification AS n
WHERE 
    n.user_id = 1 --유저 ID 입력
ORDER BY 
    n.created_at DESC;
```

QueryDSL로 수정
```java
@Repository
@RequiredArgsConstructor
public class MyPageRepositoryImpl implements MyPageRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    @Override
    public MyPageUserInfoResponse getUserInfo(Long memberId) {
        QMember member = QMember.member;
        QReview review = QReview.review;
        QMemberMission memberMission = QMemberMission.memberMission;

        return queryFactory
                .select(Projections.constructor(MyPageUserInfoResponse.class,
                        member.id,
                        member.name,
                        member.email,
                        member.phoneNumber,
                        member.status.stringValue().eq("ACTIVE"),
                        member.points,
                        memberMission.mission.rewardPoints.sum().coalesce(0),
                        review.id.count().coalesce(0L)
                ))
                .from(member)
                .leftJoin(memberMission).on(memberMission.member.eq(member).and(memberMission.completedDate.isNotNull())) // 완료된 미션만 조인
                .leftJoin(review).on(review.member.eq(member)) // 리뷰 조인
                .where(member.id.eq(memberId))
                .groupBy(member.id) // count,sum 사용하니깐 groupBy 사용
                .fetchOne();
    }

    @Override
    public List<InquiryResponse> getUserInquiries(Long memberId) {
        QInquiryNotification inquiry = QInquiryNotification.inquiryNotification;

        return queryFactory
                .select(Projections.constructor(InquiryResponse.class,
                        inquiry.id,
                        inquiry.notification.id,
                        inquiry.inquiryTitle,
                        inquiry.inquiryContent,
                        inquiry.createdAt
                ))
                .from(inquiry)
                .where(inquiry.member.id.eq(memberId)) // 해당 id의 유저만
                .orderBy(inquiry.createdAt.desc()) // 최신순으로 정렬
                .fetch();
    }

    @Override
    public List<NotificationResponse> getUserNotifications(Long memberId) {
        QNotification notification = QNotification.notification;

        return queryFactory
                .select(Projections.constructor(NotificationResponse.class,
                        notification.id,
                        notification.message,
                        notification.status.stringValue(),
                        notification.createdAt,
                        notification.updatedAt
                ))
                .from(notification)
                .where(notification.member.id.eq(memberId))
                .orderBy(notification.createdAt.desc())
                .fetch();
    }
}
```