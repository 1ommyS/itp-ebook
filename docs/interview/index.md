<div class="handbook-hero" markdown>

<span class="handbook-eyebrow">Маршрут подготовки</span>

# От первого ответа до уверенного system design

Программа превращает чек-лист из Excel в последовательное обучение. На каждом этапе нужно понять модель, проговорить ответ, закрепить его кодом и разобрать компромиссы.

[Посмотреть все темы](coverage.md){ .md-button .md-button--primary }
[Перейти к заданиям](practice-new-topics.md){ .md-button }

</div>

## Метод сильного ответа

<div class="grid cards" markdown>

-   :material-lightbulb-on-outline:{ .lg .middle } **1. Модель**

    Какую проблему решает инструмент? Какие гарантии и инварианты он создаёт?

-   :material-microphone-message:{ .lg .middle } **2. Объяснение**

    Сформулируйте ответ за 60–90 секунд: определение → механизм → пример.

-   :material-code-block-tags:{ .lg .middle } **3. Реализация**

    Напишите минимальный код или запрос и проверьте граничные случаи.

-   :material-scale-balance:{ .lg .middle } **4. Компромиссы**

    Назовите цену решения, альтернативу и случай, когда нужен другой инструмент.

</div>

!!! tip "Шаблон сильного ответа"
    **Определение → механизм → пример → ограничения → выбор.** Например: «`volatile` обеспечивает видимость записей и ограничивает переупорядочивание, но не превращает составную операцию в атомарную. Для счётчика нужен `AtomicInteger` или блокировка».

## Восемь этапов подготовки

| Этап | Фокус | Критерий готовности |
|---|---|---|
| 1. Язык | Java Core, collections, generics, Stream API | Объясняете контракты и пишете пример без подсказок |
| 2. Runtime | JVM, GC, JIT, JMM | Связываете симптом с областью памяти или механизмом исполнения |
| 3. Параллелизм | Потоки, locks, futures, virtual threads | Находите race condition и обосновываете синхронизацию |
| 4. Данные | SQL, PostgreSQL, JPA, транзакции | Читаете план запроса и объясняете границы транзакции |
| 5. Backend | Spring Core, Boot, Web, Security | Проходите путь запроса от сети до репозитория |
| 6. Интеграции | Kafka, Redis, REST, gRPC | Проектируете повторяемую и идемпотентную обработку |
| 7. Production | Микросервисы, тесты, CI/CD, observability | Называете режимы отказа и способы их обнаружить |
| 8. Архитектура | System design и практические задачи | Защищаете решение через требования и trade-offs |

## Углублённые конспекты

<div class="grid cards" markdown>

-   :material-language-java:{ .lg .middle } **Java и runtime**

    [Java Core](deep/java-core.md) · [Collections](deep/collections.md) · [Generics](deep/generics-functional.md) · [JVM](deep/jvm.md) · [Concurrency](deep/concurrency.md) · [Algorithms](deep/algorithms.md)

-   :material-database-cog:{ .lg .middle } **Данные и Spring**

    [SQL/PostgreSQL](deep/sql-postgresql.md) · [Spring Core](deep/spring-core.md) · [Boot](deep/spring-boot.md) · [Web](deep/spring-web.md) · [JPA/Hibernate](deep/jpa-hibernate.md) · [Transactions/Security](deep/transactions-security.md)

-   :material-transit-connection-variant:{ .lg .middle } **Интеграции**

    [Kafka](deep/kafka.md) · [Redis](deep/redis.md) · [REST/gRPC](deep/grpc-rest.md) · [Microservices](deep/microservices.md)

-   :material-chart-timeline-variant-shimmer:{ .lg .middle } **Production и архитектура**

    [Testing](deep/testing.md) · [Docker/CI/CD](deep/docker-cicd.md) · [Observability](deep/observability.md) · [System Design](deep/system-design.md) · [Practical Java](deep/practical-java.md)

</div>

## План на четыре недели

| Неделя | Фокус | Ежедневная практика |
|---|---|---|
| 1 | Java Core, collections, generics, JVM | Две coding-задачи и один устный ответ |
| 2 | Concurrency, SQL, PostgreSQL | Thread-safe задача и разбор `EXPLAIN ANALYZE` |
| 3 | Spring, JPA, Security, API | Небольшой CRUD-сервис с тестами |
| 4 | Kafka, Redis, микросервисы, system design | Два архитектурных кейса и пробное интервью |

Регулируйте темп по результатам практики: слабое место определяется не временем чтения, а ошибкой в объяснении, коде или выборе компромисса.

!!! success "Финишная проверка"
    Завершите маршрут [пробным интервью](mock-interview.md), а пробелы закройте через [дополнительные задания](practice-new-topics.md). Возвращайтесь к главе только после того, как сможете точно назвать свою ошибку.
