# API를 작성하는 다양한 방법

## ✅ GET API 만들기

간단히 만들수가 있다.

```java
@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {

}
```

위와 같이 설정하게 되면 api/v1/get-api 경로로 들어오는 모든 요청들이 들어오게된다.

```java

package com.springboot.api.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod; 
import org.springframework.web.bind.annotation. RestController;

@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {

    // http://localhost:8080/api/v1/get-api/hello
    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String getHello() {
        return "Hello World";
    }
}
```

위처럼 `RequestMethod.GET` 처럼 특정한 메서드를 사용해서 endpoint 역할을 할 수가 있다.

하지만 더이상 위의 방법은 사용하지 않는다.

스프링 4.3 버전 이후로는 새로운 방식을 사용하고 있다.



`GetMapping`

PostMapping

PutMapping

DeleteMapping



## ✅ PathVariable을 활용한 GET 메서드 구현

매개변수를 받을 때 쓰이는 방법 중에 URL 자체에 값을 담아 요청하는 것이다.

```java

// http://localhost:8080/api/v1/get-api/variable1/{String} 
@GetMapping(value = "/variable1/{variable}")
public String getVariable1(@PathVariable String variable) { 
    return variable;
}
```

만약 쿼리 스트링에 어떤 값이 들어올지 모른다면&#x20;

```java

// http ://localhost:8080/api/v1/get-api/request2?key1=value&key2=value2
@GetMapping(value = "/request2")
public String getRequestParam2(@RequestParam Map<String, String> param) {
    StringBuilder sb = new StringBuilder();
    param.entrySet().forEach(map -> {
        sb.append(map.getKey() + " : " + map.getValue() + "\n");
    });
    
    return sb.toString();
}
```

예를들어 회원 가입 관련 API 에서 사용자는 회원 가입을 하면서 ID 같은 필수항목이 아닌 취미 같은 선택항목에 대해서는 값을 기입하지 않는 경우가 있다. 이럴경우 Map 객체를 돌리면 좋다.

하지만 위의 코드는 추천하지 않는다.

DTO 를 사용해서 데이터 교환하는것을 추천한다.

## ✅ DTO 객체를 활용한 GET 메서드 구현

DTO 에는 별도의 로직이 포함되지 않습니다.

DTO 를 사용할때 주의점은 DTO 는 로직이 포함되지 않는다.

말 그대로 데이터 교환하기 위해 활용된다.



DTO 와 VO의 역할을 서로 엄밀하게 구분하지 않고 사용할때가 많다

VO 는 데이터 그 자체로 의미가 있는 객체를 의미한다.

VO 의 가장 특징적인 부분은 읽기전용으로 설계한다는 점이다.

즉 , VO는 값을 변경할 수 없게 만들어 데이터의 신뢰성을 유지한다.



DTO는 데이터 전송을 위해 사용되는 데이터 컨테이너로 볼 수 있다.즉 같은 애플리케이션 내부에서 사용되는 것이 아니라 다른 서버로 전달하는 경우에 사용됩니다.



책에서는 GET 뿐만 아니라 POST , PUT, DELETE 에 대해서 설명했다.

모두 GET 과 비슷한 내용이다.

## ✅ Swagger

문서화 하는것은 정말 중요하다.

명세 문서도 주기적으로 업데이트를 해야한다.

또한 명세 작업은 번거롭고 시간 또한 오래걸린다.

하지만 Swagger 를 작성하게 되면 이와같은 작업을 덜어준다.



```java
package com.example.showmeyourability.shared;

import io.swagger.v3.oas.annotations.OpenAPIDefinition;
import io.swagger.v3.oas.annotations.info.Info;
import lombok.RequiredArgsConstructor;
import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@OpenAPIDefinition(
        info = @Info(title = "SHOW ME YOUR ABILITY App",
                description = "SHOW ME YOUR ABILITY api명세",
                version = "v1"))
@Configuration
@RequiredArgsConstructor
public class SwaggerConfig {
    @Bean
    public GroupedOpenApi chatOpenApi() {
        String[] paths = {"/api/**"};

        return GroupedOpenApi.builder()
                .group("SHOW ME YOUR ABILITY API")
                .pathsToMatch(paths)
                .build();
    }
}

```

```java
@Data
public class CreateUserRequestDto {
    @Schema(description = "email", example = "이메일")
    @NotEmpty(message = "이메일을 입력해주세요.")
    private String email;

    @Schema(description = "password", example = "password")
    @NotEmpty(message = "비밀번호를 입력해주세요.")
    private String password;

    @Schema(description = "genderType", example = "FEMALE")
    private GenderType genderType;

    @Schema(description = "age", example = "나이")
    private int age;

    @Schema(description = "phone", example = "phone")
    private String phone;

    @Schema(description = "img", example = "img")
    private String img;
}
```

위처럼 작성하게 되면 `CreateUserRequestDto` 를 가져다가 사용하는 곳에서 Swagger 문서가 생기게 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-11-26 오후 12.04.07.png" alt=""><figcaption></figcaption></figure>

위의 내용말고 Logger 에 대해서도 책에서 다뤘다. 한번 찾아보면 좋을 것 같다.

