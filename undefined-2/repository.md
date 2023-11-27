# Repository

리포지토리 인터페이스 생성

Spring Data JPA가 제공하는 인터페이스 이다.

엔티티를 데이터베이스의 테이블과 구조를 생성하는 데 사용했다면 리포지토리는 엔티티가 생성한 데이터베이스에 접근하는 데 사용한다.



```java
public interface ProductRepository extends JpaRepository<Product, Long> {

}
```

<figure><img src="../.gitbook/assets/스크린샷 2023-11-27 오후 7.28.08.png" alt="" width="375"><figcaption></figcaption></figure>

ProductRepository 가 JpaRepository 를 상속받을 때는 대상 엔티티와 기본값 타입을 지정 해야한다.

다양한 JPA 쿼리에 대한 내용은 검색하면 많이 나온다.



## ✅ DAO 클래스 생성

DAO 클래스는 일반적으로 인터페이스-구현체 구성으로 생성한다.

DAO 클래스는 의존성 결합을 낮추기 위한 디자인 패턴이며, 서비스 레이어에 DAO 객체를 주입받을 때 인터페이스를 선언하는 방식으로 구성 할 수 있다.



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
    public Product insertProduct(Product product) {
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
    public void deleteProduct(Long number) throws Exception { }
}

```



DAO 객체에서도 데이터베이스에 접근하기 위해 리포지토리 인터페이스를 사용해 의존성 주입을 받아야 합니다.

6\~11번 줄과 같이 리포지토리를 정의하고 생성자를 통해 의존성 주입을 받으면 됩니다.



이제 인터페이스에 정의한 메서드를 구현해야 합니다.

먼저 13 \~ 15 번 줄에 정의돼 있는 insertProduct() 메서드를 구현하겠습니다

```java
@Override
public Product insertProduct(Product product) {
    Product savedProduct = productRepository.save(product);
    return savedProduct;
}
```

```java
@Override
public Product selectProduct(Long number) {
    Product selectedProduct = productRepository.getById(number);
    return selectedProduct;
}
```



```java
@Override
public Product updateProductName(Long number, String name) throws Exception {
    Optional<Product> selectedProduct = productRepository.findById(number);
    
    Product updatedProduct;
    if (selectdProduct.isPresent()) {
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

JPA 에서 데이터의 값을 변경할 때는 다른 메서드와는 다른 점이 있습니다.

JPA는 값을 갱신할 때 update 라는 키워드를 사용하지 않습니다.



여기서는 영속성 컨텍스트 활용해 값을 갱신해주는데, find() 메서드를 통해 데이터베이스에서 값을 가져오면 가져온 객체가 영속성 컨텍스트에 추가됩니다.

영속성 컨텍스트가 유지되는 상황에서 객체의 값을 변경하고 다시 save() 를 실행하면 JPA에서는 더티 체크라고 하는 변경 감지를 수행합니다.



SimpleJpaRepository에 구현돼 있는 save() 메서드를 살펴보면

```java
@Transactional
@Override
public <S extends T> S save(S entity) {

    Assert.notNull(entity, "Entity must not be null");
    
    if (entityInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
        return em.merge(entity);
    }
}
```

