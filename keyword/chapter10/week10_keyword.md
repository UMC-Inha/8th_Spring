- **Spring Security**

  Spring Securiy란?

    - Spring 기반 어플리케이션의 보안을 담당하는 강력한 프레임워크
    - 주로 2가지 핵심 기능인 인증과 권한 부여 제공

  Spring Security 역할

    - 누가 들어오는지 확인 (인증, Authentication)
    - 들어온 사람이 어디에 갈 수 있는지 결정 (인가, Authorization)
    - 위험한 상황으로부터 보호 (다양한 보안 위협 방어)

  인증 (Authentication)

    - "당신이 누구인지 증명하시오" 묻는 과정
    - 사용자가 제공한 Credential (보통 아이디, 비번) 을 확인하여 신원 검증

  Authentication 인터페이스 -> 인증된 사용자의 정보 담고있음

    - getPrincipal() -> 사용자의 식별 정보 반환
    - getAuthorities() -> 사용자의 권한 정보 반환

  인가 (Authorization)

    - "이 리소스에 접근할 권한이 있나요?" 확인하는 과정
    - 인증된 사용자가 특정 리소스에 접근할 수 있는지, 특정 동작 수행할 수 있는지 결정

  왜 Spring Security를 사용하는가?

    - 인증/인가 기능 제공 : 사용자 인증, 권한 관리, 세션 유지 등 복잡한 보안 문제 간단히 처리
    - Spring 생태계 : Spring Framework와 자연스럽게 통합되어 있음 -> 개발 과정에서 보안 기능 추가하는 데 용이

    - 확장성 : 기본 보안 설정을 바탕으로 다양한 요구 사항에 맞게 확장 가능

  # Spring Security 주요 컴포넌트

  → 여러 주요 컴포넌트가 협력하여 인증과 인가를 처리

    1. AuthenticationManager
    - 인증(Authentication) 과정 관리하는 중심 컴포넌트
    - 사용자가 로그인 시도할 때 AuthenticationManager 가 사용자의 자격 증명을 받아 이를 인증할 수 있는 프로세스 호출

    1. AuthenticationProvider
    - 실제 인증 로직 처리하는 역할 담당
    - 여러개 존재 가능, 각각 특정 인증 방식 처리
        - 예시로 이메일/비밀번호 인증은 하나의 AuthenticationProvider 에서 처리, 소셜 로그인의 경우 또 다른 AuthenticationProvider 가 담당

    1. UserDetailsService
    - 사용자 정보 불러오고 검증하는 서비스
    - DB 또는 다른 저장소에서 사용자 정보 가져오고, 이를 UserDetails 객체로 변환
        - UserDetails 객체 : 사용자의 아이디, 비밀번호, 권한(Role) 등 다양한 정보 포함

    1. SecurityContext
    - 인증이 완료된 사용자 정보를 저장하는 컨텍스트
    - 이 정보는 애플리케이션 전반에서 공유, SecurityContextHolder 를 통해 접근 가능 (SecurityContextHolder.getContext())

  # SecurityContextHolder

    - 현재 보안 컨텍스트에 대한 세부 정보 보관
    - 기본적으로 ThreadLocal 사용, 동일한 스레드 내에서는 각 사용자의 인증 정보를 개별적으로 유지
      -> 즉, 요청마다 인증된 사용자의 정보 보존, 다른 요청에선 다른 사용자의 정보를 처리할 수 있도록 함

  # Filter Chain

    - Spring Security 에서 HTTP 요청 처리할 때 사용하는 일련의 패턴
    - 각 패턴은 특정 보안 기능 담당, 요청이 애플리케이션에 도달하기 전에 이 필터들을 순차적으로 통과

  주요 필터들

    1. SecurityContextPersistenceFilter
    - 요청 간 SecurityContext 를 유지
    - 새 요청이 들어올 때 이전에 인증된 사용자의 정보를 복원

    1. UsernamePasswordAuthenticationFilter
    - 폼 기반 로그인 처리
    - 사용자가 제공한 username, password 확인하여 인증시도

    1. AnonymousAuthenticationFilter
    - 이전 필터에서 인증되지 않은 요청에 대해 익명 사용자 인증 제공

    1. ExceptionTranslationFilter
    - Spring Security 예외를 HTTP 응답으로 변환
    - 인증 실패 시 로그인 페이지로 리다이렉트 하거나, 인가 실패 시 403 오류 반환

    1. FilterSecurityInterceptor
    - 접근 제어 결정 내리는 마지막 필터
    - 현재 인증된 사용자가 요청한 리소스에 접근할 권리 있는지 확인

  Filter Chain 동작 방식

    1. 클라이언트로부터 요청 들어오면, 요청은 Filter Chain의 첫 번째 필터부터 순차적으로 통과
    2. 각 필터는 요청을 검사하고 필요한 작업 수행
    3. 필터는 요청을 다음 필터로 전달하거나, 특정 조건에 따라 요청 처리 중단 가능
    4. 모든 필터를 통과한 요청만이 실제 어플리케이션 로직에 도달
