# Дополнительные задания по новым темам

Задания делятся на три уровня: **A** — закрепить механизм, **B** — собрать рабочий компонент, **C** — учесть сбои и эксплуатацию. Для зачёта недостаточно кода: приложите короткое решение, тесты и объяснение trade-offs.

## JVM и concurrency

### A1. Class loading

Создайте два `URLClassLoader`, загрузите ими один class и докажите, что типы несовместимы. Выведите class loader каждого объекта. Объясните identity `(name, loader)`.

### A2. Диагностика памяти

Напишите приложение с cache без eviction и controlled allocation. Снимите JFR/heap dump, найдите dominator и retained path. Критерий: причина подтверждена данными, а не только графиком heap.

### B1. Bounded executor

Реализуйте сервис с ограниченным pool/queue и rejection policy. Нагрузите producer быстрее consumer. Добавьте метрики queue depth, wait time и rejected. Сравните unbounded и bounded варианты.

### B2. Safe publication

Покажите четыре способа безопасно опубликовать immutable config snapshot: static initializer, volatile reference, synchronized и `AtomicReference`. Напишите stress test обновления/чтения.

### C1. Deadlock laboratory

Создайте deadlock двух locks, снимите thread dump, найдите cycle. Исправьте общим order и вариантом `tryLock` с deadline. Объясните плюсы/минусы обоих исправлений.

### C2. Virtual threads

Сравните fixed platform pool и virtual-thread-per-task для 10 000 имитированных blocking операций. Затем ограничьте downstream semaphore до 50. Измерьте throughput, latency и количество активных соединений.

## Алгоритмы

### A1. Набор паттернов

Решите: two sum в sorted array, longest substring без повторов, next greater element, level order tree, shortest path unweighted graph. Для каждой задачи сформулируйте invariant до кода.

### B1. Dijkstra

Реализуйте adjacency list + priority queue с пропуском stale entries. Тесты: disconnected graph, multiple paths, zero weight, один vertex. Отдельно покажите контрпример с negative edge.

### B2. LRU

Реализуйте LRU без `LinkedHashMap`. Автотесты должны проверять capacity 0/1, update, recency и eviction. Добавьте invariant checker связности списка и согласованности map.

### C1. Rate limiter

Реализуйте token bucket с инъекцией monotonic clock. Тесты без sleep: refill, burst, long pause, concurrent acquire. Опишите distributed вариант и проблему clock/atomicity.

## SQL и PostgreSQL

### A1. Window functions

На таблице заказов выведите последний заказ, running total, разницу с предыдущим и top-3 по клиенту. Учитывайте одинаковый timestamp через ID tie-breaker.

### A2. NULL и outer join

Составьте набор данных, на котором `LEFT JOIN` + filter в WHERE теряет строку, а `NOT IN` с NULL возвращает неожиданный результат. Исправьте запросы и объясните 3-valued logic.

### B1. Индекс по workload

Сгенерируйте 1 млн orders. Для трёх запросов предложите composite/partial/INCLUDE indexes. Снимите `EXPLAIN (ANALYZE, BUFFERS)`, сравните estimates/actual, размер индекса и стоимость INSERT.

### B2. MVCC

В двух sessions воспроизведите Read Committed и Repeatable Read, row lock wait и deadlock. Найдите blocked query в системных views. Объясните, почему долгий `idle in transaction` мешает vacuum.

### C1. Worker queue

Реализуйте выдачу jobs через `FOR UPDATE SKIP LOCKED` нескольким workers. Добавьте lease/attempts, recovery зависшего job и idempotent result. Докажите отсутствие двойного terminal effect.

## Spring, JPA и Security

### A1. Lifecycle tracer

Создайте BeanPostProcessor и bean с constructor, aware, post-construct/pre-destroy. Запишите порядок. Затем добавьте transactional proxy и сравните class target/proxy.

### A2. Self-invocation

Воспроизведите неработающий `@Transactional(REQUIRES_NEW)` при `this.method()`. Напишите integration test, затем вынесите boundary в другой bean и докажите двумя transactions.

### B1. REST endpoint

Сделайте `POST /orders`: DTO, nested validation, application service, unique idempotency key, единый error contract, 201/409. Тесты MockMvc + PostgreSQL Testcontainer.

### B2. N+1 laboratory

Создайте Order→Items и endpoint списка. Зафиксируйте число SQL, воспроизведите N+1, исправьте projection/entity graph/two-step pagination. Сравните строки результата и memory.

### B3. Optimistic locking

Два concurrent запроса меняют один aggregate. Получите conflict `@Version`, верните 409 и реализуйте безопасный retry только для машинной команды. Для пользовательского редактирования покажите diff/current version.

### C1. Security

Настройте session или JWT resource server: authentication, object-level authorization, 401/403, CORS allowlist. Добавьте негативные тесты: чужой order, expired token, wrong audience, CSRF для cookie-сценария.

## Kafka

### A1. Partitioning

Опубликуйте события 100 orders с key orderId в topic с несколькими partitions. Докажите порядок внутри order и отсутствие глобального порядка. Измерьте распределение и смоделируйте hot key.

### A2. Rebalance

Запустите consumers одной group, добавляйте/убирайте instances. Логируйте assigned/revoked partitions и lag. Затем сделайте обработку дольше `max.poll.interval` и объясните повтор.

### B1. Idempotent consumer

Consumer пишет payment result в PostgreSQL и inbox event ID одной transaction, затем commit offset. Убейте процесс после DB commit до Kafka commit. После restart бизнес-эффект должен остаться один.

