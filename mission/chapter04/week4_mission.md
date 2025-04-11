# 시니어 미션

1. Servlet VS Spring MVC

-> https://jungle-friday-d03.notion.site/Servlet-VS-Spring-MVC-1d08dd1cbf478078ad28dc4e7f183d1b?pvs=4

2. AOP (Aspect-Oriented Programming) 원리 탐구

-> https://jungle-friday-d03.notion.site/AOP-1d08dd1cbf4780129f59e3a3396f49ce?pvs=4

### 도전 과제
기존 UMC에서 진행했던 프로젝트를 AOP 기반 Logging 전략을 채택하여 리팩토링할 수 있는 방안에 대해 논의

-> 실시간 로깅이 많이 필요하진 않는 상황

그렇다면 어느 상황에 쓰면 좋을까?

- 따로 로그 파일 생성
  - 예외나 에러 기록을 로그 파일에 저장
  - 예외나 에러 발생 시 빠른 이슈 파악 가능
  - 로깅은 AOP로 진행