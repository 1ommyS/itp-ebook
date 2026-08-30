<div class="handbook-hero" markdown>

<span class="handbook-eyebrow">Kotlin · JVM · backend</span>

# Kotlin для Java-разработчика: от синтаксиса до production-кода

Маршрут объясняет не только более короткий синтаксис, но и модель типов Kotlin, границы Java interop, structured concurrency и инженерные решения, необходимые серверному приложению.

[Открыть подробный конспект](01-intro-jdbc.md){ .md-button .md-button--primary }
[Перейти к практике](practice.md){ .md-button }
[Посмотреть решения](solutions.md){ .md-button }

</div>

## Что важно понять до начала

Kotlin/JVM компилируется в JVM-байткод и использует Java-библиотеки, но это не «Java с другим синтаксисом». Наиболее важные отличия находятся в системе типов и модели API:

- nullability является частью типа: `String` и `String?` — разные типы;
- `if`, `when` и `try` являются выражениями и могут возвращать значение;
- функции могут быть значениями, а extension-функции позволяют выразить предметный API без наследования;
- read-only-интерфейс коллекции не гарантирует физическую неизменяемость объекта;
- `suspend` описывает возможность приостановки, но сам по себе не запускает новый поток;
- Java platform types ослабляют гарантии null safety на границе языков;
- data/value/sealed-классы моделируют данные и конечные состояния, но не заменяют продуманную предметную модель.

!!! warning "Главная ловушка Java-разработчика"
    Механический перевод Java-кода рождает Kotlin с `!!`, mutable-состоянием, статическими utility-классами и блокирующими вызовами внутри coroutines. Цель обучения — писать идиоматичный Kotlin, а не сокращать количество символов.

## Маршрут обучения

| Этап | Темы | Результат |
|---|---|---|
| 1. Базовый язык | `val`/`var`, функции, выражения, классы, свойства | Пишете небольшой CLI без Java-паттернов |
| 2. Система типов | nullable types, smart casts, `Nothing`, generics, variance | Проектируете API без бесконтрольного `!!` |
| 3. Моделирование | data/value/sealed classes, delegation, extensions | Представляете состояния предметной области через типы |
| 4. Коллекции | read-only/mutable, sequences, scope functions | Выбираете eager или lazy pipeline осознанно |
| 5. JVM interop | platform types, SAM, checked exceptions, annotations | Безопасно вызываете Java из Kotlin и наоборот |
| 6. Асинхронность | suspend, coroutine scope, cancellation, Flow | Не теряете ошибки и не создаёте coroutine leaks |
| 7. Data access | JDBC, транзакции, resource management | Реализуете отменяемые запросы и корректный rollback |
| 8. Production | тесты, конфигурация, логирование, метрики | Собираете проверяемый backend-модуль |

## Карта раздела

<div class="grid cards" markdown>

-   :material-code-braces:{ .lg .middle } **Язык и модели данных**

    Null safety, functions, properties, smart casts, data/sealed-классы, extensions, generics и lambdas.

    [:octicons-arrow-right-24: Читать конспект](01-intro-jdbc.md)

-   :material-coffee:{ .lg .middle } **Java interoperability**

    Platform types, Java annotations, SAM, отображение коллекций и исключений, удобство вызова Kotlin API из Java.

    [:octicons-arrow-right-24: Открыть Java interop](01-intro-jdbc.md#java-interop)

-   :material-database:{ .lg .middle } **JDBC и транзакции**

    Управление ресурсами через `use`, prepared statements, commit/rollback и ошибки доступа к данным.

    [:octicons-arrow-right-24: Перейти к JDBC](01-intro-jdbc.md#18-jdbc-kotlin)

-   :material-flask-outline:{ .lg .middle } **Практический трек**

    Задания от null-safe функций до coroutine pipeline и итогового backend-проекта.

    [:octicons-arrow-right-24: Начать практику](practice.md)

    [:material-check-circle-outline: Готовые решения](solutions.md)

</div>

## Kotlin и Java: как выбирать язык

| Вопрос | Kotlin | Java |
|---|---|---|
| Nullability | Выражена в типах, кроме platform types | Зависит от соглашений и аннотаций |
| Data-oriented модели | Data/value/sealed classes | Records, sealed classes, обычные классы |
| Асинхронный код | `suspend`, coroutines, Flow | Futures, reactive API, virtual threads |
| Экосистема | Полный доступ к JVM-библиотекам | Нативная JVM-экосистема и более простой interop |
| Сборка | Нужен Kotlin compiler/plugin | Достаточно JDK toolchain |
| Риск | Скрытые аллокации, неверные coroutines, platform types | Verbosity, nullable-контракты, boilerplate |

Выбирайте Kotlin, когда команда понимает его модель типов и готова поддерживать Kotlin toolchain. Сам по себе более короткий синтаксис не компенсирует отсутствия единых conventions, тестов и наблюдаемости.

## Как проходить каждую тему

1. Перепишите небольшой Java-пример на идиоматичный Kotlin.
2. Объясните, что изменилось на уровне типов, JVM или управления ресурсами.
3. Добавьте негативный тест: `null`, исключение, cancellation или пустую коллекцию.
4. Проверьте, не скрывает ли лаконичная конструкция аллокацию, блокировку или неявное изменение состояния.
5. Посмотрите сигнатуру из Java: удобен ли созданный Kotlin API для Java-клиента?

## Итоговый результат

После раздела вы должны уметь создать Kotlin/JVM сервисный модуль, в котором:

- nullability не маскируется операторами `!!`;
- предметные состояния представлены sealed-иерархией;
- coroutine scope имеет владельца и корректно отменяется;
- блокирующая операция выполняется в подходящем контексте;
- JDBC-ресурсы закрываются, транзакции откатываются при ошибке;
- публичный API остаётся понятным Java-клиентам;
- ошибки проверяются тестами и несут предметный контекст.

## Официальные материалы

- [Null safety](https://kotlinlang.org/docs/null-safety.html)
- [Java interoperability](https://kotlinlang.org/docs/java-interop.html)
- [Coroutines guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Kotlin/JVM с Gradle](https://kotlinlang.org/docs/get-started-with-jvm-gradle-project.html)
