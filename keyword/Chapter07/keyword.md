- RestContollerAdvice
    - @RestControllerAdvicesms @ControllerAdvice와 @ResponseBody를 합친 anotation
    - @ControllerAdvice의 역할을 수행하고, @ResponseBody를 통해 객체 리턴 가능
    - 장점
        - 컨트롤러마다 예외처리 작성 필요없이, 한 곳에서 모든 예외처리 관리
        - 속성을 통해 특정 범위 세분화하여 적용 가능 ( 특정 패키지 or 클래스 or 어노테이션 )
    - 단점
        - 디버깅 난이도 있음 - 커스텀 메세지만 보여서 디버깅 늦어질 수 있음
- lombok
    - 자바 라이브러리
    - 자바 컴파일 시 어노테이션 프로세서(APT)로 동작하며 @Getter,@Setter같은 어노세이션을 읽어 필드에 자동생성 해줌
    - 가족성이 높아지고, 반복적인 코드를 없애줌
    - 생산성이 매우 좋아지는 장점이 있지만, 보이지 않는 코드가 있기에 디버깅이 어려워지는 단점도 있음
- JSON Annotation
    - @JsonProperty
        - Jackson에서 속성 이름을 지정할 때 사용하는 어노테이션
        - JSON데이터와 자바 엔티티의 이름이 같지 않은 경우 매핑시켜줌
            - ex) JSON데이터는 스네이크 케이스인데, 자바 엔티티가 케멀케이스인 경우
        - 필드에 각각 @JsonProperty를 붙여줘야하는데 필드가 많다면 코드가 지저분해질 수 있다. 이럴 땐 @JsonNaming사용
    - @JsonPropertyOrder
        - JSON 직렬화 시 속성의 순서를 지정하는 데 사용되는 어노테이션
        - JSON 직렬화 : 객체를 JSON 문자열로 변환하는 과정
        - 아래의 경우 JSON 문자열은 name, email, id 순서로 직렬화

        ```java
        import com.fasterxml.jackson.annotation.JsonPropertyOrder;
        
        @JsonPropertyOrder({"name", "email", "id"}) // 원하는 속성 순서 지정
        public class User {
            private int id;
            private String name;
            private String email;
        
            ...
        }
        ```

    - @JsonInclude
        - 자바 객체 → JSON형식으로 직렬화 때 포함할 속성을 지정할 때 사용
        - @JsonInclude를 사용해 JSON에서 포함되어야 할 속성과 제외할 속성 지정 가능
        - 옵션 Default는 Include.ALWAYS

      | 옵션 | 직렬화 대상 | 예시 |
              | --- | --- | --- |
      | **`ALWAYS`** (기본) | 모든 값 | `name: null` 도 출력 |
      | **`NON_NULL`** | `null` 제외 | `@JsonInclude(NON_NULL)` 👈 대부분 이걸 씀 |
      | **`NON_EMPTY`** | `null` + 빈 값(`""`, `[]`, `{}`) 제외 | 빈 문자열 필드도 빠짐 |
      | **`NON_DEFAULT`** | 필드가 **기본값**과 다를 때만 | `int age = 0;` → 0이면 미포함 |
      | **`NON_ABSENT`** | Optional 등 “존재하지 않음” 상태 제외 | `Optional.empty()` 미포함 |
      | **`CUSTOM` / `USE_DEFAULTS`** | 커스텀 포함 규칙 or 전역 설정 값 사용 | 고급 사용 |