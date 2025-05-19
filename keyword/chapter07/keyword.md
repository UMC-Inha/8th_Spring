# 🎯핵심키워드

---

## 1. RestContollerAdvice

## `@RestControllerAdvice`란?

@RestControllerAdvice는 전역적인 예외 처리와 응답 변환을 담당하는 AOP 기반의 컨트롤러 보조 클래스로, `@ControllerAdvice` + `@ResponseBody`가 결합된 기능이다.

즉, 모든 `@RestController` 또는 `@RequestMapping`이 붙은 클래스에서 발생하는 **예외(Exception)를 가로채고**, **응답을 통일된 형식으로 리턴**할 수 있게 도와주는 **전역 처리기(Global Handler)** 역할을 함.

## 동작 구조 (이건 내가 헷갈려서 찾아본건데 정확하지 않을수도,,)

### 1. **DispatcherServlet** → Controller 호출

요청이 컨트롤러로 가서 예외가 발생하면

### 2. **ExceptionResolver**들이 예외 처리 시도

`ExceptionHandlerExceptionResolver` → 이게 `@RestControllerAdvice`를 스캔해서 예외 처리 메서드 실행

### 3. **@ExceptionHandler로 등록된 메서드 호출**

등록된 핸들러 메서드에서 `ApiResponse` 같은 응답 객체로 가공

### 4. **HttpMessageConverter**가 JSON으로 변환해서 응답

## RestContollerAdvice을 쓰는 상황들

| 상황                           | 설명                                                                 |
|--------------------------------|----------------------------------------------------------------------|
| 모든 API 응답 형식 통일        | 예: `ApiResponse`로 감싸기                                          |
| 전역적으로 발생하는 예외 통일 처리 | 예: 400, 404, 500 등 각각 포맷 통일                                  |
| 사용자 정의 예외 처리          | 예: `GeneralException` 같은 사용자 정의 예외 처리                    |
| 유효성 검사 실패 메시지 포맷 지정 | 예: `@Valid`, `@Validated` 처리 시 메시지 포맷을 구조화하여 응답       |


## RestContollerAdvice이 왜 중요한가?

1. 클린한 컨트롤러 유지 ⇒ 컨트롤러는 핵심 로직에만 집중
2. 보안성 ⇒ 스택 트레이스 노출 방지
3. 확장성 ⇒ 다양한 예외 추가 및 재사용 가능
4. 일관성 ⇒ 응답 포맷과 HTTP 코드 통일

**한줄결론: RestContollerAdvice을 왜 쓰는가 하면**

⇒ 컨트롤러 코드가 깨끗해야 하고, 에러 응답은 통일돼야 하며, 모든 예외 상황을 한곳에서 잡고 싶기 때문!

## 2. lombok

Lombok은 Java 클래스에서 반복되는 보일러플레이트 코드를 자동으로 생성해주는 컴파일러 플러그인이다.  @Getter, @Setter, @Builder, @AllArgsConstructor 등 어노테이션 하나로 수동 작성 코드를 줄여주는 도구이다.

### Lombok을 쓰는 이유는?
```java
// 생성자, getter, setter, equals, toString 등 코드가 기하급수적으로 많아짐.
public class User {
    private String name;
    private int age;

    public User() {}

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    
}
```

lombok 사용
```java
import lombok.*;

// 컴파일 시점에 자동으로 위와 같은 메서드들이 생성해주니 매우 간편하고 코드도 간결
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class User {
    private String name;
    private int age;
}
```

### 자주 쓰는 Lombok 어노테이션 정리

| 어노테이션              | 설명                                                                 |
|------------------------|----------------------------------------------------------------------|
| `@Getter` / `@Setter`  | 모든 필드에 대해 Getter/Setter 생성                                   |
| `@ToString`            | `toString()` 자동 생성                                               |
| `@EqualsAndHashCode`   | `equals()` & `hashCode()` 자동 생성                                  |
| `@NoArgsConstructor`   | 기본 생성자                                                           |
| `@AllArgsConstructor`  | 모든 필드를 인자로 받는 생성자                                       |
| `@RequiredArgsConstructor` | `final` 또는 `@NonNull` 필드만 포함한 생성자                        |
| `@Builder`             | 빌더 패턴 지원 (Immutable 객체 생성에 유용)                          |
| `@Data`                | `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor` 종합 패키지 |
| `@Value`               | `@Data`의 Immutable 버전 (`final`, `private`, setters 없음)          |


