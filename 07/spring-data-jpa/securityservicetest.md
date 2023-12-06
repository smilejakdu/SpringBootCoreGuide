# SecurityServiceTest

테스트 코드를 진행할때 `application.properties` 에 있는 값들을 가져와서 사용해야할때가 있다.

```java
package com.example.showmeyourability.shared.Service;

import com.example.showmeyourability.shared.Exception.HttpExceptionCustom;
import com.example.showmeyourability.users.domain.User;
import com.example.showmeyourability.users.infrastructure.repository.UserRepository;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import jakarta.servlet.http.Cookie;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;

@Service
@RequiredArgsConstructor
public class SecurityService {

    @Value("${jwt.secret.key}")
    private String SECRET_KEY;
    private static final long EXPIRATION_TIME = 1000 * 60 * 60 * 24;

    private final UserRepository userRepository;

    public String createToken(String email) {

        if (EXPIRATION_TIME <= 0) {
            throw new RuntimeException("Expiratio time must be greater than zero!");
        }

        SecretKey key = Keys.hmacShaKeyFor(SECRET_KEY.getBytes(StandardCharsets.UTF_8));

        return Jwts.builder()
                .setSubject(email)
                .setExpiration(new Date(System.currentTimeMillis()+EXPIRATION_TIME))
                .signWith(key)
                .compact();
    }

    public User getSubject(String token) {
        SecretKey key = Keys.hmacShaKeyFor(SECRET_KEY.getBytes(StandardCharsets.UTF_8));

        Claims claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();

        return userRepository.findByEmail(claims.getSubject())
                .orElseThrow(
                        () -> new HttpExceptionCustom(
                                false,
                                "존재하지 않은 사용자입니다.",
                                HttpStatus.NOT_FOUND
                        )
                );
    }

    public String getTokenByCookie(Cookie[] cookies) {
        String token = null;
        for (Cookie cookie : cookies) {
            if (cookie.getName().equals("token")) {
                token = cookie.getValue();
            }
        }
        return token;
    }
}

```

위의 코드에서 `SECRET_KEY` 값은 일반 String 값이 아니라 application.properties 에서 가져온다.



단일 테스트를 진행하는 경우, 테스트의 목적과 테스트하려는 컴포넌트의 복잡성에 따라 다음 두 가지 접근 방식 중 하나를 선택할 수 있습니다:

1.  **Reflection을 사용한 `SECRET_KEY` 설정:**

    이 방법은 `SecurityService` 클래스의 실제 구현을 사용하면서 `SECRET_KEY` 필드만 테스트 환경에 맞게 설정하고자 할 때 유용합니다. 이 방법을 사용하면 `SecurityService`의 `createToken` 메서드와 같은 실제 로직을 그대로 테스트할 수 있으며, 테스트가 실제 환경에서의 동작을 더 잘 반영할 수 있습니다.

    Reflection을 사용하는 방법은 다음과 같습니다:

    ```java
    import java.lang.reflect.Field;

    @BeforeEach
    public void setUp() throws NoSuchFieldException, IllegalAccessException {
        // 기존 설정 코드 ...

        // Reflection을 사용하여 SECRET_KEY 설정
        Field secretKeyField = SecurityService.class.getDeclaredField("SECRET_KEY");
        secretKeyField.setAccessible(true);
        secretKeyField.set(securityService, SECRET_KEY);
    }
    ```
2.  **Mockito를 사용한 `SecurityService` 모의 객체 생성:**

    이 방법은 `SecurityService`의 내부 구현을 테스트하지 않고, `createToken` 메서드가 호출될 때 원하는 동작을 수행하도록 설정하고자 할 때 적합합니다. 만약 `SecurityService`의 다른 기능이나 로직이 테스트의 초점이 아니라면, Mockito를 사용하여 `createToken` 메서드의 동작을 모의로 구현하는 것이 더 간단하고 효율적일 수 있습니다.

    Mockito를 사용하는 방법은 다음과 같습니다:

    ```java
    @BeforeEach
    public void setUp() {
        // 기존 설정 코드 ...

        // SecurityService를 모의 객체로 생성
        securityService = Mockito.mock(SecurityService.class);
        when(securityService.createToken(anyString())).thenReturn("mocked_token");
    }
    ```

**결정 요소:**

* **실제 구현 테스트:** `SecurityService`의 실제 메서드 구현을 테스트하고자 한다면 Reflection을 사용하세요.
* **간결함과 단순성:** 특정 메서드의 동작만을 모의하고 싶다면 Mockito를 사용하세요.

테스트의 목적에 따라 가장 적합한 방법을 선택하시면 됩니다. Reflection은 실제 구현을 더 깊이 테스트하는 데 유용하지만, Mockito는 테스트를 더 단순하고 관리하기 쉽게 만들어 줍니다.



단일 테스트를 진행할 때 어떤 방법을 선택할지는 테스트의 목적과 테스트 대상의 복잡성에 따라 달라집니다. 각 방법의 장단점을 고려하여 결정해야 합니다:

1. **Reflection을 사용하여 `SECRET_KEY` 설정하기:**
   * **장점:**
     * 실제 `SecurityService` 클래스의 로직을 테스트할 수 있습니다.
     * 클래스의 내부 구현에 접근하여 실제 동작을 확인할 수 있어, 통합 테스트에 가까운 환경을 제공합니다.
   * **단점:**
     * Reflection 사용은 코드의 가독성을 떨어뜨리고 유지 관리를 어렵게 만들 수 있습니다.
     * 내부 구현에 의존하기 때문에, 구현이 변경되면 테스트 코드도 수정해야 할 수 있습니다.
2. **Mockito를 사용하여 `SecurityService` 모의 객체 생성하기:**
   * **장점:**
     * 테스트 코드가 더 간결하고 이해하기 쉽습니다.
     * 실제 구현에 의존하지 않기 때문에, 구현의 변경이 테스트에 미치는 영향이 적습니다.
   * **단점:**
     * 실제 로직을 테스트하지 않기 때문에, 실제 동작과의 차이가 있을 수 있습니다.
     * 모의 객체의 동작을 설정하는 과정에서 실수가 발생할 수 있습니다.

**결정 요소:**

* **테스트의 목적:** 실제 `SecurityService`의 로직을 테스트하고자 한다면 Reflection을 사용하세요. `SecurityService`의 로직을 테스트하지 않고 단순히 테스트 대상 클래스(`LoginUserApplication`)의 동작만 확인하고자 한다면 Mockito를 사용하세요.
* **유지 관리와 가독성:** 코드의 유지 관리와 가독성을 중시한다면 Mockito를 사용하는 것이 좋습니다.
* **내부 구현 테스트:** 클래스의 내부 구현을 테스트하고자 한다면 Reflection을 사용하세요.

단일 테스트의 경우, 일반적으로는 Mockito를 사용하는 것이 더 간단하고 효율적입니다. 하지만 실제 `SecurityService`의 로직을 포함한 보다 심층적인 테스트가 필요한 경우에는 Reflection을 사용하는 것이 좋을 수 있습니다.

