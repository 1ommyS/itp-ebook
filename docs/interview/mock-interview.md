# Практический прогон

## Как проводить

Один прогон занимает 75–90 минут:

1. 15 минут — короткие вопросы по Core/JVM.
2. 25 минут — coding-задача.
3. 15 минут — SQL или Spring debugging.
4. 25 минут — system design.
5. 10 минут — разбор: факты, структура ответа, код, trade-offs.

Записывайте не «знал/не знал», а конкретный пробел: «не сформулировал happens-before для volatile», «не проверил duplicate key», «не посчитал peak RPS».

## Раунд 1: Java Core

1. Объясните, почему mutable key ломает `HashMap`.
2. Спроектируйте immutable `Money` с currency и amount.
3. Сравните interface и abstract class на примере способов оплаты.
4. Покажите suppressed exception в try-with-resources.
5. Объясните pass-by-value на примере списка и переназначения ссылки.

Follow-up: какой контракт должен соблюдать `compareTo`, если объект хранится в `TreeSet`?

## Раунд 2: JVM и concurrency

1. Нарисуйте runtime areas JVM.
2. Почему достижимый, но больше не нужный объект создаёт memory leak?
3. Чем `volatile` отличается от `AtomicInteger`?
4. Найдите deadlock в коде с двумя locks и предложите порядок захвата.
5. Когда virtual threads не повысят throughput?

Follow-up: какие данные снимете до рестарта зависшего приложения?

## Coding A: LRU cache

Реализуйте `get`/`put` за O(1) без `LinkedHashMap`.

Проверьте:

- capacity 1;
- обновление существующего ключа;
- чтение меняет recency;
- eviction ровно одного элемента;
- поведение неизвестного ключа.

Расширение: предложите thread-safe вариант и объясните, почему lock нужен вокруг map и списка как единого инварианта.

## Coding B: bounded producer/consumer

Создайте несколько producers и consumers через `BlockingQueue`. Определите завершение без `Thread.stop`, корректно обработайте interruption и не теряйте последнюю задачу.

Расширение: как измерить queue lag и что делать при переполнении?

## Coding C: TTL cache

Контракт: `put(key, value, ttl)`, `get(key)`, maximum size. Используйте monotonic time abstraction, чтобы тест не зависел от реальных часов.

Обсудите lazy cleanup, background cleanup, stampede и concurrent load одного key.

## Spring debugging

Сценарий: публичный метод `checkout()` вызывает `this.reserve()`; на `reserve()` стоит `@Transactional(REQUIRES_NEW)`, но отдельной транзакции нет.

Ожидаемый разбор:

1. annotation реализована через proxy;
2. self-invocation не проходит через proxy;
3. границу операции следует вынести в отдельный bean или изменить orchestration;
4. нужно отдельно проверить, действительно ли независимый commit соответствует бизнес-atomicity.

## SQL-задача

Таблицы `customer(id)` и `orders(id, customer_id, created_at, amount, status)`. Выведите по каждому клиенту последний успешный заказ и сумму успешных заказов за 30 дней.

Ожидаются window function или аккуратный lateral/subquery, корректная обработка клиентов без заказов и индекс, соответствующий filter/order. Кандидат должен объяснить план и проверить данные с одинаковым `created_at`.

## Kafka consumer

Спроектируйте consumer платежных событий:

- at-least-once delivery;
- DB effect и deduplication в одной transaction;
- commit offset после эффекта;
- bounded retry для transient error;
- DLQ для permanent error;
- метрики lag, attempts и DLQ rate;
- trace/correlation identifiers.

Follow-up: что произойдёт при crash после DB commit и до offset commit?

## System design: notification service

Уточняющие вопросы:

- какие channels и объёмы;
- допустима ли задержка;
- нужны ли user preferences и schedule;
- как трактовать provider accepted vs delivered;
- какие ограничения и compliance.

Минимальный дизайн: API → durable command/outbox → broker → channel workers → providers → status events. Добавьте idempotency, rate limiting, retry/DLQ, provider failover, template versioning и observability.

## System design: order service

Покажите state machine, ownership данных, transaction + outbox, saga, inventory/payment compensation и пользовательский pending state. Отдельно разберите duplicate/late/out-of-order events.

## Оценка ответа

| Критерий | 0 | 1 | 2 |
|---|---|---|---|
| Корректность | Ключевая ошибка | В целом верно, есть пробел | Верно и с границами применимости |
| Структура | Поток сознания | Есть основная линия | Определение → механизм → пример → trade-off |
| Код | Не компилируется/нет границ | Основной путь работает | Учтены границы, complexity и тесты |
| Production thinking | Только happy path | Назван один сбой | Timeout, retry, idempotency, metrics |
| Коммуникация | Не уточняет | Уточняет часть | Явно фиксирует assumptions и выбор |

Максимум — 10. Для стабильного уровня повторите раунд до результата 8+ два раза с разными задачами.

## Финальный чек-лист

- [ ] Могу дать 90-секундный ответ по каждой High-теме.
- [ ] Пишу LRU, producer/consumer и SQL с window function без подсказки.
- [ ] Объясняю thread dump, `EXPLAIN ANALYZE` и Spring proxy.
- [ ] Проектирую REST endpoint с validation и error contract.
- [ ] Проектирую Kafka consumer с retry, DLQ и idempotency.
- [ ] Считаю peak RPS, storage и bandwidth.
- [ ] Называю режимы отказа и эксплуатационные сигналы.
- [ ] В каждом решении проговариваю минимум один trade-off.

