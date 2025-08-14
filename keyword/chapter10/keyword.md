### 1. **Spring Security**

---

- 스프링 애플리케이션 보안 프레임워크
- 인증, 인가, 필터 체인 기반으로 동작
    - credential 기반의 인증
        - **credential 방식**: username, password를 이용하는 방식
- HTTP 요청 전처리 필터를 통해 사용자 식별 및 권한 확인

![image.png](attachment:fdc0df31-1713-4977-82f3-9ccb67fb9bd2:image.png)

- 시큐리티 흐름

  **1. Http Request 수신**

    - > 사용자가 로그인 정보와 함께 인증 요청을 한다.

  **2. 유저 자격을 기반으로 인증토큰 생성**

    - > AuthenticationFilter가 요청을 가로채고, 가로챈 정보를 통해 UsernamePasswordAuthenticationToken의 인증용 객체를 생성한다.

  **3. FIlter를 통해 AuthenticationToken을 AuthenticationManager로 위임**

    - > AuthenticationManager의 구현체인 ProviderManager에게 생성한 UsernamePasswordToken 객체를 전달한다.

  **4. AuthenticationProvider의 목록으로 인증을 시도**

    - > AutenticationManger는 등록된 AuthenticationProvider들을 조회하며 인증을 요구한다.

  **5. UserDetailsService의 요구**

    - > 실제 데이터베이스에서 사용자 인증정보를 가져오는 UserDetailsService에 사용자 정보를 넘겨준다.

  **6. UserDetails를 이용해 User객체에 대한 정보 탐색**

    - > 넘겨받은 사용자 정보를 통해 데이터베이스에서 찾아낸 사용자 정보인 UserDetails 객체를 만든다.

  **7. User 객체의 정보들을 UserDetails가 UserDetailsService(LoginService)로 전달**

    - > AuthenticaitonProvider들은 UserDetails를 넘겨받고 사용자 정보를 비교한다.

  **8. 인증 객체 or AuthenticationException**

    - > 인증이 완료가되면 권한 등의 사용자 정보를 담은 Authentication 객체를 반환한다.

  **9. 인증 끝**

    - > 다시 최초의 AuthenticationFilter에 Authentication 객체가 반환된다.

  **10. SecurityContext에 인증 객체를 설정**

    - > Authentication 객체를 Security Context에 저장한다.

  출처 : https://velog.io/@hope0206/Spring-Security-%EA%B5%AC%EC%A1%B0-%ED%9D%90%EB%A6%84-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%97%AD%ED%95%A0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0


### 2. **인증(Authentication) vs 인가(Authorization)**

---

- 인증: **해당 사용자가 본인이 맞는지를 확인하는 절차** → `AuthenticationManager` 사용
- 인가: **인증된 사용자가 요청된 자원에 접근가능한가를 결정하는 절차** → `@PreAuthorize`, `hasRole()` 등 사용

```java
// 인가 설정 예시 (SecurityConfig)
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
);

```

- `http.authorizeHttpRequests(...)`: HTTP 요청에 대한 접근 권한을 설정함
- `requestMatchers("/admin/**")`: `/admin/`으로 시작하는 모든 URL 경로에 대해 아래 조건을 설정
- `hasRole("ADMIN")`: 해당 경로는 `ADMIN` 역할을 가진 사용자만 접근 가능
- `anyRequest().authenticated()`: 나머지 모든 요청은 **인증된 사용자**만 접근 가능 (로그인 필요).
    - 특정 권한을 얻기 위해서는 유저는 인증정보(Authentication)가 필요하고
    - 관리자는 해당 정보를 참고해 권한을 인가(Authorization)

### 3. **세션 vs 토큰**

---

- 세션: 로그인 시 서버가 세션 ID 발급 후 쿠키로 관리.
- 토큰: 서버가 JWT 발급 → 클라이언트가 요청마다 `Authorization: Bearer <token>` 헤더에 포함시킴.

```java
// JWT 필터 등록 예시
http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

```

- `jwtFilter`를 Spring Security 필터 체인에서 `UsernamePasswordAuthenticationFilter` **이전**에 등록
- JWT 토큰을 검사하는 필터를 로그인 인증 필터보다 먼저 실행해서 인증 처리
    - 이미 인증된 사용자로 인식해서 추가 로그인 없이도 접근 허용 가능
    - API 서버에서 토큰 기반 인증 방식 시, 매 요청마다 JWT로 인증 처리 가능
- JWT 토큰이란?
    - 사용자 정보를 담은 **작은 암호화된 토큰**
    - 자체적으로 신뢰할 수 있는 정보(서명 포함)를 담아 별도 세션 없이 인증 가능

### 4. **Access Token vs Refresh Token**

---

- Access Token: 짧은 수명, API 접근 권한 확인용.
- Refresh Token: Access Token 재발급용, 보안 중요 → 서버 DB에서 관리 추천.

```java
// Refresh Token으로 Access Token 재발급 예시 (Service 메서드)
if (isValidRefreshToken(refreshToken)) {
    return generateNewAccessToken(userId);
}

```

- `isValidRefreshToken(refreshToken)`:
    - 전달된 **리프레시 토큰(refreshToken)** 의 유효성을 검사
    - 만료 여부, 위조 여부, 서버 저장 정보와 일치하는지 확인 등
- `generateNewAccessToken(userId)`:
    - 유효한 리프레시 토큰이면, 해당 사용자의 **새로운 액세스 토큰(Access Token)** 을 생성해서 반환
- 리프레시 토큰이 유효하면 → 액세스 토큰을 새로 발급해줌
- 액세스 토큰은 짧은 유효기간 때문에 만료되기 쉬우므로,
- 리프레시 토큰으로 액세스 토큰을 갱신하는 방식

## 참고

---

https://docs.spring.io/spring-security/reference/index.html

https://datatracker.ietf.org/doc/html/rfc6749