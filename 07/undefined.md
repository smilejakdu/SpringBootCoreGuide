# 단위테스트 작성하기

단위테스트는 외부와의 연결을 끊어서 테스트 하게 된다.

```java
package com.springboot.test.data.repository;

import static org.junit.jupiter.api.Assertions.assertEquals;

import com.springboot.test.data.entity.Product;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

@DataJpaTest
public class ProductRepositoryTestByH2 {

    @Autowired
    private ProductRepository productRepository;

    // 예제 7.15
    @Test
    void saveTest() {
        // given
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(1000);

        // when
        Product savedProduct = productRepository.save(product);

        // then
        assertEquals(product.getName(), savedProduct.getName());
        assertEquals(product.getPrice(), savedProduct.getPrice());
        assertEquals(product.getStock(), savedProduct.getStock());
    }

    // 예제 7.16
    @Test
    void selectTest() {
        // given
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(1000);

        Product savedProduct = productRepository.saveAndFlush(product);

        // when
        Product foundProduct = productRepository.findById(savedProduct.getNumber()).get();

        // then
        assertEquals(product.getName(), foundProduct.getName());
        assertEquals(product.getPrice(), foundProduct.getPrice());
        assertEquals(product.getStock(), foundProduct.getStock());
    }
}


```

위처럼 작성하게 되면 편하게 테스트 코드를 작성할 수 있다.

하지만 위의 테스트 코드는 취지에 맞지않다.

우리는 단위 테스트를 하려고 한다.&#x20;

단위 테스트에는 Autowired 모다 모의 객체를 불러와서 테스트하는 것이 취지에 맞다.

## ✅ 단위테스트 MockBean 과 Autowired 차이

단위 테스트에서 `@Autowired`와 `@MockBean`의 사용은 테스트의 목적과 컨텍스트에 따라 달라집니다. 각각의 접근 방식은 다른 시나리오와 요구 사항에 적합합니다.

#### `@Autowired` 사용:

1. **실제 동작 테스트:** `@Autowired`는 실제 스프링 빈을 테스트 컨텍스트에 주입합니다. 이는 통합 테스트에서 더 자주 사용되며, 실제 애플리케이션의 동작을 테스트하고자 할 때 유용합니다.
2. **환경 설정:** 실제 빈을 사용하기 때문에, 데이터베이스 연결이나 외부 시스템과의 통합과 같은 실제 환경 설정이 필요할 수 있습니다.
3. **속도:** 실제 빈을 로드하고 의존성을 주입하는 데 시간이 걸릴 수 있으므로, `@Autowired`를 사용하는 테스트는 `@MockBean`을 사용하는 테스트보다 느릴 수 있습니다.

#### `@MockBean` 사용:

1. **단위 테스트에 적합:** `@MockBean`은 모의 객체를 생성하여 주입합니다. 이는 단위 테스트에 더 적합하며, 테스트하고자 하는 클래스나 메서드가 다른 컴포넌트와의 상호작용 없이 독립적으로 동작하는지 검증할 때 유용합니다.
2. **속도:** 모의 객체는 실제 객체보다 가볍고, 실제 의존성을 로드하거나 외부 시스템과 통신할 필요가 없기 때문에 테스트 실행 속도가 빠릅니다.
3. **제어와 단순성:** `@MockBean`을 사용하면 외부 시스템이나 서비스의 동작을 모의할 수 있으며, 테스트 중에 발생할 수 있는 외부 요인을 제어할 수 있습니다. 이는 테스트를 더 예측 가능하고 관리하기 쉽게 만듭니다.

#### 결론:

* **단위 테스트:** `@MockBean`을 선호합니다. 이는 테스트의 범위를 좁게 유지하고, 테스트 대상 클래스나 메서드의 동작에만 집중할 수 있도록 해줍니다.
* **통합 테스트:** `@Autowired`를 사용하여 실제 빈과의 상호작용을 테스트합니다. 이는 전체 시스템이나 애플리케이션의 여러 부분이 함께 어떻게 동작하는지 확인하는 데 유용합니다.

테스트의 목적과 필요에 따라 적절한 접근 방식을 선택하는 것이 중요합니다.

여기 책에서는 단위 테스트시 Autowired 어노테이션을 사용해서 단위테스트를 진행했다.

Autowired 를 사용해서 단위테스트를 진행할 수 있다.&#x20;

반드시 정해진것은 없고 상황과 기업들 마다 선호도의 차이가 있을것 같다.



## Mock 과 MockBean 그리고 InjectMocks 차이점

