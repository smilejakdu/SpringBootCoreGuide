# 회원가입과 로그인 구현

```java
package com.springboot.security.service;


import com.springboot.security.data.dto.SignInResultDto;
import com.springboot.security.data.dto.SignUpResultDto;

// 예제 13.24
public interface SignService {

    SignUpResultDto signUp(String id, String password, String name, String role);

    SignInResultDto signIn(String id, String password) throws RuntimeException;

}
```

SignService 인터페이스를 구현한 SignServiceImpl 클래스의 전체를 작성한다.

현재 애플리케이션에서는 ADMIN 과 USER로 권한을 구분하고 있습니다.

signUp() 메서드는 그에 맞게 전달받은 role 객체를 확인해 User 엔티티의 roles 변수에 추가해서 엔티티를 생성



패스워드는 암호화해서 저장해야 하기 때문에 PasswordEncoder를 활용해 인코딩을 수행합니다.

PasswordEncoder 는 별도의 @Configuration 클래스를 생성하고 @Bean 객체로 등록하도록 구현했습니다.

```java
@Configuration
public class PasswordEncoderConfiguration {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegationPasswordEncoder();
    }
}
        
```

나머지 회원가입과 로그인에 관한 코드 구현은 책을 참고하는것이 좋을것 같다.

