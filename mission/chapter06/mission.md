## Q 클래스

``` java
QUserMission um = QUserMission.userMission;
QMission m = QMission.mission;
QStore s = QStore.store;
QRegion r = QRegion.region;
QUser u = QUser.user;
QFoodCategory fc = QFoodCategory.foodCategory;
QReview review = QReview.review;
```

## 1-1. 진행중 미션

``` java
List<Tuple> inProgress = queryFactory
    .select(
        m.missionContent,
        m.rewardPoint,
        s.storeName,
        um.status
    )
    .from(um)
    .join(um.mission, m)
    .join(m.store, s)
    .where(
        um.status.eq(MissionStatus.IN_PROGRESS),
        um.user.id.eq(userId)
    )
    .orderBy(um.updatedAt.desc())
    .offset(page * size)
    .limit(size)
    .fetch();
```

## 1-2. 진행 완료 미션

``` java
List<Tuple> done = queryFactory
    .select(
        m.missionContent,
        m.rewardPoint,
        s.storeName,
        um.status
    )
    .from(um)
    .join(um.mission, m)
    .join(m.store, s)
    .where(
        um.status.eq(MissionStatus.DONE),
        um.user.id.eq(userId)
    )
    .orderBy(um.updatedAt.desc())
    .offset(page * size)
    .limit(size)
    .fetch();
```

## 3-1. 완료 미션 수

```java
Integer count = queryFactory
    .select(u.completedMission)
    .from(u)
    .where(u.id.eq(userId))
    .fetchOne();
```

## 3-2. 안암동에서 도전 가능한 미션 목록

``` java
List<Tuple> available = queryFactory
    .select(
        m.missionTitle,
        m.missionContent,
        m.rewardPoint,
        m.endDate,
        s.storeName,
        fc.categoryName,
        Expressions.numberTemplate(Integer.class, "DATEDIFF({0}, CURRENT_DATE)", m.endDate)
    )
    .from(m)
    .join(m.store, s)
    .join(s.region, r)
    .leftJoin(s.category, fc)
    .leftJoin(um).on(um.mission.eq(m).and(um.user.id.eq(userId)))
    .where(
        r.district.eq("안암동"),
        um.mission.isNull() // 아직 도전 안 한 미션만
    )
    .orderBy(m.endDate.asc())
    .offset(page * size)
    .limit(size)
    .fetch();
```

## 4. 마이페이지

``` java
Tuple myPage = queryFactory
    .select(
        u.name,
        u.email,
        u.isPhoneVarified,
        u.point,
        Expressions.stringTemplate("COALESCE({0}, '미인증')", u.phoneNum)
    )
    .from(u)
    .where(u.id.eq(userId))
    .fetchOne();
```