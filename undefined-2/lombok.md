# 롬복 Lombok

## ✅ entity 롬복

```java
package com.springboot.jpa.data.entity;

import java.time.LocalDateTime;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

import lombok.*;


@Entity
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
@ToString(exclude = "name")
@Table(name = "product")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

}

```

`@EqualsAndHashCode` 는 어떤 어노테이션 일까 ?

**equals, hashCode 자동 생성**

* **equals** : 두 객체의 내용이 같은지, 동등성(equality) 를 비교하는 연산자
* **hashCode** : 두 객체가 같은 객체인지, 동일성(identity) 를 비교하는 연산자

@**EqualsAndHashCode**어노테이션을 사용하면 자동으로 이 메소드를 생성할 수 있다.

**@Data**

* @Data는 위에서 설명한 @Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode를 한번에 설정해주는 어노테이션이다.
