## - 키워드 정리

#### - SOLID

- 객체지향 설계의 5대 원칙을 나타내는 약어로 SOLID는 SRP, OCP, LSP, ISP, DIP를 의미한다.

- **SRP (Single Responsibility Principle)**

    - 단일 책임 원칙

    - **하나의 클래스는 하나의 책임만 가져야 한다.**

    - 단일 책임 원칙의 목적은 프로그램의 유지보수 성을 높이기 위한 설계 기법이다.

    - ex)

      ``` java
      public class UserService {
       public void registerUser() {...}
       public void saveToDatabase() {...}
       public void sendWelcomeEmail() {...}
      }
      ```
        - 위의 클래스는 비즈니스 로직, DB, 데이터 전송까지 너무 많은 책임을 맡고 있다.

- **OCP (Open-Closed Principle)**

    - 개방 폐쇄 원칙

    - 서비스는 확장에는 열려있고, 수정에는 닫혀 있어야 한다.

    - 즉, 기능 추가 요청이 오면 클래스 확장을 통해 구현하지만, 클래스 수정은 최소화한다.

    - 즉, **추상화 사용을 통한 관계 구축을 권장하는 것이다.**

- **LSP (Liskov Substitution Principle)**

    - 리스코프 치환 원칙

    - 자식 클래스는 부모 클래스를 대체할 수 있어야 한다.

    - 상속을 받았다면, 기능이 동일하거나 더 나아야 한다.

    - 즉, **다형성의 원리를 이용하기 위한 원칙**이다.

- **ISP (Interface Segregation Principle)**

    - 인터페이스 분리 원칙

    - 클라이언트는 사용하지 않는 인터페이스에 의존하면 안 된다.

    - **클라이언트의 목적과 용도에 적합한 인터페이스만을 제공하는 것이 목표**이다.

- **DIP (Dependency Inversion Principle)**

    - 의존 역전 원칙

    - 추상화(인터페이스)에 의존하고, 구현체(클래스)에 의존하면 안 된다.

    - 상위 모듈(비즈니스 로직)은 하위 모듈(구현체)에 의존하면 안 된다.

    - ex) 나쁜 예

      ``` java
       public class OrderService {
          private final MySqlRepository repo = new MySqlRepository(); // 구현체 직접 사용
        }
      ```

    - ex) 좋은 예

      ``` java
       public class OrderService {
          private final OrderRepository repo;
  
          public OrderService(OrderRepository repo) {
              this.repo = repo;
          }
       }
      ```

#### - DI (Dependency Injection)

- 하나의 객체에 다른 객체의 의존성을 제공하는 기술 -> **의존 관계에 있는 클래스를 주입한다.**

- **객체를 직접 생성하지 않고, 외부에서 넣어주는 방식으로 주입**하는 설계 패턴이다.

- **DI를 쓰는 이유**

    - 결합도를 낮춘다.

    - 테스트에 용이하다.

    - SOLID 원칙의 DIP를 만족한다.

    - Spring Bean이 직접 관리하기 때문에 일관성이 있다.

- **DI 방식의 종류**

    - **생성자 주입(권장)**

        - 생성자를 통해 의존관계를 주입받는 방법

        - 생성자를 호출할 때 딱 1번만 호출되기 때문에 변수를 final로 관리할 수 있다. -> **불변성을 보장한다.**

        - 의존성이 없으면 컴파일 에러가 발생해 빠르게 감지할 수 있다.

        - 생성자를 통해 Mock 객체를 주입할 수 있어 테스트에 용이하다.

        - 코드 예시

          ``` java
          @Service
           public class MissionService {
    
              private final MissionRepository missionRepository;
    
              @Autowired
              public MissionService(MissionRepository missionRepository) {
                this.missionRepository = missionRepository;
              } 
           }
          ```

    - **Setter 주입**

        - setter 메서드를 통해서 의존 관계를 주입하는 방법

        - **불변성이 없다.**

        - **필드를 주입하지 않아도 감지할 수 없다.** -> 런타임 에러 발생 위험

        - 외부에서 맘대로 변경할 수 있어 보안 위험이 있다.

    - **필드 주입**

        - 필드에 바로 @Autowired를 통해 주입한다.

        - DI 프레임워크가 없으면 사용할 수 없게 된다.

        - 클래스 외부에서 접근이 불가능하다. -> **테스트하기 어렵다.**

        - **final 키워드를 사용할 수 없으므로 런타임에 변경 가능하다.**

        - 순환 참조를 감지할 수 없다.


