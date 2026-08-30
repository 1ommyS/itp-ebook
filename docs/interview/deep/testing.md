# Testing — подробный конспект

## Test design

Тест — executable specification риска. Он должен быть deterministic, isolated, readable и давать локальный диагноз.

AAA/Given-When-Then отделяет setup, действие и observable result. Проверяйте public behavior. Тест private method означает, что либо проверяется через public contract, либо responsibility стоит выделить.

## JUnit 5

Lifecycle default — новый instance на test. `@BeforeEach` не должен скрывать важные данные. Parameterized test использует ValueSource/CsvSource/MethodSource. Nested groups сценарии.

`assertAll` показывает несколько независимых свойств одного результата. `assertThrows` проверяет type и details. Timeout preemptive выполняет другой thread и может нарушить ThreadLocal/transaction; используйте осторожно.

## Mockito

Stub задаёт ответ, mock также проверяет interaction. Strict stubs находят ненужную настройку. ArgumentCaptor применяйте, когда аргумент — важный observable message.

Не mock framework internals и value objects. Mock chain `a.b().c().d()` нарушает Law of Demeter. Spy вызывает real methods, поэтому `when(spy.method())` может выполнить код; используйте `doReturn` при необходимости.

## Integration

Проверяйте mapping, constraints, transaction, serialization, broker headers и client timeouts настоящими adapters. Testcontainers фиксирует image version и dynamic properties.

DB test должен проверять реальные migrations, не только ORM create schema. Cleanup через transaction rollback не работает для async/other connections — нужны truncate/schema/database per test strategy.

## Spring tests

`@WebMvcTest` — MVC slice, `@DataJpaTest` — persistence slice, `@SpringBootTest` — full context. `@MockBean` меняет context key и может замедлять cache reuse; злоупотребление создаёт тест другой wiring.

## Contract/E2E

Contract фиксирует request/response/event compatibility. Provider verifies published contracts. E2E оставляют для критических paths и минимального smoke, потому что они медленны и имеют много причин падения.

## Test isolation

Контролируйте Clock, random seed, IDs, locale/timezone. Не используйте sleep для ожидания async; poll с deadline и meaningful condition. Независимость порядка проверяется random order/repeat.

## Критерий готовности

Вы готовы, если выбираете минимальный уровень теста, пишете deterministic async test и объясняете, что именно не поймает mock и зачем нужен container/contract.