`@Mock`, `@MockBean`, 그리고 `@InjectMocks`는 모두 테스트 환경에서 모의 객체(mock objects)를 생성하고 관리하는 데 사용되지만, 사용되는 컨텍스트와 방식이 다릅니다. 각각의 어노테이션은 다음과 같은 목적으로 사용됩니다:

#### 1. `@Mock` (Mockito 라이브러리)

* `@Mock`은 Mockito 테스팅 프레임워크에서 제공하는 어노테이션입니다.
* 이 어노테이션은 주로 단위 테스트(unit tests)에서 사용되며, Mockito가 관리하는 모의 객체를 생성합니다.
* `@Mock`으로 생성된 모의 객체는 주로 `@InjectMocks` 어노테이션이 적용된 클래스에 주입되어 의존성을 모의로 대체합니다.
* 이러한 모의 객체는 Spring 컨텍스트 밖에서 관리되며, Spring의 빈(bean) 생명주기나 컨텍스트와는 독립적입니다.

#### 2. `@MockBean` (Spring Boot 테스트)

* `@MockBean`은 Spring Boot의 테스트 모듈에서 제공하는 어노테이션입니다.
* 이 어노테이션은 Spring 애플리케이션 컨텍스트 내에서 관리되는 모의 객체를 생성합니다.
* `@MockBean`으로 생성된 모의 객체는 Spring 컨텍스트 내의 모든 빈(bean)에 자동으로 주입됩니다.
* 이는 주로 통합 테스트(integration tests)에서 사용되며, 실제 빈을 모의 객체로 대체하여 테스트 환경에서의 빈의 동작을 제어할 수 있습니다.

#### 3. `@InjectMocks` (Mockito 라이브러리)

* `@InjectMocks`는 Mockito에서 제공하는 어노테이션으로, `@Mock` 또는 `@Spy`로 생성된 모의 객체를 테스트 대상 클래스에 자동으로 주입합니다.
* 이 어노테이션은 주로 단위 테스트에서 사용되며, 테스트 대상 클래스의 의존성을 모의 객체로 채워 넣는 데 사용됩니다.
* interface 일 경우 사용할 수 없다. 즉 Repository 에서는 사용할 수 없다.
* `@InjectMocks`를 사용하면, 테스트 대상 클래스의 생성자, 세터 메소드, 또는 필드 주입을 통해 의존성을 주입할 수 있습니다.

#### 요약

* `@Mock`과 `@InjectMocks`는 Mockito 프레임워크를 사용한 단위 테스트에 주로 사용됩니다.
* `@MockBean`은 Spring Boot의 테스트 환경에서 Spring 컨텍스트 내의 빈을 모의 객체로 대체할 때 사용됩니다.

```java
package com.example.showmeyourability.users.application;

import com.example.showmeyourability.teacher.infrastructure.dto.FindTeacherDto.FindByIdTeacherRequestDto;
import com.example.showmeyourability.users.domain.User;
import com.example.showmeyourability.users.infrastructure.dto.FindUserDto.FindUserByEmailResponseDto;
import com.example.showmeyourability.users.infrastructure.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.hibernate.annotations.NotFound;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.webjars.NotFoundException;

@Service
@RequiredArgsConstructor
public class FindUserByEmailApplication {
    private final UserRepository userRepository;

    @Transactional
    public FindUserByEmailResponseDto execute(String email) {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() -> new NotFoundException("해당 유저가 존재하지 않습니다."));

        return FindUserByEmailResponseDto
                .builder()
                .id(user.getId())
                .email(user.getEmail())
                .build();
    }
}

```

위와 같은 코드가 존재한다.email 로 user 를 찾는 코드이다.&#x20;

