# Spring Core, Boot и Web

## IoC и DI

IoC означает, что контейнер управляет созданием объектов и их связями. Dependency Injection — способ передать объекту зависимости снаружи.

Constructor injection предпочтителен: зависимости обязательны и видны, объект можно создать в unit test, поля остаются `final`, а circular dependency обнаруживается раньше.

```java
@Service
final class OrderService {
    private final OrderRepository repository;
    private final PaymentGateway paymentGateway;

    OrderService(OrderRepository repository, PaymentGateway paymentGateway) {
        this.repository = repository;
        this.paymentGateway = paymentGateway;
    }
}
```

Если кандидатов одного типа несколько, используйте `@Qualifier`; `@Primary` задаёт default. Лучше, чтобы имена qualifier описывали роль, а не случайное имя реализации.

## Создание bean

- component scanning находит stereotype-аннотации;
- `@Configuration` + `@Bean` описывают явную сборку, особенно для сторонних типов;
- `ApplicationContext` хранит bean definitions, создаёт non-lazy singletons и запускает post-processors;
- `@Profile` и environment properties выбирают конфигурацию, но не должны размазывать бизнес-ветвления по коду.

### Lifecycle

Упрощённая последовательность:

1. instantiate;
2. dependency population;
3. aware callbacks;
4. `BeanPostProcessor` before initialization;
5. `@PostConstruct`/initialization callback;
6. `BeanPostProcessor` after initialization — здесь часто появляется proxy;
7. использование;
8. `@PreDestroy`/destroy callback при закрытии context.

Не выполняйте тяжёлый сетевой I/O в конструкторе: объект ещё не полностью обработан контейнером, а startup становится хрупким.

## Scopes

Singleton в Spring означает один bean на `ApplicationContext`, а не один объект на JVM. Prototype создаётся при каждом запросе из container, но контейнер не управляет его полным destruction lifecycle. Web scopes (`request`, `session`) требуют web context.

Внедрение короткоживущего bean в singleton требует provider/proxy, иначе dependency будет разрешена один раз при создании singleton.

## AOP и proxy

Aspect содержит cross-cutting behavior. Pointcut выбирает join points, advice выполняется до/после/вокруг вызова. Spring AOP обычно перехватывает вызов метода через proxy:

- JDK dynamic proxy реализует интерфейсы;
- class-based proxy создаёт подкласс.

Вызов `this.otherMethod()` не проходит через внешний proxy, поэтому `@Transactional`, `@Async` или cache advice на `otherMethod` может не сработать. Решения: вынести границу в другой bean, изменить orchestration или использовать подходящий weaving — не внедрять self без понимания цены.

## Events

`ApplicationEventPublisher` развязывает участников внутри приложения. По умолчанию listener выполняется синхронно в потоке publisher; это не durable message broker. Для бизнес-события, которое должно пережить падение процесса, нужен transactional outbox и внешняя доставка.

## Spring Boot

### Auto-configuration

Boot добавляет конфигурации по условиям: наличие классов, beans, properties и тип приложения. Пользовательский bean часто заставляет auto-configuration «отступить». Для диагностики смотрите condition evaluation report, а не угадывайте.

Starter — согласованный набор dependencies. Auto-configuration — код, создающий beans. Это связанные, но разные понятия.

### Конфигурация

`@ConfigurationProperties` группирует типизированные настройки и поддерживает validation:

```java
@ConfigurationProperties("payment")
@Validated
public record PaymentProperties(
        @NotNull URI baseUrl,
        @NotNull Duration timeout
) {}
```

Не храните секреты в repository и не логируйте environment целиком. Внешний secret store или platform secret передаёт значение приложению, а rotation проектируется отдельно.

Profiles удобны для групп конфигурации, но большое число условных профилей делает startup непредсказуемым. Старайтесь сохранять один artifact и менять externalized configuration.

### Actuator, logging и server

Actuator предоставляет health, metrics и диагностические endpoints. Liveness отвечает «процесс нужно перезапустить?», readiness — «можно ли направлять трафик?». Не делайте liveness зависимой от временно недоступного downstream, иначе получится restart storm.

Используйте SLF4J API и структурированные поля: request/trace ID, operation, result, duration. Не записывайте токены, пароли и персональные данные.

Embedded server является частью процесса: servlet stack обычно использует Tomcat/Jetty, reactive stack — Netty. Выбор модели зависит от библиотек и характера нагрузки, а не от моды.

### Conditional beans и custom starter

Собственный starter уместен для повторяемой инфраструктурной интеграции. В auto-configuration:

- используйте namespace properties;
- применяйте conditions;
- отступайте при пользовательском bean;
- не включайте неожиданное поведение без явной настройки;
- добавьте metadata и integration tests.

## Spring MVC request lifecycle

Упрощённый путь:

1. server принимает HTTP;
2. servlet filters оборачивают запрос;
3. `DispatcherServlet` ищет handler через `HandlerMapping`;
4. interceptors выполняют `preHandle`;
5. argument resolvers создают параметры и запускают validation;
6. controller вызывает application service;
7. return value handler и message converter формируют body;
8. exception resolvers/`@ControllerAdvice` преобразуют ошибку;
9. interceptor completion и filters завершают обработку.

Filter работает на servlet-уровне и подходит для общих protocol concerns. `HandlerInterceptor` знает выбранный Spring handler. AOP работает вокруг вызовов bean и не заменяет HTTP filter.

## Контракт REST endpoint

Не возвращайте JPA entity напрямую: контракт API начинает зависеть от persistence model, lazy loading и bidirectional links. Используйте DTO и явное mapping.

```java
@PostMapping("/orders")
ResponseEntity<OrderResponse> create(@Valid @RequestBody CreateOrderRequest request) {
    OrderResponse created = service.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

Bean Validation проверяет форму команды на границе. Бизнес-инвариант, зависящий от состояния системы, остаётся в domain/application service и защищается транзакцией/constraint.

Единый error response обычно содержит machine-readable code, безопасное message, field errors, correlation ID и timestamp. Не отдавайте stack trace клиенту.

## HTTP semantics

- GET безопасен и идемпотентен;
- PUT и DELETE должны быть идемпотентны по контракту;
- POST обычно не идемпотентен, но команда может поддержать idempotency key;
- 400 — форма запроса некорректна, 401 — не аутентифицирован, 403 — недостаточно прав, 404 — ресурс не найден, 409 — конфликт состояния, 422 — семантически недопустимая команда, если такой контракт принят.

## WebFlux

`Mono` моделирует 0..1 элемент, `Flux` — 0..N и backpressure. Reactive stack полезен для большого числа concurrent I/O при полностью неблокирующей цепочке. Блокирующий JDBC-вызов внутри event loop уничтожает преимущество; его нужно изолировать или выбрать servlet/virtual-thread model.

## Вопросы для самопроверки

1. Почему constructor injection улучшает дизайн, а не только тесты?
2. На каком этапе lifecycle появляется proxy?
3. Почему self-invocation обходит advice?
4. Чем filter отличается от interceptor?
5. Когда reactive controller не даёт выигрыша?
