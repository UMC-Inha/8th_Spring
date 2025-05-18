# 💡 7주차 핵심 키워드 💡
<br>

## 1. RestControllerAdvice

[정의]<br>전역적으로 예외를 처리하거나, 모든 API 응답을 하나의 형식으로 통일할 수 있게 도와주는 기능.

[역할]<br>@RestController가 붙은 Exception이 발생하는 것을 감지.

[목적]<br>
1. 예외 발생 시 공통된 JSON 응답 형태로 처리하고자 함.
2. API 응답을 일관된 구조로 만들고자 함.
3. 컨트롤러 단에서 발생하는 예외를 try-catch로 일일이 잡지 않고, 자동으로 핸들링하고자 함.

[장점]<br>
1. try-catch 없이도 예외 대응 가능
2. 에러 응답을 일관되게 처리 가능
3. 모든 응답을 ApiResponse<T> 형식으로 감쌀 수 있음
4. 에러 처리 코드가 모듈화 되어 있어서 유지보수 쉬움
5. 코드가 간결해짐.

<br>

## 2. lombok

[정의]<br>자바 클래스에서 getter/setter, 생성자, toString 등 반복 코드를 자동으로 만들어주는 라이브러리.<br>

[장점]<br>
1. 코드 절약 가능
2. 가독성 향상
3. 유지보수 쉬움

[예시]<br>
```java
// lombok을 사용하지 않는 경우
public class Member {
    private String name;
    private int age;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    public Member(String name, int age) { ... }

    @Override
    public String toString() { ... }
}
```

**lombok 적용하면**❓<br>
```java
import lombok.*;

@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Member {
    private String name;
    private int age;
}
```

[주요 어노테이션]<br>
1. `@Getter, @Setter` : 각 필드에 getter/setter 생성
2. `@ToString` : 자동으로 toString() 메서드 생성
3. `@NoArgsConstructor` : 파라미터 없는 생성자 생성
4. `@AllArgsConstructor` : 모든 필드를 받는 생성자 생성
5. `@RequiredArgsConstructor` : final 필드만 받는 생성자 생성
6. `@Builder` : 빌더 패턴 자동 생성<br>
`Builder.Default : lombok Builder가 필드의 초기값을 무시하지 않도록 하는 도구.`
7. `@Slf4j` : 자동으로 log라는 이름의 logger 객체 생성

<br>

## 3. @RequestParam

[정의]<br>URL에 포함된 쿼리 파라미터나 폼 데이터를 컨트롤러의 메서드 파라미터로 바인딩할 때 사용.

[옵션]<br>
1. value : 요청 파라미터의 이름 지정
```java
@RequestParam("user") String name
```
2. required (기본값 : true) : 파라미터가 반드시 있어야 하고, 없으면 400 Bad Request
```java
@RequestParam(required = false) String name  // 없어도 에러 안 남
```
3. defaultValue : 파라미터가 없을 때 기본값 저장
```java
@RequestParam(defaultValue = "져니") String name
```

[주의할 점]<br>JSON에는 사용 불가능하므로 `@RequestBody` 사용해야 한다.