# ORM

ORM 은 Object Relational Mapping 의 줄임말로 객체 관계 매핑을 의미한다.

자바와 같은 객체지향언어에서 의미하는 객체와 RDB 의 테이블을 자동으로 매핑하는 방법이다.

클래스는 데이터베이스의 테이블과 매핑하기 위해 만들어진 것이 아니기 때문에 RDB 테이블과 어쩔 수 없는 불일치가 존재한다.

ORM 은 이 둘의 불일치와 제약사항을 해결하는 역할이다.

<figure><img src="../.gitbook/assets/스크린샷 2023-11-26 오후 1.41.34.png" alt=""><figcaption></figcaption></figure>

ORM을 이용하면 쿼리문 작성이 아닌 코드로 데이터를 조작할 수 있다.



* ORM을 사용하면 데이터베이스 쿼리를 객체지향적으로 조작할 수 있다.
  * 쿼리문을 작성하는 양이 현저히 줄어 개발 비용이 줄어든다.
  * 객체지향적으로 데이터베이스에 접근할 수 있어 코드의 가독성을 높인다.
* 재사용 및 유지보수가 편리하다.
  * ORM을 통해 매핑된 객체는 모두 독립적으로 작성되어 재사용이 용이하다.
  * 객체들은 각 클래스로 나뉘어 있어 유지보수가 수월하다.
* 데이터베이스에 대한 종속성이 줄어든다.
  * ORM을 통해 자동 생성된 SQL 문은 객체를 기반으로 데이터베이스 테이블을 관리하기 때문에 데이터베이스에 종속적이지 않다.
  * 데이터베이스를 교체하는 상황에서도 비교적 적은 리스크를 부담한다.

## JPA

JPA 는 자바 진영의 ORM 기술 표준으로 채택된 인터페이스의 모음이다.

ORM 이 큰 개념이라면 JPA는 더 구체화된 스펙을 포함한다.

즉, JPA 또한 실제로 동작하는 것이 아니고 어떻게 동작해야 하는지 메커니즘을 정리한 표준 명세로 생각하면 된다.

<figure><img src="../.gitbook/assets/스크린샷 2023-11-26 오후 1.56.03.png" alt=""><figcaption></figcaption></figure>

JPA의 메커니즘을 보면 내부적으로 JDBC를 사용한다.

개발자가 직접 JDBC를 구현하면 SQL에 의존하게 되는 문제등이 있어 개발의 효율성이 떨어지는데,

JPA는 이 같은 문제점을 보완해서 개발자 대신 적절한 SQL을 생성하고 데이터베이스를 조작해서 객체를 자동 매핑하는 역할을 수행한다



## ✅ 영속성 컨텍스트

<figure><img src="../.gitbook/assets/스크린샷 2023-11-26 오후 2.21.23.png" alt=""><figcaption></figcaption></figure>

영속성 컨텍스트는 세션 단위의 생명주기를 가진다.

데이터베이스에 접근하기 위한 세션이 생성되면 영속성 컨텍스트가 만들어지고, 세션이 종료되면 영속성 컨텍스트도 없어진다.



## 엔티티 매니저

엔티티 매니저는 이름 그대로 엔티티를 관리하는 객체이다. 앤티티 매니저는 데이터베이스에 접근해서 CRUD 작업을 수행한다. Spring Data JPA를 사용하면 리포지토리를 사용해서 데이터베이스에 접근하는데, 실제 내부 구현체인 SimpleJpaRepository 가 리포지토리에서 엔티티 매니저를 사용하게 된다.

```java
public SimpleJpaRepository(JpaEntityInformation<T,?> entityInformation, EntityManager entityManager) {
    Aseert.notNull(entityInformation, "JpaEntityInformation must not be null");
    Assert.notNull(entityManager, "EntityManager must not be null!");
    
    this.entityInformation = entityInformation;
    this.em = entityManager;
    this.provider = PersistenceProvider.fromEntityManager(entityManager);
}
```

엔티티 매니저는 엔티티 매니저 팩토리가 만든다.

엔티티 매니저 팩토리는 데이터베이스에 대응하는 객체로서 스프링 부트에서는 자동 설정 기능이 있기때문에 application.properties 에서 작성한 최소한의 설정만으로도 동작하지만 JPA의 구현체 중 하나인 하이버네이트에서는 persistence.xml 이라는 설정 파일을 구성하고 사용해야하는 객체입니다.

