
## 1. Что такое Spring Data JPA и зачем он нужен

**Spring Data JPA** — это модуль Spring Data, который упрощает работу с JPA-провайдерами (Hibernate, EclipseLink и др.). Он позволяет:

* **Минимизировать «бойлерплейт»** CRUD-операций (DAO, шаблонные запросы).
* **Автоматически генерировать репозитории** на основе интерфейсов.
* **Поддерживать** пагинацию, сортировку, динамические запросы (Specification, Query by Example).
* **Интегрироваться** с Spring Transaction и Spring Boot автоконфигурацией.

В основе лежит **паттерн Repository**: вы описываете Java-интерфейс, а Spring Data сам генерирует реализацию в рантайме.

---

## 2. Основные интерфейсы репозиториев

Spring Data JPA поставляет набор базовых интерфейсов:

| Интерфейс                           | Описание                                                                                                       |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `Repository<T, ID>`                 | Пустой «маркер», все остальные расширяют его.                                                                  |
| `CrudRepository<T, ID>`             | CRUD-операции: `save`, `findById`, `delete`                                                                    |
| `PagingAndSortingRepository<T, ID>` | Плюс `findAll(Pageable)`, `findAll(Sort)`                                                                      |
| `JpaRepository<T, ID>`              | Расширяет `PagingAndSortingRepository`, плюс JPA-специфичные методы (`flush`, `saveAndFlush`, `deleteInBatch`) |

### Пример объявления репозитория

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // методы настраиваются далее
}
```

При старте приложения Spring создаст бины типа `UserRepository` с полной реализацией.

---

## 3. Derived Query Methods (производные методы)

По соглашению об именах Spring Data генерирует запросы автоматически:

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // поиск по одному полю
    Optional<User> findByUsername(String username);

    // составное условие
    List<User> findAllByStatusAndCreatedAtBetween(
        UserStatus status, LocalDateTime from, LocalDateTime to);

    // сортировка по параметру
    List<User> findByEmailContainingIgnoreCaseOrderByCreatedAtDesc(String emailPart);
}
```

**Ключевые слова в именах**: `And`, `Or`, `Between`, `LessThan`, `Like`, `In`, `OrderBy`, `IgnoreCase`, `Distinct`, `TopN`, и т.д.

---

## 4. JPQL и Native Queries

### 4.1. @Query (JPQL)

Для более сложных запросов вы можете писать JPQL:

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    @Query("SELECT o FROM Order o JOIN o.user u WHERE u.id = :userId AND o.status = :status")
    List<Order> findByUserAndStatus(@Param("userId") Long userId, @Param("status") OrderStatus status);
}
```

### 4.2. Native SQL

Если нужен «родной» SQL с функциями СУБД:

```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    @Query(value = "SELECT * FROM products p WHERE p.price > :minPrice", nativeQuery = true)
    List<Product> findExpensiveProducts(@Param("minPrice") BigDecimal minPrice);
}
```

---

## 5. Пагинация и сортировка

```java
// в методе репозитория
Page<Order> findByUserId(Long userId, Pageable page);

// вызов из сервиса
Pageable paging = PageRequest.of(0, 20, Sort.by("createdAt").descending());
Page<Order> page = orderRepo.findByUserId(userId, paging);
List<Order> orders = page.getContent();
long total = page.getTotalElements();
```

---

## 6. Dоmic Specifications (Criteria API)

API для динамических запросов на основе JPA Criteria:

```java
public class UserSpecifications {
    public static Specification<User> hasStatus(UserStatus status) {
        return (root, cq, cb) -> cb.equal(root.get("status"), status);
    }
    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, cq, cb) -> cb.greaterThan(root.get("createdAt"), date);
    }
}

// в репозитории
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> { }

// в сервисе
Specification<User> spec = where(hasStatus(ACTIVE)).and(createdAfter(oneMonthAgo));
List<User> users = userRepo.findAll(spec, Sort.by("username"));
```

---

## 7. Query by Example (QBE)

Простая форма динамического поиска без Criteria:

```java
User probe = new User();
probe.setStatus(UserStatus.ACTIVE);
probe.setCountry("RU");

ExampleMatcher matcher = ExampleMatcher.matching()
    .withIgnorePaths("id", "createdAt")
    .withStringMatcher(StringMatcher.CONTAINING)
    .withIgnoreCase();

Example<User> example = Example.of(probe, matcher);
List<User> users = userRepo.findAll(example);
```

---

## 8. Проекции (Projections)

### 8.1. Интерфейсные проекции

```java
public interface UserView {
    String getUsername();
    String getEmail();
    AddressView getAddress();  // вложенная проекция
}
public interface AddressView {
    String getCity();
    String getStreet();
}

// в репозитории
List<UserView> findAllByStatus(UserStatus status);
```

### 8.2. Класс-DTO проекции

```java
public class UserDto {
    private final String username;
    private final String email;
    public UserDto(String username, String email) { … }
    // геттеры…
}

// в репозитории
@Query("SELECT new com.example.dto.UserDto(u.username, u.email) FROM User u")
List<UserDto> findDtoByStatus(UserStatus status);
```

---

## 9. Кастомная реализация методов

Если нужно добавить специфичные методы:

1. **Определить интерфейс** с суффиксом `Custom`:

   ```java
   public interface UserRepositoryCustom {
     List<User> findByComplexRule(...);
   }
   ```
2. **Реализовать** его:

   ```java
   public class UserRepositoryImpl implements UserRepositoryCustom {
     @PersistenceContext
     private EntityManager em;
     @Override
     public List<User> findByComplexRule(...) { /* JPQL/Criteria */ }
   }
   ```
3. **Наследовать** в основном репозитории:

   ```java
   public interface UserRepository extends JpaRepository<User,Long>, UserRepositoryCustom { }
   ```

Spring подхватит `UserRepositoryImpl` автоматически.

---

## 10. Транзакции и Spring Data JPA

* По умолчанию CRUD-методы `save`, `delete`, `find…` обёрнуты в транзакции (`@Transactional`) с `readOnly=false` для мутаций и `readOnly=true` для чтения.
* Можно переопределить на уровне метода:

  ```java
  @Transactional(readOnly = true)
  @Override
  List<UserView> findAllByStatus(UserStatus status);
  ```

---

## 11. Настройки и оптимизация

* **batch\_size** для `saveAll` через `spring.jpa.properties.hibernate.jdbc.batch_size`.
* **Первый/второй уровень кэша** — настраивать в `application.properties`.
* **Entity Graph** — можно применить и в Spring Data JPA через `@EntityGraph` на методе репозитория:

  ```java
  @EntityGraph(attributePaths = {"orders", "address"})
  List<User> findByStatus(UserStatus status);
  ```

---

## 12. Пример Spring Boot проекта

```java
@SpringBootApplication
public class Application { public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
}}
```

```properties
spring.datasource.url=jdbc:postgresql://... 
spring.datasource.username=... 
spring.datasource.password=... 
spring.jpa.hibernate.ddl-auto=validate 
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
