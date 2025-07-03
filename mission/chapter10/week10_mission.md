- 실습 1. 간단한 로그인 및 회원가입 구현 - Session 방식

  # 실습 1. 간단한 로그인 및 회원가입 구현 - Session 방식

  SecurityConfig

    ```java
    @EnableWebSecurity // Spring Security 설정 활성화 시키는 역할
    @Configuration
    public class SecurityConfig {
    
        @Bean // 아래 메서드는 SecurityFilterChain 정의, HttpSecurity 객체를 통해 다양한 보안 설정 구성 가능
        public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
            http
                    .authorizeHttpRequests((requests) -> requests // HTTP 요청에 대한 접근 제어 설정
                            .requestMatchers("/", "/home", "/signup", "/css/**").permitAll() 
                            // 특정 URL 패턴에 대한 접근 권한 설정 / permitAll() : 인증 없이 접근 가능한 경로 지정                 
                            .requestMatchers("/admin/**").hasRole("ADMIN") // hasRole(?) : ? 역할을 가진 사용자만 접근 가능하도록 제한
                            .anyRequest().authenticated() // 그 외 모든 요청에 대한 인증 요구
                    )
                    .formLogin((form) -> form // 폼 기반 로그인에 대한 설정
                            .loginPage("/login") // 커스텀 로그인 페이지를 /login 으로 지정
                            .defaultSuccessUrl("/home", true) // 로그인 성공 시 /home 으로 리다이렉트
                            .permitAll() // 로그인 페이지는 모든 사용자가 접근할 수 있도록 설정
                    )
                    .logout((logout) -> logout // 로그아웃 처리에 대한 설정
                            .logoutUrl("/logout") // /logout 경로로 로그아웃 처리
                            .logoutSuccessUrl("/login?logout") // 로그아웃 성공 시 /login?logout 으로 리다이렉트
                            .permitAll()
                    );
    
            return http.build();
        }
    
        @Bean
        public PasswordEncoder passwordEncoder() { // 비밀번호 암호화하여 저장하기 위해 BCryptPasswordEncoder 사용
            return new BCryptPasswordEncoder();
        }
    }
    
    ```

  비밀번호 인코딩
  `newMember.encodePassword(passwordEncoder.encode(request.getPassword()));`

    - 암호화 사용 이유
        - 보안 강화
            - 평문 비밀번호를 DB에 저장하지 않음 → 보안 강화
        - 단방향 해시
            - BCryptPasswordEncoder 는 단방향 해시 함수 사용 → 원문 비밀번호 복원 불가
        - 솔트(Salt) 사용
            - BCrypt 는 자동으로 솔트를 생성하여 레인보우 테이블 공격 방지


    ![image](https://github.com/user-attachments/assets/ae50a547-ba27-487f-8a71-f77110851585)

    
    → 회원가입용 데이터 입력
    
    ![image](https://github.com/user-attachments/assets/654245dc-f022-42eb-a9e3-836f691435d2)

    
    ![image](https://github.com/user-attachments/assets/eb57ce7d-0465-4425-8e6a-b327ddacec66)

    
    → 로그인 성공
    
    ![image](https://github.com/user-attachments/assets/156dcdce-5ee5-4bf8-8f56-52002a941243)

    
    → 권한이 없어 /admin 접근 불가
    
    ![image](https://github.com/user-attachments/assets/7ab89835-a168-432a-8d17-3492a8b4b137)

    
    → Role = admin 업데이트
    
    ![image](https://github.com/user-attachments/assets/975c47c5-5a9f-4615-bf4b-a3cd3b8b3640)

    
    → 접근 가능

- 실습 2. 간단한 로그인 및 회원가입 구현 - JWT Token 방식
    - 액세스 토큰 방식으로만 인증하는 것 구현

  기본적인 세팅

    ```java
    package umc.study.config.properties;
    
    import lombok.Getter;
    import lombok.Setter;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.stereotype.Component;
    
    @Component
    @Getter
    @Setter
    @ConfigurationProperties("jwt.token")
    public class JwtProperties {
        private String secretKey="";
        private Expiration expiration;
        
        @Getter
        @Setter
        public static class Expiration{
            private Long access;
            // TODO: refreshToken
        }
    }
    ```

    ```java
    package umc.study.config.properties;
    
    public final class Constants {
        public static final String AUTH_HEADER = "Authorization";
        public static final String TOKEN_PREFIX = "Bearer ";
    }
    ```

  Jwt 관련 설정 파일 ( JwtTokenProvider )

    ```java
    package umc.study.config.security.jwt;
    
    import io.jsonwebtoken.*;
    import io.jsonwebtoken.security.Keys;
    import jakarta.servlet.http.HttpServletRequest;
    import lombok.RequiredArgsConstructor;
    import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
    import org.springframework.security.core.Authentication;
    import org.springframework.security.core.userdetails.User;
    import org.springframework.stereotype.Component;
    import org.springframework.util.StringUtils;
    import umc.study.apiPayload.code.status.ErrorStatus;
    import umc.study.apiPayload.exception.handler.MemberHandler;
    import umc.study.config.properties.Constants;
    import umc.study.config.properties.JwtProperties;
    
    import java.security.Key;
    import java.util.Date;
    import java.util.Collections;
    
    @Component
    @RequiredArgsConstructor
    public class JwtTokenProvider {
    
        private final JwtProperties jwtProperties;
    
        private Key getSigningKey() {
            return Keys.hmacShaKeyFor(jwtProperties.getSecretKey().getBytes());
        }
    
        public String generateToken(Authentication authentication) {
            String email = authentication.getName();
    
            return Jwts.builder()
                    .setSubject(email)
                    .claim("role", authentication.getAuthorities().iterator().next().getAuthority())
                    .setIssuedAt(new Date())
                    .setExpiration(new Date(System.currentTimeMillis() + jwtProperties.getExpiration().getAccess()))
                    .signWith(getSigningKey(), SignatureAlgorithm.HS256)
                    .compact();
        }
    
        public boolean validateToken(String token) {
            try {
                Jwts.parserBuilder()
                        .setSigningKey(getSigningKey())
                        .build()
                        .parseClaimsJws(token);
                return true;
            } catch (JwtException | IllegalArgumentException e) {
                return false;
            }
        }
    
        public Authentication getAuthentication(String token) {
            Claims claims = Jwts.parserBuilder()
                    .setSigningKey(getSigningKey())
                    .build()
                    .parseClaimsJws(token)
                    .getBody();
    
            String email = claims.getSubject();
            String role = claims.get("role", String.class);
    
            User principal = new User(email, "", Collections.singleton(() -> role));
            return new UsernamePasswordAuthenticationToken(principal, token, principal.getAuthorities());
        }
    
        public static String resolveToken(HttpServletRequest request) {
            String bearerToken = request.getHeader(Constants.AUTH_HEADER);
            if(StringUtils.hasText(bearerToken) && bearerToken.startsWith(Constants.TOKEN_PREFIX)) {
                return bearerToken.substring(Constants.TOKEN_PREFIX.length());
            }
            return null;
        }
    
        public Authentication extractAuthentication(HttpServletRequest request){
            String accessToken = resolveToken(request);
            if(accessToken == null || !validateToken(accessToken)) {
                throw new MemberHandler(ErrorStatus.INVALID_TOKEN);
            }
            return getAuthentication(accessToken);
        }
    }
    ```

  JWT 토큰 생성, 검증, 인증 객체 반환하는 역할 수행하는 클래스

  크게 4가지 메서드

    1. generateToken() → 인증 정보 받아서 JWT Access Token 생성, 반환
    2. validateToken() → 해당 JWT 토큰 유효한지 검증
    3. getAuthentication() → JWT 토큰에서 인증 정보 추출, Spring Security의 Authentication 객체로 변환
    4. extractAuthentication() → HttpServletRequest 에서 토큰 값 추출, getAuthentication 메서드를 이용해 Authentication 객체로 변환

  Jwt 관련 설정 파일 ( JwtAuthenticationFilter )

    ```java
    package umc.study.config.security.jwt;
    
    import jakarta.servlet.FilterChain;
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    import lombok.RequiredArgsConstructor;
    import org.springframework.security.core.Authentication;
    import org.springframework.security.core.context.SecurityContextHolder;
    import org.springframework.util.StringUtils;
    import org.springframework.web.filter.OncePerRequestFilter;
    import umc.study.config.properties.Constants;
    
    import java.io.IOException;
    
    @RequiredArgsConstructor
    public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
        private final JwtTokenProvider jwtTokenProvider;
    
        @Override
        protected void doFilterInternal(HttpServletRequest request,
                                        HttpServletResponse response,
                                        FilterChain filterChain)
            throws ServletException, IOException {
    
            String token = resolveToken(request);
    
            if(StringUtils.hasText(token) && jwtTokenProvider.validateToken(token)) {
                Authentication authentication = jwtTokenProvider.getAuthentication(token);
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
            filterChain.doFilter(request, response);
        }
    
        private String resolveToken(HttpServletRequest request) {
            String bearerToken = request.getHeader(Constants.AUTH_HEADER);
            if(StringUtils.hasText(bearerToken) && bearerToken.startsWith(Constants.TOKEN_PREFIX)) {
                return bearerToken.substring(Constants.TOKEN_PREFIX.length());
            }
            return null;
        }
    }
    ```

  토큰 기반 인증은 클라이언트 요청의 Authorization Header 에 실어서 보낸다.

  → 클라이언트가 서버에게 전달되기 전에 인증 과정을 거쳐야 함

  위의 Filter 클래스가 해당 역할을 하는 클래스

    1. resloveToken() → 순수 토큰 반환해주는 메서드 (앞에 Bearer 부분 떼주고 그런 역할)
    2. doFilterInternal() → HttpServletRequest 로 받아온 요청 객체에서 순수한 토큰을 추출 후 jwtTokenProvider의 validateToken() 에서 검증 과정 통과하면 getAuthentication() 메서드를 호출해서 Authentication 객체 만들고 SecurityContextHolder 가 인증 등록
        1. 이렇게 등록된 인증 정보는 이후 컨트롤러에서 사용할 것
    3. 마지막으로 filterChain.doFilter() 를 호출해서 다음 필터, 컨트롤러 등에 요청을 넘겨줌.

  ![image](https://github.com/user-attachments/assets/f5659bdf-51d9-4654-b915-4e679bcc271b)


  ![image](https://github.com/user-attachments/assets/5817c4b2-21c9-43fa-9571-63351fe634cc)
