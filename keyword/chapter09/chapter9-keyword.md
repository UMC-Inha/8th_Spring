# 키워드 미션

# Spring Data JPA의 Paging

## `Page<T>` vs `Slice<T>` 차이

| 항목 | Page | Slice |
| --- | --- | --- |
| 반환 정보 | **전체 개수, 페이지 정보, 콘텐츠** | **다음 페이지 존재 여부 + 콘텐츠** |
| 전체 데이터 수 | `count(*)` 쿼리로 총 개수 가져옴 (비용 ↑) | 없음 (성능 ↑) |
| 대표 메서드 | `findAll(Pageable pageable)` | `findAllBySomething(Pageable pageable)` |
| 사용 시기 | 전체 페이지 수나 개수 필요할 때 | 무한 스크롤 등, 다음 페이지 유무만 필요할 때 |

## Page

- `Page<T>`는 **전체 데이터 개수(total count)** 를 포함한 **완전한 페이징 정보**를 제공하는 객체이다.
- 내부적으로 **count 쿼리**가 추가적으로 실행되어 전체 항목 수를 계산한다.

### 사용 상황

- 전체 페이지 수(`getTotalPages()`), 총 항목 수(`getTotalElements()`)를 **UI에 표시하거나**,
- "총 n건 중 m건을 보고 있습니다"와 같은 메시지를 띄울 필요가 있을 때 사용한다.

### 주요 메서드

```

Page<User> users = userRepository.findAll(PageRequest.of(page, size));
```

- `getContent()` → 실제 데이터 목록 (`List<User>`)
- `getTotalElements()` → 전체 데이터 개수
- `getTotalPages()` → 전체 페이지 수
- `isFirst()` / `isLast()` → 첫/마지막 페이지 여부
- `hasNext()` / `hasPrevious()` → 다음/이전 페이지 존재 여부

### 장점

- UI에 필요한 모든 페이징 정보를 포함하고 있음
- 페이지네이션 UI 구성에 적합 (전체 페이지 수, 현재 페이지 등)

### 단점

- 추가적인 **count 쿼리** 실행 (대용량 데이터 시 성능 저하 우려)

---

## Slice

- `Slice<T>`는 다음 페이지 존재 여부까지만 확인할 수 있는 **경량 페이징 방식이다.**
- 전체 데이터 개수를 구하지 않고, **지금 페이지 + 다음 페이지가 있는지만 판단한다.**

### 사용 상황

- **무한 스크롤** 방식에서 주로 사용됨 (예: 인스타그램, 페이스북)
- 전체 페이지 수가 필요 없고, 사용자가 계속 스크롤할 수 있게 하면 됨

### 주요 메서드

```

Slice<User> users = userRepository.findByActiveTrue(PageRequest.of(page, size));
```

- `getContent()` → 실제 데이터 목록
- `hasNext()` → 다음 페이지 존재 여부
- `isFirst()` / `isLast()`

> 주의: getTotalPages(), getTotalElements()는 없음
>

### 장점

- count 쿼리가 없어서 빠름
- 대용량 데이터에서 매우 유리

### 단점

- 전체 데이터 수를 알 수 없음
- 페이지 번호나 전체 페이지 수를 UI에 표시할 수 없음

# 객체 그래프 탐색

JPA에서 **연관된 엔티티의 속성에 접근**하는 것 → `연관관계를 따라가는 것`

### 예시 ) 엔티티 구조

```
@Entity
public class UserMission {
    @Id
    private Long id;

    @ManyToOne
    private User user;

    @ManyToOne
    private Mission mission;

```

```
@Entity
public class Mission {
    @Id
    private Long missionId;

    private String title;
}
```

---

### 탐색 예시

```
userMission.getMission().getTitle()
```

여기서 수행되는 과정은:

1. `userMission.getMission()` → Mission 엔티티를 참조
2. `mission.getTitle()` → Mission 객체의 필드 접근

> 이런 것이 객체 그래프 탐색이다.
>

### 쿼리 메서드에도 객체 그래프 탐색 가능

```
List<UserMission> findByMission_Title(String title);
```

여기서 `Mission`은 `UserMission`의 필드이며, `title`은 그 안의 필드

→ JPA가 내부적으로 join 해서 처리함 (`join mission where mission.title = ?`)