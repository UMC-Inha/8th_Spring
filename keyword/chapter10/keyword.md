## - 키워드 정리

#### - Spring Security

- 인증과 인가를 담당하는 강력하고 보안 프레임워크

- 구성 요소

    - **Servlet Filter Chain**

        - Spring Security는 서블릿 필터를 기반으로 동작하며, 모든 HTTP 요청이 FilterChainProxy라는 최상위 필터를 통해 전달된다.

        - FilterChainProxy: HTTP 요청을 받아 요청 URL 패턴에 맞는 SecurityFilterChain 실행

        - SecurityFilterChain: UsernamePasswordAuthenticationFilter와 같은 여러 필터들의 순서를 정의하며, 어떤 URL에 어떤 필터를 적용할지 결정한다.

    - **SecurityContext & Authentication**

        - **SecurityContext**: 내부에 Authencation 객체를 담아 두며, 이 객체에 인증된 사용자 정보(Pricipal)와 권한(GrantedAuthority) 정보를 보관한다.

        - **Authencation**: 사용자가 인증된 후 생성되는 토큰. 이 객체가 SecurityContext에 저장되어야 애플리케이션 어디서든 SecurityContextHolder.getContext().getAuthentication()으로 사용자 정보를 꺼낼 수 있다.

    - **AuthenticationManager**

        - 인증 흐름의 진입점.

    - **UserDetailService / UserDetails**

        - 유저 저장소에서 사용자 정보를 조회해 Spring Security 표준 인터페이스에 맞춰 감싸는 역할

        - **UserDetailService**: loadUserByUsername() 메서드를 구현해 주어진 아이디로 UserDetails를 관리한다.

        - **UserDetails**: 사용자 이름, 암호, 권한 목록, 계정 활성화 여부와 같은 속성을 담는 인터페이스.

- Spring Security는 세션 기반 방식과 JWT 기반 방식을 모두 지원한다.

#### - 인증(Authentication)과 인가(Authorization)

- **인증(Authentication)**

    - 사용자가 내가 누구인지 증명하는 과정

    - 아이디 + 비밀번호, OTP 등으로 사용자 신원을 검증

    - 관련 키워드

        - **Principal**: 인증된 사용자 주체

        - **Credentials**: 검증 수단

        - **Authencation Token**: 인증 결과를 담는 객체

    - Spring의 인증 절차

        - 요청 가로채기(SecurityFilterChain)

            - 모든 HTTP 요청을 필터가 가로챈다.

        - 자격증명 추출(Authencation Token 생성)

            - username, pasword 파라미터를 꺼내서 사용

        - 인증 매니저 호출(AuthencationManager)

        - 유저 정보 검증(UserDetailsService)

        - 인증 성공 시 SecurityContext에 저장

- **인가(Authorization)**

    - 사용자가 무엇을 할 수 있는가를 결정하는 과정

    - 역할(Role), 권한(Authority) 기반으로 접근 제어

    - 관련 키워드

        - **GrantedAuthority**: 권한 단위

    - Spring의 인가 절차

        - 필터 기반 인가(FilterSecurityInterceptor)

        - 접근 결정(AccessDecisionManager)

        - 인가

#### - 세션과 토큰

- **세션**

    - 서버가 사용자 별로 고유 식별자인 세션 ID를 생성해 관리하고, 그 안에 인증 정보나 장바구니 같은 상태를 저장하는 방식

    - 동작 방식

        - 로그인 시 세션을 만들고, 세션 ID를 쿠키로 클라이언트에 전달

        - 클라이언트는 이후 모든 요청에 이 쿠키를 포함

        - 서버는 쿠키의 세션 ID로 세션 저장소(메모리, Redis)를 조회해 사용자 상태를 재구성

    - 장점

        - 구현이 쉽고, 세션 만료나 강제 로그아웃 같은 기능을 프레임워크에서 제공한다.

    - 단점

        - 서버에 상태를 저장하므로, **수평 확장 시 세션 공유가 필요하다.**

- **토큰**

    - 서버가 사용자 정보나 권한을 담아 암호화&서명된 토큰을 발급하고, 클라이언트가 이를 보관해 매 요청마다 전송하는 방식

    - 동작 방식

        - 로그인 성공 시 서버가 사용자 식별 정보와 클레임을 포함해 토큰 발급

        - 클라이언트는 로컬 스토리지나 메모리에 이 토큰을 저장

        - 이후 요청 헤더에 토큰을 포함해 서버에 전송

        - 서버는 토큰 서명을 검증해 사용자 정보와 권한을 재구성

    - 장점

        - 서버에 상태를 저장하지 않아 **stateless**하다. 확장성이 뛰어나고 마이크로서비스 간 연동이 용이하다.

        - 탈취된 토큰을 즉시 무효화하기 어려워 블랙리스트 관리가 필요하다. 토큰 크기 때문에 네트워크 오버헤드가 발생할 수 있다.

- **세션 방식과 토큰 방식은 각각 어떤 상황에 적합할까??**

    - **세션 방식이 적합한 경우**

        - 상태 유지를 중시하는 관리형 어플리케이션

        - 기업 내부 시스템

        - 쇼핑몰, 게시판과 같은 서비스

        - 관리자 대시보드

    - **토큰 방식이 적합한 경우**

        - 모바일 어플리케이션 -> 세션을 쓸 경우 별도의 쿠키 처리 로직 필요

        - SPA => 백엔드 API를 호출할 때, 토큰 기반으로 CORS 환경에서도 인증이 자유롭다.

        - MSA 환경

#### - 엑세스 토큰(Access Token)과 리프레쉬 토큰(Refresh Token)

- **Access Token**

    - 클라이언트가 API에 접근할 때 HTTP 요청 헤더에 포함시켜 보내는 짧은 수명의 토큰

    - 서버가 토큰의 서명, 유효기간, 클레임을 빠르게 검증한 후, 해당 요청을 승인하거나 거부할 수 있다.

    - 매 요청마다 재검증을 수행하기 때문에, 별도의 세션 저장소 없이 stateless하게 인증이 가능하다.

    - 일반적으로 **수분~수십 분** 단위로 짧게 설정해, 탈취 위험을 줄이고 토큰 남용을 방지한다.

    - 만료 시점이 빠르기 때문에 자주 갱신 로직이 필요하다.

- **Refresh Token**

    - Access Token이 만료되었을 때 새로운 Access Token을 발급받기 위해 사용하는 긴 수명의 토큰

    - 클라이언트가 만료된 Access Token으로 요청할 경우, 서버가 Refresh Token을 검증 후 새 Access Tokend을 발급해준다.

    - 사용자가 매번 로그인하지 않고도 일정 기간동안 자동으로 인증 상태를 유지해준다.

    - 일반적으로 **며칠 단위**로 설정해, 자주 로그인을 요구하지 않으면서 탈취 시에는 위험을 관리해준다.

    - Refresh Token이 외부에 노출되지 않도록 안전하게 보관해야 한다.

