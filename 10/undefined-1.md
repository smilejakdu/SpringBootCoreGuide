# 예외 처리

자바서는 try/catch , throw 구문을 활용해 처리합니다.

스프링 부트에서는 더욱 편리하게 예외 처리를 할 수있는 기능을 제공합니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-04 오후 3.28.49.png" alt=""><figcaption></figcaption></figure>

모든 예외 클래스는 Throwable 클래스를 상속받습니다.

Exception 클래스는 다양한 자식 클래스를 가지고있습니다.

이 클래스는 크게 Checked Exception 과 Unchecked Exception으로 구분할 수 있습니다.

Checked Exception 은 컴파일 단계에서 확인 가능한 예외 상황이다.

이러한 예외는 IDE에서 캐치해서 반드시 예외 처리를 할 수 있게 표시해줍니다.

반면 Unchecked Exception 은 런타임 단계에서 확인되는 예외 상황을 나타냅니다.

즉, 문법상 문제는 없지만 프로그램이 동작하는 도중 예기치 않은 상황이 생겨 발생하는 예외를 의미합니다.

간단히 분류하자면 RuntimeException 을 상속받는 Exception 클래스는 Unchecked Exception 이고 그렇지 않은 Exception 클래스는 Checked Exception 입니다.

## 예외 처리 방법

예외가 발생하면 예외를 복구해서 정상으로 처리하기보다는 요청을 보낸 클라이언트에 어떤 문제가 발생했는지 상황을 전달하는 경우가 많습니다.

* @(Rest)ControllerAdvice와 @ExceptionHandler 를 통해 모든 컨트롤러의 예외를 처리
* @ExceptionHandler 를 통해 특정 컨트롤러의 예외를 처리

먼저 @RestControllerAdvice 를 활용한 핸들러 클래스를 생성합니다.



```java
package com.springboot.valid_exception.common.exception;

import java.util.HashMap;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

// 예제 10.11
@RestControllerAdvice //(basePackages = "com.springboot.valid_exception")
public class CustomExceptionHandler {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomExceptionHandler.class);

    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleException(RuntimeException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        LOGGER.error("Advice 내 exceptionHandler 호출, {}, {}", request.getRequestURI(),
            e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }

    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleException(MethodArgumentNotValidException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }

    // 예제 10.19
    @ExceptionHandler(value = CustomException.class)
    public ResponseEntity<Map<String, String>> handleException(CustomException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        LOGGER.error("Advice 내 handleException 호출, {}, {}", request.getRequestURI(),
            e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", e.getHttpStatusType());
        map.put("code", Integer.toString(e.getHttpStatusCode()));
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, e.getHttpStatus());
    }
}


```

@Controller 나 @RestController 에서 발생하는 예외를 한 곳에서 관리하고 처리할 수 있게 하는 기능을 수행합니다.

즉, 다음과 같이 별도 설정을 통해 예외를 관제하는 범위를 지정할 수 있습니다.

@RestControllerAdvice(basePackages = "com.springboot.valid\_excetpion")

@ExceptionHandler 는 @Controller 나 @RestController가 적용된 빈에서 발생하는 예외를 잡아 처리하는 메서드를 정의할 때 사용합니다.

어떤 예외 클래스를 처리할지는 value 속성으로 등록합니다.

value 속성은 배열의 형식으로도 전달받을 수 있어 여러 예외 클래스를 등록할 수도 있습니다.

위 예제에서는 RuntimeException이 발생하면 처리하도록 코드를 작성했으므로 RuntimeException 에 포함되는 각종 예외가 발생할 경우를 포착해서 처리하게 됩니다.

ExceptionController 를 생성합니다.

```java
@RestController
@RequestMapping("/exception")
public class ExceptionController {

    private final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);

    @GetMapping
    public void getRuntimeException() {
        throw new RuntimeException("getRuntimeException 메소드 호출");
    }
}
```

getRuntimeException() 메서드는 컨트롤러로 요청이 들어오면 RuntimeException 을 발생시킵니다.

이처럼 컨트롤러에서 던진 예외는 @ControllerAdvice 또는 @RestControllerAdvice가 선언돼 있는 핸들러 클래스에서 매핑된 예외 타입을 찾아 처리하게 됩니다.

두 어노테이션은 별도 범위 설정이 없으면 전역범위에서 예외를 처리하기 때문에 특정 컨트롤러에서만 동작하는 @ExceptionHandler 메서드를 생성해서 처리할 수도 있습니다.

```java
package com.springboot.valid_exception.controller;

import com.springboot.valid_exception.common.Constants.ExceptionClass;
import com.springboot.valid_exception.common.exception.CustomException;
import java.util.HashMap;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

// 예제 10.12
@RestController
@RequestMapping("/exception")
public class ExceptionController {

    private final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);

    @GetMapping
    public void getRuntimeException() {
        throw new RuntimeException("getRuntimeException 메소드 호출");
    }

    // 예제 10.12
    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleException(RuntimeException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        responseHeaders.setContentType(MediaType.APPLICATION_JSON);
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        LOGGER.error("클래스 내 handleException 호출, {}, {}", request.getRequestURI(),
            e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }
}

```

컨트롤러 클래스 내에 @ExceptionHandler 어노테이션을 사용한 메서드를 선언하면 해당클래스에 국한해서 예외 처리를 할 수 있습니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-04 오후 4.49.24.png" alt=""><figcaption></figcaption></figure>

## 커스텀 예외

