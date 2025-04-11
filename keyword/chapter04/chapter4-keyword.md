# chapter 4 키워드
## DI(Dependency Injection) : 의존성 주입

### 정의

- 객체가 필요한 의존 객체를 직접 생성하지 않고, 외부에서 주입받는 방식을 의미한다.
- 이는 객체간 결합도를 낮추고, 유연하고 테스트하기 좋은 구조를 만들 수 있다.

### 종류

- 생성자 주입
- setter 주입
- 필드 주입

### Spring에서 DI

- `@Autowired` , `@Component` , `@RequiredArgsConstructor` 등으로 DI수행 가능하다.
- 스프링이 Bean을 관리하면서 필요한 객체를 자동으로 넣어준다.

## IoC(Inversion of Control) : 제어의 역전

### 정의

- 애플리케이션의 제어 흐름을 개발자가 아닌 외부(컨테이너)가 관리하는 구조를 의미한다.
- 객체의 생성, 생명주기, 의존성 관리 등을 프레임워크(여기서는 Spring)가 담당한다.
- 즉 개발자가 객체를 만들고 쓰고 정리하는 것을 넘어 → Spring이 알아서 해주기 때문에 → 기능에만 집중할 수 있도록 한다.

### Spring에서 IoC

- Spring Container(Application Context)가 모든 Bean생성과 의존성을 관리한다.
- 단지 `@Component`, `@Service` 등으로 등록만 해주면 자동으로 관리된다.

## 프레임워크와 API의 차이

| 구분 | 프레임워크 | API |
| --- | --- | --- |
| 주도권 | 프레임워크가 흐름을 주도 (IoC) | 개발자가 호출해서 사용 |
| 제어권 | Inversion of Control (IoC) 있음 | 없음 |
| 예시 | Spring, Django | Java Collections, JDBC, Retrofit |
- **API**는 "요리할 재료와 조리법"
- **프레임워크**는 "셰프가 알아서 요리해주는 주방"

  → 프레임워크는 코드가 그 틀 안에서 돌아간다고 보면 된다.


## AOP (Aspect Oriented Programming) : 관점 지향 프로그래밍

![image.png](attachment:613bfc57-e3ae-4929-9de2-7f44d0c0309e:image.png)

### 정의

- **핵심 로직과는 별개인 부가 로직(공통 관심사)을** 분리해서 처리하는 프로그래밍 기법이다.
- Aspect로 흩어진 관심사를 모듈화하고 핵신적인 비즈니스 로직에서 분리하여 재사용하겠다는 것이 취지이다.
- 즉 중복을 제거하여 유지보수성을 향상시킨다.

### 여기서 공통 관심사란?

- 로깅
- 트랜잭션
- 보안
- 예외처리 등을 의미한다.

### AOP 용어 정리

- **Cross-cutting concerns(횡단 관심사)** 로직 전 또는 후에 실행되어야하는 공통적인 작업.
- **Advice**aspect가 해야하는 작업과 시기.
- **Target**advice가 적용될 객체.
- **Join Point**advice를 적용할 수 있는 지점들. 메서드, 필드, 객체, 생성자 등. Spring AOP에서는 메서드가 실행될 때만 적용되도록 한정하고 있다.
- **Point Cut**실제 advice가 적용될 지점. advice가 적용될 수 있지만 적용되지 않는 곳도 존재할 것이다. point cut은 join point 중에서도 advice가 적용되는 지점을 말한다.
- **Proxy**advice를 target 객체에 적용하면 생성되는 객체. Aspect를 구현하기 위해 프레임워크로부터 만들어진 객체라고도 말한다. Spring 프레임워크에서 AOP proxy는 JDK dynamic proxy 또는 CGLIB proxy이다.
- **Aspect**AOP에서는 횡단관심사를 aspect라는 특별한 객체로 모듈화한다. 이는 advice와 point cut을 합친 것으로, 무엇을 언제 어디서 할지에 대한 정보가 모두 정의되어 있다.
- **Weaving**target 객체에 aspect를 적용해서 새로운 프록시 객체를 생성하는 절차. aspect는 target 객체의 join point로 위빙된다. 위빙은 대상 객체의 생애 중 다음과 같은 시점에서 수행될 수 있다.

### Spring에서 AOP 사용 예시

- 스프링 부트의 `@Transactional`도 내부적으로 AOP로 구현된다.
- Aspect는 Advice와 Pointcut으로 구성된다.
- Advice의 경우 아래처럼 5개의 어노테이션을 지원한다.

| 어노테이션 | 설명 |
| --- | --- |
| @Before | 비즈니스 로직 실행 전 |
| @AfterReturning | 비즈니스 로직 실행 결과가 정상적으로 반환된 후 |
| @AfterThrowing | 비즈니스 로직 실행 시 예외가 발생된 후 |
| @After | 비즈니스 로직 실행 후 |
| @Around | 비즈니스 로직 실행 전과 후 |

```java
@Aspect
@Component
public class LogAspect {
  @Before("execution(* com.example.service.*.*(..))")
  public void logBefore(JoinPoint joinPoint) {
    System.out.println("메서드 호출 전: " + joinPoint.getSignature());
  }
}

```

## 서블릿

### 정의

- **Java에서 웹 요청과 응답을 처리하는 기본 단위이다.**
- 클라이언트의 HTTP 요청을 받아서 동적으로 결과를 만들어 응답한다.

### Servlet의 핵심 메서드

| 메서드 | 설명 |
| --- | --- |
| `init()` | 서블릿 초기화 |
| `doGet()` | GET 요청 처리 |
| `doPost()` | POST 요청 처리 |
| `destroy()` | 서블릿 종료 시 호출 |

### 서블릿의 동작 구조

1. 사용자가 웹 브라우저로 요청
2. 요청이 `web.xml` 또는 애노테이션을 통해 서블릿으로 연결
3. 서블릿이 `HttpServletRequest` / `HttpServletResponse` 사용해 처리
4. 응답 반환

### Spring MVC에서의 서블릿 위치

- Spring이 서블릿 위에 추상화를 올린 것이 DispatcherServlet 이다.
- 즉, 서블릿은 여전히 Spring 안에서 동작하고 있다.