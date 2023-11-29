# DAO 설계

## ✅ DAO 클래스 생성

DAO 클래스는 일반적으로 인터페이스-구현체 구성으로 생성합니다.

DAO 클래스는 의존성 결합을 낮추기 위한 디자인 패턴이다.

<figure><img src="../.gitbook/assets/스크린샷 2023-11-29 오전 12.32.32.png" alt="" width="343"><figcaption></figcaption></figure>

```java
package com.springboot.jpa.data.dao;

import com.springboot.jpa.data.entity.Product;

public interface ProductDAO {

    Product insertProduct(Product product);
    
    Product selectProduct(Long number);
    
    Product updateProductName(Long number, String name) throws Exception;
    
    void deleteProduct(Long number) throws Exception;
}
```



일반적으로 데이터베이스에 접근하는 메서드는 리턴 값으로 데이터 객체를 전달합니다.

```java
package com.springboot.jpa.data.dao.impl;

@Component
public class ProductDAOImpl implements ProductDAO {

    private final ProductRepository productRepository;
    
    @Authwired
    public ProductDAOImpl(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }
    
    @Override
    public Product insertProduct(Product Product) {
        return null;
    }
    
    @Override
    public Product selectProduct(Long number) {
        return null;
    }
    
    @Override
    public Product updateProductName(Long number, String name) throws Exception {
        return null;
    }
    
    @Override
    public void deleteProduct(Long number) throw Exception { }
}
```

위의 코드에서도 마찬가지로 의존성 주입을 받아야한다.

생성자를 통해 의존성 주입을 받으면 된다.

```java
    @Override
    public Product insertProduct(Product product) {
        Product savedProduct = productRepository.save(product);

        return savedProduct;
    }
    
    @Override
    public Product selectProduct(Long number) {
        Product selectedProduct = productRepository.getById(number);
        return selectedProduct;
    }

```



`getById()` 내부적으로 EntityManager 의 getReference() 메서드를 호출합니다.

getReference() 메서드를 호출하면 프락시 객체를 리턴합니다. 실제 쿼리는 프락시 객체를 통해 최초로 데이터에 접근하는 시점에 실행됩니다.

이때 데이터가 존재하지 않는 경우에는 EntityNotFoundException 이 발생합니다.

JpaRepository 의 실제 구현체인 SimpleJpaRepository 의 getById() 메서드는&#x20;

```java
@Override
public T getById(ID id) {
    Assert.notNull(id, ID_MUST_NOT_BE_NULL);
    return em.getReference(getDomainClass(), id);
}
```

그리고 내부적으로 findById() 는&#x20;

```java
@Override
public Optional<T> findById(ID id) {

    Assert.notNull(id, ID_MUST_NOT_BE_NULL);
    
    Class<T> domainType = getDomainClass();
    
    if (metadata == null) {
        return Optional.ofNullable(em.find(domainType,id));
    }
    
    LockModeType type = metadata.getLockModeType();
    
    Map<String, Object> hints = new HashMap<>();
    getQueryHints().withFetchGraphs(em).forEach(hints::put);
    
    return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
}
```

조회 기능을 구현하기 위해서는 어떤 메서드를 사용하더라도 무관하다.

```java
    @Override
    public Product updateProductName(Long number, String name) throws Exception {
        Optional<Product> selectedProduct = productRepository.findById(number);

        Product updatedProduct;
        if (selectedProduct.isPresent()) {
            Product product = selectedProduct.get();

            product.setName(name);
            product.setUpdatedAt(LocalDateTime.now());

            updatedProduct = productRepository.save(product);
        } else {
            throw new Exception();
        }

        return updatedProduct;
    }

```

JPA는 값을 갱신할 때 update 라는 키워드를 사용하지 않습니다.

여기서는 영속성 컨텍스트를 활용해 값을 갱신하는데, find() 메서드를 통해 데이터베이스에서 값을 가져오면 가져온 객체가 영속성 컨택스트에 추가된다.

영속성 컨텍스트가 유지되는 상황에서 객체의 값을 변경하고 다시 save() 를 실행하면 JPA에서는 더티 체크라고 하는 변경 감지를 수행한다. SimpleJpaRepository 에 구현돼 있는 save() 메서드를 살펴보게 되면&#x20;

```java
@Transactional
@Override
public <S extends T> S save(S entity) {
    Assert.notNull(entity, "Entity must not be null.");
    
    if ( entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

@Transactional 어노테이션이 선언돼 있다.

이 어노테이션이 지정돼 있으면 메서드 내 작업을 마칠 경우 자동으로 flush() 메서드를 실행한다.



```java
    @Override
    public void deleteProduct(Long number) throws Exception {
        Optional<Product> selectedProduct = productRepository.findById(number);

        if (selectedProduct.isPresent()) {
            Product product = selectedProduct.get();

            productRepository.delete(product);
        } else {
            throw new Exception();
        }
    }