#### - IoC(Inversion Of Control)

- **객체의 생성과 제어 권한을 개발자가 아닌 프레임워크에 위임하는 것이다.**

- 다음과 같은 코드가 있을 때, Spring 컨테이너가 자동으로 repo를 생성하고 넣어준다.

  ``` java
   @Service
    public class MissionService {
       private final MissionRepository repo;

       public MissionService(MissionRepository repo) {
         this.repo = repo; // 외부에서 주입받음
       }
    }
  ```

- **IoC 방식의 장점**

    - 결합도를 낮춘다.

        - 객체를 직접 생성하지 않고 외부에서 주입 받기 때문에 느슨하게 결합된다.

        - 클래스끼리 직접 연결되지 않기 때문에 **구조가 더 유연하고 분리가 쉽다.**

    - 확장성과 유연성이 향상된다.

        - **인터페이스만 고정하면 구현체를 자유롭게 교체할 수 있다.**

    - 실제 구현체 대신 Mock 객체를 넣어서 테스트를 할 수 있기 때문에 테스트에 유리하다.

    - 재사용성과 모듈화가 증가한다.

        - 한 번 정의한 컴포넌트를 다양한 곳에서 재사용 할 수 있기 때문이다.

    - 기능 중심이 아니라 역할 중심의 설계가 가능하다.

- **Spring에서의 IoC 구현**

    - 클래스에 @Component, @Service와 같은 어노테이션을 붙이면 **스프링이 직접 객체를 생성하고, Bean으로 등록한다.**

    - 객체를 직접 만들지 않아도 자동으로 주입해준다.

    - DI를 통해 IoC를 구현한다.

#### - 프레임워크와 API의 차이

- **API**

    - 어떤 기능을 쉽게 쓸 수 있도록 제공하는 함수, 클래스, 라이브러리의 모음

    - **개발자가 직접 API를 불러서** 호출한다.

    - 개발자에게 주도권이 있다.

    - 자유도가 높다.

- **프레임워크**

    - 어플리케이션의 구조, 실행 흐름을 이미 정해두었다.

    - **제어 흐름이 프레임워크에 있다.**

    - 개발자는 프레임워크의 규칙에 따라야 한다.

#### - AOP (Aspect-Oriented Programming)

- **AOP란?**

    - **핵심 로직(비즈니스 로직)과 부가 로직(로깅, 트랜잭션, 인증 등)을 깔끔하게 분리**해서 코드 중복 없이 모듈화된 형태로 적용하는 프로그래밍 기법

- **AOP를 사용하는 이유**

    - 만약 메서드 실행 전/후에 로그를 남기고 싶다고 할 때, AOP가 없으면 아래와 같은 코드를 반복해서 적어주어야 한다.

      ``` java
      public void saveUser() {
         System.out.println("메서드 시작");
         // 비즈니스 로직
         System.out.println("메서드 종료");
       }
      ```

    - AOP를 쓰면 이러한 공통 기능을 딱 한 곳에만 구현해서 자동 적용할 수 있다.

#### - 서블릿

- **서블릿(Servlet) 이란?**

    - **HTTP 요청을 받아서 응답을 만드는 자바 클래스**

    - 자바에서 HTTP 통신을 다루는 공식 표준 API

- **서블릿이 하는 일**

    - HTTP 요청 받기

    - 요청 파라미터 읽기

    - DB 조회, 비즈니스 로직 실행 등의 로직 처리

    - HTTP 응답 생성

- **서블릿 사용 방법**

    - javax.servlet.http.HttpServlet 클래스를 상속해서 doGet(), doPost() 등을 오버라이딩해 사용한다.

    - 서블릿 컨테이너를 통해 서블릿을 실행한다. ex) **톰캣**

- **서블릿은 왜 사용할까?**

    - 정적인 HTML 파일만으로는 사용자 응답을 만들 수 없기 때문이다.

    - **서블릿은 자바로 동적인 웹 응답을 처리하기 위해 만든 표준 기술이다.**

- **서블릿의 장점**

    - Java 기반 동적 웹 페이지 처리가 가능하다.

    - HTTP 요청/응답 구조를 추상화한 API를 제공한다.

    - 서블릿 컨테이너가 HTTP 요청 수신, 서블릿 객체 관리, 멀티스레드로 요청 처리 등의 역할을 다 해주기 때문에 우리는 요청 처리 로직만 신경 쓰면 된다.

