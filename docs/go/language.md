# Go: язык, типы и дизайн API

## 1. Toolchain и модуль

Go-проект начинается с модуля. Module path является стабильным пространством имён для импортов, а `go.mod` фиксирует минимальную семантику языка и прямые зависимости.

```bash
mkdir orders
cd orders
go mod init example.com/orders
go fmt ./...
go test ./...
go vet ./...
```

Минимальная структура backend-модуля:

```text
orders/
├── cmd/api/main.go          # composition root
├── internal/order/          # предметный пакет
├── internal/postgres/       # adapter данных
├── internal/httpapi/        # HTTP transport
├── migrations/
├── go.mod
└── go.sum
```

`internal` запрещает импорт пакета извне родительского дерева. Это compile-time граница, а не просто convention. Не создавайте `pkg`, `util`, `common` автоматически: пакет должен иметь конкретную ответственность и имя, описывающее предоставляемую возможность.

## 2. Packages, visibility и initialization

Идентификатор экспортируется, если начинается с заглавной буквы. Экспорт — часть API-контракта: публичное имя труднее изменить, поэтому начинайте с минимальной поверхности.

```go
package order

type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service {
    return &Service{repo: repo}
}
```

Package initialization выполняется в порядке зависимостей: сначала импортируемые пакеты, затем package variables и функции `init`. Избегайте скрытых I/O и goroutines в `init`: их трудно конфигурировать и тестировать.

## 3. Values, zero value и присваивание

Каждый тип имеет zero value: `0`, `false`, `""`, `nil` для pointers, slices, maps, channels, functions и interfaces. Хорошо спроектированный тип полезен сразу после объявления без обязательного конструктора.

```go
var count int          // 0
var names []string     // nil slice: len=0, append разрешён
var cache map[string]int // nil map: чтение разрешено, запись panic
```

Присваивание копирует значение. Для struct копируются все поля; slice, map и channel содержат descriptors, указывающие на общие runtime-структуры. Поэтому «передача по значению» не означает глубокую копию.

## 4. Arrays и slices

Array `[N]T` включает длину в тип. Slice `[]T` — descriptor из указателя на backing array, длины и capacity.

```go
func appendOrder(ids []int, id int) []int {
    ids = append(ids, id)
    return ids // новый descriptor нужно вернуть владельцу
}
```

### Важные инварианты slice

- `len` — доступная длина, `cap` — пространство до конца backing array;
- `append` может использовать прежний array или выделить новый;
- subslice удерживает весь backing array и способен продлить жизнь большого объекта;
- два slice могут неожиданно видеть изменения друг друга;
- `nil` и пустой slice обычно одинаковы для `len`/`range`, но могут по-разному сериализоваться в JSON.

Чтобы отделить данные от исходного массива:

```go
copyOfIDs := append([]int(nil), ids...)
// или в современном Go:
copyOfIDs := slices.Clone(ids)
```

## 5. Maps

Map — hash table reference-like типа. Zero value равен `nil`; чтение отсутствующего ключа возвращает zero value, поэтому используйте comma-ok, когда отсутствие отличается от нулевого значения.

```go
price, ok := prices[sku]
if !ok {
    return fmt.Errorf("sku %q: %w", sku, ErrNotFound)
}
```

Порядок iteration намеренно не определён. Map не безопасна для конкурентных записи/чтения. В зависимости от модели владения используйте mutex, single-owner goroutine или специализированную concurrent structure.

## 6. Structs, methods и pointer receiver

Method принадлежит типу receiver, но в Go нет классов. Receiver выбирается по семантике:

- pointer receiver нужен для изменения состояния, больших structs и types с mutex;
- value receiver подходит маленькому immutable-like значению;
- не смешивайте receivers без причины: method sets влияют на реализацию interface.

```go
type Counter struct {
    mu sync.Mutex
    n  int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.n++
}
```

Копировать значение с `sync.Mutex` после первого использования нельзя. Поэтому такие types почти всегда используют pointer receiver и передаются по указателю.

## 7. Composition и embedding

Embedding продвигает methods/fields, но не создаёт отношение подтипов. Используйте его, если embedded capability действительно является частью публичного поведения.

```go
type AuditWriter struct {
    io.Writer
    Clock Clock
}
```

Не встраивайте concrete type только ради экономии нескольких forwarding methods: так вы случайно расширяете публичный API.

## 8. Interfaces и structural typing

Тип реализует interface неявно, если имеет необходимый method set.

```go
type OrderStore interface {
    Save(ctx context.Context, order Order) error
    ByID(ctx context.Context, id ID) (Order, error)
}
```

