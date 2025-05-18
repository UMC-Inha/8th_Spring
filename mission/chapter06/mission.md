- 미션 1

    ```java
    public Page<MemberMissionDto> findByMemberAndStatus(
                Long memberId,
                MissionStatus status,
                Pageable pageable
        ) {
            QueryResults<MemberMissionDto> qr = jpaQueryFactory
                    .select(Projections.constructor(
                            MemberMissionDto.class,
                            qMission.id,
                            qMission.missionSpec,
                            qMission.reward,
                            qMission.deadline,
                            qMission.store.name,
                            qMemberMission.status.stringValue()
                    ))
                    .from(qMemberMission)
                    .join(qMemberMission.mission, qMission)
                    .join(qMission.store, qStore)
                    .where(qMemberMission.member.id.eq(memberId)
                            .and(qMemberMission.status.eq(status)))
                    .orderBy(qMission.deadline.asc())
                    .offset(pageable.getOffset())
                    .limit(pageable.getPageSize())
                    .fetchResults();
    
            List<MemberMissionDto> content = qr.getResults();
            long total = qr.getTotal();
    
            return new PageImpl<>(content, pageable, total);
        }
    ```

- 미션 2 → 단순 저장인데 굳이 querydsl이 필요없을 것이라고 생각했다.

    ```java
    reviewRepository.save(review);
    ```

- 미션 3

    ```java
    public Page<RegionMissionDto> findByRegion(Long regionId, Pageable pageable) {
            QueryResults<RegionMissionDto> qr;
            qr = jpaQueryFactory
                    .select(Projections.constructor(
                            RegionMissionDto.class,
                            qMission.id,
                            qMission.missionSpec,
                            qMission.reward,
                            qMission.deadline,
                            qStore.name
                    ))
                    .from(qMission)
                    .join(qMission.store, qStore)
                    .join(qStore.region, qRegion)
                    .where(qRegion.id.eq(regionId))
                    .orderBy(qMission.deadline.asc())
                    .offset(pageable.getOffset())
                    .limit(pageable.getPageSize())
                    .fetchResults();
    
            List<RegionMissionDto> content = qr.getResults();
            long total = qr.getTotal();
    
            return new PageImpl<>(content, pageable, total);
        }
    ```

- 미션 4

    ```java
    public MemberResponseDto findMemberDetail(Long memberId) {
            return jpaQueryFactory
                    .select(Projections.constructor(
                            MemberResponseDto.class,
                            qMember.id,
                            qMember.name,
                            qMember.email,
                            qMember.profileImageUrl,
                            qMember.phoneVerified,
                            qMember.phoneNumber,
                            qMember.point
                    ))
                    .from(qMember)
                    .fetchOne();
        }
    ```