커스텀 예외를 사용하면 예외 상황에 대한 처리도 용이합니다.

앞에서 @ControllerAdvice 와 @ExceptionHandler 에 대해 알아봤는데,

이러한 어노테이션을 사용해 애플리케이션에서 발생하는 예외 상황들을 한곳에서 처리할 수 있었습니다.

예를 들어 RuntimeException에 대해 ControllerAdvice 의 내부에서 표준 예외 처리를 하는 로직을 작성한 경우 개발자가 의도한 RuntimeException 부분이 아닌 의도하지 않은 부분에서 발생하는 에러들이 존재할 수 있다.

표준 예외를 사용하면 이처럼 의도하지 않은 예외 상황도 정해진 예외 처리 코드에서 처리하기 때문에 어디에서 문제가 발생했는지 확인하기가 어렵습니다.

이때 커스텀 예외로 관리하면 의도하지 않았던 부분에서 발생한 예외는 개발자가 관리하는 예외 처리 코드가 처리하지 않으므로 개발 과정에서 혼동할 여지가 줄어듭니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-04 오후 5.01.51.png" alt=""><figcaption></figcaption></figure>

예외 클래스의 상속 구조를 보면 Exception 클래스는 Throwable 클래스를 상속받습니다.

그중 필수적으로 사용되는 message 변수를이용해 Exception 클래스의 커스텀 예외를 만들도록 합니다.

* 에러 타입: HttpStatus 의 responsePhrase
* 에러코드: HttpStatus 의 value
* 메시지 : 상황별 상세 메시지

위와 같은 구성으로 커스텀 예외 클래스를 생성하겠습니다.

추가로 애플리케이션에서 가지고 있는 도메인 레벨을 메시지에 표현하기 위해 ExceptionClass 열거형 타입을 생성합니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-04 오후 5.14.41.png" alt=""><figcaption></figcaption></figure>

이후 열거형을 생성하도록 한다.

```java
package com.springboot.valid_exception.common;

// 예제 10.17
public class Constants {

    public enum ExceptionClass {

        PRODUCT("Product");

        private String exceptionClass;

        ExceptionClass(String exceptionClass) {
            this.exceptionClass = exceptionClass;
        }

        public String getExceptionClass() {
            return exceptionClass;
        }

        @Override
        public String toString() {
            return getExceptionClass() + " Exception. ";
        }

    }

}

```

열거형을 생성했으면 커스텀 예외 클래스를 생성하도록 한다.

```java
package com.springboot.valid_exception.common.exception;

import com.springboot.valid_exception.common.Constants;
import org.springframework.http.HttpStatus;

// 예제 10.18
public class CustomException extends Exception{

    private static final long serialVersionUID = 4300333310379239987L;

    private Constants.ExceptionClass exceptionClass;
    private HttpStatus httpStatus;

    public CustomException(Constants.ExceptionClass exceptionClass, HttpStatus httpStatus,
        String message) {
        super(exceptionClass.toString() + message);
        this.exceptionClass = exceptionClass;
        this.httpStatus = httpStatus;
    }

    public Constants.ExceptionClass getExceptionClass() {
        return exceptionClass;
    }

    public int getHttpStatusCode() {
        return httpStatus.value();
    }

    public String getHttpStatusType() {
        return httpStatus.getReasonPhrase();
    }

    public HttpStatus getHttpStatus() {
        return httpStatus;
    }

}

```

커스텀 예외 클래스는 앞에서 만든 ExceptionClass 와 HttpStatus 를 필드로 가집니다.

두 객체를 기반으로 예외 내용을 정의하며, 클래스를 초기화 합니다.

```java
package com.springboot.valid_exception.common.exception;

import java.util.HashMap;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

// 예제 10.11
@RestControllerAdvice //(basePackages = "com.springboot.valid_exception")
public class CustomExceptionHandler {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomExceptionHandler.class);

    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleException(RuntimeException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();

        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        LOGGER.error("Advice 내 exceptionHandler 호출, {}, {}", request.getRequestURI(),
            e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }

    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleException(MethodArgumentNotValidException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;

        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }

    // 예제 10.19
    @ExceptionHandler(value = CustomException.class)
    public ResponseEntity<Map<String, String>> handleException(CustomException e,
        HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        LOGGER.error("Advice 내 handleException 호출, {}, {}", request.getRequestURI(),
            e.getMessage());

        Map<String, String> map = new HashMap<>();
        map.put("error type", e.getHttpStatusType());
        map.put("code", Integer.toString(e.getHttpStatusCode()));
        map.put("message", e.getMessage());

        return new ResponseEntity<>(map, responseHeaders, e.getHttpStatus());
    }
}

```

기존에 작성했던 핸들러 메서드와 달리 예외 발생 시점에 HttpStatus 를 정의해서 전달하기 때문에 클라이언트 요청에 따라 유동적인 응답 코드를 설정할 수 있다는 장점이 있습니다.

```java
    // 예제 10.19
    @GetMapping("/custom")
    public void getCustomException() throws CustomException {
        throw new CustomException(ExceptionClass.PRODUCT, HttpStatus.BAD_REQUEST, "getCustomException 메소드 호출");
    }

```

CustomException 을 throw 키워드로 던지면 커스텀 예외가 발생합니다.

ExceptionClass 에서 도메인을 비롯해 HttpStatus를 통해 어떤 응답 코드를 사용할지와 세부 메시지를 전달 합니다.&#x20;
