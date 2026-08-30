# Microservices — подробный конспект

## Границы и ownership

Сервис владеет бизнес-возможностью, моделью и данными. Другие сервисы не пишут его таблицы напрямую. Граница учитывает language, team ownership, change cadence и consistency needs.

Distributed monolith имеет раздельные процессы, но требует coordinated deploy, shared DB и synchronous chain. Он сложнее монолита без автономности микросервисов.

## Sync/async

Sync даёт немедленный result и простой control flow, но связывает availability/latency. Async decouples time and absorbs bursts, но требует state machine, duplicates, ordering и reconciliation.

Решение по interaction, не по сервису целиком. Query часто sync, domain event async, command выбирается по нужному acknowledgment.

## Saga

Каждый шаг — local transaction. Failure запускает compensation. Orchestrator хранит current step, attempts и outcome; choreography распределяет transition logic по consumers.

Saga должна обрабатывать duplicate command/event, timeout без ясного outcome, late success после compensation и manual intervention. Состояние terminal/transition защищают optimistic version.

## Outbox/Inbox

Outbox row содержит event ID, aggregate ID/version, type, payload, created time, publish status. Relay читает через polling/CDC и публикует. Mark-published не атомарен с broker ack, поэтому duplicate допустим.

Inbox unique event ID + business effect в одной transaction. Для ordering проверяют aggregate version; пропуск можно ждать/replay или применить commutative model.

## Resilience chain

Deadline сверху распределяет budget. Timeout каждого hop меньше remaining budget. Retry только transient/idempotent, exponential backoff+jitter, ограничен attempts и retry budget.

Circuit breaker считает failures/slow calls в sliding window, переходит open, затем limited half-open probes. Bulkhead отделяет pools/semaphores. Rate limit защищает capacity. Все механизмы без observability лишь меняют форму отказа.

## Eventual consistency

Показывайте пользователю `PENDING`, operation resource и status. Read model может отставать; read-your-writes обеспечивается sticky/version wait/primary read или явным pending response.

Conflict policy: last-write-wins редко соответствует бизнесу. Лучше version vector/domain merge/single writer/optimistic reject.

## CAP и network partition

При partition coordinated nodes либо отказывают части операций для consistency, либо принимают их с риском divergence. Latency/failure detector превращают теорию в практический timeout choice.

## Distributed locks

Lock service не контролирует защищаемый ресурс автоматически. Lease expiry и pause требуют fencing. Альтернативы: unique constraint, partitioned single consumer, optimistic version, idempotent command.

## Критерий готовности

Вы готовы, если определяете границу, рисуете saga/outbox/inbox, считаете deadline/retry amplification и описываете recovery для duplicate, late и out-of-order events.

