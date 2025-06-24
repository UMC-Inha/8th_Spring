# 🎯핵심키워드

---

# 1. Spring Security

### 1. Spring Security란?

**Spring Security**는 Spring 애플리케이션을 위한 **보안 프레임워크**로, 주로 아래 두 가지 기능을 수행해:

- **인증 (Authentication):** 사용자가 누구인지 확인 (예: 로그인)
- **인가 (Authorization):** 해당 사용자가 무엇을 할 수 있는지 확인 (예: 어떤 URL, 메서드 접근 가능 여부)

Spring Security는 서블릿 필터 기반 구조를 가지고 있음으로, 요청이 Controller에 도달하기 전에 필터 체인을 통해 보안 처리가 이뤄짐.

### 2. Spring Security 핵심 컴포넌트


| 개념                  | 실제 역할                                              | 사용 위치                                 | 예시                                                                |
|---------------------|------------------------------------------------------|----------------------------------------|-------------------------------------------------------------------|
| Authentication      | 인증 정보를 담는 객체                                     | SecurityContextHolder 내부                | 로그인 성공 시 Authentication 객체 생성                                   |
| Authorization       | 권한 검사 (역할 기반 접근 제어)                              | URL 접근 제어, 메서드 보안                    | `.hasRole("ADMIN")`, `@PreAuthorize("hasRole('USER')")`           |
| SecurityContextHolder | 현재 로그인한 사용자의 Authentication 객체를 저장하는 장소 (ThreadLocal 기반) | 어디서든 사용자 정보 접근 가능                  | `SecurityContextHolder.getContext().getAuthentication()`          |
| AuthenticationManager | 인증을 수행하는 인터페이스                                   | 내부적으로 ProviderManager가 구현            | 로그인 시 이 객체가 사용자 인증 처리                                     |
| UserDetailsService  | DB나 외부 API에서 사용자 정보를 로드                           | 커스텀 가능                                | `loadUserByUsername(String username)` 오버라이딩                   |
| UserDetails         | 사용자 객체 도메인 (아이디, 비번, 권한 포함)                       | UserDetailsService가 반환                  | `User implements UserDetails`                                     |
| GrantedAuthority    | 사용자 권한(ROLE_*) 표현                                   | `UserDetails.getAuthorities()`에서 제공   | `ROLE_ADMIN`, `ROLE_USER`                                         |
| SecurityFilterChain | 인증/인가 필터들의 체인 구조                                   | 모든 요청 전에 실행됨                         | `UsernamePasswordAuthenticationFilter`, `JwtAuthenticationFilter` 등 |
| PasswordEncoder     | 비밀번호 암호화/검증                                      | 회원가입, 로그인 비번 비교 시                   | `BCryptPasswordEncoder().encode(password)`                        |



이것들을 통해서 인증/인가 과정에서 어떻게 연결되는가 하면

1. **로그인 요청**이 들어오면
2. `UsernamePasswordAuthenticationFilter`가 가로채고
3. 입력한 사용자 정보는 `AuthenticationManager`로 전달
4. `AuthenticationManager`는 `UserDetailsService`를 통해 사용자를 찾고 (`loadUserByUsername`)
5. `UserDetails` 객체를 반환 → 비밀번호 일치 확인 (`PasswordEncoder`)
6. 인증 성공 → `Authentication` 객체 생성 → `SecurityContextHolder`에 저장
7. 이후 요청은 이 `SecurityContextHolder`에 저장된 `Authentication` 정보를 통해 인가됨

### Spring Security 구조

```markdown
[요청]
   |
   v
[SecurityFilterChain]
   |
   v
[UsernamePasswordAuthenticationFilter]  --> (로그인 시 작동)
   |
   v
[AuthenticationManager]
   |
   v
[UserDetailsService] --> DB 조회
   |
   v
[UserDetails]
   |
   v
인증 성공 → SecurityContext에 저장
```


## 인증(Authentication) 흐름

1. 로그인 요청 (`POST /login`)
2. `UsernamePasswordAuthenticationFilter`가 요청을 가로챔
3. 아이디/비밀번호로 `AuthenticationManager`에 인증 위임
4. `UserDetailsService`가 사용자 조회
5. 비밀번호 비교 → 성공하면 `Authentication` 객체 생성
6. `SecurityContextHolder`에 `Authentication` 저장

## 인가(Authorization) 흐름

요청이 들어오면,

- Spring Security는 `Authentication`의 `GrantedAuthority`를 확인하여
- 요청 URL, 메서드에 대한 접근 권한이 있는지 검사

Spring Security에는 여러가지 인증 방식인

- **폼 로그인** (`formLogin()`)
- **HTTP Basic Auth** (`httpBasic()`)
- **JWT 인증** (Custom Filter 필요)
- **OAuth2 인증** (`spring-security-oauth2-client`)
- **세션 기반 인증 vs 토큰 기반 인증 등이 존재함.**

