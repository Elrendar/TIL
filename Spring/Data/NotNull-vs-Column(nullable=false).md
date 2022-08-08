# NotNull vs Column(nullable = false)

[목록으로 돌아가기](/README.md)

## 개요

JPA를 사용할 때, `@NotNull`과 `@Column(nullable = false)` 은 둘 다 db에 null 값이 저장되는 것을 막는 용도로 사용된다.

`@Column`을 사용하려면 `Spring Data JPA`가, `@NotNull`을 사용하려면 `Spring Validation`이 필요하다.

```Gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

하지만 이 둘에는 중요한 차이가 있다.

### @NotNull

```Java
@Entity
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    @NotNull
    private BigDecimal price;
}
```

```Java
@SpringBootTest
public class ItemIntegrationTest {

    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void shouldNotAllowToPersistNullItemsPrice() {
        itemRepository.save(new Item());
    }
}
```

테스트를 실행하면 `javax.validation.ConstraintViolationException`이 throw된다.

여기서 중요한 점은, `Hibernate`가 SQL 삽입을 시도하지 않았다는 점이다.

db에 쿼리를 보내기 전에 `bean validation`에 의해 차단되기 때문이다.

### 검증

먼저, `application.properties` 파일에 아래와 같은 옵션을 추가한다.

```application.properties
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

`spring.jpa.hibernate.ddl-auto`는 db 초기화 전략을 정하는 옵션이다.

`create-drop`은 서버가 실행될 때 drop 및 생성을 실행하고, 종료될 때 drop을 실행한다. in-memory db의 경우 기본값이다.

`spring.jpa.show-sql`은 `Hibernate`가 db에 날리는 모든 쿼리 로그를 보여줄지 정하는 옵션이다.

> *`spring.jpa.properties.hibernate.format_sql=true` 까지 사용하면 조금 더 예쁘게 정리된 형태로 볼 수 있다.*

이 상태로 아까의 테스트를 실행하면 DDL문의 형태를 볼 수 있다.

```SQL
create table item (
   id bigint not null,
    price decimal(19,2) not null,
    primary key (id)
)
```

보다시피, `Hibernate`가 자동으로 price 컬럼에 `not null` 제약조건을 추가했다.

## @Column(nullable = false)

`@Column` 어노테이션은 JPA에 정의되어있다. 이것은 `Hibernate`가 db 스키마를 자동 생성할 때, `not null` 제약조건을 컬럼에 추가한다.

아까의 코드를 아래와 같이 바꿔보자.

```Java
@Entity
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private BigDecimal price;
}
```

```Java
@SpringBootTest
public class ItemIntegrationTest {

    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void shouldNotAllowToPersistNullItemsPrice() {
        itemRepository.save(new Item());
    }
}
```

이번에는 로그에 이렇게 나온다.

```sql
Hibernate: 
    
    create table item (
       id bigint not null,
        price decimal(19,2) not null,
        primary key (id)
    )

(...)

Hibernate: 
    insert 
    into
        item
        (price, id) 
    values
        (?, ?)

[main] o.h.engine.jdbc.spi.SqlExceptionHelper   : 
NULL not allowed for column "PRICE"
```

`@NotNull`때와 마찬가지로, 이번에도 역시 null값이 db에 들어가지 않았다.

하지만 이번에는 SQL 삽입 쿼리가 생성되고, 삽입 시도가 행해졌다는 게 차이점이다.

## 결론

이 둘의 차이는 결국 데이터에 대한 검증을 자바 코드 단계에서 시행하느냐, 아니면 db가 직접 하느냐의 차이이다.

그리고 늘 그렇듯이, 이런 문제는 조금이라도 일찍, 빠르게 알아낼수록 좋다.

조금 늦어지는 그 사이에 어떤 다른 동작이 수행되고, 또 다른 문제가 발생할지 예측하기가 어렵기 때문이다.

따라서 null값 방지가 목적이라면 `@Column(nullable = false)`보다는 `@NotNull`을 사용하는 것이 권장된다.
