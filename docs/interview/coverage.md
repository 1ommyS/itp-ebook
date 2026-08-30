# Покрытие чек-листа

Excel-чек-лист содержит 255 тем в 21 содержательном разделе. Новая структура группирует их по связям, которые обычно проверяются на собеседовании.

| Раздел чек-листа | Тем | Основная глава |
|---|---:|---|
| Java Core | 16 | [Java Core](deep/java-core.md) |
| Collections | 12 | [Collections](deep/collections.md) |
| Generics & Functional | 12 | [Generics и функциональный стиль](deep/generics-functional.md) |
| JVM | 15 | [JVM](deep/jvm.md) |
| Concurrency | 17 | [Concurrency](deep/concurrency.md) |
| Algorithms | 12 | [Algorithms](deep/algorithms.md) |
| SQL & PostgreSQL | 22 | [SQL и PostgreSQL](deep/sql-postgresql.md) |
| Spring Core | 12 | [Spring Core](deep/spring-core.md) |
| Spring Boot | 10 | [Spring Boot](deep/spring-boot.md) |
| Spring Web | 10 | [Spring Web](deep/spring-web.md) |
| JPA & Hibernate | 12 | [JPA и Hibernate](deep/jpa-hibernate.md) |
| Spring Transactions & Security | 10 | [Transactions и Security](deep/transactions-security.md) |
| Kafka | 15 | [Kafka](deep/kafka.md) |
| Redis | 9 | [Redis](deep/redis.md) |
| gRPC & REST | 10 | [REST и gRPC](deep/grpc-rest.md) |
| Microservices | 13 | [Microservices](deep/microservices.md) |
| Testing | 7 | [Testing](deep/testing.md) |
| Docker & CI/CD | 8 | [Docker и CI/CD](deep/docker-cicd.md) |
| Observability | 8 | [Observability](deep/observability.md) |
| System Design | 13 | [System Design](deep/system-design.md) |
| Practical Java | 12 | [Practical Java](deep/practical-java.md) |
| **Итого** | **255** | |

## Что было добавлено

В прежней навигации почти отсутствовали Kafka, Redis, gRPC, микросервисные паттерны, тестовая стратегия, CI/CD, observability и system design. Эти блоки добавлены как самостоятельные главы. Существующие материалы по Java, SQL, Spring и Hibernate сохранены как углублённый справочник и практика.

## Глубина покрытия

Основные главы дают модель ответа, механизм, ограничения, пример и вопросы для самопроверки. Детали библиотечного API и длинные упражнения остаются в справочных главах. Если тема из чек-листа встречается в нескольких разделах — например idempotency, retry или transaction isolation — она объясняется в основном контексте и повторно применяется в интеграционном или архитектурном кейсе.

## Поэлементный трекер

Отмечайте пункт после устного ответа, небольшого примера и разбора хотя бы одного ограничения. Формулировки сохранены из исходного Excel-чек-листа.

### Java Core

- [ ] 4 принципа OOP — Encapsulation, inheritance, polymorphism, abstraction
- [ ] Composition vs inheritance — Когда предпочитать композицию
- [ ] SOLID — Все принципы и практические примеры
- [ ] equals/hashCode — Контракт и последствия нарушения
- [ ] immutability — Как сделать immutable class
- [ ] abstract class vs interface — Различия и use cases
- [ ] static — Статические поля, методы, блоки
- [ ] final — Класс, метод, переменная
- [ ] nested classes — Inner, static nested, anonymous
- [ ] checked vs unchecked — Когда использовать каждый тип
- [ ] exception hierarchy — Throwable, Error, Exception, RuntimeException
- [ ] try-with-resources — AutoCloseable и suppression
- [ ] custom exceptions — Проектирование собственных исключений
- [ ] String pool — Interning и immutable String
- [ ] StringBuilder vs StringBuffer — Отличия и thread safety
- [ ] pass-by-value — Как Java передаёт аргументы

### Collections

