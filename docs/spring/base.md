
## 1. Основные принципы Spring

1. **Inversion of Control (IoC)**
   Контейнер управляет жизненным циклом объектов (bean-ами), их созданием, связями и уничтожением.
2. **Dependency Injection (DI)**
   Контейнер внедряет зависимости в ваши объекты вместо того, чтобы объекты сами создавали нужные им сервисы.
3. **Aspect-Oriented Programming (AOP)**
   Сквозная функциональность (логирование, транзакции, безопасность) выносят в “аспекты”, а не размазывают по коду.
4. **Принцип разделения конфигурации и кода**
   Настройки и определения bean-ов можно держать в XML, аннотациях или Java-классе, отдельно от бизнес-логики.

---

## 2. Модули Spring

Spring состоит из набора взаимосвязанных модулей, которые можно использовать по отдельности:

| Модуль             | Назначение                                                 |
| ------------------ | ---------------------------------------------------------- |
| **Core Container** | IoC-контейнер: `BeanFactory` / `ApplicationContext`        |
| **Spring AOP**     | Реализация аспектов поверх JDK-прокси или CGLIB            |
| **Data Access**    | JDBC-шаблон, ORM-интеграция (Hibernate, JPA), DAO-шаблоны  |
| **Transaction**    | Декларативный и программный менеджмент транзакций          |
| **Web / Web MVC**  | Веб-слой: контроллеры, DispatcherServlet, ViewResolver     |
| **Messaging**      | Поддержка JMS, STOMP, WebSocket                            |
| **Cache**          | Абстракция кэша поверх разных реализаций (EhCache, Redis…) |
| **Integration**    | Spring Integration, Enterprise Integration Patterns        |
| **Security**       | Spring Security (аутентификация, авторизация)              |
| **Batch**          | Обработка больших объёмов данных партиями                  |
| **Social**         | Подключение OAuth, социальные сети                         |
| **Boot**           | Автоконфигурация, упрощённый старт приложений              |

Все модули строятся поверх **Core Container** и могут дополняться друг другом.

---

## 3. IoC-контейнер и bean-ы

### 3.1. BeanFactory vs ApplicationContext

* **BeanFactory**
  – базовый интерфейс контейнера. Загружает bean-ы “лениво” — по первому обращению.
* **ApplicationContext**
  – расширяет BeanFactory:

    * Пре-инстанцирует singleton-bean-ы при старте
    * Поддерживает публикацию событий, ресурс-лоадер, MessageSource (i18n)
    * Позволяет легко интегрироваться с веб-контекстом (WebApplicationContext)

### 3.2. Конфигурация bean-ов

1. **Аннотации (Java-config + Component-scan):**

   ```java
   @Configuration
   @ComponentScan("com.example.app")
   public class AppConfig {
     @Bean
     public DataSource dataSource() {
       HikariDataSource ds = new HikariDataSource();
       ds.setJdbcUrl("jdbc:postgresql://localhost:5432/db");
       ds.setUsername("user");
       ds.setPassword("pass");
       return ds;
     }
   }

   @Service
   public class UserService {
     private final UserRepository repo;
     @Autowired
     public UserService(UserRepository repo) {
       this.repo = repo;
     }
   }

   @Repository
   public interface UserRepository extends JpaRepository<User,Long> { }
   ```

2. **XML-конфигурация:**

   ```xml
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:tx="http://www.springframework.org/schema/tx"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="...">
     <context:component-scan base-package="com.example.app"/>
     <bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource">
       <property name="jdbcUrl" value="jdbc:..."/>
       <property name="username" value="user"/>
       <property name="password" value="pass"/>
     </bean>
     <tx:annotation-driven transaction-manager="txManager"/>
     <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource"/>
     </bean>
   </beans>
   ```

3. **Java-конфигурация (без XML):**

   ```java
   @SpringBootApplication  // включает @Configuration, @ComponentScan и автоконфиг
   public class Application {
     public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
     }
   }
   ```

### 3.3. Жизненный цикл bean

