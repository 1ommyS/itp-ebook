# Practical Java — подробный конспект

## Как оценивать практическую задачу

Сначала contract и edge cases, затем простой correct вариант, потом complexity/production requirements. Не начинайте concurrency/distribution до однопоточного invariant.

## LRU

Map даёт key→node, doubly linked list — recency. Head most recent, tail least. Get: найти, detach, move front. Put: update/move или add; при overflow remove tail из list и map атомарно.

Thread-safe вариант lock вокруг обеих structures. ConcurrentHashMap отдельно не защищает общий invariant.

## Counter и concurrent structure

Выберите linearizability requirements. AtomicLong для точного get/increment, LongAdder для high-contention metrics, lock для multi-field invariant. Опишите memory visibility и snapshot.

## Producer/consumer

Bounded BlockingQueue, cancellation/poison pill, interruption, multiple producers termination. Метрики: depth, oldest age, throughput, rejection.

## Rate limiter/TTL cache

Rate limiter: monotonic clock, refill arithmetic, burst, contention, distributed atomicity. TTL cache: entry expiry, lazy/background cleanup, max size, stampede, clock abstraction tests.

## REST endpoint

DTO/validation/domain transaction/error contract/idempotency/authorization. Tests: invalid, duplicate, concurrent, downstream timeout, retry.

## Kafka consumer

Manual commit after DB inbox/effect, bounded retry, DLQ metadata/replay, order by aggregate version, lag alerts.

## SQL

Join+aggregation+window; deterministic ordering, NULL, index, EXPLAIN. Проверяйте duplicates от joins и tie rows.

## Design cases

URL shortener, notification, order — обязательны capacity, data model, failure modes и SLI. Diagram без numbers/recovery считается незавершённым.

## Критерий готовности

Вы готовы, если пишете ключевой код за 25 минут, тестируете границы и ещё за 5 минут объясняете production extension без разрушения базового решения.

