## Spring Data JPA의 Paging
- Spring Data JPA는 페이징 처리를 돕기 위한 관련 클래스와 인터페이스를 제공한다.


- 요청과 응답 관점으로 분류한 페이징 클래스 및 인터페이스
    - 요청 시 : Pageable, PageRequest
    - 응답 시 : Page, Slice


- Pageable
    - 페이징 요청 시 사용되는 인터페이스
    - 원하는 페이지 번호, 크기, 정렬 방식 등의 정보를 전달한다.


- PageRequest
    - Pageable 인터페이스의 대표적인 구현체
    - PageRequest.of(...) 형태로 생성


- Pageable의 주요 메서드
```java
public interface Pageable {
    int getPageNumber(); // 현재 페이지 번호 (0부터 시작)
    int getPageSize(); // 페이지에 포함될 데이터 개수
    long getOffset(); // 데이터를 가져올 시작점(offset)
    Sort getSort(); // 정렬 방식을 반환

    Pageable next(); // 다음 페이지에 대한 Pageable 객체 반환
    Pageable previousOrFirst(); // 이전 페이지에 대한 Pageable 반환, 없다면 첫 번째 페이지
    Pageable first(); // 첫 번째 페이지에 대한 Pageable 반환
    
    boolean hasPrevious(); // 이전 페이지가 존재하는지 여부
}
```
```java
// 1번째 페이지, 10개, name 기준 오름차순 정렬
Pageable pageable = PageRequest.of(0, 10, Sort.by("name").ascending());
```


- Page
    - 페이지 응답 시 사용되는 인터페이스
    - 구현체로는 PageImpl이 있다.
    - 현재 페이지의 데이터와 페이지 관련 정보를 포함하고 있다.
    - Slice 상속 + 전체 데이터 개수를 조회하는 쿼리가 나간다.
```java
// Page 인터페이스
public interface Page<T> extends Slice<T> {
    int getTotalPages(); // 전체 페이지 수
    long getTotalElements(); // 전체 데이터 개수
}
```
```java
// PageImpl 클래스
public class PageImpl<T> extends Chunk<T> implements Page<T> {
    
  public int getTotalPages() {
    return this.getSize() == 0 ? 1 : (int)Math.ceil((double)this.total / (double)this.getSize());
  }

  public long getTotalElements() {
    return this.total;
  }

  public boolean hasNext() {
    return this.getNumber() + 1 < this.getTotalPages();
  }
}
```


- Slice
    - 페이지 응답 시 사용되는 인터페이스
    - Page와 유사하나, 전체 데이터 개수를 조회하지 않는다.
```java
public interface Slice<T> extends Streamable<T> {
  int getNumber(); // 현재 페이지의 번호
  int getSize(); // 요청한 데이터의 개수
  int getNumberOfElements(); // 실제 조회된 개수

  List<T> getContent(); // 현재 데이터를 List로 반환 
  
  boolean isFirst(); // 첫 번째 페이지 여부
  boolean isLast(); // 마지막 페이지 여부
  boolean hasNext(); // 다음 페이지 존재 여부
  boolean hasPrevious(); // 이전 페이지 존재 여부
}
```

- Page와 Slice의 차이점
    - Page는 전체 데이터 개수를 조회하기 위해 추가적인 count 쿼리가 발생한다.
    - Slice는 다음 페이지가 존재하는지 여부만 필요하므로 limit에서 size+1로 쿼리가 나간다.
    - Page는 주로 게시판과 같이 총 데이터 개수가 필요한 환경에서 사용한다.
    - Slice는 모바일 어플리케이션 같이 무한스크롤 등을 구현할 때 사용한다.


## 객체 그래프 탐색
- 객체 간 맺어진 연관관계를 통해 객체를 탐색하는 것


- 객체 지향에서의 객체 그래프 탐색
    - 다른 객체에 대한 참조를 필드로 가져 객체 그래프 탐색이 자유롭다.
    - 직접적으로 메서드나 데이터를 호출하여 탐색한다.
```java
Customer customer = getCustomer();
List<Order> orders = customer.getOrders(); // 객체 그래프 탐색
```


- 관계형 데이터 베이스에서의 객체 그래프 탐색
    - 테이블 간의 관계를 외래키로 표현한다.
    - 연관된 데이터는 JOIN 연산으로 가져온다.
    - SQL을 직접 사용한다면 조회되는 객체까지만 탐색할 수 있다는 범위 제약이 생긴다.
    - 만약 어플리케이션에서 추가적인 객체 그래프 탐색이 필요하다면 새로운 쿼리를 작성해야 한다.


- 객체-관계 패러다임 불일치
    - 객체 지향에서는 참조를 통해 자유롭게 객체 그래프 탐색이 가능해진다.
    - 관계형 데이터베이스에서는 조회 범위에 제약이 생겨 객체 그래프 탐색에 제한이 생긴다.


- JPA의 해결법
    - 연관 관계 매핑
        - 객체의 연관 관계를 테이터베이스 테이블과 매핑해준다.
    - 객체 그래프 탐색 지원
        - JPA에서는 즉시로딩, 지연로딩으로 연관된 데이터를 가져오는 방법을 제공한다.
        - 필요한 시점에 데이터베이스에 쿼리를 발생시켜 객체 그래프 탐색이 가능하도록 한다.
    - 쿼리 최적화
        - Fetch Join과 같은 기법을 제공하여 N+1과 같은 성능 문제를 해결할 수 있도록 한다.
    - 캐싱
        - 쿼리를 최소화하기 위해 1차, 2차 캐시를 제공한다.
        - 한 트랜잭션에서 동일한 엔티티를 여러 번 조회하는 경우, insert는 한번만 발생한다.