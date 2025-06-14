
## 1. Что такое Hibernate и зачем он нужен

**Hibernate** – это фреймворк, реализующий паттерн «Data Mapper», предоставляющий разработчику возможность работать с объектами Java, а не напрямую с SQL-запросами. Он автоматически переводит операции с объектами в SQL-команды и наоборот.

**Ключевые цели Hibernate:**

1. **Абстракция работы с БД**: забыть о JDBC-бойлерплейте (Connection, Statement, ResultSet).
2. **Ускорение разработки**: маппинг POJO ↔ таблицы, декларативная настройка связей.
3. **Кросс-СУБД**: большинство SQL-диалектов «под капотом».
4. **Кэширование и оптимизация**: первый и второй уровни кэша, батчи, lazy-загрузка.

---

## 2. Архитектура Hibernate

```
+--------------------+
|  Приложение (DAO)  |
+--------------------+
          │
          ▼
+--------------------+
|   SessionFactory   |  ← создаётся единожды
+--------------------+
          │   ▲
          ▼   │
+--------------------+
|      Session       |  ← short-lived, per-transaction
+--------------------+
    │     │     │
    ▼     ▼     ▼
  save  query  delete
    │     │     │
    ▼     ▼     ▼
+--------------------+
|    JDBC Connection |
+--------------------+
          │
          ▼
+--------------------+
|    База данных     |
+--------------------+
+--------------------+
```

* **SessionFactory** – потокобезопасный фабричный объект, конфигурируется один раз при старте приложения.
* **Session** – не потокобезопасный, «рабочий» объект для CRUD-операций и транзакций; живёт недолго (обычно per-request).
* **Transaction** – обёртка над JDBC-транзакцией; управляет `commit`/`rollback`.
* **Query / Criteria API** – DSL для работы с HQL/SQL или критериальный API.
* **Cache (L1, L2)** – первый уровень кэш привязан к Session, второй – к SessionFactory (при необходимости совместно для нескольких процессов).

---

## 3. Конфигурация Hibernate

### 3.1. XML-конфигурация (`hibernate.cfg.xml`)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<hibernate-configuration>
  <session-factory>
    <!-- JDBC параметры -->
    <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>
    <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/mydb</property>
    <property name="hibernate.connection.username">user</property>
    <property name="hibernate.connection.password">pass</property>

    <!-- Диалект СУБД -->
    <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQLDialect</property>

    <!-- Управление схемой -->
    <property name="hibernate.hbm2ddl.auto">validate</property>
    <!-- possible: validate | update | create | create-drop -->

    <!-- Кэш -->
    <property name="hibernate.cache.use_second_level_cache">true</property>
    <property name="hibernate.cache.region.factory_class">
      org.hibernate.cache.ehcache.EhCacheRegionFactory
    </property>

    <!-- Пакет с Entity -->
    <mapping package="com.example.model"/>
  </session-factory>
</hibernate-configuration>
```

### 3.2. Java-конфигурация (JPA + Spring)

```java
@Configuration
@EnableTransactionManagement
public class HibernateConfig {
  @Bean
  public LocalSessionFactoryBean sessionFactory(DataSource dataSource) {
    LocalSessionFactoryBean sf = new LocalSessionFactoryBean();
    sf.setDataSource(dataSource);
    sf.setPackagesToScan("com.example.model");
    Properties props = new Properties();
    props.put("hibernate.dialect", "org.hibernate.dialect.PostgreSQLDialect");
    props.put("hibernate.hbm2ddl.auto", "validate");
    props.put("hibernate.show_sql", "true");
    sf.setHibernateProperties(props);
    return sf;
  }

  @Bean
  public HibernateTransactionManager transactionManager(SessionFactory sessionFactory) {
    HibernateTransactionManager tm = new HibernateTransactionManager();
    tm.setSessionFactory(sessionFactory);
    return tm;
  }
}
```

---

## 4. Маппинг сущностей

### 4.1. Аннотации JPA

```java
@Entity
@Table(name = "users")
public class User {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(name = "username", nullable = false, unique = true, length = 50)
  private String username;

  @Column(name = "email", nullable = false, unique = true)
  private String email;

  @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
  private List<Order> orders = new ArrayList<>();

  // конструкторы, геттеры/сеттеры
}
```

### 4.2. XML-маппинг (`User.hbm.xml`)

```xml
<hibernate-mapping>
  <class name="com.example.model.User" table="users">
    <id name="id">
      <generator class="identity"/>
    </id>
    <property name="username" column="username" not-null="true" unique="true" length="50"/>
    <property name="email" column="email" not-null="true" unique="true"/>
    <set name="orders" inverse="true" cascade="all-delete-orphan">
      <key column="user_id"/>
      <one-to-many class="com.example.model.Order"/>
    </set>
  </class>
