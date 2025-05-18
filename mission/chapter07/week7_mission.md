# 일반 미션

![image](https://github.com/user-attachments/assets/568a5eb3-7b12-4c77-8d1b-dd34d16f1e6f)


**RestControllerAdvice 장점**

- 전역적인 예외 처리 → 애플리케이션 전역에서 발생하는 예외를 한 곳에서 처리
- 응답 포맷 일관성 유지 → 예외 발생 시 JSON 형식의 표준화된 응답 보장
- 중복 코드 감소 및 가독성과 유지보수성 증가
- 에러 로그 처리 및 추적

**없으면 불편한 점**

- 예외 처리를 각 컨트롤러 내에서 개별적으로 처리해야 함
- 응답 포맷의 일관성 하락
- 글로벌 예외 처리 누락 → NullPointer, BadRequest 등등을 각 컨트롤러마다 처리해야 함
- 에러 추적 어려움
- 단위 테스트 및 통합 테스트 복잡성 증가

![image](https://github.com/user-attachments/assets/36413353-cc14-4646-b1fe-7507035b235d)


# 시니어 미션

https://makeus-challenge.notion.site/1eeb57f4596b80e5841ad2a80639a204?pvs=4

사진이 너무 많아서 링크로 대체