```java
package com.example.showmeyourability.service.userApplication;

import com.example.showmeyourability.users.application.FindUserByEmailApplication;
import com.example.showmeyourability.users.domain.GenderType;
import com.example.showmeyourability.users.domain.User;
import com.example.showmeyourability.users.infrastructure.dto.FindUserDto.FindUserByEmailResponseDto;
import com.example.showmeyourability.users.infrastructure.repository.UserRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.security.crypto.bcrypt.BCrypt;
import org.webjars.NotFoundException;

import java.util.Optional;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
public class FindUserApplicationTest {
    @Mock // 스프링 부트 테스트에서 사용하는 목(mock) 객체를 생성하는 어노테이션입니다
    private UserRepository userRepository;

    @InjectMocks
    private FindUserByEmailApplication findUserByEmailApplication;

    private User user1;
    private User user2;

    @BeforeEach // 각 테스트 메소드 실행 전에 호출되는 메소드를 지정
    public void setup() {
        String hashedPassword = BCrypt.hashpw("1234", BCrypt.gensalt());

        // User 객체 생성
        user1 = User.builder()
                .id(1L)
                .email("robertvsd1@gmail.com")
                .genderType(GenderType.MALE)
                .age(20)
                .img("img")
                .password(hashedPassword)
                .build();

        user2 = User.builder()
                .id(2L)
                .email("robertvsd2@gmail.com")
                .genderType(GenderType.MALE)
                .age(20)
                .img("img_two")
                .password(hashedPassword)
                .build();
    }


    @Test
    public void notFoundUserByEmailTest() {
        // given
        String notFoundEmail = "nonexistentemail@example.com";

        // userRepository.findByEmail에 대한 모의 동작 정의
        // 이메일로 사용자를 찾을 수 없을 때 빈 Optional 반환
        when(userRepository.findByEmail(notFoundEmail)).thenReturn(Optional.empty());

        // then
        assertThrows(NotFoundException.class, () -> {
            // when
            findUserByEmailApplication.execute(notFoundEmail);
        });
    }

    @Test
    public void successTestFindUserByEmail() {
        // given
        String email = "robertvsd1@gmail.com";
        when(userRepository.findByEmail(email)).thenReturn(Optional.of(user1));

        FindUserByEmailResponseDto expectedResponse = new FindUserByEmailResponseDto();
        expectedResponse.setId(1L);
        expectedResponse.setEmail(email);

        // when
        FindUserByEmailResponseDto actualResponse = findUserByEmailApplication.execute(email);

        // then
        Assertions.assertEquals(expectedResponse.getId(), actualResponse.getId());
        Assertions.assertEquals(expectedResponse.getEmail(), actualResponse.getEmail());    }
}

```

테스트 코드를 작성할때 실패 케이스부터 먼저 작성하게 된다.&#x20;

그리고 난뒤에 성공 케이스 테스트를 작성한다.

@InjectMocks 는 Mockito 테스트 환경에서 사용되므로, Spring 컨텍스트에 의해 관리되는 빈과는 별개로 작동합니다.&#x20;

따라서 @InjectMocks 를 사용하면 FindUserByEmailApplication 인스턴스가 Spring 컨텍스트에 의해 생성되고 관리 되지 않습니다.

그래서 InjectMocks를 사용한다면 Mock 과 같이사용하게 됩니다.

## Mock 과 InjectMocks

단위 테스트(Unit Test)를 수행할 때는 테스트 대상 클래스의 기능을 독립적으로 검증하는 것이 중요합니다. 이를 위해 테스트 대상 클래스가 의존하는 다른 컴포넌트들을 모의 객체(Mock Objects)로 대체하는 것이 일반적입니다. 이 경우, Mockito와 같은 모킹 프레임워크를 사용하여 의존성을 모의 객체로 대체할 수 있습니다.

#### 단위 테스트에서 사용하는 주요 어노테이션:

1. **`@Mock`:** Mockito 프레임워크를 사용하여 의존성에 대한 모의 객체를 생성합니다. `@Mock` 어노테이션은 테스트 대상 클래스가 의존하는 컴포넌트에 대한 모의 객체를 만들 때 사용됩니다.
2. **`@InjectMocks`:** `@InjectMocks` 어노테이션은 Mockito가 관리하는 모의 객체들을 테스트 대상 클래스에 자동으로 주입합니다. 이는 테스트 대상 클래스가 생성될 때 `@Mock`으로 생성된 모의 객체들을 해당 클래스의 필드에 주입하는 데 사용됩니다.
3. **`@ExtendWith(MockitoExtension.class)`:** JUnit 5에서 Mockito를 사용할 때, 이 어노테이션을 테스트 클래스에 추가하여 Mockito의 초기화와 통합을 관리합니다.

위의 내용으로 단위 테스트 예시를 보면&#x20;

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

@ExtendWith(MockitoExtension.class)
public class MyServiceTest {

    @Mock
    private DependencyRepository dependencyRepository;

    @InjectMocks
    private MyService myService;