</hibernate-mapping>
```

### 4.3. Типы связей

* **@OneToMany / @ManyToOne**
* **@OneToOne**
* **@ManyToMany** (с промежуточной таблицей `@JoinTable`)
* **FetchType.LAZY vs EAGER** – по умолчанию `OneToMany` и `ManyToMany` LAZY, `ManyToOne` EAGER.

---

## 5. CRUD-операции и жизненный цикл объекта

1. **Transient** – объект создан, но не привязан к сессии и не сохраняется в БД.
2. **Persistent** – объект привязан к сессии; изменения автоматически «снятчекены» (dirty checking).
3. **Detached** – объект отсоединён (Session закрыта или явно `evict`), но имеет идентификатор.
4. **Removed** – объект помечен на удаление в текущей сессии.

### Пример работы с сессией

```java
try (Session session = sessionFactory.openSession()) {
  Transaction tx = session.beginTransaction();

  // Создание нового пользователя
  User u = new User();
  u.setUsername("ivan");
  u.setEmail("ivan@example.com");
  session.save(u);                // переходит в Persistent

  // Извлечение и изменение
  User persisted = session.get(User.class, u.getId());
  persisted.setEmail("new@mail"); // автоматически зафиксируется при flush

  // Удаление
  session.delete(persisted);

  tx.commit();                    // flush + commit
}
```

---

## 6. HQL и Criteria API

### 6.1. HQL (Hibernate Query Language)

```java
// Параметризованный запрос
List<User> users = session.createQuery(
    "from User u where u.username = :name", User.class)
  .setParameter("name", "ivan")
  .list();

// JOIN и агрегаты
List<Object[]> stats = session.createQuery(
    "select u.username, count(o) from User u " +
    "join u.orders o group by u.username")
  .list();
```

### 6.2. Criteria API (JPA)

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> root = cq.from(User.class);
cq.select(root)
  .where(cb.equal(root.get("email"), "ivan@example.com"));
List<User> list = session.createQuery(cq).getResultList();
```

---

## 7. Транзакции и изоляция

* Hibernate поддерживает **декларативные** (`@Transactional` в Spring) и **программные** (`session.beginTransaction()`) транзакции.
* **Уровни изоляции** контролируются на стороне БД (`READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`).
* Важный момент: **коммит** вызывает `flush()`, при котором все изменения persistent-объектов материализуются в SQL-запросы.

---

## 8. Кэширование

### 8.1. Первый уровень (L1 Cache)

* По умолчанию привязан к **Session**.
* Гарантирует, что два `session.get(User, id)` в одной сессии не будут дважды читать БД.

### 8.2. Второй уровень (L2 Cache)

* Опционален, на уровне **SessionFactory**.
* Сохраняет данные между сессиями.
* Требует настройки провайдера (EhCache, Infinispan, Hazelcast).

```xml
<property name="hibernate.cache.use_second_level_cache">true</property>
<property name="hibernate.cache.region.factory_class">
  org.hibernate.cache.ehcache.EhCacheRegionFactory
</property>
<cache usage="read-write"/>
```

### 8.3. Query Cache

* Кэширует результаты HQL-запросов, но зависит от L2 Cache для сущностей.
* Включается `hibernate.cache.use_query_cache=true` и `query.setCacheable(true)`.

---

## 9. Оптимизация производительности

1. **Batch Inserts/Updates**

   ```properties
   hibernate.jdbc.batch_size=50
   ```
2. **StatelessSession** – обход L1 Cache для массовых операций.
3. **Fetch Strategies**

    * `JOIN FETCH` в HQL
    * `@BatchSize(size=...)` для коллекций
4. **Avoid N+1 problem** с помощью `fetch join` или `EntityGraph`.
5. **Second Level Cache** для «слабо меняющихся» сущностей.
6. **Read-Only Entities**: `session.setDefaultReadOnly(true)` для оптимизации.
7. **Connection Pooling** через HikariCP или c3p0.

---

## 10. Миграции схемы и версионирование

* Hibernate умеет сам генерировать DDL (`hbm2ddl.auto`), но на продакшене лучше использовать Liquibase/Flyway для контроля версий.
* **Схема + обратная совместимость**: не полагаться на автоматическое `update`, а писать миграции вручную.

---

## 11. Расширенные возможности

* **Envers** – аудит изменений сущностей.
* **Interceptor / EventListener** – точечная логика при загрузке/сохранении.
* **Custom Types** – маппинг нестандартных типов (`@Type`).
* **Multi-Tenancy** – поддержка нескольких схем или баз в одном приложении.

