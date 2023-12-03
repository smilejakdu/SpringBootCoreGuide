# 일대다 다대일 매핑

##

## ✅ 다대일 일대다 매핑

<figure><img src="../.gitbook/assets/스크린샷 2023-12-02 오전 10.59.59.png" alt="" width="563"><figcaption></figcaption></figure>

```java
package com.springboot.relationship.data.entity;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

// 예제 9.9
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

}
```

공급업체는 Provider 라는 도메인을 사용해서 정의했습니다. 더 추가할 수 있지만 우선 id 와 name 만 추가했습니다.

EqualsAndHashCode 어노테이션은 `equals()` 와 `hashCode()` 메서드를 클래스에 대해 생성해준다.

Java의 Spring Boot 프레임워크에서 `@EqualsAndHashCode`는 Lombok 라이브러리의 어노테이션입니다. 이 어노테이션은 자동으로 `equals()`와 `hashCode()` 메서드를 클래스에 대해 생성해줍니다. 이 두 메서드는 객체의 동등성을 비교하고 객체의 해시코드를 생성하는 데 사용됩니다.

`@EqualsAndHashCode` 어노테이션을 사용할 때 주의할 점:

1. **필드 선택**: 모든 필드가 `equals()`와 `hashCode()` 계산에 포함됩니다. 특정 필드를 제외하려면 `exclude` 속성을 사용할 수 있습니다.
2. **상속 문제**: 이 어노테이션을 사용할 때 상속 구조에서 문제가 발생할 수 있습니다. 부모 클래스와 자식 클래스 모두에서 `@EqualsAndHashCode`를 사용하면 예기치 않은 결과를 초래할 수 있습니다.
3. **성능 고려**: `equals()`와 `hashCode()` 메서드는 컬렉션에서 객체를 비교하고 해시 기반 구조에서 사용되므로 성능에 중요한 영푑을 미칠 수 있습니다.
4. **사이드 이펙트 방지**: 객체의 동등성을 결정하는 데 사용되는 필드가 변경될 수 있는 경우, 이를 고려하여 설계해야 합니다.

`@EqualsAndHashCode` 어노테이션은 코드를 간결하게 유지하면서 중복을 줄이는 데 도움이 되지만, 그 사용법과 영향을 잘 이해하고 사용해야 합니다.



```java
package com.springboot.relationship.data.entity;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToMany;
import javax.persistence.ManyToOne;
import javax.persistence.OneToMany;
import javax.persistence.OneToOne;
import javax.persistence.Table;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @OneToOne(mappedBy = "product") // 예제 9.6, 9.7
    @ToString.Exclude // 예제 9.8
    private ProductDetail productDetail;

    // 예제 9.10
    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;

    // 예제 9.23
    @ManyToMany
    @ToString.Exclude
    private List<Producer> producers = new ArrayList<>();

    public void addProducer(Producer producer){
        this.producers.add(producer);
    }

}

```

외래키를 갖는 쪽이 주인의 역할을 수행하기 때문에 이 경우 상품 엔티티가 공급업체 엔티티의 주인이 됩니다.

이후 공급업체 repository도 생성합니다.

```java
public interface ProviderRepository extends JpaRepository<Provider, Long> {
}
```

상품 엔티티가 주인이기 때문에 ProductRepository 를 활용해서 테스트를 진행합니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.Provider;
import org.assertj.core.util.Lists;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.transaction.Transactional;
import java.util.List;

@SpringBootTest
class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    // 예제 9.12
    @Test
    void relationshipTest1() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("ㅇㅇ물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        // 테스트
        System.out.println(
                "product : " + productRepository.findById(1L).orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L).orElseThrow(RuntimeException::new).getProvider());
    }

}
```

연관관계를 테스트하기 위해 테스트 데이터를 생성합니다.

그렇기 때문에 두 repository 에 대해 의존성 주입을 받습니다.

```java
package com.springboot.relationship.data.entity;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