Правила хорошего interface:

- объявляйте его рядом с потребителем;
- делайте маленьким и ориентированным на одну роль;
- не создавайте interface «на будущее»;
- принимайте interface, если функция действительно допускает варианты поведения;
- возвращайте concrete type, если abstraction не нужна клиенту.

### Typed nil внутри interface

Interface состоит из dynamic type и dynamic value. Он равен `nil`, только если обе части nil.

```go
var p *MyError = nil
var err error = p
fmt.Println(err == nil) // false
```

Не возвращайте typed nil как `error`. Возвращайте literal `nil` по успешной ветке.

## 9. Errors как значения

Исключения не используются для обычного control flow. Функция возвращает `error`, клиент выбирает обработку.

```go
var ErrNotFound = errors.New("order not found")

func (s *Service) Find(ctx context.Context, id ID) (Order, error) {
    order, err := s.repo.ByID(ctx, id)
    if err != nil {
        return Order{}, fmt.Errorf("find order %s: %w", id, err)
    }
    return order, nil
}
```

Используйте:

- `errors.Is` для sentinel/wrapped error;
- `errors.As` для извлечения typed error;
- `%w` для сохранения cause chain;
- собственный error type, если клиенту нужны структурированные поля;
- stable domain error codes на внешней API-границе.

Не сравнивайте текст `err.Error()` и не логируйте одну ошибку на каждом слое: это создаёт дубли. Обычно ошибка оборачивается внутри и логируется один раз на границе запроса.

## 10. `defer`, `panic` и `recover`

`defer` регистрирует вызов при выходе из текущей функции. Аргументы deferred call вычисляются сразу, а вызовы выполняются в обратном порядке.

```go
rows, err := db.QueryContext(ctx, query)
if err != nil {
    return nil, err
}
defer rows.Close()
```

`panic` подходит нарушенному внутреннему инварианту или невозможности продолжить запуск приложения, но не validation/domain error. `recover` работает только внутри deferred function той же goroutine. На HTTP-границе recovery middleware может не дать упасть процессу, но обязан залогировать stack trace и вернуть контролируемый ответ.

## 11. Functions, closures и loop variables

Functions — значения. Closure захватывает переменные, а не моментальные значения. В конкурентном коде явно передавайте iteration value в функцию, если поддерживаемая версия и читаемость не позволяют однозначно понять capture semantics.

```go
for _, id := range ids {
    id := id
    go func() {
        process(id)
    }()
}
```

## 12. Generics

Generics нужны для алгоритма или container, одинакового для набора типов. Constraint определяет разрешённые операции.

```go
func Map[T, R any](in []T, f func(T) R) []R {
    out := make([]R, 0, len(in))
    for _, value := range in {
        out = append(out, f(value))
    }
    return out
}
```

Не заменяйте domain interfaces сложными constraints. Если операция описывает поведение (`Read`, `Save`, `Validate`), обычный interface чаще понятнее. Если повторяется структура алгоритма над типами, generics могут быть уместны.

## 13. Pointers и escape analysis

Pointer позволяет разделять и изменять значение, но не поддерживает arithmetic. Компилятор решает, разместить значение на stack или heap; возврат адреса local variable безопасен.

```go
func NewConfig() *Config {
    cfg := Config{Timeout: 3 * time.Second}
    return &cfg
}
```

Не выбирайте pointer только из страха копирования. Сначала определите identity, mutability и optionality, затем измеряйте performance. `go build -gcflags=-m` помогает изучить escape decisions, но это диагностический инструмент, а не руководство к микроптимизации.

## 14. Идиоматичный API checklist

- package name короткое и предметное;
- exported declarations имеют doc comments;
- constructor возвращает готовый к использованию type;
- zero value полезен либо невозможность объяснена;
- ownership slice/map понятен: borrowed, consumed или copied;
- context является первым параметром и не хранится в struct;
- error сохраняет cause и предметный контекст;
- interface объявлен потребителем;
- configuration проверяется при запуске;
- goroutine не запускается без lifecycle contract.

## Вопросы для самопроверки

1. Почему изменение slice после `append` иногда видно вызывающему коду, а иногда нет?
2. Чем `nil` map отличается от `nil` slice?
3. Когда значение с pointer receiver реализует interface, а значение без указателя — нет?
4. Почему interface, содержащий typed nil, не равен `nil`?
5. Где следует оборачивать ошибку и где логировать её?
6. Чем embedding отличается от наследования?
7. Когда generics делают API проще, а когда скрывают domain behavior?
