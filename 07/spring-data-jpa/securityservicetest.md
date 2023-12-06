# SecurityServiceTest

테스트 코드를 진행할때 `application.properties` 에 있는 값들을 가져와서 사용해야할때가 있다.

application.properties 에 값을 직접 가져오기보다.



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



통합테스트 일경우 Autowired 로 해서 가져올 수 있지만 단일 테스트일 모의 객체만 불러와서 테스트해야한다.

상황에 따라서 회사 규칙에따라서  적절히 사용하면 될것 같다.