    @Test
    public void testMyServiceMethod() {
        // 여기서 dependencyRepository의 메소드를 모의로 정의하고
        // myService의 메소드를 테스트합니다.
    }
}
```

MyService 는 DependencyRepository 에 의존하고 있습니다. @Mock 어노테이션을 사용하여 DependencyRepository의 모의 객체를 생성하고, @InjectMocks 를 사용하여 이 모의 객체를 MyService에 주입합니다. 이렇게 하면 MyService 의 메소드를 독립적으로 테스트 할 수 있습니다.

#### 결론

단위 테스트에서는 `@Mock`과 `@InjectMocks`를 사용하여 의존성을 모의 객체로 대체하고, 테스트 대상 클래스의 기능을 독립적으로 검증하는 것이 좋습니다. 이 방법은 테스트 대상 클래스가 외부 컴포넌트와의 상호작용 없이 올바르게 작동하는지 확인하는 데 도움이 됩니다.



## MockBean 과 Autowired

물론이죠, `@InjectMocks`와 관련된 동작에 대해 더 자세히 설명드리겠습니다.

#### `@InjectMocks`의 작동 방식

`@InjectMocks`는 Mockito 테스팅 프레임워크에서 제공하는 어노테이션으로, 지정된 클래스의 인스턴스를 생성하고, 이 클래스에 필요한 의존성들을 Mockito가 생성한 모의(Mock) 객체로 자동 주입합니다. 이는 주로 단위 테스트(unit tests)에서 사용되며, 테스트 대상 클래스가 의존하는 다른 컴포넌트들을 모의 객체로 대체하여, 테스트 대상 클래스의 기능을 독립적으로 검증할 수 있게 해줍니다.

#### `userRepository`의 모의 객체 생성

예를 들어, `FindUserByEmailApplication` 클래스가 `UserRepository`에 의존하고 있다고 가정해 보겠습니다. `@InjectMocks`를 사용하여 `FindUserByEmailApplication`의 인스턴스를 생성하면, Mockito는 `UserRepository`의 모의 객체를 생성하고, 이를 `FindUserByEmailApplication` 인스턴스에 주입합니다.



#### 문제 발생 가능성

이 접근 방식에서 문제가 발생하는 경우는 주로 통합 테스트(integration tests)에서 발생합니다. 통합 테스트의 목적은 애플리케이션의 다양한 컴포넌트들이 실제 환경과 유사한 조건에서 어떻게 상호작용하는지를 검증하는 것입니다. 이 경우, 실제 `UserRepository`와 같은 Spring 데이터 리포지토리의 실제 동작을 테스트하는 것이 중요합니다.

* **실제 동작과의 차이:** `@InjectMocks`를 사용하면 `UserRepository`의 실제 동작이 아닌, Mockito가 생성한 모의 객체의 동작을 테스트하게 됩니다. 이 모의 객체는 실제 데이터베이스와의 상호작용을 모방하지 않으므로, 데이터베이스와 관련된 실제 동작의 정확성을 검증할 수 없습니다.
* **테스트 정확성 저하:** 모의 객체를 사용하면 데이터베이스 연결, 트랜잭션 관리, JPA/Hibernate와 같은 ORM 도구의 동작 등 실제 애플리케이션에서 발생할 수 있는 다양한 상황들을 완전히 재현할 수 없습니다. 따라서, 모의 객체를 사용하는 테스트는 실제 애플리케이션의 동작을 완벽하게 반영하지 못할 수 있으며, 이는 테스트의 정확성을 저하시킬 수 있습니다.

#### 결론

따라서, 통합 테스트에서는 `@MockBean`과 `@Autowired`를 함께 사용하는 것이 권장됩니다. `@MockBean`을 사용하면 Spring 컨텍스트 내의 특정 빈을 모의 객체로 대체할 수 있으며, `@Autowired`를 사용하면 테스트 대상 클래스에 실제 빈을 주입할 수 있습니다. 이 방법은 Spring 컨텍스트와의 일관성을 유지하면서 필요한 부분에 대해서만 모의 동작을 정의할 수 있게 해줍니다.



## H2 인메모리 테스트&#x20;

인-메모리 데이터베이스 사용 -> H2 와 같은 인-메모리 데이터베이스를 사용하여 테스트를 실행합니다.

인-메모리 데이터베이스는 테스트가 끝나면 데이터가 모두 사라지기 때문에, 실제 데이터베이스를 오염시키지 않습니다.

그러면 Spring Boot 에서 설정하지 않아도 자동으로 H2 와 같은 인-메모리 데이터베이스가 실행이 되는걸까 ??

## ✅ 테스트를 위한 H2 인-메모리 데이터베이스 사용

인-메모리 데이터베이스, 예를 들어 H2 데이터베이스를 사용하기 위해서는 몇 가지 설정이 필요합니다. 단순히 테스트 파일을 실행한다고 해서 자동으로 인-메모리 데이터베이스가 사용되지는 않습니다. 주로 다음과 같은 단계를 거쳐야 합니다:

1.  **의존성 추가:** 먼저, `build.gradle` (Gradle을 사용하는 경우) 또는 `pom.xml` (Maven을 사용하는 경우) 파일에 H2 데이터베이스 관련 의존성을 추가해야 합니다. 예를 들어, Gradle을 사용하는 경우 `build.gradle` 파일에 다음과 같은 의존성을 추가합니다:

    ```gradle
    dependencies {
        testImplementation 'com.h2database:h2'
        // 기타 필요한 의존성들...
    }
    ```
2.  **테스트 환경 설정:** 테스트 환경에서 인-메모리 데이터베이스를 사용하도록 설정해야 합니다. 이는 `src/test/resources` 디렉토리 아래에 위치한 `application.properties` 또는 `application.yml` 파일을 통해 설정할 수 있습니다. 예를 들어, 다음과 같이 설정할 수 있습니다:

    ```properties
    # src/test/resources/application.properties
    spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;MODE=MySQL
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=your_username
    spring.datasource.password=your_password
    spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
    ```

    이 설정은 테스트 실행 시 H2 인-메모리 데이터베이스를 사용하도록 지시합니다.
3. **테스트 실행:** 이제 테스트를 실행하면, 설정된 인-메모리 데이터베이스를 사용하여 테스트가 진행됩니다. 테스트가 종료되면 인-메모리 데이터베이스에 저장된 모든 데이터는 사라집니다.

이러한 설정을 통해, 테스트 실행 시 실제 운영 데이터베이스가 아닌 H2와 같은 인-메모리 데이터베이스를 사용하여 데이터베이스 관련 테스트를 안전하게 수행할 수 있습니다.



만약 로컬 환경에서는 개발데이터베이스가 연결을 하고 싶고 , 테스트 실행시에는 H2 인-메모리 데이터베이스를 실행하고 싶을땐 어떻게 해야할까??



## ✅ 개발 데이터베이스와 H2 인-메모리 데이터베이스 설정 분리

Gradle의 `build.gradle` 파일에 `testImplementation 'com.h2database:h2'`를 추가하는 것은 테스트 환경에서 H2 데이터베이스를 사용할 수 있도록 하는 설정입니다. 그러나 이 설정 자체만으로는 테스트 실행 시 자동으로 H2 데이터베이스가 사용되는 것은 아닙니다. 테스트 환경에서 H2를 사용하도록 명시적으로 설정해야 합니다.

테스트 코드 실행 시 H2 데이터베이스를 사용하고, 일반 애플리케이션 실행(예: 로컬에서 애플리케이션을 실행) 시 MySQL과 같은 다른 데이터베이스를 사용하려면 다음과 같이 설정해야 합니다:

1.  **테스트 환경 설정:** `src/test/resources` 디렉토리에 `application.properties` 또는 `application.yml` 파일을 만들고, 여기에 H2 데이터베이스 관련 설정을 추가합니다.

    ```properties
    # src/test/resources/application.properties
    spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;MODE=MySQL
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=your_username
    spring.datasource.password=your_password
    spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
    ```
2.  **개발/운영 환경 설정:** `src/main/resources` 디렉토리에 있는 `application.properties` 또는 `application.yml` 파일에 MySQL 또는 다른 데이터베이스에 대한 설정을 추가합니다.

    ```properties
    # src/main/resources/application.properties
    spring.datasource.url=jdbc:mysql://localhost:3306/your_database
    spring.datasource.username=your_username
    spring.datasource.password=your_password
    spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
    ```

이렇게 설정하면, 테스트를 실행할 때는 `src/test/resources`의 설정이 적용되어 H2 데이터베이스를 사용하고, 애플리케이션을 일반적으로 실행할 때는 `src/main/resources`의 설정이 적용되어 MySQL 또는 지정한 다른 데이터베이스를 사용하게 됩니다.

즉, `build.gradle`에 의존성을 추가하는 것은 해당 라이브러리를 프로젝트에서 사용할 수 있도록 하는 것이며, 실제 어떤 데이터베이스를 사용할지는 애플리케이션의 설정 파일에서 결정됩니다.

## ✅ TDD

* 실패테스트 작성 : 실패하는 경우의 테스트 코드를 먼저 작성한다.
* 테스트를 통과하는 코드 작성: 테스트 코드를 실행 시키기 위한 실제 코드 작성
* 리팩토링: 중복 코드 제거한다.

TDD 개발하게 되면 여러 장점이 존재한다.

TDD 장점에 대해선 서칭하면 많이 나오지만 작성하면서 본인이 경험해보는것이 가장 좋다.



위의 Product 를 TDD 기준으로 작성하게 된다면 실패하는 코드부터 작성하고 난뒤에 통과하는 코드를 작성해야 한다.

