
## 1. RestTemplate

### 1.1. Что это такое

* **RestTemplate** — синхронный HTTP-клиент от Spring (до Spring 5 включительно), реализующий паттерн **Template Method**.
* Позволяет делать GET/POST/PUT/DELETE-запросы, автоматически (де)сериализовать JSON/XML в Java-объекты через `HttpMessageConverter`.

> **Важно:** начиная с Spring 5 рекомендуется использовать **WebClient** (из Spring WebFlux), но многие проекты до сих пор активно используют RestTemplate.

### 1.2. Подключение и конфигурация

В Spring Boot достаточно добавить зависимость:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Затем объявить бин:

```java
@Configuration
public class RestConfig {
  @Bean
  public RestTemplate restTemplate(RestTemplateBuilder builder) {
    return builder
      .setConnectTimeout(Duration.ofSeconds(5))
      .setReadTimeout(Duration.ofSeconds(5))
      .errorHandler(new DefaultResponseErrorHandler()) // можно свой
      .build();
  }
}
```

* **RestTemplateBuilder** — удобный способ централизованно задавать таймауты, базовый URL, MessageConverter’ы, интерсепторы.

### 1.3. Основные методы

```java
@Autowired
private RestTemplate rest;

public void example() {
  // GET → объект
  User user = rest.getForObject("https://api.example.com/users/{id}", User.class, 42);

  // GET → ResponseEntity (с доступом к статусу и заголовкам)
  ResponseEntity<User> resp = rest.getForEntity("/users/{id}", User.class, 42);
  HttpStatus status = resp.getStatusCode();
  User body = resp.getBody();

  // POST → объект
  User newUser = new User("Ivan", "ivan@example.com");
  User created = rest.postForObject("/users", newUser, User.class);

  // Обмен произвольными HTTP-методами
  HttpHeaders headers = new HttpHeaders();
  headers.setContentType(MediaType.APPLICATION_JSON);
  HttpEntity<User> request = new HttpEntity<>(newUser, headers);
  ResponseEntity<Void> result = rest.exchange(
    "/users/{id}", HttpMethod.PUT, request, Void.class, 42);

  // DELETE
  rest.delete("/users/{id}", 42);
}
```

### 1.4. Кастомизация

1. **Интерсепторы** (`ClientHttpRequestInterceptor`) — для логирования, добавления заголовков:

   ```java
   rest.getInterceptors().add((req, body, exec) -> {
     req.getHeaders().add("X-Request-ID", UUID.randomUUID().toString());
     return exec.execute(req, body);
   });
   ```
2. **Свой обработчик ошибок** (`ResponseErrorHandler`) — чтобы перехватывать 4xx/5xx и бросать свои исключения.
3. **MessageConverters** — по умолчанию Jackson для JSON, но можно добавить XML, protobuf и т.д.

### 1.5. Плюсы и минусы

| Плюсы                                            | Минусы                                             |
| ------------------------------------------------ | -------------------------------------------------- |
| Простота и прозрачность вызовов                  | Синхронный, блокирует поток                        |
| Нет дополнительных зависимостей                  | Шаблон менее декларативен, чем Feign               |
| Гибкая кастомизация через Builder и интерсепторы | При большом числе API-методов много кода «обвязки» |

---

## 2. OpenFeign

### 2.1. Что это такое

* **OpenFeign** — декларативный HTTP-клиент, интегрированный в Spring Cloud.
* Позволяет описывать REST-клиенты как Java-интерфейсы с аннотацией `@FeignClient`, а Spring генерирует реализацию «на лету».

### 2.2. Подключение и настройка

1. Зависимости (например, для Spring Boot + Spring Cloud):

   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```
2. Включить Feign-клиенты:

   ```java
   @SpringBootApplication
   @EnableFeignClients(basePackages = "com.example.clients")
   public class Application { public static void main(String[] args) {
     SpringApplication.run(Application.class, args);
   }}
   ```

### 2.3. Объявление клиента

```java
@FeignClient(
  name = "user-service",      // логическое имя сервиса
  url = "${user.service.url}",// можно брать из application.properties
  configuration = UserClientConfig.class,
  fallback = UserClientFallback.class
)
public interface UserClient {

  @GetMapping("/users/{id}")
  UserDto getUser(@PathVariable("id") Long id);

  @PostMapping("/users")
  UserDto createUser(@RequestBody CreateUserRequest req);

  @GetMapping("/users")
  List<UserDto> findByStatus(@RequestParam("status") String status);
}
```

* **name** — сервисное имя (используется также для интеграции с Eureka/Consul).
* **configuration** — класс, определяющий отдельные настройки (кодеки, логгер, retry).
* **fallback / fallbackFactory** — для Hystrix-стиля отказоустойчивости.

### 2.4. Конфигурация клиента

```java
@Configuration
public class UserClientConfig {
  @Bean
  public Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;  // BASIC / HEADERS / FULL
  }
  @Bean
  public ErrorDecoder errorDecoder() {
    return (methodKey, response) -> {
      if (response.status() == 404) {
        return new UserNotFoundException("User not found");
      }
      return new Default();
    };
  }
  @Bean
  public Request.Options options() {
    return new Request.Options(5000, 10000); // connectTimeoutMillis, readTimeoutMillis
  }
}
```

### 2.5. Fallback-механизм

```java
@Component
public class UserClientFallback implements UserClient {
  @Override
  public UserDto getUser(Long id) {
    return new UserDto(id, "Unknown", "unknown@example.com");
  }
  // ...
}
```

### 2.6. Расширенные возможности

* **Interceptors** (`RequestInterceptor`) — добавление заголовков (например, токен авторизации).
* **Decoders/Encoders** — поддержка нестандартных форматов (XML, protobuf).
* **Contract** — можно определять свои правила аннотаций вместо Spring MVC.

### 2.7. Плюсы и минусы

| Плюсы                                        | Минусы                                                  |
| -------------------------------------------- | ------------------------------------------------------- |
| Декларативный, мало «обвязки» кода           | Зависит от Spring Cloud, дополнительный слой абстракции |
| Автоматический биндинг запросов/ответов      | Сложнее отлаживать (генерация прокси)                   |
| Встроенная поддержка Circuit Breaker и Retry | Менее гибок в кастомных сценариях, чем RestTemplate     |

---

## 3. Когда что использовать

1. **RestTemplate**

    * Простые проекты без Spring Cloud
    * Не требуется декларативных клиентов и отказоустойчивости «из коробки»
    * Нужно полное управление HTTP-вызовами

2. **OpenFeign**

    * Микросервисная архитектура с Spring Cloud (Eureka, Config, Circuit Breaker)
    * Много REST-методов — декларативный стиль снижает бойлерплейт
    * Нужны fallback/Retry/Request tracing (Sleuth)

> **Совет:** если вы уже используете Spring Boot без Cloud, выбирайте **RestTemplate** (или лучше — **WebClient**). Если же строите распределённую систему с сервис-дискавери и паттернами Resilience, **OpenFeign** даёт более «сквозной» и декларативный опыт.