1. **Instantiation** — создание экземпляра.
2. **Populate properties** — внедрение зависимостей.
3. **Pre-init** — интерфейс `BeanPostProcessor``#postProcessBeforeInitialization`.
4. **InitializingBean.afterPropertiesSet** или метод, помеченный `@PostConstruct`.
5. **Post-init** — `BeanPostProcessor#postProcessAfterInitialization`.
6. **Destruction** — `DisposableBean#destroy` или метод `@PreDestroy`.

---

## 4. Dependency Injection

Способы внедрения зависимостей:

* **Constructor Injection** (рекомендуется)

  ```java
  @Component
  public class OrderService {
    private final PaymentService payment;
    @Autowired
    public OrderService(PaymentService payment) {
      this.payment = payment;
    }
  }
  ```

* **Setter Injection**

  ```java
  @Component
  public class OrderService {
    private PaymentService payment;
    @Autowired
    public void setPaymentService(PaymentService p) {
      this.payment = p;
    }
  }
  ```

* **Field Injection** (менее гибко, усложняет тестирование)

  ```java
  @Component
  public class OrderService {
    @Autowired
    private PaymentService payment;
  }
  ```

---

## 5. AOP и прокси

### 5.1. Как работает

* Spring генерирует прокси-объекты (JDK-интерфейсные или CGLIB-классы), перехватывающие вызовы методов.
* В точках “advice” (до/после/вокруг метода) может выполняться дополнительный код.

### 5.2. Пример аспекта

```java
@Aspect
@Component
public class LoggingAspect {
  @Pointcut("execution(* com.example.service.*.*(..))")
  public void serviceMethods(){}

  @Before("serviceMethods()")
  public void logBefore(JoinPoint jp) {
    System.out.println("Entering: " + jp.getSignature());
  }

  @AfterReturning(pointcut="serviceMethods()", returning="ret")
  public void logAfter(JoinPoint jp, Object ret) {
    System.out.println("Exiting: " + jp.getSignature() + ", returned=" + ret);
  }
}
```

---

## 6. Работа с данными и транзакции

### 6.1. JDBC-шаблон и ORM

* **JdbcTemplate** — упрощает работу с JDBC, автоматически закрывает ресурсы.
* **ORM-шаблоны** — интеграция с Hibernate, JPA, MyBatis.

### 6.2. Транзакции

* **Программные** через `PlatformTransactionManager` и `TransactionTemplate`.

* **Декларативные** через аннотацию `@Transactional`:

  ```java
  @Service
  public class PaymentService {
    @Transactional
    public void transfer(Account from, Account to, BigDecimal amt) {
      debit(from, amt);
      credit(to, amt);
    }
  }
  ```

* **Propagation**, **Isolation**, **ReadOnly**, **Timeout** — настраиваются в параметрах `@Transactional`.

---

## 7. Веб-слой (Spring MVC / WebFlux)

### 7.1. Spring MVC

* **DispatcherServlet** — “фронт-контроллер”, маршрутизирует запросы к контроллерам.

* **Controller** + **@RequestMapping**:

  ```java
  @Controller
  public class HomeController {
    @GetMapping("/home")
    public String home(Model model) {
      model.addAttribute("msg", "Hello, Spring!");
      return "home";  // имя View (home.jsp или Thymeleaf шаблон)
    }
  }
  ```

* **ViewResolver**, **HandlerInterceptor**, **DataBinder**, **MessageConverter** — расширяемые точки интеграции.

### 7.2. Spring WebFlux

* Реактивный стек на базе Project Reactor.
* **@RestController**, **Mono/Flux**, **WebClient**.

---

## 8. Spring Boot

Spring Boot строится поверх Spring и обеспечивает:

1. **Автоконфигурацию** — разбирает classpath и настройки, сам настраивает DataSource, MVC, Security, Actuator.
2. **Встроенный сервер** (Tomcat/Jetty/Undertow) — можно запускать как jar.
3. **Opinionated Defaults** — умолчания, подходящие “для большинства случаев”.
4. **Actuator** — эндпойнты для метрик, health-checks, логирования.
5. **Starter-зависимости** (`spring-boot-starter-web`, `-data-jpa`, `-security` и т.д.) — подтягивают нужные артефакты.

Пример `application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/db
spring.datasource.username=user
spring.datasource.password=pass
spring.jpa.hibernate.ddl-auto=update
server.port=8081
```