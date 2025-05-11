1. 
```sql
QMemberMission mm = QMemberMission.memberMission;
QMission m = QMission.mission;
QStore s = QStore.store;

List<Tuple> result = queryFactory
    .select(
        m.id,
        m.title,
        m.missionBody,
        s.name,
        mm.status,
        m.reward
    )
    .from(mm)
    .leftJoin(mm.mission, m)
    .leftJoin(m.store, s)
    .where(mm.member.id.eq(memberId))
    .orderBy(mm.createdAt.desc())
    .offset((page - 1) * 5)
    .limit(5)
    .fetch();
```


2. 
```sql
Review review = Review.builder()
    .store(store)                   
    .member(member)                   
    .starRating(0.0)                  
    .reviewBody("맛있어요")             
    .build();

reviewRepository.save(review);
```


3. 
```sql
QMission m = QMission.mission;
QMemberMission mm = QMemberMission.memberMission;
QStore s = QStore.store;
QFoodCategory fc = QFoodCategory.foodCategory;

List<Tuple> results = queryFactory
    .select(s.name, fc.name, m.missionBody, m.reward,
        Expressions.numberTemplate(Integer.class, "DATEDIFF({0}, CURRENT_DATE)", mm.dueDate).as("daysRemaining"))
    .from(m)
    .leftJoin(mm).on(mm.mission.eq(m))
    .leftJoin(s).on(s.eq(m.store))
    .leftJoin(fc).on(fc.eq(s.category))
    .where(s.address.eq(targetAddress)
        .and(mm.mission.isNull()))
    .orderBy(m.createdAt.desc())
    .offset((page - 1) * 5)
    .limit(5)
    .fetch();

```


4. 
```sql
Tuple result = queryFactory
    .select(
        m.id,
        m.name,
        m.email,
        m.point,
        certificationStatus
    )
    .from(m)
    .where(m.id.eq(memberId))
    .fetchOne();

//certificationStatus는 CaseBuilder()로 정의되어 있다고 가정
```