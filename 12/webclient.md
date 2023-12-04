# WebClient 란 ?

Spring WebFlux는 HTTP 요청을 수행하는 클라이언트로 WebClient 를 제공합니다.

WebClient 는 리액터 기반으로 동작하는 API 입니다.

리액터 기반이므로 스레드와 동시성 문제를 벗어나 비동기 형식으로 사용할 수 있습니다.

* 논블로킹 I/O를 지원합니다.
* 리액티브 스트림의 백 프레셔를 지원합니다.
* 적은 하드웨어 리소스로 동시성을 지원합니다.
* 함수형 API를 지원합니다.
* 동기, 비동기 상호작용을 지원합니다.
* 스트리밍을 지원합니다.

이 책에서는 WebClient 를 사용할수 있는 환경을 구성하고 사용하는 방법에 대해서만 다룰 생각이다.

사용하려면 WebFlux 모듈에 대한 의존성 추가를 해야합니다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

이후 서버 프로젝트의 GET 메서드 컨트롤러에 접근할 수 있는 WebClient 를 생성해봅니다.

## WebClient 기본

Spring Boot에서 `WebClient`를 사용하는 방법에는 여러 단계가 포함됩니다. `WebClient`는 비동기적인 HTTP 요청을 보내고 응답을 처리할 수 있는 Spring WebFlux 모듈의 일부입니다. 다음은 기본적인 사용 방법입니다:

#### 1. `WebClient` 인스턴스 생성

먼저, `WebClient` 인스턴스를 생성해야 합니다. 이는 빈(bean)으로 등록하거나 직접 인스턴스화할 수 있습니다. 빈으로 등록하는 방법이 좀 더 권장됩니다.

```java
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebClientConfig {

    @Bean
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}
```

그리고, 필요한 곳에서 `WebClient.Builder`를 주입받아 사용할 수 있습니다.

```java
@Service
public class MyService {

    private final WebClient webClient;

    public MyService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder
                            .baseUrl("http://example.com")
                            .build();
    }
}
```

#### 2. HTTP 요청 보내기

`WebClient`를 사용하여 다양한 HTTP 요청을 보낼 수 있습니다. 예를 들어, GET 요청은 다음과 같이 보낼 수 있습니다:

```java
public Mono<String> performGetRequest() {
    return this.webClient.get()
                         .uri("/resource")
                         .retrieve()
                         .bodyToMono(String.class);
}
```

#### 3. 응답 처리

`WebClient`는 `Mono` 또는 `Flux`를 반환하여 비동기적으로 작업을 처리합니다. `Mono`는 0 또는 1개의 결과를, `Flux`는 0개 이상의 결과를 나타냅니다.

예를 들어, `Mono<String>`를 반환하는 메소드를 사용하여 응답을 처리할 수 있습니다:

```java
public void useGetRequest() {
    performGetRequest()
        .subscribe(response -> System.out.println("Response: " + response));
}
```

#### 4. 오류 처리

오류 처리는 `WebClient`의 중요한 부분입니다. `onErrorMap`이나 `doOnError`와 같은 메소드를 사용하여 오류를 처리할 수 있습니다.

```java
public Mono<String> performGetRequestWithErrorHandling() {
    return this.webClient.get()
                         .uri("/resource")
                         .retrieve()
                         .onStatus(HttpStatus::is4xxClientError, response -> Mono.error(new RuntimeException("4xx error")))
                         .bodyToMono(String.class)
                         .doOnError(e -> System.out.println("Error: " + e.getMessage()));
}
```

#### 5. 응답 본문의 커스텀 객체로 변환

종종 응답을 사용자 정의 객체로 변환해야 할 필요가 있습니다. 이는 `bodyToMono` 또는 `bodyToFlux` 메소드를 사용하여 할 수 있습니다.

```java
public Mono<MyCustomObject> performGetRequestForCustomObject() {
    return this.webClient.get()
                         .uri("/custom-object")
                         .retrieve()
                         .bodyToMono(MyCustomObject.class);
}
```

`WebClient`는 매우 유연하고 강력한 도구이며, 다양한 요구 사항에 맞게 사용자 정의할 수 있습니다. 위의 예제들은 기본적인 사용 방법을 보여주며, 실제 사용 시에는 프로젝트의 요구 사항에 맞게 조정이 필요합니다.



책의 내용으로는 많이 부족하다. 따로 WebFlux 에 대해 공부해야한다.



