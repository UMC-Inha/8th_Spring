# 미션

### 기준 ERD
![image](https://github.com/user-attachments/assets/3e2b6f95-bd56-476f-a160-3740b6109679)


### 미션 1.
-> 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리 (페이징 포함)
![image](https://github.com/user-attachments/assets/e7a5aa58-2ccf-4743-b893-c1a076c30dfc)



### 미션 2.
-> 리뷰 작성하는 쿼리 (사진 배제)
![image](https://github.com/user-attachments/assets/99cc5f87-27a9-4105-bf4e-2e24dba02f87)



### 미션 3.
-> 홈 화면 쿼리 (현재 선택된 지역에서 도전이 가능한 미션 목록, 페이징 포함)
![image](https://github.com/user-attachments/assets/5cf9e589-e242-4f49-b2e7-0137776b6b6e)



### 미션 4.
-> 마이 페이지 화면 쿼리
![image](https://github.com/user-attachments/assets/47050677-8576-4920-af99-4a3cca2d0aca)



# 시니어 미션

### 미션 1.
-> 기본 미션 1에서 짠 쿼리문 디벨롭하기 (커서 페이징, 정렬 기준 1순위 포인트, 2순위 최신순)
![image](https://github.com/user-attachments/assets/bd9fd7e7-f4a3-414c-9283-310bce9212e4)
- WHERE m.point < ?
    - ?에는 이전 페이지의 마지막 point 값이 들어가고 포인트가 내림차순으로 정렬될 테니 이전 페이지의 마지막 point 값보다 작은 point 값을 가져와야 해서 `m.point < ?`
- OR ( m.point = ? AND mm.updated_at < ? )
    - 만약 이전 페이지의 마지막 point 와 현재 페이지의 첫 번째 point가 같다면? → 누락되는 데이터가 생길 수 있음
    - 위의 문제를 해결하기 위해 2순위로 최신순이 있는 것이기에 OR을 사용하여 만약 같다면 updated_at 을 비교
        - updated_at은 처음 생성 시 created_at과 같은 값을 추가하고 미션을 완료하여 status 값이 변한다면 그 시각으로 변하기에 도전중이거나 도전을 완료한 데이터의 시간을 가져오기에 가장 적합하다고 생각하여 선택


### 미션 2.
-> SQL Injection 조사 및 예방법
SQL Injection 이란?

- 웹 애플리케이션의 취약점을 악용 → 악의적인 SQL 코드를 삽입
    - 이를 통해 DB 조작 및 접근하는 공격 기법
- 보통 입력값 검증이 제대로 이루어지지 않는 경우 발생한다.
- 종류로는
    - Error based SI
        - DB에서 발생하는 오류 메시지를 통해 구조나 정보 파악하는 기법
    - Union SI
        - UNION 명령어를 통해 원래 쿼리에 악의적인 쿼리 삽입하여 정보 탈취하는 기법
    - Blind SI
        - 참/거짓 판단을 통해 DB 정보 유추하는 기법
    - Stored Procedure SI
        - 저장 프로시저의 취약점 이용하는 기법
- 등 이 있다.

대응법

- 파라미터화된 쿼리
    - 쿼리문을 템플릿화, 사용자 입력은 파라미터로 전달 → 구조 변경 차단
- 입력값 필터링
    - 사용자 입력을 허용된 패턴으로 제한하여 위험한 문자 제거
- DB 계정 권한 최소화
    - 웹앱과 관리자 계정을 분리, 최소 권한만 부여 → 탈취되더라도 제한된 권한으로 DB 전체 장악은 어렵게 만들 수 있다.
- 보안 도구 활용
    - 버프 스위트 같은 보안도구를 통해 SI 시도가 가능한 파라미터를 탐지가능
    - 다양한 입력값을 반복적으로 시도하며 반응을 분석하여 취약점 찾아낼 수 있음

### 미션 3.
-> 키워드에 정리되어 있습니다.
