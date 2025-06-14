
## 1. Зачем нужен Entity Graph

По умолчанию JPA-провайдер загружает ассоциации в зависимости от настроек `FetchType` (`EAGER`/`LAZY`), что может приводить к проблемам:

* **N+1-проблема**: при `LAZY`-загрузке “дочерних” коллекций большое число дополнительных запросов.
* **Перегрузка данных**: при `EAGER`-загрузке могут подтягиваться ненужные связи.

**Entity Graph** позволяет гибко указывать, какие связи нужно забрать **единственным** SQL-ом, а какие оставить `LAZY`, без изменения аннотаций в модели.

---

## 2. Виды Entity Graph

1. **Named Entity Graph** — статически объявляется в аннотациях `@NamedEntityGraph` над сущностью.
2. **Dynamic Entity Graph** — строится «на лету» через API `EntityManager.createEntityGraph(...)`.

Для каждого графа можно указать:

* **fetch graph** — «полный» граф, JPA будет `EAGER`-подгружать только указанные атрибуты, остальные — `LAZY`.
* **load graph** — JPA будет `EAGER`-подгружать указанные атрибуты **дополнительно** к тем, что уже `EAGER` в модели.

---

## 3. Объявление Named Entity Graph

Предположим у нас есть сущности `User` и связанная с ним коллекция `orders`:

```java
@Entity
@Table(name = "users")
@NamedEntityGraph(
  name = "User.withOrders",
  attributeNodes = @NamedAttributeNode("orders")
)
public class User {
    @Id
    @GeneratedValue
    private Long id;

    private String username;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<Order> orders = new ArrayList<>();

    // getters/setters...
}
```

* **`@NamedEntityGraph`** с именем `User.withOrders` включает атрибут `orders`.
* По умолчанию остальные `LAZY`-поле останутся `LAZY`.

Можно описать **глубокий граф** (несколько уровней) через `subgraph`:

```java
@NamedEntityGraph(
  name = "User.withOrdersAndProducts",
  attributeNodes = {
    @NamedAttributeNode(value = "orders", subgraph = "ordersWithItems")
  },
  subgraphs = {
    @NamedSubgraph(
      name = "ordersWithItems",
      attributeNodes = @NamedAttributeNode("orderItems")
    )
  }
)
```

Здесь мы говорим: при загрузке `User` подгрузить `orders`, а в каждой `Order` — ещё и `orderItems`.

---

## 4. Использование Named Entity Graph в запросах

### 4.1. JPQL с hint’ом

```java
EntityGraph<?> graph = em.getEntityGraph("User.withOrders");

List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE u.username LIKE :pattern", User.class)
  .setParameter("pattern", "ivan%")
  .setHint("javax.persistence.fetchgraph", graph)  // или "javax.persistence.loadgraph"
  .getResultList();
```

* `"javax.persistence.fetchgraph"` — все свойства, **не** указанные в graph, будут загружаться `LAZY`.
* `"javax.persistence.loadgraph"` — все свойства, указанные в graph, будут загружаться `EAGER` в дополнение к дефолтным `EAGER`.

### 4.2. Criteria API

```java
EntityGraph<User> graph = em.createEntityGraph(User.class);
graph.addAttributeNodes("orders");

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> root = cq.from(User.class);
cq.select(root).where(cb.equal(root.get("username"), "ivan"));

List<User> result = em.createQuery(cq)
  .setHint("javax.persistence.loadgraph", graph)
  .getResultList();
```

---

## 5. Dynamic Entity Graph

Если не хочется или невозможно заранее объявлять Named Graph, его можно собрать программно:

```java
// Создаём граф для User
EntityGraph<User> dynGraph = em.createEntityGraph(User.class);
dynGraph.addAttributeNodes("orders");

// Можно добавить subgraph:
Subgraph<Order> ordersSub = dynGraph.addSubgraph("orders");
ordersSub.addAttributeNodes("orderItems");

// Используем его в запросе
User single = em.find(
    User.class, 
    123L, 
    Collections.singletonMap("javax.persistence.fetchgraph", dynGraph)
);
```

Варианты использования:

* `em.find(...)` с параметрами `Map<String, Object> properties`.
* `em.createQuery(...)` + `setHint(...)`, как в разделе выше.

---

## 6. Пример полного кода

```java
// 1. Объявление EntityGraph в классе
@Entity
@NamedEntityGraph(
  name = "User.full",
  attributeNodes = {
    @NamedAttributeNode(value = "orders", subgraph = "ordersItems")
  },
  subgraphs = @NamedSubgraph(
    name = "ordersItems",
    attributeNodes = @NamedAttributeNode("orderItems")
  )
)
public class User { /* … */ }

@Entity
public class Order {
  @Id @GeneratedValue private Long id;
  @ManyToOne(fetch = FetchType.LAZY) private User user;
  @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
  private List<OrderItem> orderItems;
  // …
}

@Entity
public class OrderItem {
  @Id @GeneratedValue private Long id;
  @ManyToOne(fetch = FetchType.LAZY) private Order order;
  private Integer quantity;
  // …
}

// 2. Загрузка с NamedEntityGraph
EntityGraph<?> graph = em.getEntityGraph("User.full");

List<User> list = em.createQuery(
    "SELECT u FROM User u WHERE u.id IN :ids", User.class)
  .setParameter("ids", Arrays.asList(1L,2L,3L))
  .setHint("javax.persistence.fetchgraph", graph)
  .getResultList();

// Теперь у каждого User будут подгружены и orders, и orderItems в них,
// но другие ленивые связи останутся ленивыми.

// 3. Программный EntityGraph (динамический)
EntityGraph<User> dyn = em.createEntityGraph(User.class);
dyn.addAttributeNodes("orders");

List<User> users = em.createQuery("FROM User u", User.class)
  .setHint("javax.persistence.loadgraph", dyn)
  .getResultList();
```

---

## 7. Когда использовать Entity Graph

* **Избежать N+1** без глобального перевода `FetchType` в `EAGER`.
* **Гибко собирать** разные варианты загрузки в одном сервисе.
* **Переиспользовать** часто встречающиеся комбинации связей через `@NamedEntityGraph`.