- **서블릿의 단점**

    - 매번 req.getParameter(), res.setContentType()과 같은 코드를 반복해야 한다.

    - 인증, 로깅, 에러 처리 등을 직접 처리해야 한다.

    - MVC가 분리되어 있지 않아 유지보수가 어렵다.


#### - 컴포넌트 스캔 어노테이션

- 컴포넌트 스캔 어노테이션이란?

    - **스프링은 컴포넌트 스캔 대상이 붙은 객체를 스프링 컨테이너에 등록하고, 필요한 곳에 자동으로 주입해준다.**

    - 이렇게 컴포넌트 스캔을 위해 서비스에 적용하는 어노테이션을 말한다.

- @Configuration + @Bean

    - @Configuration은 @Bean을 포함하는 설정 클래스이다.

    - @Bean을 통해 수동으로 Bean에 등록할 수 있다.

- @Component

    - 범용 컴포넌트 등록 어노테이션이다.

    - 공용 유틸, 일반 클래스 등에 사용한다.

- @Service

    - Service 클래스와 같이 비즈니스 로직 계층에 사용한다.

    - AOP의 대상이 되기 쉽다.

- @Repository

    - 데이터 접근 계층에 사용한다.

- @Controller

    - 웹 요청 처리 클래스(@RequestMapping)에 사용한다.

- @RestController

    - @Controller + @ResponseBody

    - JSON API 응답용 클래스에 사용한다.

#### - 생성자 어노테이션

- @AllArgsConstructor

    - 클래스 내에 있는 모든 필드를 파라미터로 받는 전체 생성자를 자동 생성해주는 어노테이션

- @NoArgsConstructor

    - 기본 생성자를 만들어준다.

    - **JPA의 Entity는 생성자 접근 제어자가 protected 이상이어야 한다.**

    - 따라서 @NoArgsConstructor(access = AccessLevel.PROTECTED)와 같은 방식으로 많이 쓰인다.

- @RequiredArgsConstructor

    - **final이나 @NonNull이 붙은 필드를 대상으로 생성자를 자동으로 생성해주는 어노테이션**

    - 이 어노테이션을 통해 **생성자 주입 방식을 사용할 수 있다.**

## - 시니어 미션

#### - 서블릿 VS Spring MVC 비교하기

- **전통적인 서블릿 기반 개발과 Spring MVC의 차이**

    - **전통적인 서블릿 기반 개발 방식**은 HttpServlet 클래스를 상속받아 doGet(), doPost() 메서드를 오버라이드 하는 방식이다.

    - 또한 **요청 파라미터를 직접 파싱하고, 응답을 직접 작성한다.**

    - Spring MVC 방식은 @Controller와 같은 어노테이션을 사용하며, @RequestMapping, @GetMapping, @PostMapping 등의 어노테이션으로 URI와 메서드를 매핑한다.

    - 또한 @RequestParam, @RequestBody, DTO 등으로 요청을 바인딩하고, 응답은 return으로 반환한다.

- **Spring MVC가 서블릿보다 편리한 이유**

    - 전통적인 서블릿 방식은 요청 파라미터를 수동으로 파싱하며 응답을 수동으로 작성해야했다. 또한 비즈니스 로직도 컨트롤러 로직에 포함되는 문제가 있었다.

    - 하지만 **Spring MVC 방식을 통해 요청과 응답, 파라미터 바인딩, 에러 처리, 유효성 검사 등이 모두 자동으로 처리되면서 테스트, 확장성, 유지보수 면에서 훨씬 간결해지게 되었다.**