# 2. 인증(Authentication)과 인가(Authorization)

## 1. 인증(Authentication)
: 사용자의 신원을 검증하는 과정

### 절차

1. 사용자가 `아이디/비밀번호` 또는 `토큰` 등을 제출
2. 시스템이 해당 정보로 **본인 여부**를 확인
3. 본인이 맞다면 `Authentication` 객체 생성 및 저장

### 결과

- 로그인 성공 → 인증된 사용자 정보 유지됨 (세션/토큰 등)
- 로그인 실패 → `401 Unauthorized`


## 2. 인가(Authorization)
: 인증된 사용자가 어떤 자원에 접근 가능한지 판단하는 과정

### 절차

1. 사용자가 **인증된 상태**로 요청을 보냄
2. 시스템이 사용자의 `권한(Role, Permission)`을 확인
3. 요청한 자원에 접근 권한이 있는지 판단

### 결과

- 권한 있음 → 요청 처리
- 권한 없음 → `403 Forbidden`

Spring Security 실제 작동 구조를 대강 알아본다면

```markdown
[클라이언트 요청]
    |
    V
[Spring Security Filter Chain]
    |
    V
[인증(Authentication) 여부 확인]
    |
    ├── 인증 안됨 → 로그인 페이지로 이동 (401)
    └── 인증 됨
           |
           V
    [인가(Authorization) 체크 → ROLE 확인]
           |
           ├── 권한 없음 → 403 Forbidden
           └── 권한 있음 → Controller 실행
```

### 그렇다면 둘의 차이는?

## 핵심 차이 요약

| 구분              | 인증 (Authentication)                             | 인가 (Authorization)                            |
|-----------------|--------------------------------------------------|------------------------------------------------|
| 의미              | "너 누구야?"                                       | "너 이거 해도 돼?"                                  |
| 목적              | 사용자의 신원 확인                                    | 사용자의 권한 검증                                   |
| 시점              | 시작 전 (로그인 시)                                   | 요청 처리 중/후                                     |
| 방식              | 아이디/비밀번호, OTP, 인증서, 지문, JWT 등               | 역할(ROLE), 권한, 정책 기반 접근                          |
| 실패 시            | 로그인 실패                                         | 접근 거부 (403 Forbidden)                         |
| Spring 구성 요소   | `Authentication`, `AuthenticationManager`, `UserDetailsService` 등 | `@PreAuthorize`, `hasRole()`, URL 접근 제어 등   |
| 예시              | "이 계정은 UMC입니다"                                | "UMC /admin 페이지에 접근할 수 없습니다"              |

## 인증과 인가를 분리해야 하는 이유

- **보안성 강화:** 인증만 있다고 아무나 모든 리소스를 접근하게 하면 당연히 매우 위험
- **유연한 권한 설계:** 한 사용자가 다양한 권한을 가질 수 있음
- **유지보수성 향상:** 인증/인가 로직을 나눔으로써 변경에 유연하게 대응 가능

# 3. 세션과 토큰

## 1. 세션 기반 인증

### 동작 원리

1. 사용자가 로그인 (`POST /login`)
2. 서버가 인증 성공 → 사용자 정보를 **서버에 저장**하고, **세션 ID 생성**
3. 세션 ID를 **쿠키에 저장해서 클라이언트에게 전송**
4. 이후 모든 요청에서 클라이언트는 이 **세션 ID 쿠키**를 함께 전송
5. 서버는 세션 ID로 사용자 상태를 복원해서 요청 처리

### 동작 흐름 예시
```markdown
[Client] ⇄ [Server]
1. 로그인 → username/password 전송
2. 서버 인증 성공 → 세션 저장 + 세션 ID 생성
3. 응답에 Set-Cookie: JSESSIONID=abc123
4. 이후 요청 시 Cookie: JSESSIONID=abc123 전송
5. 서버가 세션 ID로 사용자 상태 유지
```
**하지만,, 세션 기반 인증은**

- 세션 탈취(Session Hijacking)에 취약 → HTTPS 필수
- CSRF 취약점 존재 → 토큰 or SameSite 쿠키 정책 필요

## 2. 토큰 기반 인증 (특히 JWT)

### 동작 원리

1. 사용자가 로그인 (`POST /login`)
2. 서버가 인증 성공 → 사용자 정보로 **JWT(토큰)** 생성
3. 클라이언트에 **JWT 토큰 응답**
4. 이후 요청 시 `Authorization: Bearer <JWT>` 헤더로 전송
5. 서버는 **토큰만으로 인증 판단 (무상태 Stateless)**

