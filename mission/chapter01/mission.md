### 구현 과정

구현하고자 하는 기능을 DB에 담아내야 하므로 **상위 카테고리**인 미션, 지도, 마이페이지, 알람을 먼저 구

- 미션 → mission
- 지도 → location
- 마이페이지 → user
- 알람 → alarm

이렇게 기본 기능을 table로 먼저 구현한 후, column 추가 및 table 생성 및 연관관계 지정 과정을 통해서 erd 모델링을 했습니다.

기본적으로는 온보딩 내용에 있는 기능을 다 구현했고, 선호도 관련해서 있을 법한 table을 3개 만들어 추천 기능까지 생각해봤습니다

### 기본 테이블

### user

- 회원가입 시 받을 정보(username, password, gender 등)
- point (mission 성공 시 획득할 point를 user table에서 관리)

### mission

- 미션의 기본 정보(이름, 설명 등)
- 같은 지역의 미션을 했는지 체크해야 하기 때문에 location과 일대일의 연관관계를 맺었습니다

### location

- province(도)에 해당하는 값들을 미리 insert 시키기

 → 경기도, 전라도, 경상도, 충청도, 강원도, 제주도 row 추가하기

### alarm

- user 정보가 필요하므로 일대다 관계
- 읽었는지 유무 등의 기본정보

→ 향후 alarm을 슈퍼 테이블로 두고, 서브 테이블로 alarm_mission(mission 관련 알람), alarm_review(review 관련 알림) 설정

**슈퍼 타입과 서브 타입으로 alarm을 설정한 이유?**

→ 모든 알람이 확인했는지 유무가 필요하므로 알람의 기본 정보를 슈퍼 타입으로 빼고 각 영역 별 알람은 서브 타입으로 분리했다

**alarm_mission, alarm_review로 설정한 이유?**

→ 현재 2가지의 알람이 존재하지만 미션 완료, 리뷰 수정 등의 새로운 알람 기능이 추가될 여지가 있으므로 확장성을 고려해 테이블을 설계했으며 각각의 상황에 enum 값으로 value 지정

**만약, 지속적으로 값이 update 된다면 enum에서 따로 테이블을 둬 데이터 관리기**
----------

### 추가 테이블

### user_mission

- user와 mission 관계를 맺어주는 mapping table
- 내가 받은 미션 기능

### review

- restaurant과 1대다 관계
- 리뷰 작성 기능

### restaurant

- location과 1대다 관계
- 가게 리스트 / 가게 정보 기능

### restaurant_food

- 가게별 음식 리스트 정리
- 음식 이름, 음식 가격 정보 제공


----------
### 추가 기능(선호도 관련)

### user_preference

- 선호도 조사 기능

→ 선호도 항목을 restaurant와 user을 연결지어 유저 선호도에 맞는 추천 기능 도입

### cuisine_type

 음식 종류 → 한식, 양식, 중식, 일식, 카페 칼럼 추가

['KOREAN_FOOD', 'WESTERN_FOOD', 'CHINESE_FOOD', 'JAPANESE_FOOD', 'CAFE’]

### price_range

가격 범위 → 저렴한, 보통, 비싼 칼럼 추가

['LOW_PRICE', 'MEDIUM_PRICE', 'HIGH_PRICE’]

### visit_purpose

 방문 목적 → 혼밥, 데이트, 술, 가족(외식) 칼럼 추가

['SOLO', 'DATE', 'FAMILY', 'DRINKING’] 
