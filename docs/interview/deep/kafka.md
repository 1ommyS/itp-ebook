# Kafka — подробный конспект

## Log и partition

Topic partition — последовательность records с monotonically increasing offset. Offset не глобален и не идентификатор сообщения. Record состоит из key, value, headers, timestamp.

Partition — единица порядка, replication и consumer parallelism. Keying по aggregate ID сохраняет порядок событий aggregate, но hot key ограничивает throughput одной partition.

## Brokers и replication

Leader принимает produce/fetch. Replicas копируют log. In-sync replica достаточно близка к leader по правилам cluster; ISR влияет на подтверждение `acks=all` и durability.

Replication factor 3 не означает нулевую потерю: важны `min.insync.replicas`, unclean leader election, producer acks и disk/region failures.

## Producer batching

Producer сериализует, выбирает partition, накапливает batch per partition, compresses и отправляет. `linger.ms` даёт время собрать batch; `batch.size` ограничивает. Compression часто снижает network/storage и CPU broker ценой CPU producer/consumer.

`acks=0/1/all` меняет latency/durability. Retries обязательны для transient errors, но deadline ограничивает их. Idempotent producer использует producer ID и sequence numbers, предотвращая duplicate/out-of-order при retry в session.

## Ordering

Порядок гарантируется в partition. Несколько producers не создают бизнес-порядок без version/sequence. Retry при неправильных in-flight/idempotence settings может переупорядочить records.

Consumer должен обрабатывать order-aware: параллелизм внутри partition требует partitioning по key или sequencing buffer.

## Consumer loop

`poll` одновременно получает records и поддерживает group protocol. Consumer не thread-safe. Обычно один consumer принадлежит одному thread; обработку можно передать workers, но offset commit должен учитывать завершённые offsets без пропусков.

`max.poll.records` ограничивает batch, `max.poll.interval.ms` — максимум между poll, session timeout/heartbeat обнаруживают погибший consumer.

## Consumer groups и rebalance

Coordinator назначает partitions members. Eager rebalance отзывает всё, cooperative — постепенно. На revoke завершите/зафиксируйте безопасную позицию и освободите partition state.

Static membership уменьшает rebalance при кратком restart, но stale instance fencing и уникальный identity важны.

## Offset semantics

Committed offset обычно обозначает следующий record к чтению. Auto-commit по времени может подтвердить ещё не завершённую обработку.

At-most-once: commit до effect. At-least-once: effect до commit. Crash между effect и commit повторит record.

Idempotent consumer:

```text
BEGIN
  INSERT processed_event(event_id) ON CONFLICT DO NOTHING
  if inserted: apply business change
COMMIT
commit Kafka offset
```

Unique constraint и effect должны быть в одной DB transaction.

## Kafka transactions

Producer transaction атомарно записывает records в несколько Kafka partitions и offsets group. Consumers `read_committed` не видят aborted/uncommitted records.

Она не включает обычную PostgreSQL transaction. Для DB→Kafka применяют outbox; Kafka→DB — idempotent consumer/inbox.

## Retention и compaction

Retention удаляет целые segments по time/size. Consumer lag больше retention приводит к data loss для consumer.

Compaction сохраняет по возможности последнее record key; old versions удаляются асинхронно. Tombstone означает delete и хранится период. Snapshot consumer должен читать весь compacted log и корректно применять tombstones.

## Retry topics и DLQ

Blocking retry задерживает partition. Retry topic переносит record с attempt/next-time headers и освобождает основной поток, но может нарушить order.

DLQ contract: original topic/partition/offset, key, payload/schema ID, exception class/message, attempts, timestamps, trace ID. Нужны alert, owner, retention и replay tool с исправленной причиной.

## Schema evolution

Field IDs/names и defaults определяют compatibility. Consumer-first deployment требует backward/forward strategy. Unknown fields должны сохраняться/игнорироваться по формату. Breaking semantic change иногда требует нового event type, а не только schema version.

## Наблюдаемость

Producer: error/retry rate, request latency, batch/compression. Consumer: lag per partition, poll/process latency, rebalance count, commit errors. Broker: under-replicated/offline partitions, ISR changes, disk, request queue.

## Критерий готовности

Вы готовы, если объясняете путь record от producer до replica, связываете partition с order/parallelism, проектируете at-least-once consumer и отличаете Kafka transaction от DB atomicity.

