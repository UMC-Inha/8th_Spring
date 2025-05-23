**일반 미션**

- 깃허브 관련 미션

![image](https://github.com/user-attachments/assets/a1f4d4b5-8aaf-48cc-a2aa-51492308fec5)
![image](https://github.com/user-attachments/assets/4b1af452-541e-44af-a278-f30290455fe8)
![image](https://github.com/user-attachments/assets/3858dce4-ea2a-40d2-9bc6-fc7b33134fd4)


- 1번 가게 추가 API

![image](https://github.com/user-attachments/assets/a4640e65-d2e2-4ce8-a66a-db595196f222)


- 2번 리뷰 추가 API

![image](https://github.com/user-attachments/assets/33fffde5-0074-4585-a669-717c676b19cc)
    - 리뷰 작성 시 해당 가게의 scoreAvg 값도 갱신됨

- 3번 미션 추가 API

![image](https://github.com/user-attachments/assets/db124ebc-a786-4d5a-ba37-01a44d696491)


- 4번 미션 도전 API

![image](https://github.com/user-attachments/assets/7fad3af8-da80-4c7e-8cf5-1b25f419d22d)


**시니어 미션**

### 1️⃣ @DynamicInsert와 @DynamicUpdate가 어떻게 작동되는 지 파악하기

- 기존 JPA 쿼리 문이 어떻게 만들어지는 지 알아보기
    - 엔티티 분석 → 컬럼 목록, PK 등 정보 수집
    - 애플리케이션 시작 시 고정 SQL을 각 엔티티별로 한 번 생성 후 캐시
    - 런타임에 값만 바인딩하여 실행
- @DynamicInsert, @DynamicUpdate 적용 시 어떻게 바뀌는 지 알아보기
    - Insert/Update 직전에 현재 속성값을 보고 동적으로 SQL 조합
    - null인 값 제외 → DB Default 사용 가능
    - SQL을 매번 새로 만든다는 단점 존재

### 2️⃣ 기존과 @DynamicInsert, @DynamicUpdate 적용 시 장단점 파악하기

- 찾아본 원리를 토대로 서로의 장단점 적어보기


- 기존에 Dynamic 어노테이션이 없는 상황

![image](https://github.com/user-attachments/assets/f269107d-416b-457e-bcee-e0a329f86a49)

![image](https://github.com/user-attachments/assets/cb4ba386-2b0d-4aef-9848-0efbc05e4dbc)


    ```java
    Hibernate: 
        /* insert umc.domain.Member
            */ insert 
        into
            member (address, birth, created_at, gender, inactive_at, is_phone, login_type, mail, mission_complete, name, phone_num, point, status, updated_at) 
        values
            (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    Hibernate: 
        /* insert umc.domain.mapping.PreferCategory
            */ insert 
        into
            prefer_category (created_at, food_category_id, member_id, updated_at) 
        values
            (?, ?, ?, ?)
    Hibernate: 
        /* insert umc.domain.mapping.PreferCategory
            */ insert 
        into
            prefer_category (created_at, food_category_id, member_id, updated_at) 
        values
            (?, ?, ?, ?)
    Hibernate: 
        /* insert umc.domain.mapping.PreferCategory
            */ insert 
        into
            prefer_category (created_at, food_category_id, member_id, updated_at) 
        values
            (?, ?, ?, ?)
    ```

  → default인 mission_complete, point, status 에 값이 들어가는 것 확인 가능

  →→ 근데 왜 DB Default 가 아닌 null 인가?

  →→→ JPA는 애플리케이션 가동 시 고정 SQL 을 1개 미리 컴파일/캐시 진행

  →→→→ Default 또한 null 값을 삽입하여 DB Default가 무력화됨.


- Dynamic 어노테이션 추가

![image](https://github.com/user-attachments/assets/2c8da2f1-e146-401b-9e3b-1478261f2200)

![image](https://github.com/user-attachments/assets/4589330e-d4b7-4310-a2d5-c5f071ecd1f5)


    ```java
    Hibernate: 
        /* insert umc.domain.Member
            */ insert 
        into
            member (address, birth, created_at, gender, is_phone, login_type, mail, name, phone_num, updated_at) 
        values
            (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    Hibernate: 
        /* insert umc.domain.mapping.PreferCategory
            */ insert 
        into
            prefer_category (created_at, food_category_id, member_id, updated_at) 
        values
            (?, ?, ?, ?)
    Hibernate: 
        /* insert umc.domain.mapping.PreferCategory
            */ insert 
        into
            prefer_category (created_at, food_category_id, member_id, updated_at) 
        values
            (?, ?, ?, ?)
    ```

  → Insert문에 default 값인 속성들이 포함되지 않은 것 확인 가능

  →→ DB에 정상적으로 Default 값 Insert 된 것 확인 가능


- 장단점


  장점

- Insert
- null 값 제외 → DB Default 반영 가능
- 대용량 테이블에서 불필요한 I/O | REDO 줄어듦
- 컬럼이 수십 개인데 실값은 몇 개 뿐인 레코드에 유리


- Update
- 변경된 컬럼만 SET → 쓰기 트래픽, 락 경쟁 최소화
- 감사 컬럼 한두 개만 수정할 때 효율 좋아짐


  단점

- Insert
- PreparedStatement 캐시 miss 높아짐
- JDBC Batch 불가능 → 대량 적재 시 속도 하락


- Update
- PreparedStatement 캐시 miss 높아짐
- 여러 필드 한꺼번에 자주 수정하면 이득 낮아짐


### 3️⃣ 언제 적용하면 좋을 지 파악하기

- 장단점을 토대로 언제 @DynamicInsert, @DynamicUpdate를 적용하면 좋을 지 적기
    - DB Default / Trigger 살리고 싶으면 → @DynamicInsert 사용
    - 테이블 폭이 넓고 부분 업데이트가 잦다 → @DynamicUpdate 사용
    - 동일 SQL을 대량 배치로 넣어야 한다 → 두 어노테이션 지양해야함
    - 캐시 Miss 비용 < I/O 절감 효과 일 때 사용하면 좋음

### 1️⃣ Rest Docs가 무엇인지 알아보기

- Rest Docs가  무엇인지 검색해보기
    - 통합 | 단위 테스트 과정에서 실제 HTTP 요청 / 응답을 캡처하여 Asciidoctor(adoc) 또는 Markdown(md) 조각(snippet) 을 생성, 빌드 시 이를 묶어 정적 HTML | PDF 문서를 만드는 도구
    - 테스트 실패 시 문서 갱신되지 않음 → API 동작과 문서의 정확도 자동으로 보장
    - 런타임 의존성 X, 생성된 산출물은 웹 서버 없이도 배포 가능

### 2️⃣ Swagger와 Rest Docs의 장단점 비교하기

- Swagger와 Rest Docs의 장단점 적어보기
    - Swagger
        - Rest Docs 에 비해 설정 간편
        - API 문서에서 테스트 가능
        - 다양한 진영에서 사용 가능
        - Production 코드에 Swagger 관련 코드가 함께 들어감
        - 테스트 기반이 아니여서 문서의 안정성 보장 X
    - Rest Docs
        - Swagger와 달리 Production 코드에 영향 없음
        - 테스트를 통과한 API만 문서화 → 안정성 보장
        - Swagger 보다 설정 까다롭고 레퍼런스가 적음
        - 테스트 코드 아래에 이어 붙이는 형식 → 테스트 코드의 양이 많아짐

### 3️⃣ 언제 적용하면 좋을 지 파악하기

- 장단점을 토대로 언제 Swagger와 Rest Docs를 적용하면 좋을 지 적기
    - 속도가 중요하다 ( 외부 개발자가 바로 테스트 할 수 있는 콘솔을 제공해야 한다) → Swagger
    - 보안이 더 중요하다 → Rest Docs
    - 레이어별 테스트가 확립되어 있고 API 정확도가 가장 중요하다 → Rest Docs

  즉

    - 빠른 시각화 | 인터렉티브 를 원한다면 Swagger
    - 테스트 기반 정확도 | 정적 배포 | 보안 을 원한다면 Rest Docs
