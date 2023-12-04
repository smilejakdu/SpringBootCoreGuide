# 영속성 전이

영속성 전이(Persistence Transitivity)는 JPA(Java Persistence API)에서 사용되는 개념으로, 엔티티의 상태 변화를 관련 엔티티에 전파하는 것을 의미합니다. 즉, 한 엔티티의 생명주기 상태 변경이 연관된 엔티티에도 영향을 미칩니다. 이를 통해 관련 엔티티들을 개별적으로 처리하는 번거로움을 줄일 수 있습니다.

영속성 전이는 `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany` 등의 관계 매핑 어노테이션과 함께 사용됩니다. `cascade` 속성을 통해 다양한 영속성 전이 유형을 지정할 수 있습니다. 주요 영속성 전이 유형은 다음과 같습니다:

1. `CascadeType.PERSIST`: 부모 엔티티를 저장할 때 연관된 자식 엔티티도 함께 저장합니다.
2. `CascadeType.MERGE`: 부모 엔티티 상태를 병합할 때 연관된 자식 엔티티의 상태도 함께 병합합니다.
3. `CascadeType.REMOVE`: 부모 엔티티를 삭제할 때 연관된 자식 엔티티도 함께 삭제합니다.
4. `CascadeType.REFRESH`: 부모 엔티티를 새로고침할 때 연관된 자식 엔티티도 함께 새로고침합니다.
5. `CascadeType.DETACH`: 부모 엔티티를 영속성 컨텍스트에서 분리할 때 연관된 자식 엔티티도 함께 분리합니다.
6. `CascadeType.ALL`: 모든 종류의 영속성 전이를 적용합니다.

예를 들어, `@OneToMany` 관계에서 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장하고 싶다면, 다음과 같이 `cascade` 속성을 설정할 수 있습니다:

```java
@Entity
public class Parent {
    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(cascade = CascadeType.PERSIST)
    private List<Child> children;
}
```

이 경우, `Parent` 엔티티를 저장할 때 `children` 리스트에 포함된 `Child` 엔티티들도 자동으로 저장됩니다.

영속성 전이를 사용할 때는 주의가 필요합니다. 무분별한 사용은 예상치 못한 데이터의 변경이나 삭제를 초래할 수 있기 때문입니다. 따라서, 영속성 전이를 적용할 때는 엔티티 간의 관계와 애플리케이션의 비즈니스 로직을 충분히 고려해야 합니다.

<figure><img src="../.gitbook/assets/스크린샷 2023-12-04 오전 11.09.45.png" alt=""><figcaption></figcaption></figure>

영속성 전이를 설정해준다.



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
    @OneToMany(
        mappedBy = "provider", 
        cascade = CascadeType.PERSIST,
        orphanRemoval = true // 고아 객체를 제거하는 기능
    )
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
}
```

영속성 전이를 사용하게 되면,&#x20;

부모 엔티티가 되는 Provider 엔티티만 저장되면 코드에 작성돼 있는 cascade = CascadeType.PERSIST 에 의해 상품 엔티티도 함께 저장이 된다.

## 고아 객체

JPA 에서 고아란 부모 엔티티와 연결관계가 끊어진 엔티티를 의미한다.

JPA 에는 이러한 고아 객체를 자동으로 제거하는 기능이 있다.

물론 자식 엔티티가 다른 엔티티와 연관관계를 가지고 있다면 이 기능은 사용하지 않는것이 좋다.

```java
 @OneToMany(
        mappedBy = "provider", 
        cascade = CascadeType.PERSIST,
        orphanRemoval = true // 고아 객체를 제거하는 기능
    )
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
```

orphanRemoval = true 속성은 고아 객체를 제거하는 기능이다.&#x20;

어떻게 동작하는지 테스트를 해보겠다.

```java
    // 예제 9.28
    @Test
    @Transactional
    void orphanRemovalTest() {
        Provider provider = savedProvider("새로운 공급업체");

        Product product1 = savedProduct("상품1", 1000, 1000);
        Product product2 = savedProduct("상품2", 500, 1500);
        Product product3 = savedProduct("상품3", 750, 500);

        // 연관관계 설정
        product1.setProvider(provider);
        product2.setProvider(provider);
        product3.setProvider(provider);

        provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

        providerRepository.saveAndFlush(provider);

        System.out.println("## Before Removal ##");
        System.out.println("## provider list ##");
        providerRepository.findAll().forEach(System.out::println);

        System.out.println("## product list ##");
        productRepository.findAll().forEach(System.out::println);

        // 연관관계 제거
        Provider foundProvider = providerRepository.findById(1L).get();
        foundProvider.getProductList().remove(0);

        System.out.println("## After Removal ##");
        System.out.println("## provider list ##");
        providerRepository.findAll().forEach(System.out::println);

        System.out.println("## product list ##");
        productRepository.findAll().forEach(System.out::println);
    }

```

위의 테스트코드를 실행하게 되면 , 연관관계가 제거되면서 상태 감지를 통해 삭제하는 쿼리가 수행되는 것을 볼 수 있다.

