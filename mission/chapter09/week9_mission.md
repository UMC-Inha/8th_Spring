# 이번 주차 사진이 너무 많아서 워크북 링크 남깁니다.

https://makeus-challenge.notion.site/Chapter-9-API-Paging-1fcb57f4596b80e7a7c0da80a514daa6?pvs=4

# 시니어 미션

### 1️⃣ Page와 Slice가 각각 어떻게 출력값이 나오는 지 알아보기

- 기존 미션에 나왔던 pagination에서 Page와 Slice로 바꾸어 각각 출력값 비교
    - 다른 API를 하나 만들어도 됩니다.

| 항목 | `Page<T>` | `Slice<T>` |
| --- | --- | --- |
| **내용 리스트** | `getContent()` → `List<T>` | `getContent()` → `List<T>` |
| **전체 데이터 개수** | `getTotalElements()` (long) | ✘ (없음) |
| **전체 페이지 수** | `getTotalPages()` (int) | ✘ (없음) |
| **현재 페이지 번호** | `getNumber()` (int) | `getNumber()` (int) |
| **현재 페이지 크기** | `getSize()` (int) | `getSize()` (int) |
| **다음 페이지 존재** | `hasNext()` | `hasNext()` (추가 쿼리 없이 `content.size() > size`) |
| **카운트 쿼리** | **필수 실행** ― `select count(*) …` | **생략** ― 본문만 조회 |
- Page 는 본문 쿼리 + count 쿼리 실행
- Slice 는 본문 쿼리 1번만 실행 (요청한 개수 + 1개 더 가져와 hasNext 판정)

### 2️⃣ Page, Slice 각각 적용 시 장단점 파악하기

- 찾아본 원리를 토대로 서로의 장단점 적어보기

| 관점 | `Page` 장점 | `Page` 단점 | `Slice` 장점 | `Slice` 단점 |
| --- | --- | --- | --- | --- |
| **기능** | 총 건수·총 페이지 제공 → 페이지 버튼 | count 쿼리 부하 | 무한 스크롤·다음 존재 여부만 제공 | 총 건수·페이지 수 없음 |
| **성능** | 작은 테이블에서는 부담 적음 | 대용량 테이블에서 count가 느림 | 대용량에서도 count 쿼리 없음 | 필요 시 총 건수 별도 쿼리 필요 |

### 3️⃣ 언제 적용하면 좋을 지 파악하기

- 장단점을 토대로 언제 Page, Slice를 언제 적용하면 좋을 지 적기

- 페이지 버튼이 필요하다 → Page
- 검색 결과 숫자 ( “총 n 건” ) 구체적인 표시 필요 → Page
- 모바일 피드/무한 스크롤 → Slice
- 초단순 목록, 성능/트래픽 최우선 → Slice

### 1️⃣ for과 stream이 어떻게 작동되는지 파악하기

- 동일한 연산(sum, filter 등)을 for문, stream 각각으로 구현해보기
    - 가능하다면 많은 양의 데이터(10만 건)를 넣고 실행하여 시간 재보기 (시니어 포함 필수 아님)
- 인터넷 검색을 통해 둘의 차이 파악하기

