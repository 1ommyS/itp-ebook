
## 1. Принцип Inversion of Control и DI

* **Inversion of Control (IoC)** — контейнер управляет созданием и жизненным циклом компонентов (bean-ов), а не ваша программа вручную.
* **Dependency Injection (DI)** — контейнер «впрыскивает» (inject) в bean зависимости, которые ему требуются, вместо того чтобы bean сам их доставал.

Преимущества:

1. Слабая связанность классов.
2. Упрощение тестирования (можно подменять зависимости на моки).
3. Централизованная конфигурация и управление объектами.

---

## 2. Способы инъекции зависимостей

### 2.1. Constructor Injection

```java
@Component
public class OrderService {
    private final PaymentService paymentService;
    
    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

* **Плюсы:** гарантирует, что зависимость не может быть `null`, удобен для immutable-классов, способствует явной документации API класса.
* **Рекомендация:** используйте constructor injection по умолчанию.

### 2.2. Setter Injection

```java
@Component
public class OrderService {
    private PaymentService paymentService;
    
    @Autowired
    public void setPaymentService(PaymentService ps) {
        this.paymentService = ps;
    }
}
```

* **Плюсы:** подходит для необязательных зависимостей.
* **Минусы:** bean может оказаться в неполностью сконфигурированном состоянии.

### 2.3. Field Injection

```java
@Component
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

* **Плюсы:** меньше кода.
* **Минусы:** нельзя делать `final`, сложнее тестировать (нужен Reflection), неявно видно зависимости.

---

## 3. Конфигурация DI

### 3.1. Аннотационная

* `@Component`, `@Service`, `@Repository`, `@Controller` — маркировка классов для component-scan.
* `@Configuration` + `@Bean` — Java-конфигурация:

  ```java
  @Configuration
  public class AppConfig {
    @Bean
    public PaymentService paymentService() {
      return new StripePaymentService();
    }
  }
  ```

### 3.2. XML-конфигурация

```xml
<beans xmlns="http://www.springframework.org/schema/beans" …>
  <bean id="paymentService" class="com.example.StripePaymentService"/>
  <bean id="orderService" class="com.example.OrderService">
    <constructor-arg ref="paymentService"/>
  </bean>
</beans>
```

---

## 4. Автовнедрение и разрешение неоднозначностей

### 4.1. @Autowired

* По умолчанию **по типу**; если несколько кандидатов — ошибка.
* `required=false` сделает зависимость опциональной.

### 4.2. @Qualifier и @Primary

```java
@Component @Primary
public class PaypalPaymentService implements PaymentService {…}

@Component
@Qualifier("stripe")
public class StripePaymentService implements PaymentService {…}

@Component
public class OrderService {
    @Autowired
    public OrderService(@Qualifier("stripe") PaymentService ps) {…}
}
```

* `@Primary` — предпочтительный бин при автосвязывании без уточнения.
* `@Qualifier("name")` — конкретизирует, какой именно бин нужно ввести.

### 4.3. @Resource и @Inject

* `@Resource(name="beanName")` из JSR-250 — по имени.
* `@Inject` (JSR-330) аналог `@Autowired`, но без Spring-специфических атрибутов.

---

## 5. Инъекция коллекций и карт

Spring умеет внедрять все бины заданного типа:

```java
@Component
public class ReportService {
    private final List<ReportGenerator> generators;

    @Autowired
    public ReportService(List<ReportGenerator> generators) {
        this.generators = generators;
    }
}
```

* `List<Interface>` — внедряются все реализации в порядке объявления.
* Можно также `Set<…>` или `Map<String, BeanType>` — тогда ключом станет имя бина.

---

## 6. Профили и условная загрузка

* `@Profile("dev")` — бин активен только при указанном профиле.
* `@ConditionalOnProperty`, `@ConditionalOnMissingBean` (в Spring Boot) — гибкое условие создания бина.

```java
@Bean
@Profile("prod")
public DataSource prodDataSource() { … }

@Bean
@Profile("dev")
public DataSource devDataSource() { … }
```

---

## 7. Scopes (области видимости) bean-ов

| Scope         | Описание                                                |
| ------------- | ------------------------------------------------------- |
| `singleton`   | единый экземпляр на `ApplicationContext` (по умолчанию) |
| `prototype`   | новый экземпляр при каждом запросе из контейнера        |
| `request`     | один экземпляр на HTTP-запрос (web-scope)               |
| `session`     | один экземпляр на HTTP-сессию                           |
| `application` | один экземпляр на `ServletContext`                      |

```java
@Component
@Scope("prototype")
public class TaskProcessor { … }
```

---

## 8. Пост-обработка bean-ов

* **`BeanPostProcessor`** — перехватывает все bean-ы до/после инициализации:

  ```java
  @Component
  public class AuditBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String name) { … }
    @Override
    public Object postProcessAfterInitialization(Object bean, String name) { … }
  }
  ```

* **`BeanFactoryPostProcessor`** — модифицирует определения bean-ов до создания экземпляров (e.g. `PropertyPlaceholderConfigurer`).

---

## 9. Circular Dependencies

* Spring умеет разрешать круговые зависимости через **setter** и **proxy**, но **constructor-cycle** без опоры на `@Lazy` приведёт к ошибке.
* Для прерывания цикла можно:

    * Использовать setter injection вместо constructor.
    * Пометить один из бинов `@Lazy`.
