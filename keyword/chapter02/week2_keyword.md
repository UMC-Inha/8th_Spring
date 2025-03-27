# 키워드

### Join
Join이란?

- DB 내의 여러 테이블의 레코드를 조합하여 하나의 열로 표현한 것
- 정규화를 진행하면 서로 관계있는 데이터가 여러 테이블로 나뉘어 저장됨
    - 각 테이블에 저장된 데이터를 효과적으로 검색하기 위해 필요

종류

- INNER JOIN
    - EQUAL JOIN
    - NATURAL JOIN
    - CROSS JOIN
- OUTER JOIN
    - LEFT OUTER JOIN
    - RIGHT OUTER JOIN
    - FULL OUTER JOIN

INNER JOIN

![image](https://github.com/user-attachments/assets/9ddf8ec5-0b2b-4c63-bda0-e1e44f03f4c9)


- MySQL 에선 JOIN, INNER JOIN, CROSS JOIN이 모두 같은 의미로 사용됨
- 조인하는 테이블의 ON 절의 조건이 일치하는 결과만 출력
- FROM 절에 쉼표를 사용하여 JOIN 생략 가능

LEFT/RIGHT OUTER JOIN

- 두 테이블이 합쳐질 때 왼쪽/오른쪽을 기준에 따라 기준 테이블의 것은 모두 출력이 되어야 함
- 보통 LEFT OUTER JOIN 을 사용하며 FULL 은 성능상 거의 사용하지 않음

Left Outer Join
![image](https://github.com/user-attachments/assets/ab39688d-9bf5-4b11-9e9b-3bbb30d3c540)

Right Outer Join
![image](https://github.com/user-attachments/assets/e68e38a0-6df3-404c-8bc3-e3ed655720f6)


### N + 1 문제
N + 1 문제란

- ORM 기술에서 특정 객체를 대상으로 수행한 쿼리가 해당 객체의 연관관계로 퍼져나가면서 N 번의 추가적인 쿼리가 발생하는 문제

원인

- RDBMS 와 객체지향 언어간의 패러다임 차이
    - 객체는 언제든 메모리 내에서 접근 가능
    - RDBMS는 Select 으로 조회 가능

예시

- A라는 객체가 B라는 객체와 1:N 으로 연관관계가 설정되어 있을 때
- JPA에서 A라는 객체를 Select 하기 위해 쿼리문을 날릴 경우 A 객체에 List<B> 에 들어있는 모든 객체들을 Select 하는 쿼리문이 같이 날아감

### SpringBoot 에서 N + 1 문제를 대응하는 방법
SpringBoot 에는 FetchType 이란 것이 존재

FetchType 이란?

- JPA에서 엔티티를 조회할 때 연관된 ‘엔티티 조회 방법을 결정하는 전략’ 을 의미
- 즉시 로딩(EAGER) 와 지연 로딩 (LAZY) 가 있음


  즉시 로딩
  - FetchType.EAGER
  - A 라는 엔티티를 조회할 때 연관된 B 라는 엔티티도 함께 조회하는 방식
  - 즉, 필요하지 않은 데이터까지 모두 조회하게 됨 → 실무에서 지양

  지연 로딩
  - FetchType.LAZY
  - A 라는 엔티티를 조회할 때 연관된 B라는 엔티티도 있지만 A만 조회하는 방식
  - B는 필요한 경우 조회
  - 실무에서 지향

  여기서 지연 로딩을 사용하여 최대한 예방할 수 있다.

  하지만, 지연 로딩을 써도 결국 필요한 경우 N개의 쿼리가 날아가는 것은 변함이 없다.

  즉, 시기의 차이일 뿐 N + 1 을 완벽하게 막진 못한다 → DTO 를 사용하여 필요한 데이터만 최소한으로 가져옴으로써 최적화할 순 있다.