- **DispatcherServlet이 내부적으로 요청을 처리하는 방식**

    - DispatcherServlet이란?

        - **Spring MVC에서 모든 HTTP 요청을 받아서 어떤 컨트롤러가 처리할지 결정하고, 응답까지 책임지는 컨트롤러**

    - DispatcherServlet의 요청 처리 단계

        1. DispatcherServlet이 모든 요청을 가로챈다.

        2. HandlerMapping을 조회해 URL과 매핑된 Controller 메서드를 찾는다.

        3. 찾은 핸들러를 어떻게 실행할지 결정하기 위해 HandlerAdapter를 선택한다. (이 과정에서 파라미터 바인딩, 메시지 컨버팅 등의 과정이 수행된다.)

        4. 바인딩 된 파라미터와 함께 실제 Controller 메서드가 호출된다.

        5. ModelAndView 혹은 객체와 같은 리턴값을 수신한다.

        6. ViewResolver로 뷰를 결정한다. (REST API이면 JSON으로 변환한다.)

        7. 렌더링한 결과를 HttpServletResponse에 기록하고 반환한다.

      ![](https://velog.velcdn.com/images/sm011212/post/e515f2f5-a740-463f-acc2-473bf4826503/image.png)


#### - AOP 원리 탐구

- **OOP와 AOP의 차이**

    - **AOP란?**

        - **핵심 로직(비즈니스 로직)과 부가 로직(로깅, 트랜잭션, 인증 등)을 깔끔하게 분리**해서 코드 중복 없이 모듈화된 형태로 적용하는 프로그래밍 기법. **AOP는 공통적으로 처리되는 횡단 관심사**를 따로 관리한다.

    - **OOP란?**

        - **객체 단위**로 역할과 책임을 나누는 설계 방식. **OOP는 현실 세계의 객체처럼 모델링**하고, 각 객체는 고유한 역할과 책임을 가지고 동작한다.

- **AOP의 핵심 개념 정리**

    - **Advice**

        - 공통 기능을 실행할 실제 로직으로, **언제 실행할지를 담당**한다.

        - @Before, @AfterRunning, @AfterThrowing, @After, @Around 등의 어노테이션이 있다.

    - **JoinPoint**

        - Advice가 **실행될 수 있는 위치(메서드 실행 지점, 예외 발생 등의 지점)를 지정**한다.

    - **PointCut**

        - **어떤 JoinPoint에 Advice를 적용할지 정하는** 조건이다.

        - 아래 어노테이션과 같은 표현식을 사용해 정의한다.

          ``` java
           // 모든 Service 클래스의 모든 메서드에 대해 수행
           @Before("execution(* com.example.service..*(..))")
          ```

    - **Aspect**

        - **AOP의 핵심 단위로, Advice와 Pointcut을 묶은 모듈**이다.

        - 즉, **클래스에 @Aspect를 선언하고, 내부 메서드에 @Advice를, 메서드 조건에 @Pointcut을 선언한다.**

    - **Weaving**

        - Pointcut과 Advice를 실제 실행 코드에 결합하는 작업이다.

        - 위빙 방식에는 크게 런타임 위빙 방식과 컴파일 타임 위빙 방식이 있다.

- **런타임 위빙 vs 컴파일 타임 위빙**

    - 런타임 위빙

        - **실행 시점에 프록시 객체를 동적으로 생성해서 Advice를 적용한다.**

        - 실제 클래스 코드는 안 바뀌고, 프록시가 대신 감싸서 동작한다.

        - 즉, 메서드 실행 시 프록시가 Advice를 실행하고, 원래 메서드를 호출한다.

        - Spring AOP 프레임워크를 이용한다.

    - 컴파일 타임 위빙

        - **Java 코드를 컴파일 할 때, 바로 Advice가 적용된 바이트코드로 변환해 적용한다.**

        - 직접 코드를 삽입하기 때문에 속도가 빠르지만, 설정이 많고 복잡하다는 단점이 있다.

        - AspectJ 프레임워크를 사용한다.

- **Spring에서 AOP가 프록시 패턴을 활용해 동작하는 원리**

    - 프록시 패턴이란?

        - **원래 객체 대신 가짜 객체(프록시)**를 앞에 세워서, **요청을 가로채고 추가 작업을 한 뒤에 실제 객체를 호출하는 패턴**

    - 프록시 패턴을 사용하는 이유

        - **자바는 런타임에 코드를 끼워 넣을 수 없다.** 대신 **프록시 객체로 핵심 객체를 감싸서** AOP 기능을 삽입할 수 있는데, 이 구조를 통해 **핵심 구조를 손대지 않고 AOP 기능을 삽입할 수 있다.**

    - Spring AOP의 프록시 생성 과정

        1. @Aspect 클래스를 스캔한다.

        2. Pointcut에 맞는 Bean을 찾아낸다.

        3. 해당 Bean에 대해 프록시 객체를 생성한다.

        4. 프록시가 Advice를 감싸도록 설정한다.

        5. 프록시가 스프링 컨테이너에 등록되고 주입된다.

    - 프록시 기반 Spring AOP의 동작 구조

        1. 클라이언트는 진짜 객체가 아니라 프록시 Bean을 주입받는다.

        2. 프록시는 메서드 실행 시 먼저 Advice를 실행한다.

        3. 그 다음에 진짜 타겟 객체의 메서드를 호출한다.