**동작 흐름 예시**
```markdown
[Client] ⇄ [Server]
1. 로그인 요청 → username/password
2. 서버 인증 성공 → JWT 생성 후 응답
3. 클라이언트가 JWT 저장 (보통 LocalStorage)
4. 이후 요청 → Authorization: Bearer <JWT>
5. 서버는 JWT 검증 → 사용자 인증
```
| 파트       | 설명                                 |
|------------|--------------------------------------|
| Header     | 알고리즘 + 타입 (예: HS256, JWT)         |
| Payload    | 사용자 데이터 (예: `sub`, `role` 등)     |
| Signature  | 비밀키로 서명 (위조 방지)                  |

JWT의 구조는 간단하게 위와 같이 설명 가능하고, 서버는 **Payload**를 복호화하고, **Signature**로 진짜인지 검증한다.

### 세션(Session) vs 토큰(Token) 인증의 차이
| 구분                 | 세션 기반 인증                                   | 토큰 기반 인증                                     |
|--------------------|----------------------------------------------|--------------------------------------------------|
| 인증 상태 저장 위치     | 서버 (서버 메모리 또는 DB)                          | 클라이언트 (보통 JWT 토큰)                          |
| 서버 확장성            | 낮음 (상태를 서버가 저장해야 함)                     | 높음 (무상태 Stateless 구조)                        |
| 인증 정보 전달 방식     | 세션 ID가 쿠키에 저장됨                             | JWT 또는 Access Token이 HTTP 헤더에 포함됨             |
| 인증 처리 방식         | 세션 ID로 서버에서 사용자 상태를 조회함                | 토큰 자체에 사용자 상태 정보가 포함됨                  |
| 주로 쓰이는 환경       | 웹 애플리케이션, 전통적인 MVC 아키텍처                  | SPA(Single Page Application), 모바일, RESTful API |
| 보안 요소             | 세션 탈취 방지 (CSRF 방어 필요)                        | 토큰 위조 방지 (서명), 만료 시간 설정, 토큰 재발급 관리     |


# 4. 액세스 토큰(Access Token)과 리프레시 토큰(Refresh Token)

## 1. Access Token

: Access Token은 사용자가 인증되었음을 나타내며, API 접근 권한을 부여받는 토큰

### 포함 정보 (JWT 기준)

- `sub`: 사용자 ID
- `exp`: 만료 시간
- `roles`: 권한 정보
- `iat`: 발급 시간
- 일반적으로 짧게 유지됨 (5~30분 정도)
- 짧아야 보안상 유리 (탈취 위험 감소)

## 2.  Refresh Token

### : **Access Token이 만료되었을 때**, 사용자가 다시 로그인하지 않고 **Access Token을 재발급 받을 수 있도록 해주는 토큰**

포함 정보

- 최소한의 사용자 정보 (예: userId, exp)
- **재발급 전용**, 절대 API 요청에 사용 X
- 일반적으로 **몇 시간~며칠** 유지
- 길게 설정하지만, 보안에 더 민감하게 관리해야 함

### 토큰 흐름 (Access Token, Refresh Token)

- **로그인 요청** (`POST /login`)
- 서버가 `Access Token` + `Refresh Token` 발급
- 클라이언트:
    - Access Token: 메모리 or localStorage
    - Refresh Token: **HttpOnly 쿠키 or Secure Storage**
- 사용자가 API 호출 → Access Token으로 인증
- **Access Token 만료 시**:
    - Refresh Token으로 새로운 Access Token 요청
    - 유효하면 재발급, 아니면 로그인 페이지로 유도

## Refresh Token 관리 방식

### 1. 서버 저장 방식 (Secure)

- Refresh Token을 DB or Redis에 저장
- 클라이언트가 보낼 때 서버에서 대조 확인
- 탈취 당해도 서버에서 무효화 가능

### 2. Stateless 방식 (간편하나 위험)

- Refresh Token도 JWT로 발급
- 클라이언트에만 저장 (서버 저장 X)
- 발급/무효화 관리 어려움 → 보안 위험도 ↑

⇒ 실무에서는 대부분 1번 방식을 채택한다.

### Access Token과 Refresh Token의 비교

| 항목             | Access Token                                    | Refresh Token                                          |
|------------------|--------------------------------------------------|--------------------------------------------------------|
| 목적             | API 접근 권한 부여                                 | Access Token 재발급용                                     |
| 수명             | 짧음 (몇 분 ~ 수십 분)                              | 김 (수 시간 ~ 수 일)                                      |
| 저장 위치         | 클라이언트 (보통 메모리 또는 localStorage)            | 클라이언트 (보안 필요, HttpOnly 쿠키 권장)                  |
| 사용 대상         | API 서버                                          | 인증 서버                                                 |
| 인증 포함 여부      | 포함됨 (예: `sub`, `exp`, `role` 등)                  | 사용자 식별 정보만 포함                                    |
| 만료 후           | API 호출 불가 → Refresh Token으로 Access 재발급 시도     | 다시 로그인 필요 또는 Refresh 토큰 정책에 따라 재발급 가능     |