- [ ] List / Set / Map — Назначение и основные реализации
- [ ] внутреннее устройство — Массив, resize, complexity
- [ ] внутреннее устройство — Когда реально полезен
- [ ] внутреннее устройство — Buckets, hash, collision
- [ ] resize — Load factor и capacity
- [ ] treeification — Когда bucket превращается в дерево
- [ ] внутреннее устройство — Связь с HashMap
- [ ] внутреннее устройство — Red-black tree и ordering
- [ ] внутреннее устройство — Отличия от synchronized map
- [ ] iterator — Fail-fast и ConcurrentModificationException
- [ ] Comparable vs Comparator — Сортировка объектов
- [ ] Queue / Deque — Основные реализации

### Generics & Func

- [ ] type erasure — Что происходит с generic types
- [ ] wildcards — ? extends / ? super
- [ ] PECS — Producer Extends Consumer Super
- [ ] generic methods — Ограничения и inference
- [ ] functional interfaces — Function, Predicate, Consumer, Supplier
- [ ] lambda — Как работают lambda expressions
- [ ] stream pipeline — Intermediate и terminal operations
- [ ] map/filter/flatMap — Отличия и use cases
- [ ] collect — Collectors и grouping
- [ ] lazy evaluation — Почему intermediate operations lazy
- [ ] parallelStream — Когда использовать и почему осторожно
- [ ] Optional — Правильное использование и антипаттерны

### JVM

- [ ] JVM architecture — Class Loader, Runtime Data Areas, Execution Engine
- [ ] class loading — Loading, linking, initialization
- [ ] ClassLoader — Bootstrap, Platform, Application
- [ ] Heap — Young/Old generations
- [ ] Stack — Stack frames и локальные переменные
- [ ] Metaspace — Что хранится и где находится
- [ ] Garbage Collection — Как JVM определяет garbage
- [ ] G1 GC — Regions, pause targets
- [ ] ZGC — Основные особенности
- [ ] GC roots — Какие объекты являются roots
- [ ] JIT compiler — HotSpot, profiling, compilation
- [ ] warmup — Почему производительность меняется со временем
- [ ] escape analysis — Scalar replacement и stack allocation
- [ ] bytecode — Основы JVM instructions
- [ ] jstack / jmap / jcmd — Диагностика JVM

### Concurrency

- [ ] Thread lifecycle — Состояния потока
- [ ] Runnable vs Callable — Отличия
- [ ] ExecutorService — Thread pools
- [ ] Future / CompletableFuture — Асинхронные вычисления
- [ ] synchronized — Monitor и lock
- [ ] volatile — Visibility и ordering
- [ ] atomic classes — CAS и AtomicInteger
- [ ] ReentrantLock — Отличия от synchronized
- [ ] ReadWriteLock — Когда использовать
- [ ] deadlock — Причины и предотвращение
- [ ] race condition — Примеры и устранение
- [ ] livelock / starvation — Отличия
- [ ] Java Memory Model — Visibility, ordering, atomicity
- [ ] happens-before — Основные правила
- [ ] safe publication — Как безопасно публиковать объект
- [ ] virtual threads — Как работают и когда применять
- [ ] ForkJoinPool — Work stealing

### Algorithms

- [ ] Big O — Time и space complexity
- [ ] two pointers — Типовые задачи
- [ ] sliding window — Типовые задачи
- [ ] hash-based problems — Two Sum, frequency maps
- [ ] monotonic stack — Типовые задачи
- [ ] DFS / BFS — Обходы
- [ ] DFS / BFS — Поиск в графе
- [ ] Dijkstra — Кратчайший путь
- [ ] QuickSort / MergeSort — Complexity и особенности
- [ ] Binary Search — Варианты применения
- [ ] Heap / PriorityQueue — Top K и scheduling
- [ ] Trie — Поиск по префиксу

### SQL & PostgreSQL

- [ ] JOINs — INNER, LEFT, RIGHT, FULL
- [ ] GROUP BY / HAVING — Агрегации
- [ ] subqueries — Correlated и обычные
- [ ] CTE — WITH и recursive CTE
- [ ] window functions — ROW_NUMBER, RANK, LAG, LEAD
- [ ] NULL — 3-valued logic
- [ ] B-Tree — Устройство и use cases
- [ ] GIN — Полнотекстовый поиск и массивы
- [ ] BRIN — Когда выгоден
- [ ] composite index — Порядок колонок
- [ ] covering index — INCLUDE
- [ ] EXPLAIN — Как читать план
- [ ] EXPLAIN ANALYZE — Фактическое выполнение
- [ ] MVCC — Версии строк и visibility
- [ ] VACUUM — Почему нужен
- [ ] ACID — Atomicity, Consistency, Isolation, Durability
- [ ] isolation levels — Read Committed, Repeatable Read, Serializable
- [ ] locks — Row/table locks
- [ ] deadlocks — Как возникают в БД
- [ ] partitioning — Range, List, Hash
- [ ] replication — Основы streaming replication
- [ ] connection pool — HikariCP и ограничения

