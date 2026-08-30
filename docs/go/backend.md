# Go backend: concurrency, HTTP, database и production

## 1. Concurrency не равна parallelism

Concurrency описывает структуру программы как независимо продвигающиеся задачи. Parallelism означает физическое одновременное выполнение. Goroutine может выполняться параллельно, но главная ценность модели — возможность выразить множество одновременно ожидающих операций.

Goroutine дёшева, но не бесплатна. Она удерживает stack, ссылки на heap, timer, connection или channel. Поэтому для каждого запуска ответьте:

1. Кто владеет goroutine?
2. Как она узнаёт о завершении?
3. Куда возвращает ошибку?
4. Что ограничивает количество экземпляров?
5. Что произойдёт, если consumer перестанет читать?

## 2. Channels и ownership

Channel передаёт значения и синхронизирует goroutines.

- unbuffered channel завершает send только при готовом receiver;
- buffered channel допускает ограниченное расхождение скоростей;
- закрывает channel producer, а не consumer;
- receive из закрытого channel возвращает buffered values, затем zero value с `ok=false`;
- send в закрытый channel вызывает panic;
- `nil` channel навсегда блокирует send/receive и полезен для отключения case в `select`.

```go
func generate(ctx context.Context, ids []int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, id := range ids {
            select {
            case out <- id:
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}
```

## 3. `select`, timeout и cancellation

`select` ждёт готовности одного из communication cases. Если готовы несколько, выбирается один. `default` превращает ожидание в non-blocking poll и часто создаёт busy loop, поэтому используйте его только с ясной причиной.

```go
select {
case result := <-results:
    return result, nil
case <-ctx.Done():
    return Result{}, ctx.Err()
}
```

Не создавайте `time.After` в горячем цикле: каждый вызов создаёт timer. Для повторяющегося события используйте `time.NewTicker` и `defer ticker.Stop()`.

## 4. `context.Context`

Context переносит deadline, cancellation signal и request-scoped values через API-границы.

Правила:

- принимайте `ctx` первым параметром;
- не храните context в struct;
- всегда вызывайте `cancel` у derived context;
- передавайте request context в DB/RPC вызовы;
- values используйте только для cross-cutting request metadata, не для обязательных dependencies;
- не заменяйте cancellation обычной business error.

```go
ctx, cancel := context.WithTimeout(parent, 2*time.Second)
defer cancel()

order, err := repo.ByID(ctx, id)
if err != nil {
    return Order{}, fmt.Errorf("load order: %w", err)
}
```

## 5. Worker pool без утечек

```go
type Job struct{ ID int }
type Result struct {
    ID  int
    Err error
}

func Run(ctx context.Context, workers int, jobs <-chan Job) <-chan Result {
    results := make(chan Result)
    var wg sync.WaitGroup

    worker := func() {
        defer wg.Done()
        for {
            select {
            case <-ctx.Done():
                return
            case job, ok := <-jobs:
                if !ok {
                    return
                }
                result := Result{ID: job.ID, Err: process(ctx, job)}
                select {
                case results <- result:
                case <-ctx.Done():
                    return
                }
            }
        }
    }

    wg.Add(workers)
    for i := 0; i < workers; i++ {
        go worker()
    }
    go func() {
        wg.Wait()
        close(results)
    }()
    return results
}
```

Проверяемые свойства: workers ограничены; закрытие jobs завершает pool; cancellation разблокирует receive и send; results закрывается ровно один раз после всех workers.

## 6. Mutex, atomic или channel

| Механизм | Когда использовать | Риск |
|---|---|---|
| `sync.Mutex` | Несколько полей образуют общий инвариант | Deadlock, длинная critical section |
| `sync.RWMutex` | Реально измеренное доминирование чтений | Overhead и writer starvation assumptions |
| `atomic` | Простой counter/flag с понятной memory semantics | Нельзя легко поддержать составной инвариант |
| Channel owner | Состояние естественно обрабатывается как поток команд | Сложный протокол, backpressure, goroutine leak |

Запускайте `go test -race ./...`. Race detector обнаруживает только races, проявившиеся во время конкретного запуска, поэтому нужны конкурентные тесты с достаточным числом interleavings.

## 7. HTTP server на standard library

```go
type Server struct {
    orders *OrderService
}

func (s *Server) routes() http.Handler {
    mux := http.NewServeMux()
    mux.HandleFunc("GET /orders/{id}", s.getOrder)
    mux.HandleFunc("POST /orders", s.createOrder)
    return requestID(recoverPanic(accessLog(mux)))
}
```

Handler должен:

- ограничить размер body через `http.MaxBytesReader`;
- декодировать ровно один JSON object и запрещать неизвестные поля, если контракт строгий;
- отделять transport DTO от domain model;
- отображать domain errors в стабильные HTTP status/error codes;
- передавать `r.Context()` вниз;
- не писать заголовки после начала response body;
- не раскрывать internal error text клиенту.

## 8. Timeouts и HTTP hardening

```go
srv := &http.Server{
    Addr:              ":8080",
    Handler:           handler,
    ReadHeaderTimeout: 5 * time.Second,
    ReadTimeout:       10 * time.Second,
    WriteTimeout:      15 * time.Second,
    IdleTimeout:       60 * time.Second,
}
```

Timeouts защищают разные этапы. Один общий timeout не заменяет отдельные budgets для чтения заголовков, body, бизнес-операции и downstream-вызовов. За reverse proxy учитывайте его собственные limits и forwarded headers только от доверенного proxy.

