# 회원가입과 로그인 구현

User 객체를 통해 인증하는 방법을 구현했는데, 이번 절에서는 User 객체를 생성하기 위해 회원가입을 구현하고 User 객체로 인증을 시도하는 로그인을 구현하겠습니다.



회원가입과 로그인의 도메인은 Sign으로 통합해서 표현할 예정이며, 각각 Sign-up, Sign-in 으로 구분해서 기능을 구현합니다.

먼저 서비스 레이어를 구현하겠습니다. SignService 인터페이스에 정의된 메서드는 예제 13.24 와 같습니다.



```java
package com.springboot.security.service.impl;

import com.springboot.security.common.CommonResponse;
import com.springboot.security.config.security.JwtTokenProvider;
import com.springboot.security.data.dto.SignInResultDto;
import com.springboot.security.data.dto.SignUpResultDto;
import com.springboot.security.data.entity.User;
import com.springboot.security.data.repository.UserRepository;
import com.springboot.security.service.SignService;
import java.util.Collections;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;


// 예제 13.25
@Service
public class SignServiceImpl implements SignService {

    private final Logger LOGGER = LoggerFactory.getLogger(SignServiceImpl.class);

    public UserRepository userRepository;
    public JwtTokenProvider jwtTokenProvider;
    public PasswordEncoder passwordEncoder;

    @Autowired
    public SignServiceImpl(UserRepository userRepository, JwtTokenProvider jwtTokenProvider,
        PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.jwtTokenProvider = jwtTokenProvider;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public SignUpResultDto signUp(String id, String password, String name, String role) {
        LOGGER.info("[getSignUpResult] 회원 가입 정보 전달");
        User user;
        if (role.equalsIgnoreCase("admin")) {
            user = User.builder()
                .uid(id)
                .name(name)
                .password(passwordEncoder.encode(password))
                .roles(Collections.singletonList("ROLE_ADMIN"))
                .build();
        } else {
            user = User.builder()
                .uid(id)
                .name(name)
                .password(passwordEncoder.encode(password))
                .roles(Collections.singletonList("ROLE_USER"))
                .build();
        }

        User savedUser = userRepository.save(user);
        SignUpResultDto signUpResultDto = new SignInResultDto();

        LOGGER.info("[getSignUpResult] userEntity 값이 들어왔는지 확인 후 결과값 주입");
        if (!savedUser.getName().isEmpty()) {
            LOGGER.info("[getSignUpResult] 정상 처리 완료");
            setSuccessResult(signUpResultDto);
        } else {
            LOGGER.info("[getSignUpResult] 실패 처리 완료");
            setFailResult(signUpResultDto);
        }
        return signUpResultDto;
    }

    @Override
    public SignInResultDto signIn(String id, String password) throws RuntimeException {
        LOGGER.info("[getSignInResult] signDataHandler 로 회원 정보 요청");
        User user = userRepository.getByUid(id);
        LOGGER.info("[getSignInResult] Id : {}", id);

        LOGGER.info("[getSignInResult] 패스워드 비교 수행");
        if (!passwordEncoder.matches(password, user.getPassword())) {
            throw new RuntimeException();
        }
        LOGGER.info("[getSignInResult] 패스워드 일치");

        LOGGER.info("[getSignInResult] SignInResultDto 객체 생성");
        SignInResultDto signInResultDto = SignInResultDto.builder()
            .token(jwtTokenProvider.createToken(String.valueOf(user.getUid()),
                user.getRoles()))
            .build();

        LOGGER.info("[getSignInResult] SignInResultDto 객체에 값 주입");
        setSuccessResult(signInResultDto);

        return signInResultDto;
    }

    // 결과 모델에 api 요청 성공 데이터를 세팅해주는 메소드
    private void setSuccessResult(SignUpResultDto result) {
        result.setSuccess(true);
        result.setCode(CommonResponse.SUCCESS.getCode());
        result.setMsg(CommonResponse.SUCCESS.getMsg());
    }

    // 결과 모델에 api 요청 실패 데이터를 세팅해주는 메소드
    private void setFailResult(SignUpResultDto result) {
        result.setSuccess(false);
        result.setCode(CommonResponse.FAIL.getCode());
        result.setMsg(CommonResponse.FAIL.getMsg());
    }
}
```

위의 코드에서 @Autowired 으로 의존성 주입을 했습니다.

Spring 컨테이너가 자동으로 지정된 타입의 Bean 을 찾아 해당 필드, 생성자 또는 메서드에 주입하는 역할을 합니다.



이 코드에서 @Autowired 는 SignServiceImpl 클래스의 생성자에 사용되고 있으며, 이는 생성자 기반의 의존성 주입을 나타냅니다. @Autowired 가 생성자에 붙어있을 때,&#x20;

Spring 은 UserRepository, JwtTokenProvider, PasswordEncoder 타입에 맞는 빈을 컨테이너에서 찾아 SignServiceImpl 의 생성자 인자로 자동으로 주입합니다.



다시 한번 @Autowired 의 역할에 대해서 알아보면&#x20;



signUp() 메서드는 그에 맞게 전달받은 role 객체를 확인해 User 엔티티의 roels 변수에 추가해서 엔티티를 생성합니다. 패스워드는 암호화해서 저장해야 하기 때문에 PasswordEncoder 를 활용해 인코딩을 수행합니다.&#x20;

PasswordEncoder 는 @Configuration 클래스를 생성하고 @Bean 객체로 등록하도록 구현했습니다.



빈 객체를 등로갛기 위해서 생성된 클래스이기 때문에 SecurityConfiguraation 클래스 같은 이미 생성된 @Configuration 클래스 내부에 passwordEncoder() 메서드를 정의해도 충분합니다.



이렇게 생성된 엔티티를 UserRepository를 통해 저장합니다.

```java
package com.springboot.security.common;

// 예제 13.27
public enum CommonResponse {

    SUCCESS(0, "Success"), FAIL(-1, "Fail");

    int code;
    String msg;

    CommonResponse(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }

}

```



## SignUpResultDto 클래스 와 SignInResultDto 클래스

```java
package com.springboot.security.data.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

// 예제 13.29
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class SignUpResultDto {

    private boolean success;

    private int code;

    private String msg;

}
```



```java
package com.springboot.security.data.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

// 예제 13.30
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class SignInResultDto extends SignUpResultDto {

    private String token;

    @Builder
    public SignInResultDto(boolean success, int code, String msg, String token) {
        super(success, code, msg);
        this.token = token;
    }
}
```

