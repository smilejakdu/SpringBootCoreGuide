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