## 9. JSON и validation

```go
func decodeJSON(w http.ResponseWriter, r *http.Request, dst any) error {
    r.Body = http.MaxBytesReader(w, r.Body, 1<<20)
    dec := json.NewDecoder(r.Body)
    dec.DisallowUnknownFields()
    if err := dec.Decode(dst); err != nil {
        return fmt.Errorf("decode request: %w", err)
    }
    if err := dec.Decode(&struct{}{}); err != io.EOF {
        return errors.New("request body must contain one JSON object")
    }
    return nil
}
```

Syntactic validation проверяет форму запроса; domain validation — бизнес-инварианты. Не привязывайте предметный слой к HTTP validation tags.

## 10. `database/sql` — это pool handle

`*sql.DB` обычно не является одним соединением. Это concurrent-safe handle, управляющий pool. `sql.Open` может не устанавливать соединение немедленно; readiness проверяют `PingContext`.

```go
db.SetMaxOpenConns(20)
db.SetMaxIdleConns(20)
db.SetConnMaxIdleTime(5 * time.Minute)
db.SetConnMaxLifetime(30 * time.Minute)
```

Настройки выводятся из capacity БД, числа replicas сервиса и профиля нагрузки. Слишком высокий лимит перегружает БД, слишком низкий превращает pool в bottleneck. Наблюдайте `DB.Stats()`.

## 11. Query lifecycle

```go
func (r *Repository) List(ctx context.Context, customerID int64) ([]Order, error) {
    rows, err := r.db.QueryContext(ctx, `
        SELECT id, status, created_at
        FROM orders
        WHERE customer_id = $1
        ORDER BY created_at DESC`, customerID)
    if err != nil {
        return nil, fmt.Errorf("query orders: %w", err)
    }
    defer rows.Close()

    orders := make([]Order, 0)
    for rows.Next() {
        var order Order
        if err := rows.Scan(&order.ID, &order.Status, &order.CreatedAt); err != nil {
            return nil, fmt.Errorf("scan order: %w", err)
        }
        orders = append(orders, order)
    }
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("iterate orders: %w", err)
    }
    return orders, nil
}
```

Всегда закрывайте rows и проверяйте `rows.Err()`. Для одной строки используйте `QueryRowContext`; отсутствие результата сравнивайте через `errors.Is(err, sql.ErrNoRows)`.

## 12. Transactions

```go
func inTx(ctx context.Context, db *sql.DB, fn func(*sql.Tx) error) (err error) {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("begin tx: %w", err)
    }
    defer func() {
        if err != nil {
            _ = tx.Rollback()
        }
    }()

    if err = fn(tx); err != nil {
        return err
    }
    if err = tx.Commit(); err != nil {
        return fmt.Errorf("commit tx: %w", err)
    }
    return nil
}
```

Все операции транзакции выполняйте через `*sql.Tx`, не через исходный `*sql.DB`. Транзакция удерживает connection, поэтому не выполняйте внутри медленный внешний HTTP-вызов без архитектурной необходимости.

## 13. Graceful shutdown

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()

errCh := make(chan error, 1)
go func() { errCh <- srv.ListenAndServe() }()

select {
case <-ctx.Done():
case err := <-errCh:
    if !errors.Is(err, http.ErrServerClosed) {
        return err
    }
}

shutdownCtx, cancel := context.WithTimeout(context.Background(), 15*time.Second)
defer cancel()
return srv.Shutdown(shutdownCtx)
```

Shutdown должен прекратить admission новых запросов, дать активным запросам budget на завершение, остановить background workers, flush telemetry и закрыть внешние ресурсы.

## 14. Testing strategy

- table-driven unit tests проверяют варианты входа;
- `httptest` проверяет handler без реального порта;
- integration tests поднимают настоящую БД и миграции;
- fuzz tests ищут edge cases parser/decoder;
- race detector проверяет выполненные concurrent paths;
- benchmarks используют `b.ReportAllocs()` и реалистичные данные;
- test doubles реализуют маленькие consumer-owned interfaces.

```go
func TestValidateStatus(t *testing.T) {
    tests := []struct {
        name    string
        status string
        wantErr bool
    }{
        {"valid", "paid", false},
        {"empty", "", true},
        {"unknown", "lost", true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := ValidateStatus(tt.status)
            if (err != nil) != tt.wantErr {
                t.Fatalf("ValidateStatus() error = %v", err)
            }
        })
    }
}
```

## 15. Production checklist

- HTTP и downstream timeouts заданы явно;
- request body и concurrency ограничены;
- все goroutines имеют owner и cancellation path;
- `go test -race ./...` проходит;
- errors сохраняют cause, но internal details не уходят клиенту;
- logs структурированы и содержат request/trace ID;
- RED-метрики покрывают rate, errors и duration;
- DB pool metrics наблюдаются;
- `pprof` недоступен из публичной сети;
- readiness учитывает критические зависимости;
- shutdown протестирован под активной нагрузкой.

## Официальные материалы

- [Effective Go: concurrency](https://go.dev/doc/effective_go#concurrency)
- [Context](https://go.dev/blog/context)
- [Работа с relational databases](https://go.dev/doc/database/)
- [Управление соединениями](https://go.dev/doc/database/manage-connections)
- [Отмена DB-операций](https://go.dev/doc/database/cancel-operations)
