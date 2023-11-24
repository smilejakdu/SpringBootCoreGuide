# 스프링 프레임워크 vs 스프링 부트



스프링 프레임워크에서 모듈들을 추가하다보면 설정이 복잡해지는데&#x20;

이러한 문제를 해결하기 위해서 등장한 것이 스프링 부트이다.

```markdown
Spring Boot makes it easy to create stand-alone.
production-grade Spring based Applications that you can "just run"
```



## 의존성 관리

스프링 부트에서는 `spring-boot-starter` 라는 의존성을 제공한다.

* spring-boot-starter-web: 스프링 MVC 를 사용하는 RESTful application 을 만들기 위한 의존성
* spring-boot-starter-test: JUnit Jupiter, Mockito 등의 테스트용 라이브러리를 포함
* spring-boot-starter-security: 스프링 시큐리티(인증, 권한, 인가 등) 기능을 제공한다.
* spring-boot-starter-data-jpa: 하이버네이트를 활용한 JPA 기능을 제공
* spring-boot-starter-cache: 스프링 프레임워크의 캐시 기능을 지원