### Lombok 동작방식

1. Lombok은 **애노테이션 프로세서(annotation processor)** 를 사용해서 컴파일 타임에 클래스 파일에 코드를 직접 추가함.
2. 즉, 실제 소스 코드에는 없지만 `.class` 파일에는 들어가 있음.
3. 그래서 **IDE와 컴파일러가 Lombok을 지원해야** 정상 작동 가능

### 그렇다면 이러한 lombok을 언제쓰면 좋을까?

⇒ 사실 내가 공부하거나 직접 실습을 할때는 lombok은 거의 필수적으로 사용했었는데, 그냥 당연히 사용하는거지라고 생각하면서 언제쓰면 좋은가는 많이 고민해보진 않았던것 같다.

그래서 찾아본결과

1. DTO, Entity, Response 객체 ⇒ 웬만하면 사용
2. 서비스, 비즈니스 로직 클래스 ⇒ 최소한의 사용
3. 코어 도메인 모델 ⇒ 너무 자동화를 하면 설계가 흐릿해질수 있기때문에 적절하게 사용

(이렇게 말해도 사실 아직까지는 언제 굳이 lombok을 사용 안해야하는지는 좀 헷갈리는것 같다.)

## 3. Excepton 핸들링

### 예외 처리(Exception Handling)란?

⇒ 프로그램 실행 중 발생할 수 있는 예기치 못한 상황에 적절히 대응하기 위한 구조적 코드 처리 기법

자바에서는 대표적으로 `try-catch`, `throw`, `throws`, 그리고 사용자 정의 예외(Exception 클래스 상속) 등을 사용한다.

### 예외 처리를 사용해야 하는 이유? 

| 목적               | 설명                                                                 |
|--------------------|----------------------------------------------------------------------|
| 서비스 안정성 확보 | 예외로 인해 전체 서비스가 죽는 걸 방지                              |
| 사용자 친화적인 응답 | 개발자 에러 메시지가 아닌 사용자에게 의미 있는 메시지 제공             |
| 디버깅 용이성       | 예외 로그를 통해 버그를 빠르게 추적 가능                              |
| 유지보수성 향상     | 일관된 방식으로 예외를 처리하면 시스템 전반적으로 통제 가능             |

### Spring의 예외 처리 방식

Spring은 예외를 단순히 try-catch로 처리하지 않고, **전역 예외 처리 전략**을 제공한다.

| 기술 요소                     | 설명                                                           |
|------------------------------|----------------------------------------------------------------|
| `@ExceptionHandler`          | 특정 예외를 잡아주는 메서드 단위 처리                         |
| `@ControllerAdvice`          | 전역적으로 예외를 처리하는 클래스                             |
| `@RestControllerAdvice`      | REST API 전용 전역 예외 처리 (JSON 응답)                      |
| `ResponseEntityExceptionHandler` | Spring 기본 예외 처리기 상속받아 커스터마이징 가능              |


## Spring에서의 예외 처리 동작 흐름

1. 컨트롤러 실행 중 예외 발생
2. `@RestControllerAdvice`로 등록된 클래스에서 예외 감지
3. 해당 예외를 처리하는 `@ExceptionHandler` 메서드 실행
4. 공통 응답 포맷(`ApiResponse`, `ResponseEntity`)으로 변환
5. 사용자에게 JSON 응답 전송

### 결론: 예외 처리는 **오류를 잡는 것**이 아니라, **예외를 통제하는 것으로,** Spring에서는 `@RestControllerAdvice` + `@ExceptionHandler`로 전역 예외 처리를 함

예외 처리의 주 목적은 사용자에게 의미 있는 메시지를 전달하고, 시스템의 안정성과 유지보수성을 높이는것. 때문에 체계적으로 계층을 나누고 통일된 응답을 사용하는것이 좋다.


