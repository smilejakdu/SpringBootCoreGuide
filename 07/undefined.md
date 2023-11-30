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

`@DataJpaTest` 는 무슨 어노테이션 일까 ?

{% embed url="https://0soo.tistory.com/40" %}

## ✅ TDD

* 실패테스트 작성 : 실패하는 경우의 테스트 코드를 먼저 작성한다.
* 테스트를 통과하는 코드 작성: 테스트 코드를 실행 시키기 위한 실제 코드 작성
* 리팩토링: 중복 코드 제거한다.

TDD 개발하게 되면 여러 장점이 존재한다.

TDD 장점에 대해선 서칭하면 많이 나오지만 작성하면서 본인이 경험해보는것이 가장 좋다.



위의 Product 를 TDD 기준으로 작성하게 된다면 실패하는 코드부터 작성하고 난뒤에 통과하는 코드를 작성해야 한다.

