# SecurityConfiguration 구현



```java
package com.springboot.security.config.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

/**
 * 어플리케이션의 보안 설정
 * 예제 13.19
 *
 * @author Flature
 * @version 1.0.0
 */
@Configuration
//@EnableWebSecurity // Spring Security에 대한 디버깅 모드를 사용하기 위한 어노테이션 (default : false)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    private final JwtTokenProvider jwtTokenProvider;

    @Autowired
    public SecurityConfiguration(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.httpBasic().disable() // REST API는 UI를 사용하지 않으므로 기본설정을 비활성화

            .csrf().disable() // REST API는 csrf 보안이 필요 없으므로 비활성화

            .sessionManagement()
            .sessionCreationPolicy(
                SessionCreationPolicy.STATELESS) // JWT Token 인증방식으로 세션은 필요 없으므로 비활성화

            .and()
            .authorizeRequests() // 리퀘스트에 대한 사용권한 체크
            .antMatchers("/sign-api/sign-in", "/sign-api/sign-up",
                "/sign-api/exception").permitAll() // 가입 및 로그인 주소는 허용
            .antMatchers(HttpMethod.GET, "/product/**").permitAll() // product로 시작하는 Get 요청은 허용

            .antMatchers("**exception**").permitAll()

            .anyRequest().hasRole("ADMIN") // 나머지 요청은 인증된 ADMIN만 접근 가능

            .and()
            .exceptionHandling().accessDeniedHandler(new CustomAccessDeniedHandler())
            .and()
            .exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())

            .and()
            .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),
                UsernamePasswordAuthenticationFilter.class); // JWT Token 필터를 id/password 인증 필터 이전에 추가
    }

    /**
     * Swagger 페이지 접근에 대한 예외 처리
     *
     * @param webSecurity
     */
    @Override
    public void configure(WebSecurity webSecurity) {
        webSecurity.ignoring().antMatchers("/v2/api-docs", "/swagger-resources/**",
            "/swagger-ui.html", "/webjars/**", "/swagger/**", "/sign-api/exception");
    }
}
```

SecurityConfiguration 클래스의 주요 메서드는 두가지로 WebSecurity 파라미터로 받은 configure() 메서드와 HttpSecurity 파라미터를 받은 configure() 메서드입니다.

HttpSecurity의 대표적인 기능은

* 리소스 접근 권한 설정
* 인증 실패시 발생하는 예외 처리
* 인증 로직 커스터마이징
* csrf, cors 등의 스프링 시큐리티 설정



지금부터 configure() 메서드에 작성돼 있는 코드를 설정별로 구분해 설명하겠습니다.

모든 설정은 전달 받은 HttpSecurity 에 설정하게 됩니다.



*   httpBasic().disable()

    UI를 사용하는 것을 기본값으로 가진 시큐리티 설정을 비활성화 합니다.
* csrf().disalbe()\
  CSRF는 공격자가 사용자의 의지와는 무관하게 사용자가 인증된 웹사이트에 악의적인 요청을 보내도록 하는 공격 방법입니다. 예를 들어, 사용자가 로그인한 상태에서 악의적인 링크를 클릭하면, 사용자의 인증 정보를 사용하여 민감한 작업을 수행할 수 있습니다.
* sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)\
  REST API 기반 애플리케이션의 동작 방식을 설정합니다.\
  지금 진행 중인 프로젝트에서는 JWT 토큰으로 인증을 처리하며, 세션은 사용하지 않기 때문에 STATELESS 로 설정합니다.
* authorizeRequest()\
  애플리케이션에 들어오는 요청에 대한 사용 권한을 체크합니다. 이어서 사용한 antMatchers() 메서드는 antPattern 을 통해 권한을 설정하는 역할을 합니다.
* exceptionHandling().accessDeniedHandler()\
  권한을 확인하는 과정에서 통과하지 못하는 예외가 발생할 경우 예외를 전달합니다.
* exceptionHandling().authenticationEntryPoint()\
  인증 과정에서 예외가 발생할 경우 예외를 전달합니다.

스프링 시큐리티는 각각의 역할을 수행하는 필터들이 체인 형태로 구성돼 순서대로 동작합니다.

JWT로 인증하는 필터를 생성했으며, 이 필터의 등록은 HttpSecurity 설정에서 진행합니다.

addFilterBefore() 메서드를 사용해 어느 필터 앞에 추가할 것인지 설정할 수 있는데, 현재 구현돼 잇는 설정은 스프링 시큐리티에서 인증을 처리하는 필터인 UsernamePasswordAuthenticationFilter 앞에 앞에서 생성한 JwtAuthenticationFilter를 추가하겠다는 의미입니다. 추가된 필터에서 인증이 정상적으로 처리되면 UsernamePasswordAuthenticationFilter는 자동으로 통과되기 때문에 위와 같은 구성을 선택했습니다.

WebSecurity는 HttpSecurity 앞단에 적용되며, 전체적으로 스프링 시큐리티의 영향권 밖에 있습니다. 즉, 인증과 인가가 모두 적용되기 전에 동작하는 설정입니다.

그렇기 때문에 다양한 곳에서 사용되지 않고 인증과 인가가 적용되지 않는 리소스 접근에 대해서만 사용합니다.



## 커스텀 AccessDeniedHandler, AuthenticationEntryPoint 구현



CustomAccessDeniedHandler 와 CustomAuthenticationEntryPoint 로 예외를 전달하고 있었습니다.

먼저 AccessDeniedHandler 인터페이스의 구현체 클래스를 생성하겠습니다.

```java
package com.springboot.security.config.security;

import java.io.IOException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;
import org.springframework.stereotype.Component;

/**
 * 권한이 없는 예외가 발생했을 경우 핸들링하는 클래스
 * 예제 13.20
 */
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomAccessDeniedHandler.class);

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
        AccessDeniedException exception) throws IOException {
        LOGGER.info("[handle] 접근이 막혔을 경우 경로 리다이렉트");
        response.sendRedirect("/sign-api/exception");
    }
}
```

AccessDeniedException 은 액세스 권한이 없는 리소스에 접근할 경우 발생하는 예외입니다.

이 예외를 처리하기 위해 AccessDeniedHandler 인터페이스가 사용되며, SecurityConfiguration 에도 exceptionHandling() 메서드를 통해 추가했습니다.

AccessDeniedHandler 의 구현 클래스인 CustomAccessDeniedHandler 클래스는 handle() 메서드를 오버라이딩 합니다.

이 메서드는 HttpServletRequest 와 HttpServletResponse, AccessDeniedException 을 파라미터로 가져옵니다.

이번 예제에서는 response 에서 리다이렉트하는 sendRedirect() 메서드를 활용하는 방식으로 구현했습니다.

```java
package com.springboot.security.config.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.springboot.security.data.dto.EntryPointErrorResponse;
import java.io.IOException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

/**
 * 인증 실패시 결과를 처리해주는 로직을 가지고 있는 클래스
 * 예제 13.21, 예제 13.32
 */
@Component
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomAuthenticationEntryPoint.class);

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
        AuthenticationException ex) throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        LOGGER.info("[commence] 인증 실패로 response.sendError 발생");

        EntryPointErrorResponse entryPointErrorResponse = new EntryPointErrorResponse();
        entryPointErrorResponse.setMsg("인증이 실패하였습니다.");

        response.setStatus(401);
        response.setContentType("application/json");
        response.setCharacterEncoding("utf-8");
        response.getWriter().write(objectMapper.writeValueAsString(entryPointErrorResponse));
        // 예제 13.23
        //response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
    }
}
```

