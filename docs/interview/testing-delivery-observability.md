# Тестирование, доставка и наблюдаемость

## Пирамида тестов

| Уровень | Проверяет | Цена |
|---|---|---|
| Unit | правило/класс в изоляции | быстро, узкая уверенность |
| Component/slice | адаптер или слой с частью framework | средняя |
| Integration | взаимодействие с реальной БД/broker | выше |
| Contract | совместимость consumer/provider | требует процесса публикации контрактов |
| End-to-end | критический путь всей системы | медленно и хрупко |

Пирамида — ориентир по обратной связи, а не квота. Сложность должна тестироваться на минимальном уровне, где проявляется реальный риск.

## JUnit 5

Хороший тест имеет одну наблюдаемую причину падения, читаемое arrange–act–assert и проверяет поведение, а не строки реализации.

```java
@Test
void rejectsOrderWhenCreditIsInsufficient() {
    var account = new Account(Money.of(100));

    var result = account.reserve(Money.of(150));

    assertThat(result).isEqualTo(Reservation.rejected("INSUFFICIENT_CREDIT"));
}
```

Parameterized tests полезны для таблицы граничных случаев. Не объединяйте случайные сценарии так, чтобы падение одного скрывало остальные.

## Mockito

Mock заменяет collaborator и задаёт ответы; spy оборачивает настоящий объект и часто связывает тест с деталями. `verify` нужен, когда interaction и есть контракт: публикация команды, отсутствие charge, аудит. Для обычного возвращаемого результата лучше проверять state/output.

Не mock-айте value objects, collections и код самого unit. Если тест требует десяток mocks, объект, вероятно, имеет слишком много обязанностей.

## Spring tests и Testcontainers

`@SpringBootTest` поднимает полный context и подходит для небольшого числа сквозных component tests. Slice tests (`@WebMvcTest`, `@DataJpaTest`) дают более быстрый фокус.

Testcontainers запускает настоящую версию PostgreSQL/Kafka/Redis и ловит несовместимость SQL, driver и configuration, которую не видит in-memory fake. Контейнер можно переиспользовать между тестами, но данные каждого теста должны быть изолированы.

Изоляция: уникальные данные, rollback/cleanup, независимость порядка, контролируемые часы и отсутствие общего mutable singleton state.

## Contract testing

Consumer contract фиксирует минимальные ожидания клиента, provider проверяет их перед выпуском. Contract tests не заменяют compatibility policy schema и небольшой end-to-end smoke test.

## Docker

Image — неизменяемый шаблон слоёв, container — запущенный процесс с writable layer. Данные хранят во volume/external storage, а не в container layer.

Качественный Dockerfile:

- multi-stage build;
- маленький runtime image;
- non-root user;
- pinned base image policy и vulnerability scanning;
- сначала копируются редко меняющиеся dependency descriptors для cache layers;
- один foreground process и корректная обработка signals;
- healthcheck согласован с orchestration.

Container networking использует собственное namespace; `localhost` внутри container — сам container. Service name/DNS связывает containers в сети.

## CI/CD pipeline

Минимальный поток:

1. compile + static checks;
2. unit tests;
3. integration/contract tests;
4. build один immutable artifact/image;
5. security/license scan;
6. deploy того же artifact;
7. smoke/health verification;
8. автоматический stop/rollback по сигналам.

Blue-green держит две среды и переключает трафик; rollback быстрый, но стоимость выше и миграции БД должны быть совместимы. Canary постепенно увеличивает долю новой версии и требует качественных метрик и автоматических критериев.

## Три сигнала наблюдаемости

- **Metrics** показывают агрегированное состояние и тренды.
- **Logs** объясняют дискретные события с контекстом.
- **Traces** связывают путь запроса между компонентами.

OpenTelemetry унифицирует instrumentation и передачу context; backend хранения может меняться.

### Метрики

Counter только растёт и подходит для количества запросов/ошибок. Gauge показывает текущее значение: queue depth, connections. Histogram распределяет наблюдения по buckets и позволяет агрегировать latency percentiles.

Average скрывает хвост. p95 означает, что 95% наблюдений не превышают значение, но не объясняет отдельного пользователя и зависит от окна/агрегации.

### Structured logging

Логируйте event name, timestamp, severity, trace/correlation ID, operation, safe entity ID, result и duration отдельными полями. Не собирайте JSON строковой конкатенацией и не допускайте высокой cardinality в labels metrics.

## SLI, SLO, SLA и error budget

- SLI — измеряемый показатель, например доля успешных запросов.
- SLO — целевая граница SLI за окно.
- SLA — внешнее соглашение и последствия нарушения.
- Error budget — допустимая доля неуспеха: `1 - SLO`.

SLO должен отражать пользовательский результат. «Процесс запущен» слабее, чем «валидный заказ создан не дольше X для Y% запросов».

Timeouts и retries влияют на tail latency и нагрузку. Один исходный запрос может умножиться на много downstream attempts — учитывайте retry budget и метрики попыток отдельно от пользовательских операций.

## Вопросы для самопроверки

1. Какой риск оправдывает integration test с настоящей БД?
2. Когда `verify` полезнее проверки результата?
3. Почему image должен собираться один раз?
4. Чем histogram лучше client-side вычисленного среднего?
5. Как error budget влияет на скорость релизов?

