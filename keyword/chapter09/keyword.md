## Spring Data JPA의 Paging

### Page

---

- 전체 데이터 개수와 함께 페이지 정보를 포함하는 페이징 기능
- 전체 데이터 개수 조회를 위한 추가 쿼리 발생
- 페이지 번호, 페이지 크기, 정렬 정보 등 메타데이터 제공
- `PageImpl` 클래스로 직접 구현 가능

예시:

```java
// Repository
public interface ReviewRepository extends JpaRepository<Review, Long> {
    Page<Review> findAllByStore(Store store, Pageable pageable);
    Page<Review> findAllByMember(Member member, Pageable pageable);
}

// QueryDSL
@Override
public Page<MemberMissionIsCompletedResponseDto> findByMemberIdAndMissionStatus(Long memberId, MissionStatus status, Pageable pageable) {
    // 전체 개수 조회
    long total = jpaQueryFactory
            .select(memberMission.count())
            .from(memberMission)
            .where(조건)
            .fetchOne();

    // 페이지 내용 조회
    List<DTO> content = jpaQueryFactory
            .select(Projections.constructor(...))
            .from(memberMission)
            .where(조건)
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();

    // PageImpl 객체
    return new PageImpl<>(content, pageable, total);
}

// 서비스 레이어
@Override
public Page<DTO> getCompletedAndInProgressMission(Long memberId, MissionStatus status, Pageable pageable) {
    Page<DTO> memberMissions = repository.findByMemberIdAndMissionStatus(memberId, status, pageable);
    return memberMissions.map(mm -> DTO.builder()...build());
}

```

- 레포지토리에서 페이징된 데이터를 조회 후, DTO로 변환하는 메소드
- `Page` 객체의 `map()` 메소드를 활용하여 데이터 변환
- 동작 과정
    - `repository.findByMemberIdAndMissionStatus()` 호출로 페이징된 데이터 조회
    - 조회된 `Page<DTO>` 객체에 `map()` 메소드 적용
- 장점
    - 페이지 메타데이터 유지하면서 내부 데이터 변환 가능
    - 페이징 엔티티에서 DTO로 변환 가능
    - 불필요한 반복문 작성 x

### Slice

---

- 전체 데이터 개수를 조회하지 않고 다음 페이지 존재 여부만 확인
- 요청한 페이지 크기보다 하나 더 많은 데이터를 조회하여 다음 페이지 존재 여부 확인
- 전체 데이터 개수 조회 쿼리가 없어 Page보다 성능이 좋음
- 무한 스크롤, "더 보기" 버튼 등의 UI 패턴에 적합

```java
@Repository
@RequiredArgsConstructor
public class MissionRepositoryImpl implements MissionRepositoryCustom {
    private final JPAQueryFactory queryFactory;
    
    @Override
    public Slice<MissionDto> findMissionsByLocation(String location, Pageable pageable) {
        // 페이지 크기 + 1개 조회
        int pageSize = pageable.getPageSize();
        
        List<MissionDto> content = queryFactory
                .select(Projections.constructor(MissionDto.class,
                        mission.id,
                        mission.missionSpec,
                        store.name))
                .from(mission)
                .join(mission.store, store)
                .where(mission.local.eq(location))
                .orderBy(mission.deadline.asc())
                .offset(pageable.getOffset())
                .limit(pageSize + 1) // 요청 크기 + 1 조회
                .fetch();
        
        // 다음 페이지 존재 확인
        boolean hasNext = content.size() > pageSize;
        
        // 실제 반환
        if (hasNext) {
            content = content.subList(0, pageSize);
        }
        
        return new SliceImpl<>(content, pageable, hasNext);
    }
}
```

## 객체 그래프 탐색

---

- JPA에서 연관된 엔티티를 탐색하는 방법
- 엔티티 간 관계를 통해 객체 그래프를 따라 연관 엔티티에 접근

**연관 관계 설정**

```java
// MemberMission 엔티티
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "mission_id")
private Mission mission;

@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "member_id")
private Member member;

// Member 엔티티
@OneToMany(mappedBy = "member", cascade = CascadeType.ALL)
private List<MemberMission> memberMissionList = new ArrayList<>();

@OneToMany(mappedBy = "member", cascade = CascadeType.ALL)
private List<PointMember> pointMemberList = new ArrayList<>();

```

- 특징
    - **지연 로딩(Lazy Loading)**
        - `fetch = FetchType.LAZY`로 설정
        - 필요할 때만 연관 엔티티 로딩
        - N+1 문제 발생 가능성 있음
    - **즉시 로딩(Eager Loading)**
        - `FetchType.EAGER`가 기본값
        - 엔티티 로딩 시 연관 엔티티도 함께 로딩
        - 불필요한 데이터까지 함께 로딩될 수 있음
    - **N+1 문제**
        - 지연 로딩 사용 시 연관 엔티티 조회마다 추가 쿼리 발생
        - 성능 저하의 주요 원인
    - **해결책**
        - JPQL의 fetch join
        - EntityGraph
        - BatchSize 설정
        - QueryDSL 사용

### 예시:

```java
jpaQueryFactory
    .select(Projections.constructor(DTO.class, ...))
    .from(memberMission)
    .join(memberMission.mission, mission)  // MemberMission → Mission
    .join(mission.store, store)            // Mission → Store
    .join(pointMission.point, point)       // PointMission → Point
    .where(조건)
    .fetch();

```

- QueryDSL을 사용하여 필요한 엔티티들을 조인
- DTO로 직접 변환하는 방식으로 N+1 문제 해결
    - JOIN을 사용하므로 한 번의 쿼리로 부모와 자식 데이터를 모두 가져오고
    - 엔티티가 아닌 DTO로 결과를 바로 매핑하므로 LAZY 발생 x