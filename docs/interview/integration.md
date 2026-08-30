# Kafka, Redis, REST и gRPC

## Kafka: модель данных

Cluster состоит из brokers. Topic делится на partitions, каждая partition — упорядоченный append-only log с offsets. Replicas повышают доступность; leader обслуживает чтение/запись, followers копируют данные.

Порядок гарантирован только внутри partition. Key обычно выбирает partition, поэтому события одной сущности получают один key. Больше partitions повышает parallelism, но увеличивает стоимость metadata, replication и rebalance.

## Producer

Ключевые настройки связаны:

- `acks` определяет подтверждение;
- retries повторяют временный сбой;
- idempotent producer предотвращает дубликаты из-за retry в пределах producer session и partition;
- batching/compression повышают throughput ценой небольшой задержки;
- delivery timeout ограничивает общее время попыток.

Смысл exactly-once ограничен границей Kafka transaction. Запись в внешнюю БД не становится атомарной с Kafka автоматически.

## Consumer groups и offsets

В одной group каждая partition назначается одному consumer. Число активно читающих consumers не превышает число partitions. Consumer вызывает `poll`, обрабатывает records и управляет commit offsets.

| Момент commit | Риск |
|---|---|
| До эффекта | сообщение можно потерять — at-most-once |
| После эффекта | эффект может повториться — at-least-once |

Практический выбор — at-least-once + идемпотентный consumer. Храните event ID/operation key рядом с бизнес-изменением или проектируйте естественно идемпотентную операцию.

Rebalance перераспределяет partitions при изменении group или таймаутах. Долгая обработка нарушает `max.poll.interval`; heartbeat и poll-настройки должны соответствовать workload. Cooperative strategy уменьшает остановку, но не отменяет необходимость корректного revoke/commit.

## Retention, compaction и schema evolution

Retention удаляет старые segments по времени/размеру независимо от того, прочитал ли их consumer. Log compaction сохраняет последнее значение для key и tombstones по правилам очистки; это не мгновенный map snapshot.

Schema должна иметь compatibility policy. Добавление optional поля обычно безопаснее удаления/переименования required поля. Schema Registry помогает проверять эволюцию Avro/Protobuf/JSON Schema.

## Retry и DLQ

Не повторяйте бесконечно в основном consumer loop. Разделите:

- transient ошибки — bounded retry с exponential backoff и jitter;
- permanent ошибки данных — quarantine/DLQ;
- сбой downstream — circuit breaker и контролируемая пауза;
- неизвестная ошибка — alert и достаточный диагностический контекст.

DLQ без владельца, метрик, replay-процедуры и исходного payload/headers лишь прячет потерю обработки.

## Redis

### Структуры

| Тип | Пример |
|---|---|
| String | cache value, counter |
| Hash | компактный набор полей объекта |
| List | простая последовательность/очередь с ограничениями |
| Set | membership, уникальность |
| Sorted Set | leaderboard, schedule по score |

TTL задаёт время жизни ключа. Expiration может быть lazy и active, поэтому истёкший ключ не обязан физически исчезнуть в ту же миллисекунду.

### Cache-aside

1. читать cache;
2. при miss читать source of truth;
3. записать cache с TTL;
4. при изменении данных инвалидировать или обновить cache.

Трудные места: stale data, cache stampede, hot keys, negative caching и согласование TTL. Защиты: jitter, single-flight/lock на загрузку, stale-while-revalidate и лимиты.

Eviction policy срабатывает при memory limit. Выберите LRU/LFU/random/TTL policy по access pattern и обязательно настройте `maxmemory`; иначе процесс может упереться в память раньше ожидаемой политики.

RDB даёт периодические snapshots, AOF журналирует writes с выбранной fsync policy. Replication может отставать. Redis Cluster распределяет hash slots; multi-key operation требует совместимости slots или изменения модели ключей.

Distributed lock требует уникального token, atomic acquire с TTL и release только владельцем. Для строгой защиты внешнего ресурса одной блокировки часто мало — fencing token позволяет ресурсу отвергнуть устаревшего владельца.

## REST

REST моделирует resources и использует HTTP semantics. Хороший контракт определяет:

- URI существительных, методы и status codes;
- validation и единый error format;
- idempotency повторяемых команд;
- pagination, filtering и sorting;
- optimistic concurrency (`ETag`/version), если есть конфликт изменений;
- backwards compatibility.

Offset pagination проста, но при больших offsets дорогая и нестабильна при вставках. Cursor/keyset pagination использует стабильный ordered key и лучше масштабируется, но сложнее для произвольного перехода на страницу.

Versioning — крайняя мера после совместимой эволюции. Версию можно выражать в URI/header/media type; важнее единая политика deprecation и срок миграции.

## gRPC

Protobuf задаёт строгую schema. Field numbers нельзя переиспользовать после удаления; удалённые номера/имена резервируйте. Новые поля должны иметь безопасные defaults.

Типы RPC: unary, server streaming, client streaming и bidirectional streaming. Streaming требует flow control, отмены и понимания границ сообщения.

Deadline должен идти от входящего запроса к downstream и уменьшаться по пути. Без deadline зависшая зависимость удерживает ресурсы неопределённо долго. Retry допустим только для подходящих status codes и идемпотентных операций.

Interceptors подходят для authentication, tracing, metrics и общей политики ошибок. Бизнес-ошибки переводите в стабильные gRPC status/details, не отдавая внутренний stack trace.

## REST или gRPC

| Критерий | REST/JSON | gRPC/Protobuf |
|---|---|---|
| Публичные/browser-клиенты | Проще | Требует дополнительной поддержки/gateway |
| Строгий контракт | Через OpenAPI | Встроен в `.proto` |
| Streaming | Ограниченнее | Нативный |
| Отладка вручную | Проще | Нужны инструменты |
| Внутренний low-latency RPC | Возможен | Часто удобнее |

Выбор не должен быть религиозным: важны клиенты, инфраструктура, эволюция контракта и операционная компетенция.

## Вопросы для самопроверки

1. Почему Kafka гарантирует порядок только в partition?
2. Где заканчивается exactly-once Kafka transaction?
3. Чем retention отличается от compaction?
4. Как cache stampede проявится в БД?
5. Почему deadline важнее локального timeout одного клиента?

