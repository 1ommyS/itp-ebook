# Практика Go: 30 заданий и итоговый backend-проект

## Правила выполнения

Для каждого задания обязательны:

- `go fmt ./...`;
- `go vet ./...`;
- `go test ./...`;
- `go test -race ./...` для concurrent-кода;
- table-driven tests для классов эквивалентности;
- README с контрактом, ошибками и принятыми компромиссами;
- отсутствие бесконтрольных goroutines и unbounded queues.

## Уровень 1. Toolchain и язык

### 1. Модуль конвертации температур

Создайте module с library package и CLI. Экспортируйте только необходимые types/functions, добавьте examples, которые запускаются как тесты документации.

### 2. Анализатор текста

Посчитайте Unicode-символы, слова и частоты. Не смешивайте bytes и runes. Результат с одинаковой частотой сортируйте детерминированно.

### 3. Slice ownership

Реализуйте `AppendUnique`, `CloneAndSort` и `Window`. Для каждой функции письменно определите, может ли она изменять input и удерживать исходный backing array.

### 4. LRU cache

Создайте cache фиксированной ёмкости на map + doubly linked list. Сначала single-threaded, затем concurrent-safe. Проверьте zero capacity, update существующего ключа и eviction order.

### 5. Domain types

Опишите `Order`, `OrderID`, `Money`, `Status`. Constructor валидирует инварианты. Не используйте float для денег. Zero value каждого типа должен быть либо полезен, либо явно отклонён validation.

### 6. Маленькие interfaces

Есть сервис, которому нужны чтение заказа и отправка события. Объявите interfaces рядом с сервисом, напишите in-memory adapters и докажите compile-time реализацию.

### 7. Error chain

Создайте sentinel `ErrNotFound`, typed `ValidationError` и infrastructure error. Оборачивайте причины через `%w`; тесты используют `errors.Is`/`errors.As`, а не строки.

### 8. Generic Set

Реализуйте `Set[T comparable]` и операции union/intersection. Объясните, почему constraint `comparable` необходим и почему generic repository обычно является плохой domain abstraction.

## Уровень 2. Concurrency

### 9. Ограниченный parallel map

Реализуйте generic `ParallelMap` с лимитом workers, сохранением порядка результатов, cancellation и возвратом первой ошибки.

### 10. Fan-out/fan-in pipeline

Постройте pipeline чтения файлов, вычисления checksum и агрегации. Закрытие channels должно происходить ровно одним владельцем.

### 11. Отменяемый worker pool

Обработайте очередь задач несколькими workers. Проверьте cancellation при blocked input и blocked output, закрытие results и отсутствие goroutine leaks.

### 12. Rate limiter

Сделайте token bucket с burst capacity. Тестируйте с controllable clock, не заставляя тест реально ждать.

### 13. Single-flight loader

При одновременном запросе одного ключа expensive load выполняется один раз. Ошибка также доставляется всем waiters. После завершения ключ можно загрузить повторно.

### 14. Concurrent ledger

Поддерживайте составной инвариант «сумма денег постоянна». Реализуйте mutex-вариант и owner-goroutine вариант; прогоните race detector и benchmark.

### 15. Leak investigation

Вам дан pipeline, который прекращает чтение после первого результата. Найдите заблокированных producers через goroutine profile, добавьте cancellation и regression test.

### 16. Graceful background job

Периодическая задача запускается по ticker, не допускает overlapping execution и завершается по context. Ошибка последнего запуска доступна health endpoint.

## Уровень 3. HTTP API

### 17. Strict JSON endpoint

Создайте `POST /orders`: ограничьте body, запретите неизвестные поля и второй JSON object, разделите syntax и domain validation.

### 18. Middleware chain

Реализуйте request ID, access log, panic recovery, timeout и authentication middleware. Проверьте порядок выполнения и сохранение status code.

### 19. Idempotent create

Повторный запрос с тем же idempotency key и payload возвращает исходный результат; другой payload с тем же ключом — conflict. Параллельные запросы не создают два объекта.

