# Observability — подробный конспект

## Сигналы и вопросы

Metrics отвечают «насколько часто/сильно», logs — «какое событие», traces — «где прошёл конкретный запрос». Наблюдаемость начинается с пользовательских SLI и гипотез диагностики.

## Prometheus model

Time series идентифицируется metric name + labels. Counter rate считает скорость, gauge — текущий level, histogram — bucket counts/sum/count.

Высокая cardinality (`user_id`, URL с ID, exception message) взрывает series и стоимость. Используйте route template и bounded dimensions.

Counter не уменьшают; restart обнуляет, `rate` учитывает reset. Gauge не используйте для накопительного total.

## Percentiles

Histogram quantile агрегируется между instances, если buckets согласованы. Client summary quantile часто нельзя корректно агрегировать. p99 нестабилен при малом sample; указывайте окно и traffic.

Tail latency end-to-end не равна сумме p99 каждого hop. Нужны trace и critical path.

## Tracing

Trace состоит spans с parent/child, start/duration, status, attributes/events. Context передаётся HTTP headers и message headers.

Sampling head принимает решение в начале, tail — после просмотра trace и лучше сохраняет errors/slow, но дороже. Не помещайте PII/secrets в attributes.

Async messaging создаёт producer/consumer spans и links, если causal relation не строгий parent-child.

## Structured logs

Стабильные поля: event, service/version, severity, trace/span, request/operation ID, result, duration. Error stack один раз на границе. Sampling/rate limit защищает от log storm.

## SLI/SLO/SLA

SLI задаёт numerator/denominator и окно. Availability лучше считать good events / valid events, исключая заранее определённый невалидный traffic.

SLO 99.9% за 30 дней даёт error budget 0.1%. Multi-window burn-rate alert обнаруживает быстрый и медленный расход бюджета без шума одного порога.

SLA — внешний договор, может быть слабее internal SLO, чтобы осталось время реакции.

## Retry observability

Разделяйте logical requests и attempts. Иначе retry улучшит внешнюю success rate, но скроет нагрузку. Метрики: attempts per operation, exhausted, timeout, circuit state, queue depth/age.

## Критерий готовности

Вы готовы, если проектируете RED/USE metrics, избегаете cardinality, связываете trace через Kafka и строите SLO/error-budget alerts.

