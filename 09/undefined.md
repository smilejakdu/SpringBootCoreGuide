# 일대일 매핑

## 일대일 매핑

<figure><img src="../.gitbook/assets/스크린샷 2023-12-02 오전 9.58.55.png" alt=""><figcaption></figcaption></figure>

상품정보 엔티티를 작성한다.

```java
package com.springboot.relationship.data.entity;

import lombok.*;

import javax.persistence.*;

// 예제 9.1
@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToOne //(optional=false) 예제 9.5
    @JoinColumn(name = "product_id")
    private Product product;
}

```

JoinColumn 어노테이션은 기본값이 설정돼 있어 자동으로 이름을 매핑하지만 의도한 이름이 들어가지 않기 때문에 name 속성을 사용해 원하는 칼럼명을 지정하는 것이 좋다.

책에서는   @JoinColumn(name = "product\_number") 로 되어있는데 개인적으로 identity 기준으로 작성하는게 좋다고 생각합니다.

그래서 저는&#x20;

@JoinColumn(name = "product\_id") 로 수정하고 나서 작업을 진행했습니다.



name: 매핑할 외래키의 이름을 설정합니다.

referencedColumnName: 외래키가 참조할 상대 테이블의 칼럼명을 지정합니다.

foreignKey: 외래키를 생성하면서 지정할 제약조건을 설정합니다. ( unique, nullable, insertable, updatable 등)

엔티티를 작성했으면 이제 테스트 코드를 작성합니다.

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.ProductDetail;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ProductDetailRepositoryTest {

    @Autowired
    ProductDetailRepository productDetailRepository;

    @Autowired
    ProductRepository productRepository;

    // 예제 9.3
    @Test
    public void saveAndReadTest1() {
        Product product = new Product();
        product.setName("스프링 부트 JPA");
        product.setPrice(5000);
        product.setStock(500);

        productRepository.save(product);

        ProductDetail productDetail = new ProductDetail();
        productDetail.setProduct(product);
        productDetail.setDescription("스프링 부트와 JPA를 함께 볼 수 있는 책");

        productDetailRepository.save(productDetail);

        // 생성한 데이터 조회
        System.out.println("savedProduct : " + productDetailRepository.findById(
                productDetail.getId()).get().getProduct());

        System.out.println("savedProductDetail : " + productDetailRepository.findById(
                productDetail.getId()).get());
    }
}
```

테스트코드를 작성하기 위해서 상품과 상품정보에 대한 repository 에 대해 의존성 주입을 받아야합니다.

34\~35줄 : ProductDetail 객체에서 Product 객체를 일대일 단방향 연관관계를 설정했기 때문에 ProductDetailRepository 에서 ProductDetail 객체를 조회한 후 연관 매핑된 Product 객체를 조회 할 수 있습니다.

34\~35번 줄과 37\~38번 줄에서 조회하는 쿼리는 다음과 같이 표현된다.

이후 select 구문을 작성하게 될때 ProductDetail 객체와 Product 객체가 함께 조회가 됩니다.

엔티티를 조회할 때 연관된 엔티티도 함께 조회하는 것을 `즉시 로딩` 이라고 합니다.

```java
package com.springboot.relationship.data.entity;

import lombok.*;

import javax.persistence.*;

// 예제 9.1
@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToOne //(optional=false) 예제 9.5
    @JoinColumn(name = "product_number")
    private Product product;
}

```

위와 같이 optional = false 속성을 설정하면 , product 가 null 인 값을 허용하지 않게 된다.

위와 같이 설정하고 애플리케이션을 실행하면 다음과 같이 테이블을 생성하는 쿼리에서 not null 이 설정 되는 것을 확인 할 수 있습니다.

OneToOne 어노테이션에 optional = false 속성을 지정한 경우에는 left join 이 inner join 으로 바뀌어 실행이 된다.

객체에 대한 설정에 따라 최적의 쿼리를 생성해서 실행 할 수 있다.

## 일대일 양방향 매핑

객체에서의 양방향 개념은 양쪽에서 단방향으로 서로를 매핑하는 것을 의미한다.

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

```java
@OneToOne
private ProductDetail productDetail;
```

위처럼 설정하고 application 을 실행하면 product 테이블에도 칼럼이 하나 추가된다.

JPA 에서 실제 데이터베이스의 완관관계를 반영해서 한쪽의 테이블에서만 외래키를 바꿀 수 있도록 정하는것이 좋습니다.

엔티티는 양방향으로 매핑하되 한쪽에서만 외래키를 줘야하는데 , 이때 사용되는 속성 값이 mappedBy 입니다.

mappedBy 는 어떤 객체가 주인인지 표기하는 속성이라고 보면 됩니다.

<pre class="language-java"><code class="lang-java"><strong>@OneToOne(mappedBy = "product")
</strong>private ProductDetail productDetail;
</code></pre>

그리고 위와 같이 양방향 설정을 하게 되면 순환참조가 발생할 수 있습니다.

그래서 필요할 경우에는 순환 참조 제거하기 위해 exclude 를 사용해 ToString 에서 제외 설정을 해야하고&#x20;

아니면 대체로 단방향 설정을 해야합니다.

```java
@OneToOne(mappedBy = "product")
@ToString.Exclude
private ProductDetail productDetail;
```