### B2. Retry/DLQ

Разделите transient и permanent errors. Создайте retry topics с backoff и DLQ с original metadata. Напишите replay tool, который повторяет выбранные events после исправления.

### C1. Outbox

Order + outbox в одной DB transaction, relay polling/CDC, Kafka producer. Смоделируйте crash до/после publish и duplicate. Consumer должен корректно обработать повтор.

### C2. Schema evolution

Создайте protobuf/Avro versions: additive optional field, rename/removal, enum extension. Проверьте backward/forward compatibility и порядок deploy producer/consumer.

## Redis

### A1. Cache-aside

Кэшируйте product с TTL+jitter и negative cache. Метрики hit/miss/load latency. Покажите stale window при update/invalidate.

### B1. Stampede

100 concurrent запросов после expiry не должны сделать 100 DB calls. Реализуйте single-flight или stale-while-revalidate. Тест должен считать фактические loads.

### B2. Eviction/persistence

Ограничьте maxmemory, сравните LRU/LFU на skewed access. Затем RDB/AOF everysec: перезапуск, restore time, допустимая потеря. Зафиксируйте RSS и logical memory.

### C1. Lock и fencing

Реализуйте acquire `SET NX PX` с unique token и Lua release. Смоделируйте pause дольше TTL и покажите, почему lock без fencing допускает stale writer. Добавьте monotonically increasing fencing token в resource.

## REST и gRPC

### A1. Cursor pagination

API сортирует `(createdAt DESC, id DESC)`. Cursor opaque, содержит оба значения. Между страницами вставьте новые rows и сравните с offset pagination.

### A2. Protobuf evolution

Создайте v1/v2 message, зарезервируйте удалённые номера, добавьте presence и `oneof`. Старый client должен читать новый response без падения.

### B1. Deadline propagation

Service A вызывает B→C. Передавайте remaining deadline; C задерживается. Проверьте cancellation и отсутствие работы после отмены. Метрики по hop и end-to-end.

### C1. Dual API

Один use case доступен REST и gRPC. Согласуйте validation, idempotency, authentication и mapping errors. Contract tests должны доказывать одинаковую бизнес-семантику.

## Microservices

### A1. Failure matrix

Для order/payment/inventory составьте таблицу: точка сбоя, наблюдаемый outcome, retry, idempotency, compensation, alert, manual repair.

### B1. Saga

Реализуйте orchestration state machine. Сценарии: inventory reject, payment timeout с поздним success, duplicate event, compensation failure. State transitions защищены version.

### B2. Resilience

Добавьте end-to-end deadline, timeout, retry+jitter, circuit breaker и bulkhead. Нагрузочным тестом покажите retry amplification и исправьте retry budget.

### C1. Reconciliation

Создайте periodic job, сравнивающий order и payment states, и безопасные repair commands. Любое исправление идемпотентно и аудируется.

## Testing, Docker и CI/CD

### A1. Test pyramid audit

Для существующего сервиса классифицируйте тесты и риски. Удалите один избыточный E2E, заменив unit+contract/integration с более быстрым feedback, не теряя проверку риска.

### B1. Production image

Multi-stage, non-root, read-only filesystem где возможно, graceful signals, health, SBOM/scan. Сравните размер и CVE с исходным image.

### B2. Pipeline

Build one image digest, integration/contract tests, scan/sign, deploy staging, smoke, promote same digest. Добавьте cache без зависимости результата от cache.

### C1. Canary

Разверните canary 5→25→100%. Автоматическое продвижение по error rate и p95, rollback по burn rate. Миграция БД по expand-contract совместима с обеими версиями.

## Observability

### A1. RED dashboard

Для HTTP и Kafka consumer: rate, errors, duration/lag. Уберите high-cardinality labels. Добавьте ссылки из alert в dashboard/runbook.

### B1. Distributed trace

HTTP→DB→Kafka producer→consumer→HTTP provider. Передайте context через headers, используйте links для messaging. По trace найдите самый медленный hop.

### B2. SLO

Определите SLI good/valid requests, SLO и 30-дневный budget. Реализуйте multi-window burn-rate alerts. Смоделируйте краткий сильный и долгий слабый сбой.

### C1. Incident drill

Сценарий: p99 вырос, average нормален, consumer lag растёт, CPU умеренный. Проведите диагностику metrics→trace→logs→thread/DB pool, оформите timeline и corrective actions.

## System design

### B1. URL shortener

Обязательны: расчёт RPS/storage, API, code generation, cache, DB key, analytics, expiry, abuse, replication/failover, SLI.

### B2. Notification service

Channels, preferences, templates, schedule, idempotency, provider quotas, retry/DLQ, callback status, GDPR/PII, operational dashboard.

### C1. Order platform

State machine, payment/inventory saga, outbox/inbox, read model, reconciliation, peak estimates, shard/partition keys, RPO/RTO и degraded mode.

### C2. Защита решения

Для каждого проекта запишите 20-минутное объяснение. В нём должны прозвучать assumptions, два альтернативных решения, bottleneck, три failure modes и причины выбранных trade-offs.

## Формат сдачи

Для каждого задания приложите:

1. README с контрактом и assumptions;
2. diagram для распределённых задач;
3. запускаемые tests;
4. команды локального запуска;
5. evidence: plan, metrics, trace, thread dump или benchmark;
6. раздел «что сломается первым» и план улучшения.