- **인증(Authentication)과 인가(Authorization)**

  인증 (Authentication)

    - "당신이 누구인지 증명하시오" 묻는 과정
    - 사용자가 제공한 Credential (보통 아이디, 비번) 을 확인하여 신원 검증

  Authentication 인터페이스 -> 인증된 사용자의 정보 담고있음

    - getPrincipal() -> 사용자의 식별 정보 반환
    - getAuthorities() -> 사용자의 권한 정보 반환

  인가 (Authorization)

    - "이 리소스에 접근할 권한이 있나요?" 확인하는 과정
    - 인증된 사용자가 특정 리소스에 접근할 수 있는지, 특정 동작 수행할 수 있는지 결정

  # Spring Security 인증 인가 흐름

  인증 흐름

    - 사용자의 신원 확인하는 과정

    1. 사용자 로그인 요청
        1. 사용자가 로그인 폼에 Credentials 입력하고 제출
    2. AuthenticationFilter
        1. usernamePasswordAuthenticationFilter 가 요청 가로채고 Authentication 객체 생성
    3. AuthenticationManager
        1. AuthenticationManager가 적절한 AuthenticationProvider 선택하여 인증 위임
    4. AuthenticationProvider
        1. 선택된 AuthenticationProvider는 UserDetailsService 를 사용하여 사용자 정보 로드
        2. 로드된 정보를 바탕으로 비밀번호 검증
    5. UserDetailsService
        1. DB나 다른 저장소에서 사용자 정보 조회
    6. SecurityContext
        1. 인증 성공하면, Authentication 객체가 SecurityContext에 저장됨

  인가 흐름

    - 인증된 사용자가 특정 리소스에 접근할 권한 있는지 확인하는 과정

    1. 리소스 접근 요청
        1. 인증된 사용자가 보호된 리소스에 접근 시도
    2. FilterSecurityInterceptor
        1. FilterSecurityInterceptor가 요청을 가로채고 권한 검사 시작
    3. AccessDecisionManager
        1. AccessDecisionManager 는 현재 사용자의 권한과 요청된 리소스의 필요 권한 비교
    4. 권한 확인
        1. SecurityContext 에서 현재 인증된 사용자의 권한 정보 조회
    5. 접근 결정
        1. 사용자의 권한이 충분하면 리소스 접근이 허용됨
        2. 권한이 부족하면 AccessDeniedException 이 발생하고 접근 거부됨

  ![image](https://github.com/user-attachments/assets/7892d23f-a2e7-4331-80af-d749a718bf30)


  → 위에 설명한 그대로 인증과 인가 처리되는 과정

- **세션과 토큰**

  # 세션 기반 VS 토큰 기반 로그인 방식

  사용자가 ‘로그인 했는지 아닌지’ 를 서버가 기억하는 방식

  → 세션 기반 인증과 토큰 기반 인증의 가장 큰 차이점

  ### 세션 기반 인증

  특징

    - 로그인 시 서버가 세션 객체 생성, 클라이언트에 쿠키 전달
        - 이후 클라이언트의 요청마다 쿠키를 통해 사용자 식별
    - 사용자의 정보가 서버의 메모리에 저장됨 (Stateful)
    - 클라이언트는 쿠키로 인증을 계속 유지
        - 로그아웃 시에는 세션 삭제해야 함

  로그인 시 formLogin() 방식으로 세션 생성

    ```java
    .formLogin((form) -> form
    			.loginPage("/login")
    			.defaultSuccessUrl("/home", true)
    			.permitAll()
    )
    ```

  인증 정보는 서버 세션에 저장, 브라우저는 JSESSIONID 쿠키를 전송

  서버는 해당 ID 로 세션을 찾아 사용자 인증 상태 유지

  UserDetailsService 를 이용하여 DB 내의 사용자 정보를 세션에 올림

    ```java
    @Service
    @RequiredArgsConstructor
    public class CustomUserDetailsService implements UserDetailsService {
    
        private final MemberRepository memberRepository;
    
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            Member member = memberRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("해당 이메일을 가진 유저가 존재하지 않습니다: " + username));
    
            return org.springframework.security.core.userdetails.User
                .withUsername(member.getEmail())
                .password(member.getPassword())
                .roles(member.getRole().name())
                .build();
        }
    }
    ```

  여기서 CustomUserDetailsService 는 loadByUsername 메서드로 AuthenticationManager 의 호출을 받아 DB에서 findByEmail 메서드로 사용자 정보를 조회

  조회된 Member 엔티티를 기반으로 Spring Security가 요구하는 UserDetails 객체 반환

  ### 토큰 기반 인증

  특징

    - 로그인 시 서버가 토큰 발급, 클라이언트는 이를 저장
        - 이후 요청에 Authorization 헤더로 전송
    - 사용자의 정보는 토큰 자체에 포함, 서버는 이를 저장하지 않음
        - 로그아웃 시엔 토큰을 삭제하거나 서버에서 블랙리스트 관리 필요

  두 인증의 가장 큰 차이는

    - 서버가 상태를 유지하느냐 유지하지 않느냐 이다.
- **액세스 토큰(Access Token)과 리프레시 토큰(Refresh Token)**

  액세스 토큰

    - 짧은 수명의 자격 증명용 토큰
    - 클라이언트가 API 서버에 요청을 보낼 때 이 토큰을 포함하여 자신이 인증된 사용자이며 특정 작업을 수행할 권한이 있음을 증명

  특징:

    - 짧은 수명 (Short-lived): 보안을 위해 유효 기간이 매우 짧습니다 (예: 몇 분에서 몇 시간). 토큰이 탈취되더라도 악용될 수 있는 시간이 제한됩니다.
    - 직접적인 권한 부여: 이 토큰 자체에 사용자 정보, 권한(스코프), 유효 기간 등이 포함될 수 있으며, 리소스 서버는 이 토큰을 검증하여 요청을 허용할지 결정합니다. (JWT - JSON Web Token 형태가 흔함)
    - 휴대성 (Portable): 별도의 DB 조회 없이 토큰 자체만으로 검증이 가능하여 확장성(Scalability)에 유리합니다.
    - 탈취 시 위험: 짧은 수명이지만, 만약 탈취된다면 해당 유효 기간 동안은 악용될 수 있습니다.

  주요 역할:

    - 클라이언트가 보호된 API 리소스에 접근할 때 인증 및 권한 부여.
    - 세션 관리.

  리프레시 토큰

    - 액세스 토큰이 만료되었을 때 새로운 액세스 토큰을 발급받기 위해 사용되는 긴 수명의 토큰
    - 사용자가 매번 로그인할 필요 없이 장기간 세션을 유지할 수 있도록 도움

  **특징:**

    - **긴 수명 (Long-lived):** 액세스 토큰보다 훨씬 긴 유효 기간을 가집니다 (예: 며칠, 몇 주, 몇 달 또는 만료 기한 없음).
    - **새로운 액세스 토큰 발급:** 클라이언트는 리프레시 토큰을 인증 서버(Authorization Server)에 보내 새로운 액세스 토큰을 요청합니다. 이때 보통 리프레시 토큰은 한 번만 사용되거나, 재사용 가능한 경우에도 탈취에 대한 보안 장치가 마련됩니다.
    - **보안 강화:** 액세스 토큰에 비해 훨씬 민감한 정보이므로, 일반적으로 더 안전한 저장소(예: HttpOnly 쿠키)에 저장되고, 사용될 때마다 엄격한 검증을 거칩니다.
    - **취소 가능 (Revocable):** 서버에서 언제든지 리프레시 토큰을 무효화할 수 있습니다. 사용자가 로그아웃하거나 비밀번호를 변경했을 때, 또는 비정상적인 활동이 감지될 때 서버에서 강제로 리프레시 토큰을 만료시켜 세션을 끊을 수 있습니다.

  **주요 역할:**

    - 액세스 토큰 만료 시 사용자 재인증 없이 새 액세스 토큰 발급.
    - 사용자의 장기간 세션 유지.
    - 액세스 토큰 탈취 시 공격의 노출 시간을 줄이고 보안을 강화.

  왜 두 토큰을 사용할까? ( 이중 토큰의 필요성 )

    - 액세스 토큰의 짧은 수명
        - 액세스 토큰이 탈취되더라도 짧은 유효 기간 때문에 공격자가 시스템에 접근할 수 있는 시간이 제한됨.
        - 모든 API 요청에 사용되므로, 노출될 위험이 높음
    - 리프레시 토큰의 긴 수명 및 보안 강화
        - 사용자는 액세스 토큰이 만료될 때마다 다시 로그인할 필요 없이 백그라운드에서 자동으로 새 토큰을 받을 수 있어 편함
        - 리프레시 토큰은 오직 인증 서버와만 통신하며, 보호된 리소스에 직접 접근하는 데 사용되지 않습니다. 따라서 노출될 위험이 상대적으로 적음
        - 탈취되더라도, 서버에서 해당 리프레시 토큰을 무효화할 수 있는 강력한 통제 메커니즘이 있습니다 (블랙리스트, 화이트리스트, 리프레시 토큰 회전 등).
