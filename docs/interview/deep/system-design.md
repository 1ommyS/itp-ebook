# System Design — подробный конспект

## Requirements

Functional use cases ранжируют: create/read/search/notify. Non-functional измеримы: peak RPS, p95 latency, availability, RPO/RTO, consistency, retention, regions, compliance.

Фиксируйте assumptions. Если интервьюер не дал числа, выберите разумный порядок и покажите формулу.

## Capacity

Считайте average и peak, read/write split, payload, replication, indexes, growth и retention. Concurrency по Little's Law: in-flight ≈ throughput × latency.

1000 RPS × 0.2 s = около 200 concurrent requests. Это связывает pool/connection capacity с latency.

## Architecture baseline

Client → edge/LB → stateless service → primary storage. Затем добавляйте cache, queue, replicas, shards только для конкретного требования/bottleneck.

Для каждого компонента назовите state, replication, failover, backpressure и observability.

## Load balancing/discovery

L4 дешевле и transport-level, L7 понимает routes/TLS/headers. Алгоритм учитывает long connections и unequal instances. Discovery registry требует health/lease; stale endpoint обрабатывается timeout/retry.

## Storage

Access pattern определяет schema/key/index. SQL для constraints/transactions/joins, key-value для key access, document для aggregate, search для inverted queries, object store для blobs.

Polyglot persistence добавляет dual writes; нужен source of truth и sync path (outbox/CDC).

## Scaling data

Replica увеличивает reads и availability, но lag. Shard key определяет routing и hotspot. Resharding/consistent hashing перемещают data, но global query/unique/transaction усложняются.

## Cache/messaging/consistency

Cache design включает invalidation и stampede. Queue design — delivery, order, retention, DLQ, lag. Eventual consistency — user state, version/conflict/reconciliation.

## Rate limit/gateway

Token bucket хранит tokens/refill, distributed implementation требует atomic state и clock. Gateway выполняет edge policy, но бизнес authorization остаётся сервису.

## Failure analysis

Для каждого remote call: timeout, retryable errors, idempotency, circuit/bulkhead. Для data: replica loss, backup restore, corruption. Для overload: bounded queue, shed low priority, degraded response.

## Design cases

URL shortener: code generation/collision, redirect cache, analytics async, abuse/expiry.

Notification: durable intent, per-channel workers, preference/rate limit, provider callback, retry/DLQ/status.

Order: state machine, local transaction/outbox, saga payment/inventory, idempotency, compensation/manual repair.

## Критерий готовности

Вы готовы, если за 40 минут проходите requirements→numbers→API/data→baseline→bottleneck→failure, а каждое добавление обосновываете trade-off.