### 20. Pagination contract

Реализуйте cursor pagination. Cursor должен быть opaque, подписан и стабилен при добавлении новых записей. Проверьте malformed и expired cursor.

### 21. HTTP client

Создайте клиент downstream-сервиса с общим `http.Client`, timeout budget, ограниченным retry только для безопасных временных ошибок и закрытием response body.

### 22. Graceful shutdown test

Запустите реальный HTTP server, начните медленный запрос, отправьте shutdown и докажите, что активный запрос завершился в пределах budget, а новый не принят.

## Уровень 4. PostgreSQL

### 23. Repository

Реализуйте create/find/list через `database/sql`. Закрывайте rows, проверяйте `rows.Err`, отображайте `sql.ErrNoRows` в domain error.

### 24. Transactional transfer

Переведите деньги между счетами с row locking, проверкой баланса и audit record. Проверьте rollback и concurrent transfers.

### 25. Outbox

В одной транзакции измените aggregate и сохраните событие. Отдельный publisher читает outbox, публикует и отмечает запись. Обеспечьте at-least-once и идемпотентного consumer.

### 26. Pool under load

Создайте нагрузочный тест с маленьким `MaxOpenConns`. Наблюдайте `WaitCount`, `WaitDuration`, latency и database capacity. Обоснуйте итоговые настройки.

## Уровень 5. Качество и эксплуатация

### 27. Fuzz JSON parser

Добавьте fuzz test к парсеру cursor или входного DTO. Seed corpus должен включать валидные, пустые, Unicode и граничные значения.

### 28. Benchmark и аллокации

Сравните две реализации serializer/aggregator. Используйте `ReportAllocs`, realistic payload и профилирование. Не принимайте оптимизацию без измеримого выигрыша.

### 29. Observability

Добавьте structured logs, HTTP RED metrics, DB pool metrics и trace propagation. Не используйте user-controlled значение как безграничный label.

### 30. Failure drill

Смоделируйте недоступность БД, зависший downstream, SIGTERM под нагрузкой и заполнение очереди. Для каждого сценария зафиксируйте ожидаемые сигналы и восстановление.

## Итоговый проект: Order Service

### Функциональность

- создать заказ с idempotency key;
- добавить позиции и подтвердить заказ;
- отменить допустимый заказ;
- получить заказ и историю изменений;
- опубликовать `OrderConfirmed` через transactional outbox;
- cursor pagination списка заказов.

### Архитектурные требования

- composition root в `cmd/api`;
- domain package не импортирует HTTP и SQL adapters;
- consumer-owned interfaces;
- PostgreSQL migrations;
- явные transaction boundaries;
- HTTP server/client timeouts;
- bounded background workers;
- graceful shutdown;
- configuration из environment с startup validation.

### Обязательные проверки

```bash
go fmt ./...
go vet ./...
go test ./...
go test -race ./...
go test -fuzz=Fuzz -fuzztime=20s ./...
```

### Матрица приёмки

| Область | Готово, если |
|---|---|
| Correctness | Domain invariants и rollback покрыты тестами |
| Concurrency | Нет races/leaks, очереди и workers ограничены |
| HTTP | Body limits, validation, stable errors и timeouts работают |
| Database | Rows закрываются, pool наблюдаем, запросы отменяются context |
| Delivery | Миграции воспроизводимы, binary/container запускается одной командой |
| Operations | Есть logs, metrics, traces, health/readiness и shutdown budget |

## Вопросы для защиты проекта

1. Кто владеет каждой goroutine и чем она завершается?
2. Почему выбран channel, mutex или atomic?
3. Что произойдёт при повторной доставке outbox event?
4. Где заканчивается request timeout и начинается downstream timeout?
5. Как рассчитан DB pool относительно replicas и лимита БД?
6. Какие ошибки безопасно показать клиенту?
7. Как доказано отсутствие потери денег при конкурентных транзакциях?
8. Как сервис ведёт себя во время SIGTERM под нагрузкой?
