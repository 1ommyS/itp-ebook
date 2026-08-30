# Микросервисные паттерны

## Сначала — границы

Микросервис — не «маленький Spring Boot проект», а independently deployable граница с ownership данных и операционной ответственностью. Разделяйте по бизнес-возможностям и модели изменений, а не по техническим слоям (`user-controller-service`, `user-repository-service`).

Признаки плохой границы: совместный deploy для каждого изменения, общая схема БД без ownership, длинные синхронные цепочки и распределённая транзакция для обычного use case.

## Синхронно или асинхронно

Синхронный вызов проще, когда клиенту нужен немедленный ответ и зависимость доступна в latency budget. Асинхронное сообщение развязывает время, сглаживает пики и позволяет повтор, но приносит eventual consistency, дубликаты, ordering и сложную диагностику.

Считайте end-to-end timeout. Если каждый из пяти сервисов ждёт по две секунды, клиентский budget не становится десятью секундами автоматически.

## Saga

Saga — последовательность локальных транзакций с compensating actions при неуспехе.

- Choreography: сервисы реагируют на события; меньше центральной логики, но поток трудно увидеть.
- Orchestration: coordinator выдаёт команды и хранит state; поток прозрачен, но coordinator становится важным компонентом.

Compensation не всегда возвращает прошлое: отправленное письмо нельзя «не отправить», а возврат платежа — новая операция. Проектируйте semantic compensation и ручной разбор крайних случаев.

## Transactional outbox и inbox

Outbox сохраняет бизнес-изменение и запись события в одной локальной DB transaction. Отдельный relay публикует outbox в broker. Это закрывает окно «БД committed, publish не произошёл», но возможна повторная публикация.

Inbox/processed-message table делает consumer идемпотентным: event ID и бизнес-эффект фиксируются атомарно. Очистка таблиц, ordering и повторная доставка входят в дизайн.

## Идемпотентность

Идемпотентная операция при повторе с тем же намерением не создаёт новый эффект. Способы:

- natural key и `UNIQUE` constraint;
- idempotency key с сохранённым результатом;
- compare-and-set по версии состояния;
- deduplication event IDs;
- upsert с чёткой семантикой.

Один key должен быть привязан к identity операции и payload; повтор того же key с другим payload — конфликт.

## Timeout, retry и circuit breaker

Timeout освобождает ресурсы, но не доказывает, что downstream ничего не сделал. Поэтому retry команды требует idempotency.

Retry:

- только для transient failure;
- ограничен числом попыток и общим deadline;
- exponential backoff + jitter;
- учитывает `Retry-After`/rate limit;
- не повторяется на каждом слое независимо.

Circuit breaker быстро отклоняет вызовы к явно больной зависимости и пробует восстановление в half-open. Он не заменяет timeout, а fallback должен быть бизнес-корректным.

## Bulkhead и backpressure

Bulkhead разделяет pools/semaphores, чтобы один downstream не исчерпал все ресурсы приложения. Bounded queue и отказ лучше бесконечного накопления. Backpressure означает, что producer получает сигнал замедлиться, а не просто переносит перегрузку в память.

## Eventual consistency и CAP

Eventual consistency допустима, если бизнес понимает окно рассогласования и конфликт. Пользователю можно показать pending state, operation ID и статус вместо ложного синхронного успеха.

CAP рассматривает поведение при network partition: нельзя одновременно гарантировать linearizable consistency и availability каждой операции. Это не выбор «двух букв» для системы навсегда — разные операции могут иметь разные компромиссы.

## Distributed locks

Распределённый lock сложен из-за пауз процесса, network partition, истечения lease и split brain. Даже если клиент считает lock действующим, ресурс может уже принять нового владельца. Fencing tokens и проверка версии на самом ресурсе дают более сильную защиту.

Часто lock можно заменить partitioning по key, unique constraint, optimistic concurrency или single-writer model.

## Сценарий: создание заказа

1. API принимает команду с idempotency key.
2. Order service проверяет/создаёт заказ и outbox в одной transaction.
3. Relay публикует `OrderCreated`.
4. Payment и Inventory обрабатывают событие идемпотентно.
5. Saga меняет статус заказа или запускает compensation.
6. Клиент читает статус; trace ID связывает HTTP, events и логи.

Режимы отказа, которые нужно проговорить: повтор POST, crash после commit, duplicate event, out-of-order event, недоступный broker, зависший payment, poison message и ручное восстановление.

## Вопросы для самопроверки

1. Почему shared database ослабляет границы сервисов?
2. Как outbox закрывает dual-write и какой новый риск оставляет?
3. Почему timeout требует идемпотентности команды?
4. Где хранить состояние orchestration saga?
5. Чем fencing token сильнее простого TTL lock?

