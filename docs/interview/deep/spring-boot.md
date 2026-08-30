# Spring Boot — подробный конспект

## Что именно делает Boot

Spring Boot не заменяет Spring Framework. Он стандартизирует bootstrap, dependency sets, configuration, server packaging, production endpoints и conditional auto-configuration.

`@SpringBootApplication` объединяет configuration, component scan и enable auto-configuration. Package класса задаёт default root scanning, поэтому неправильное расположение может скрыть components/entities.

## Auto-configuration

Auto-configuration classes импортируются как candidates и активируются conditions:

- class есть/нет в classpath;
- bean есть/нет;
- property имеет значение;
- web application определённого типа;
- resource/JNDI и другие условия.

`@ConditionalOnMissingBean` позволяет пользователю заменить default. Порядок auto-config описывает зависимости, но не должен использоваться как случайная гарантия runtime order.

Диагностика: condition evaluation report показывает positive/negative matches. Если bean не появился, сначала проверьте classpath, property binding и уже существующий bean.

## Starters и dependency management

Starter — удобный dependency descriptor. BOM/dependency management согласует версии transitive libraries. Не задавайте вручную случайные версии поверх managed set без проверки compatibility.

Starter не «включает магию» сам: classpath создаёт условия, auto-configuration регистрирует beans.

## Externalized configuration

Один key может иметь несколько sources. Более приоритетный переопределяет меньший. Нужно знать active profiles и origin значения.

YAML удобен для hierarchy, но:

- tabs запрещены;
- похожие типы/значения могут интерпретироваться неожиданно;
- duplicate keys опасны;
- environment variable mapping преобразует точки/дефисы;
- secret не хранится в git.

## ConfigurationProperties

Типизированная binding лучше россыпи `@Value`: единый namespace, metadata, validation, testability.

```java
@ConfigurationProperties("client.catalog")
@Validated
public record CatalogClientProperties(
        @NotNull URI baseUrl,
        @NotNull Duration connectTimeout,
        @NotNull Duration readTimeout,
        @Min(1) int maxConnections
) {}
```

Validation fail-fast останавливает startup. Для cross-field rule добавьте custom validator/constructor invariant.

## Embedded server

Servlet container управляет acceptor, connection handling, request threads и servlet lifecycle. Настройки max threads/connections/queue должны соответствовать downstream pool и latency.

Если HTTP threads 200, а DB pool 10 и каждый request ждёт БД, дополнительные threads увеличивают очередь, а не throughput.

Graceful shutdown перестаёт принимать новый трафик и ждёт in-flight requests в пределах timeout. Orchestrator readiness должен убрать instance раньше termination.

## Actuator

Endpoints: health, metrics, info, env/configprops (с осторожностью), loggers, mappings, threaddump, heapdump. Не публикуйте административные endpoints наружу без authentication/network policy.

Health contributors агрегируются. Readiness включает зависимости, без которых instance не обслуживает запрос; liveness обычно не зависит от внешней БД, иначе её сбой перезапустит все instances.

## Observability integration

Micrometer абстрагирует metrics registry. Observation связывает metrics и tracing. Tag values должны иметь bounded cardinality: status/method допустимы, userId/orderId — нет.

Logging facade SLF4J отделяет API от backend. Parameterized logging не строит строку, если level выключен. Exception передаётся отдельным последним аргументом.

## Conditional beans

Сложные trees conditions делают конфигурацию неочевидной. Используйте properties с ясными defaults, fail-fast при частичной настройке и tests на каждый meaningful mode.

Не злоупотребляйте `@ConditionalOnMissingBean` для бизнес-выбора: случайный bean сторонней библиотеки может отключить вашу конфигурацию.

## Custom starter

Содержит:

1. autoconfigure module;
2. properties;
3. conditions;
4. registration metadata auto-configuration;
5. optional dependency starter;
6. documentation default/override;
7. context runner tests.

Auto-config не должна сканировать произвольные packages пользователя и выполнять сетевой вызов на construction без явного opt-in.

## Profiles

Profile — group configuration, не environment inheritance system. Избегайте комбинаций десятков profiles (`prod & eu & kafka & blue`). Лучше orthogonal properties и platform configuration.

Test profile не должен полностью отличаться от production wiring, иначе тестируется другая система. Testcontainers позволяет сохранять реальные adapters.

## Startup failures

Частые причины:

- port занят;
- property не bound/validation failed;
- circular dependency;
- migration/DB unavailable;
- incompatible dependencies;
- duplicate bean;
- classpath condition неожиданно активировала config.

Читайте самую нижнюю meaningful cause и condition report, а не только верхний `ApplicationContextException`.

## Критерий готовности

Вы готовы, если объясняете starter vs auto-config, можете найти причину conditional bean, спроектировать validated properties и разделить liveness/readiness без restart storm.