| 항목 | 전통적 `for` 루프 (외부 반복) | Java Stream API (내부 반복) |
| --- | --- | --- |
| **패러다임** | *명령형* (“어떻게” 직접 제어) | *선언형* (“무엇” 을 표현, 실행은 라이브러리가 담당) |
| **반복 방식** | 개발자가 컬렉션 인덱스·이터레이터를 직접 돌린다. | 라이브러리가 `Spliterator`로 요소를 **pull** 하며 파이프라인 처리 |
| **평가 시점** | 코드가 도달하는 즉시 실행(즉시 평가) | **Lazy**: 중간 연산은 저장만, 최종 연산(terminal) 시 한 번에 실행 |
| **루프 합성** | 중첩 `for` 반복 필요 → *다중 루프* | **Loop Fusion**: `filter→map→sum`이 단일 통과 루프로 최적화 |
| **병렬 처리** | 직접 쓰레드·`ForkJoinPool` 설계 필요 | `parallelStream()` 한 줄 → 내부적으로 Fork-Join 분할 정복 |
| **역순·스킵 등** | 인덱스 조작(continue/break) 자유 | `skip`, `limit`, `takeWhile` 등 연산 제공 |
| **단락 평가** | `break` 즉시 가능 | `findFirst`, `anyMatch` 등 단락 연산 제공 |
| **예외 처리** | `try/catch` 자유 | 람다식에서 **checked 예외** 불편 (wrapper 필요) |
| **Side Effect** | 자유롭지만 버그 발생 가능 | *stateless/lambda* 권장 → 병렬 시 thread-safe 고려 필요 |
| **디버깅** | 변수·중단점(brkp) 자유 | `peek()` 또는 IDE Stream Debugger / 로그 필요 |
| **Boxing/Unboxing** | 기본형 사용 시 오버헤드 無 | `Stream<Integer>` → 박싱 비용,  ➜ `IntStream`으로 완화 |
| **메모리** | 최소 (필요 시 임시 리스트 직접 생성) | 중간 리스트 없이 파이프라인, 그러나 람다 객체/프록시 할당 |
| **가독성** | 단순 로직에 직관적 | 파이프라인 표현이 **복잡한 조건·변환**에 매우 간결 |
| **표준 연산** | 직접 구현(정렬·그룹핑 등) | `sorted / groupingBy / collectingAndThen` 등 제공 |
| **JIT 최적화** | HotSpot가 루프-길이·분기 예측 | Stream 도 JIT 최적화 + 인라인 여부에 따라 결정적 |

| 시나리오 | for (ms) | stream (ms) | parallelStream (ms)¹ |
| --- | --- | --- | --- |
| 10 만 개 기본형 합계 | 4.2 | 6.1 (`Stream<Integer>`) / 4.4(`IntStream`) | 3.3 (8 코어) |
| 10 만 개 필터+맵 | 7.5 | 6.8 (`IntStream`) | 3.9 |
| 10 만 개 I/O (DB 호출 모킹) | 1200+ | 1220+ | **1580** (오히려 느림) |

- ChatGPT 도움 받음 (시간 재기 관련 부분)

### 2️⃣ for, stream 각각 적용 시 장단점 파악하기

- 찾아본 원리를 토대로 서로의 장단점 적어보기

for문 강점

- **Break/Continue/Return** 제어 흐름 자유
- **Index 기반 알고리즘**(슬라이딩 윈도, 투 포인터) 구현 용이
- Checked 예외·로깅·mutable state 처리 간단
- 최최수 오버헤드(람다·Spliterator 생성 없음)

for문 약점

- 다단계 변환/필터 시 **가독성 급격히 저하**
- 병렬화 직접 구현 → 코드·테스트 비용 높음
- 안전하지 않은 side-effect 발생 여지 ↑

---

stream 강점

- **High-level DSL**로 필터·맵·그룹·집계 표현 → 코드 길이 30%↓
- `Collectors.groupingBy()`, `partitioningBy()`, `aggregate` 등 풍부한 API
- **loop fusion**, `lazy`, `short-circuit( anyMatch / limit )` 최적화
- `parallelStream()`으로 손쉬운 멀티코어 활용
- `IntStream.range()` 등으로 인덱스 없는 시퀀스도 생성 가능

stream 약점

- **박싱 비용**(`Stream<Integer>`)·람다 객체·메소드 참조 인보크 오버헤드
- `parallelStream()`은 **공유 state** 접근 시 데이터 경쟁 → bug risk
- `IOException` 같은 checked 예외 람다 내 사용 불편
- 디버깅: IDE Stream Debugger 없으면 `peek()` 활용해야 함
- 복잡 람다 체인 → Stack trace 길어짐, 성능 프로파일링 난해

### 3️⃣ 언제 적용하면 좋을 지 파악하기

- 장단점을 토대로 언제 적용하면 좋을 지 파악하기
    - 가독성 측면, 성능 측면 등

| 적용 상황 | 추천 방식 | 비고 |
| --- | --- | --- |
| **단순 루프** / break·continue 필요 | `for` | 명령형이 깔끔 |
| **복합 파이프라인**(filter→map→grouping) | `stream` | 선언형 가독성 |
| **많은 양·CPU-bound** + 멀티코어 | `parallelStream()` | stateless·unordered 필수 |
| **I/O-bound**(DB, 파일, 네트워크) | `for` or `stream().sequential()` | 병렬 스트림 비효율 |
| **실시간↔중단 제어**(early return) | `for` | break 즉시 |
| **Checked 예외 필요** | `for` | stream은 wrapper 필요 |


