- **Servlet vs Spring MVC**
    1. **Servlet**
    
    전통적인 Servlet 방식을 이해하려면 web.xml 파일의 역할을 아는 것이 중요하다. web.xml은 서블릿을 등록하고, 어떤 URL로 요청이 들어오면 어떤 서블릿이 처리해야하는 지 등의 매핑 정보를 담고 있다
    
    따라서 요청을 받기 위해서는 web.xml에 먼저 정보를 작성해야 하며, 이후 실행 로직 또한 매 요청 마다의 요청-응답 로직을 작성해야 한다.  get요청의 경우 doGet()을 이용, post요청의 경우 doPost()을 구현을 해서 사용한다
    
    2. **Spring MVC**
    
    서블릿 방식에 비하면 구조화 된 방식으로 요청을 처리할 수 있으며 코드가 간결하고 유지보수에 용이하다. 
    
    URL요청에 대한 정보를 @Controller, @RequestMapping으로 정보를 전달하면, 스프링 내부에 DispatchServlet이 있는데 이곳에서 자동으로 어떤 컨트롤러에서 요청을 처리해야 할지 자동으로 매핑해준다.
    
     
    
- **Spring MVC가 편한 이유**
    
    전통적인 서블릿 방식은 모든 요청 처리를 doGet(), doPost()에 다 담아서 결국 하나의 요청에 
    
    “비즈니스 로직 + 요청 처리 + 출력” 코드가 한 번에 섞여있음 → 유지보수가 힘듦
    
    또한, 어느 정도 같은 코드가 계속 반복된다는 불편함도 존재
    
    결국 Spring MVC가 편한 이유는 **프레임 워크가 위 부분 중 일정 부분을 대신 관리해주기 때문이다.** 
    
    - Controller
    - ReqeustMapping
    - DispatcherServlet

- **DispatcherServlet 내부 분석**
    
    DispatcherServlet은 Spring이 제공하는 특수한 서블릿으로, **모든 요청을 중앙에서 받아 분기하고 흐름을 제어하는 역할**을 한다.
    
    > DispatcherServlet = 요청의 진입점이자 전체 흐름의 조율자
    > 
    
    **동작 흐름**
    
    [요청] → [DispatcherServlet] → [HandlerMapping](어떤 Controller인지 찾기) → [HandlerAdapter](어떻게 호출하지 결정) → Controller → ViewResolver → View → 응답


  - **AOP**
    
    **AOP는 관점 지향 프로그래밍**을 의미한다. OOP는 핵심 관심사(각각의 특징)에 중점을, AOP는 공통 관심사에 중점을 둔 프로그래밍이다. 결국 절차, 객체 처럼 다른 개념이라고 보기 보다는 객체 지향 프로그래밍에서 공통된 부분을 더 효율적으로 다루기 위한 보안 기술이 AOP라고 생각하면 된다.
    
    **AOP의 핵심 개념**
    
    - Advice : 언제, 어떤 공통 로직을 실행할지 정의(ex. before, after)
    - JoinPoint: Advice가 언제 실행될지 정의
    - Pointcut: Advice를 실행할 조건
    - Aspect: Advice와 Pointcut을 합친 모듈
    - Weaving: Advice에 핵심 코드를 끼워 넣는 작업
    
    런타임 위빙 vs 컴파일 타임 위빙
    
    - 컴파일 타임 위빙은 컴파일 시점에 AOP 코드가 주입 되며, 런타임 위빙은 런타임에 프록시 객체를 만들어 동적으로 주입한다. 실제로 런타임 위빙이 많이 사용 된다.
   
  - **AOP의 프록시 패턴**
    - 프록시(Proxy)를 번역하면 대리자, 대변인의 의미를 갖고 있다. 대리자, 대변인은 누군가를 대신해서 그 역할을 수행하는 존재이다. 즉, 이를 프로그램에도 똑같이 적용하면 프록시란 어떤 일을 대신 하는 것을 의미한다
    
    이를 Spring AOP에도 적용해 보면, spring은 AOP를 위해 프록시 객체를 만드는데 client로부터 요청이 오면 그 후에 진짜 객체의 메서드를 가져와서 실행시킨다.

  - **도전 과제**
   
    - 프로젝트 AOP 기반 Logging 전략을 적용하면, Security의 JWT 인증 검증 및 예외 처리 부분, 그리고 테스트를 위한 메서드 실행 시간 측정 등의 공통 로직을 분리하면, 핵심 비즈니스 로직과 관심사를 명확히 분리할 수 있어서 가독성이 좋아지고 유지보수성 또한 향상될 것이다.
   
          
