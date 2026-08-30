<div class="handbook-hero" markdown>

<span class="handbook-eyebrow">Go · backend · concurrency</span>

# Go для backend-разработчика

Последовательный маршрут от модели языка и интерфейсов до конкурентных HTTP-сервисов, работы с PostgreSQL, тестирования и production-диагностики.

[Начать с языка](language.md){ .md-button .md-button--primary }
[Открыть практику](practice.md){ .md-button }
[Посмотреть решения](solutions.md){ .md-button }

</div>

!!! info "Версия примеров"
    Материалы ориентированы на Go 1.27, выпущенный 19 августа 2026 года, и на гарантию совместимости Go 1. Перед использованием новой возможности проверьте директиву `go` в `go.mod` и версию toolchain проекта.

## Зачем Go backend-разработчику

Go сочетает небольшую спецификацию, быстрый toolchain, статическую типизацию, сборщик мусора и стандартную библиотеку с production-ready HTTP, profiling и testing API. Сильная сторона языка — не количество конструкций, а предсказуемая композиция простых механизмов.

Go особенно полезен для:

- сетевых сервисов, API gateway, control plane и инфраструктурных утилит;
- приложений с большим количеством одновременно ожидающих I/O-операций;
- автономных бинарников и контейнеров с простой доставкой;
- команд, которым важны быстрые сборки и единый formatter;
- сервисов, где явные зависимости и небольшие интерфейсы важнее framework magic.

Go не отменяет проектирование. Goroutines без владения создают утечки, channels без протокола — deadlock, а лаконичная обработка ошибок легко превращается в потерю контекста.

## Ментальная модель после Java/Kotlin

| Java/Kotlin | Go | Практическое следствие |
|---|---|---|
| Класс и наследование | Struct, embedding, composition | Поведение собирается из маленьких компонентов |
| `implements` | Неявная реализация interface | Интерфейс удобно объявлять рядом с потребителем |
| Exception | Значение `error` | Ошибка является частью обычного control flow |
| Thread/executor/coroutine | Goroutine | Запуск дешёвый, но lifecycle всё равно нужен |
| Queue/future | Channel | Channel задаёт коммуникацию и синхронизацию |
| `try-with-resources` / `use` | `defer` | Освобождение ресурса располагается рядом с захватом |
| Generics с type erasure | Type parameters и constraints | Обобщение применяют только при реальной повторяемости алгоритма |
| Framework DI | Constructor functions + interfaces | Граф зависимостей обычно собирается явно |

## Маршрут обучения

| Этап | Темы | Результат |
|---|---|---|
| 1. Toolchain | `go mod`, `go test`, `go fmt`, `go vet` | Создаёте воспроизводимый модуль |
| 2. Язык | values, slices, maps, structs, methods | Понимаете zero value и reference-like descriptors |
| 3. Контракты | interfaces, errors, generics | Проектируете маленькие API без лишних абстракций |
| 4. Concurrency | goroutines, channels, `select`, `sync` | Строите отменяемый pipeline без leaks и races |
| 5. HTTP | handlers, middleware, JSON, timeouts | Создаёте защищённый HTTP server |
| 6. Data | `database/sql`, pool, transaction, context | Корректно управляете соединениями и rollback |
| 7. Quality | table-driven tests, fuzzing, race detector | Проверяете свойства и конкурентное поведение |
| 8. Production | logs, metrics, traces, pprof, shutdown | Диагностируете сервис под нагрузкой |

## Карта раздела

<div class="grid cards" markdown>

-   :material-code-braces:{ .lg .middle } **Язык и система типов**

    Modules, values, slices, maps, structs, interfaces, errors, generics и идиоматичный API design.

    [:octicons-arrow-right-24: Читать](language.md)

-   :material-transit-connection-variant:{ .lg .middle } **Backend и concurrency**

    Goroutines, channels, context, HTTP server, `database/sql`, testing и graceful shutdown.

    [:octicons-arrow-right-24: Читать](backend.md)

-   :material-flask-outline:{ .lg .middle } **Практический трек**

    30 заданий: от slices и errors до worker pool, REST API, PostgreSQL и итогового сервиса.

    [:octicons-arrow-right-24: Решать](practice.md)

    [:material-check-circle-outline: Готовые решения](solutions.md)

</div>

## Как изучать Go правильно

1. Начинайте со standard library; добавляйте framework только после понимания `net/http`.
2. Запускайте `go test ./...`, `go vet ./...` и `go test -race ./...` постоянно, а не перед релизом.
3. Для каждой goroutine называйте владельца, условие завершения и канал передачи ошибки.
4. Оборачивайте ошибки через `%w`, но не превращайте каждую функцию в новый «слой сообщения».
5. Принимайте interface на стороне потребителя; возвращайте конкретный тип, если нет причины скрывать реализацию.
6. Измеряйте аллокации и contention профилировщиком, а не догадками.

## Официальные материалы

- [История релизов Go](https://go.dev/doc/devel/release)
- [Официальные tutorials](https://go.dev/doc/tutorial/)
- [Effective Go](https://go.dev/doc/effective_go)
- [Документация `database/sql`](https://go.dev/doc/database/)
