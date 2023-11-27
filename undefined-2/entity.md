# Entity

Spring Data JPA를 사용하면 데이터베이스에 테이블을 생성하기 위해 직접 쿼리를 작성할 필요가 없습니다.

이 기능을 가능하게 하는 것이 엔티티입니다.

JPA 에서 엔티티는 데이터베이스의 테이블에 대응하는 클래스이다.

엔티티에는 데이터베이스에 쓰일 테이블과 칼럼을 정의한다.



```java
package com.springboot.jpa.data.entity;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name="product")
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
    
```



클래스 자체는 테이블과 일대일로 매칭되며, 해당 클래스의 인스턴스는 매핑되는 테이블에서 하나의 레코드를 의미한다.