### Spring Core

- [ ] IoC / DI — Принцип инверсии управления
- [ ] Bean lifecycle — Создание, initialization, destruction
- [ ] Bean scopes — singleton, prototype, request и т.д.
- [ ] constructor injection — Почему предпочтительнее
- [ ] qualifier / primary — Разрешение неоднозначности
- [ ] Java config — @Configuration / @Bean
- [ ] component scanning — @ComponentScan
- [ ] AOP concepts — Aspect, Pointcut, Advice
- [ ] proxy — JDK dynamic proxy vs CGLIB
- [ ] self-invocation — Почему @Transactional может не сработать
- [ ] ApplicationEvent — Spring events
- [ ] profiles — Environment и @Profile

### Spring Boot

- [ ] auto-configuration — Как Spring Boot подбирает конфигурацию
- [ ] starters — Зачем нужны starters
- [ ] application.yml — Конфигурация приложения
- [ ] configuration properties — @ConfigurationProperties
- [ ] Actuator — Health, metrics, info
- [ ] embedded server — Tomcat / Jetty / Netty
- [ ] conditional beans — @Conditional
- [ ] custom starter — Как написать starter
- [ ] logging — SLF4J, Logback
- [ ] profiles — Разные environments

### Spring Web

- [ ] request lifecycle — Как HTTP request проходит через Spring MVC
- [ ] DispatcherServlet — Роль и устройство
- [ ] Controller — @RestController
- [ ] DTO — Почему не стоит отдавать entity
- [ ] validation — @Valid / Bean Validation
- [ ] exception handling — @ControllerAdvice
- [ ] filters — Servlet Filter
- [ ] interceptors — HandlerInterceptor
- [ ] HTTP — Методы, status codes, idempotency
- [ ] reactive — Mono / Flux и backpressure

### JPA & Hibernate

- [ ] Entity lifecycle — Transient, managed, detached, removed
- [ ] Persistence Context — First-level cache
- [ ] dirty checking — Как Hibernate видит изменения
- [ ] N+1 — Причины и способы решения
- [ ] fetch types — LAZY vs EAGER
- [ ] fetch join — Решение N+1
- [ ] cascade — Cascade types
- [ ] orphanRemoval — Поведение удалений
- [ ] optimistic locking — @Version
- [ ] pessimistic locking — Lock modes
- [ ] second-level cache — Когда применять
- [ ] JPQL — Основы

### Spring Tx & Security

- [ ] @Transactional — Как работает
- [ ] propagation — REQUIRED, REQUIRES_NEW и другие
- [ ] isolation — Связь со transaction isolation
- [ ] rollback — Когда Spring делает rollback
- [ ] readOnly — Что реально означает
- [ ] authentication vs authorization — Различия
- [ ] SecurityFilterChain — Как устроена цепочка
- [ ] JWT — Access/refresh tokens
- [ ] OAuth2 — Основы
- [ ] CSRF / CORS — Различия и защита

### Kafka

- [ ] architecture — Broker, topic, partition, replica
- [ ] partitioning — Как выбирается partition
- [ ] ordering — Гарантии порядка
- [ ] consumer groups — Как распределяются partitions
- [ ] offset — Commit и позиционирование
- [ ] delivery semantics — At-most-once, at-least-once, exactly-once
- [ ] rebalancing — Причины и последствия
- [ ] producer — acks, retries, idempotence
- [ ] consumer — poll, max.poll.interval, heartbeat
- [ ] retention — Как Kafka хранит сообщения
- [ ] compaction — Log compaction
- [ ] transactions — Kafka transactions
- [ ] DLQ — Dead Letter Queue
- [ ] schema evolution — Avro / Protobuf / Schema Registry
- [ ] ordering vs scalability — Trade-offs