```

삭제하고자 하는 레코드와 매핑된 영속 객체를 영속성 컨텍스트에 가져와야 한다.

deleteProduct() 메서드는 findById() 메서드를 통해 객체를 가져오는 작업을 수행하고, delete() 메서드를 통해 해당 객체를 삭제하게끔 삭제 요청을 한다



```java
@Override
@Transactional
@SuppressWarnings("unchecked")
public void delete(T entity) {

    Assert.notNull(entity, "Entity must not be null!");
    
    if (entityInformation.isNew(entity)) {
        return;
    }
    
    class<?> type = ProxyUtils.getUserClass(entity);
    T existing = (T) em.find(type, entityInformation.getId(entity));
    
    // if the entity to be deleted doesn't exist, delete is a NOOP
    if (existing == null) {
        return;
    }
    
    em.remove(em.contains(entity) ? entity: em.merge(entity));
    
```



```java
package com.springboot.jpa.data.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

// 예제 6.19
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ProductDto {

    private String name;
    private int price;
    private int stock;

}

```

```java
package com.springboot.test.data.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.ToString;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class ProductResponseDto {

    private Long number;
    private String name;
    private int price;
    private int stock;
}

```



```java
package com.springboot.valid_exception.service;


import com.springboot.valid_exception.data.dto.ProductDto;
import com.springboot.valid_exception.data.dto.ProductResponseDto;

public interface ProductService {

    ProductResponseDto getProduct(Long number);
    ProductResponseDto saveProduct(ProductDto productDto);
    ProductResponseDto changeProductName(Long number, String name) throws Exception;
    void deleteProduct(Long number) throws Exception;
}
```



<figure><img src="../.gitbook/assets/스크린샷 2023-11-29 오후 10.35.17.png" alt=""><figcaption></figcaption></figure>

```java
package com.springboot.jpa.service.impl;

import com.springboot.jpa.data.dao.ProductDAO;
import com.springboot.jpa.data.dto.ProductDto;
import com.springboot.jpa.data.dto.ProductResponseDto;
import com.springboot.jpa.data.entity.Product;
import com.springboot.jpa.service.ProductService;
import java.time.LocalDateTime;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

// 예제 6.22
@Service
public class ProductServiceImpl implements ProductService {

    private final ProductDAO productDAO;

    @Autowired
    public ProductServiceImpl(ProductDAO productDAO) {
        this.productDAO = productDAO;
    }

    // 예제 6.23
    @Override
    public ProductResponseDto getProduct(Long number) {
        Product product = productDAO.selectProduct(number);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(product.getNumber());
        productResponseDto.setName(product.getName());
        productResponseDto.setPrice(product.getPrice());
        productResponseDto.setStock(product.getStock());

        return productResponseDto;
    }

    // 예제 6.24
    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {
        Product product = new Product();
        product.setName(productDto.getName());
        product.setPrice(productDto.getPrice());
        product.setStock(productDto.getStock());
        product.setCreatedAt(LocalDateTime.now());
        product.setUpdatedAt(LocalDateTime.now());

        Product savedProduct = productDAO.insertProduct(product);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(savedProduct.getNumber());
        productResponseDto.setName(savedProduct.getName());
        productResponseDto.setPrice(savedProduct.getPrice());
        productResponseDto.setStock(savedProduct.getStock());

        return productResponseDto;
    }

    // 예제 6.25
    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {
        Product changedProduct = productDAO.updateProductName(number, name);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(changedProduct.getNumber());
        productResponseDto.setName(changedProduct.getName());
        productResponseDto.setPrice(changedProduct.getPrice());
        productResponseDto.setStock(changedProduct.getStock());

        return productResponseDto;
    }

    // 예제 6.26
    @Override
    public void deleteProduct(Long number) throws Exception {
        productDAO.deleteProduct(number);
    }
}

```

인터페이스 구현체 클래스에서는 DAO 인터페이스를 선언하고 @Autowired 를 지정한 생성자를 통해 의존성을 주입받습니다. 그리고 인터페이스에서 정의한 메서드를 오버라이딩 합니다.

조회 메서드에 해당하는 getProduct() 메서드를 구현합니다.

현재 서비스 레이어에는 DTO 객체와 엔티티 객체가 공존하도록 설계돼 있어 변호나 작업이 필요합니다.

DTO 객체를 생성하고 값을 넣어 초기화하는 작업을 수행하는데, 이런 부분은 빌더 패턴을 활용하거나 엔티티 객체나 DTO 객체 내부에 변환하는 메서드를 추가해서 간단하게 전환 할 수 있다.



```java
package com.springboot.jpa.service.impl;

import com.springboot.jpa.data.dao.ProductDAO;
import com.springboot.jpa.data.dto.ProductDto;
import com.springboot.jpa.data.dto.ProductResponseDto;
import com.springboot.jpa.data.entity.Product;
import com.springboot.jpa.service.ProductService;
import java.time.LocalDateTime;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

// 예제 6.22
@Service
public class ProductServiceImpl implements ProductService {

    private final ProductDAO productDAO;

    @Autowired
    public ProductServiceImpl(ProductDAO productDAO) {
        this.productDAO = productDAO;
    }

    // 예제 6.23
    @Override
    public ProductResponseDto getProduct(Long number) {
        Product product = productDAO.selectProduct(number);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(product.getNumber());
        productResponseDto.setName(product.getName());
        productResponseDto.setPrice(product.getPrice());
        productResponseDto.setStock(product.getStock());

        return productResponseDto;
    }

    // 예제 6.24
    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {
        Product product = new Product();
        product.setName(productDto.getName());
        product.setPrice(productDto.getPrice());
        product.setStock(productDto.getStock());
        product.setCreatedAt(LocalDateTime.now());
        product.setUpdatedAt(LocalDateTime.now());

        Product savedProduct = productDAO.insertProduct(product);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(savedProduct.getNumber());
        productResponseDto.setName(savedProduct.getName());
        productResponseDto.setPrice(savedProduct.getPrice());
        productResponseDto.setStock(savedProduct.getStock());

        return productResponseDto;
    }

    // 예제 6.25
    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {
        Product changedProduct = productDAO.updateProductName(number, name);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(changedProduct.getNumber());
        productResponseDto.setName(changedProduct.getName());
        productResponseDto.setPrice(changedProduct.getPrice());
        productResponseDto.setStock(changedProduct.getStock());

        return productResponseDto;
    }

    // 예제 6.26
    @Override
    public void deleteProduct(Long number) throws Exception {
        productDAO.deleteProduct(number);
    }
}

```

위는 DAO 를 통해서 product 를 CRUD 하는 코드이다.

<figure><img src="../.gitbook/assets/스크린샷 2023-11-29 오후 10.58.52.png" alt="" width="375"><figcaption></figcaption></figure>

컨트롤러는 클라이언트로부터 요청을 받고 해당 요청에 대해 서비스 레이어에 구현된 적절한 메서드를 호출해서 결과값을 받습니다.

이처럼 컨트롤러는 요청과 응답을 전달하는 역할만 맡는 것이 좋습니다.

```java
package com.springboot.valid_exception.controller;

import com.springboot.valid_exception.data.dto.ChangeProductNameDto;
import com.springboot.valid_exception.data.dto.ProductDto;
import com.springboot.valid_exception.data.dto.ProductResponseDto;
import com.springboot.valid_exception.service.ProductService;
import javax.validation.Valid;
import javax.validation.constraints.Min;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Validated
@RestController
@RequestMapping("/product")
public class ProductController {

    private final Logger LOGGER = LoggerFactory.getLogger(ProductController.class);

    private final ProductService productService;

    @Autowired
    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping()
    public ResponseEntity<ProductResponseDto> getProduct(@Min(value = 5) Long number) {

        ProductResponseDto productResponseDto = productService.getProduct(number);

        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @PostMapping()
    public ResponseEntity<ProductResponseDto> createProduct(@RequestBody ProductDto productDto) {

        ProductResponseDto productResponseDto = productService.saveProduct(productDto);

        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @PutMapping()
    public ResponseEntity<ProductResponseDto> changeProductName(
        @Valid @RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {

        ProductResponseDto productResponseDto = productService.changeProductName(
            changeProductNameDto.getNumber(),
            changeProductNameDto.getName());

        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @DeleteMapping()
    public ResponseEntity<String> deleteProduct(Long number) throws Exception {
        productService.deleteProduct(number);

        return ResponseEntity.status(HttpStatus.OK).body("정상적으로 삭제되었습니다.");
    }
}

```

`@Validated` 메서드는 무엇일까 ?

위의 어노테이션을 알기전에 먼저 `@Valid` 를 알아야한다.

@Valid와 @Validated의 가장 큰 차이점은 검증 항목을 그룹으로 나눠서 검증할 수 있는지, 즉 Group Validation이 가능한 지 여부다. 자바에서 객체를 검증할 때 사용하도록 구현된 것이 javax.validation의 @Valid라면 여기서 Group Validation까지 가능하도록 구현된 것이 스프링 프레임워크의 @Validated라고 할 수 있다.

<figure><img src="../.gitbook/assets/스크린샷 2023-11-29 오후 11.12.14.png" alt="" width="375"><figcaption></figcaption></figure>