// 예제 9.9
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // 예제 9.14, 9.25, 9.27
    @OneToMany(mappedBy = "provider", cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();

}
```



OneToMany 가 붙은 쪽에서 JoinColumn 어노테이션을 사용하면 상대 엔티티에 외래키가 설정된다.

fetch = FetchType.EAGER 로 설정한 것은 OneToMany 의 기본 fetch 전략이 Lazy 이기 때문에 즉시 로딩으로 조정한 것입니다.



```java
package com.springboot.relationship.data.entity;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

// 예제 9.9
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // 예제 9.14, 9.25, 9.27
    @OneToMany(mappedBy = "provider", cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();

}
```

entity 를 작성하고 테스트 코드를 작성한다.

```java
    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest() {
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("ㅇㅇ상사");

        providerRepository.save(provider);

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(2000);
        product1.setStock(100);
        product1.setProvider(provider);

        Product product2 = new Product();
        product2.setName("가방");
        product2.setPrice(20000);
        product2.setStock(200);
        product2.setProvider(provider);

        Product product3 = new Product();
        product3.setName("노트");
        product3.setPrice(3000);
        product3.setStock(1000);
        product3.setProvider(provider);

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        System.out.println("check 1");

        List<Product> products = providerRepository.findById(provider.getId()).get()
                .getProductList();

        for (Product product : products) {
            System.out.println(product);
        }

    }

```



```java
package com.springboot.relationship.data.entity;

import java.util.ArrayList;
import java.util.List;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

// 예제 9.17
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString
@EqualsAndHashCode
@Table(name = "category")
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true)
    private String code;

    private String name;

    @OneToMany(fetch = FetchType.EAGER)
    @JoinColumn(name = "category_id")
    private List<Product> products = new ArrayList<>();

}

```

## oneToMany FetchType

Java의 Spring Boot에서 `@OneToMany` 관계를 사용할 때 `FetchType`은 데이터를 로드하는 방식을 지정합니다. `FetchType`에는 주로 두 가지 옵션이 있습니다: `EAGER`와 `LAZY`.

1. **`FetchType.EAGER`**:
   * `EAGER` 타입은 관계가 있는 모든 데이터를 즉시 로드합니다. 즉, 부모 엔티티를 로드할 때 연관된 자식 엔티티도 함께 로드됩니다.
   * 이 방식은 연관된 데이터가 항상 필요할 때 유용하지만, 많은 양의 데이터를 불필요하게 로드하여 성능 저하를 일으킬 수 있습니다.
   * 예를 들어, 한 사용자에 대한 모든 주문 정보가 항상 필요한 경우 `FetchType.EAGER`를 사용할 수 있습니다.
2. **`FetchType.LAZY`**:
   * `LAZY` 타입은 관계가 있는 데이터를 필요할 때만 로드합니다. 즉, 부모 엔티티는 필요할 때까지 자식 엔티티를 로드하지 않습니다.
   * 이 방식은 성능 최적화에 유리하지만, 실제 데이터가 필요할 때 지연이 발생할 수 있습니다.
   * 예를 들어, 사용자 정보를 조회할 때마다 모든 주문 정보가 필요하지 않은 경우 `FetchType.LAZY`를 사용하는 것이 좋습니다.

**중요한 고려사항**:

* `EAGER` 로딩은 데이터베이스와의 불필요한 라운드트립이 발생할 수 있으므로 성능에 부정적인 영향을 미칠 수 있습니다. 특히 대규모 데이터셋과 복잡한 조인이 있는 경우에는 더욱 그렇습니다.
* `LAZY` 로딩은 초기 쿼리 성능은 개선하지만, 필요한 시점에 데이터를 가져오기 위해 추가 쿼리가 발생할 수 있습니다. 이는 `N+1` 쿼리 문제를 일으킬 수 있습니다.

`@OneToMany` 관계에서 적절한 `FetchType`을 선택하는 것은 애플리케이션의 성능과 요구사항에 크게 의존하므로, 사용하는 비즈니스 케이스의 특성을 이해하고 최적의 전략을 선택해야 합니다.



