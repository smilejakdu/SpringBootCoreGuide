# JwtAuthenticationFilter 구현

Spring Boot에서 JWT를 이용한 `JwtAuthenticationFilter`를 구현하려면, Spring Security의 필터 체인에 JWT 인증 필터를 추가하고, 해당 필터에서 JWT를 검증하여 사용자 인증을 수행해야 합니다. 이를 위해 몇 가지 주요 단계를 따르면 됩니다

#### 1. Spring Security 의존성 추가

`build.gradle` 파일에 Spring Security 의존성을 추가합니다:

```gradle
dependencies {
    // Spring Security
    implementation 'org.springframework.boot:spring-boot-starter-security'
    // JWT 라이브러리 (예: jjwt)
    implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
    runtime 'io.jsonwebtoken:jjwt-impl:0.11.2'
    runtime 'io.jsonwebtoken:jjwt-jackson:0.11.2'
    // 기타 필요한 의존성들...
}
```

#### 2. JWT 검증 서비스 구현

JWT를 검증하기 위한 서비스를 구현합니다. 이 서비스는 토큰의 유효성을 검사하고, 필요한 경우 사용자의 정보를 로드하는 기능을 담당합니다.

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Service;

@Service
public class JwtService {

    private String secretKey = "verySecretKey"; 
    // 실제 환경에서는 보다 안전하게 관리해야 함

    // JWT 검증
    public Claims validateToken(String token) {
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody();
    }
}
```

#### 3. `JwtAuthenticationFilter` 구현

Spring Security의 `OncePerRequestFilter`를 상속받아 `JwtAuthenticationFilter`를 구현합니다. 이 필터에서 JWT를 추출하고 검증합니다.

```java
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private JwtService jwtService;

    public JwtAuthenticationFilter(JwtService jwtService) {
        this.jwtService = jwtService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String token = extractToken(request);
        if (token != null && jwtService.validateToken(token)) {
            // 토큰 검증 로직 및 인증 정보 설정
            // 예: SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        filterChain.doFilter(request, response);
    }

    private String extractToken(HttpServletRequest request) {
        // 헤더에서 JWT 추출 로직
        // 예: request.getHeader("Authorization");
    }
}
```

#### 4. Spring Security 설정에 필터 추가

`WebSecurityConfigurerAdapter`를 상속받는 설정 클래스를 통해`JwtAuthenticationFilter`를 Spring Security의 필터 체인에 추가합니다.

```java
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtService jwtService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // 기본 설정...
            .addFilterBefore(new JwtAuthenticationFilter(jwtService), UsernamePasswordAuthenticationFilter.class);
    }
}
```

이렇게 하면, Spring Boot 애플리케이션에 JWT 기반 인증 메커니즘이 구현됩니다. 각 요청은 `JwtAuthenticationFilter`를 통과하여 JWT의 유효성이 검증되며, 유효한 토큰을 가진 요청만이 서버의 리소스에 접근할 수 있습니다.