### Redis

- [ ] data structures — String, Hash, List, Set, Sorted Set
- [ ] TTL — Expiration
- [ ] cache-aside — Основной паттерн
- [ ] cache invalidation — Проблемы и решения
- [ ] distributed lock — Основы
- [ ] persistence — RDB / AOF
- [ ] replication — Primary / Replica
- [ ] cluster — Sharding
- [ ] eviction — LRU, LFU и policies

### gRPC & REST

- [ ] REST principles — Ресурсы и HTTP semantics
- [ ] idempotency — GET, PUT, DELETE, POST
- [ ] pagination — Offset vs cursor
- [ ] versioning — API versioning
- [ ] protobuf — Основы schema
- [ ] Unary / Streaming — Типы RPC
- [ ] deadlines — Timeout propagation
- [ ] interceptors — Middleware
- [ ] error handling — Status codes
- [ ] REST vs gRPC — Когда что выбрать

### Microservices

- [ ] service boundaries — Как разделять сервисы
- [ ] sync vs async — Когда использовать
- [ ] Saga — Distributed transactions
- [ ] Outbox — Надёжная публикация событий
- [ ] Inbox — Обработка сообщений
- [ ] Circuit Breaker — Защита зависимостей
- [ ] Retry — Правильный retry
- [ ] Timeout — Timeout propagation
- [ ] Bulkhead — Изоляция ресурсов
- [ ] Idempotency — Idempotency keys
- [ ] eventual consistency — Когда допустима
- [ ] CAP theorem — Trade-offs
- [ ] distributed locks — Проблемы

### Testing

- [ ] JUnit 5 — Основы
- [ ] Mockito — Mock, spy, verify
- [ ] test pyramid — Unit / integration / e2e
- [ ] Spring Boot Test — @SpringBootTest
- [ ] Testcontainers — Тестирование реальных зависимостей
- [ ] contract testing — Consumer/provider contracts
- [ ] test isolation — Независимость тестов

### Docker & CI-CD

- [ ] image vs container — Различия
- [ ] Dockerfile — Основные инструкции
- [ ] layers — Кэширование слоёв
- [ ] networking — Container networking
- [ ] volumes — Persistence
- [ ] pipeline — Build, test, deploy
- [ ] blue-green — Стратегия деплоя
- [ ] canary — Стратегия деплоя

### Observability

- [ ] Prometheus — Counters, gauges, histograms
- [ ] percentiles — p50, p95, p99
- [ ] distributed tracing — Trace / span
- [ ] OpenTelemetry — Инструментирование
- [ ] structured logging — JSON и correlation ID
- [ ] SLI / SLO / SLA — Различия
- [ ] error budget — Что это и зачем
- [ ] timeouts / retries — Влияние на latency

### System Design

- [ ] requirements — Functional / non-functional
- [ ] capacity estimation — RPS, storage, bandwidth
- [ ] horizontal vs vertical — Trade-offs
- [ ] cache strategies — Cache-aside, write-through
- [ ] load balancers — L4 / L7
- [ ] SQL vs NoSQL — Как выбирать
- [ ] queue vs log — Kafka vs traditional queue
- [ ] strong vs eventual — Trade-offs
- [ ] replication — Failover
- [ ] sharding — Распределение данных
- [ ] rate limiting — Token bucket / leaky bucket
- [ ] API Gateway — Зачем нужен
- [ ] service discovery — Как сервисы находят друг друга

### Practical Java

- [ ] LRU Cache — Реализовать через HashMap + linked structure
- [ ] Thread-safe counter — Несколько вариантов
- [ ] Producer / Consumer — BlockingQueue
- [ ] Rate Limiter — Token bucket
- [ ] TTL Cache — Кэш с expiration
- [ ] Concurrent data structure — Спроектировать потокобезопасную структуру
- [ ] REST endpoint — DTO, validation, error handling
- [ ] Kafka consumer — Retry, DLQ, idempotency
- [ ] SQL query — JOIN + aggregation + window function
- [ ] URL shortener — API, storage, cache, scaling
- [ ] notification service — Kafka, retries, idempotency
- [ ] order service — Transactions, outbox, events
