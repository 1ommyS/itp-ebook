# Spring Web — подробный конспект

## DispatcherServlet lifecycle запроса

1. Container создаёт request/response и запускает FilterChain.
2. DispatcherServlet находит HandlerExecutionChain через HandlerMapping.
3. HandlerAdapter знает, как вызвать handler.
4. ArgumentResolvers создают параметры.
5. Validation проверяет request model.
6. Controller вызывает application layer.
7. ReturnValueHandlers формируют response; HttpMessageConverters сериализуют body.
8. ExceptionResolvers ищут обработчик.
9. Interceptor callbacks и filters завершаются.

Понимание chain помогает выбрать extension point.

## Controller и DTO

Controller отвечает за transport: parsing, validation, status/header, mapping. Бизнес-правила принадлежат application/domain service.

Entity как DTO опасна:

- persistence annotations становятся API contract;
- lazy proxy сериализуется вне transaction;
- bidirectional relation создаёт recursion;
- mass assignment позволяет менять лишнее;
- schema evolution БД ломает клиентов.

Request и response DTO могут различаться. Не принимайте server-generated fields во входной команде.

## Validation

Bean Validation покрывает локальные структурные правила: null, size, format, numeric range. Nested object требует `@Valid`.

Бизнес-проверка «email уникален» не должна быть annotation, делающей скрытый DB request для каждого объекта. Она выполняется use case и защищается unique constraint от race.

Validation groups добавляют сложность; отдельные command types часто яснее.

## Error contract

Единая модель:

```json
{
  "code": "ORDER_STATE_CONFLICT",
  "message": "Order cannot be paid in CANCELLED state",
  "traceId": "...",
  "fieldErrors": []
}
```

HTTP status выражает класс результата, code — стабильную бизнес-причину. Не отдавайте class name, SQL, stack trace и token.

`@ControllerAdvice` централизует mapping. Catch-all должен логировать unexpected error и возвращать generic 500; ожидаемые domain errors обычно не требуют ERROR stack trace.

## Filter vs Interceptor vs AOP

- Filter видит raw request/response и может оборачивать body, CORS, correlation context.
- Interceptor видит handler method и MVC lifecycle.
- AOP видит вызов Spring bean независимо от HTTP.

Security filters должны идти в корректном порядке. Body caching wrapper увеличивает memory и требует limit.

## HTTP methods

Safe означает не менять server state по намерению. Idempotent означает одинаковый intended effect после повторов.

PUT заменяет/задаёт resource по известному URI, PATCH изменяет часть с определённой patch semantics, POST создаёт subordinate resource/command. Практический API может отклоняться, но semantics должны быть документированы.

## Status codes

- 200 — успешный ответ с body;
- 201 — создано, полезен Location;
- 202 — принято асинхронно, ещё не выполнено;
- 204 — успех без body;
- 400 — malformed/validation;
- 401 — нужна authentication;
- 403 — identity известна, access запрещён;
- 404 — нет ресурса или скрытие existence;
- 409 — конфликт текущего state/version/idempotency;
- 429 — rate limited;
- 5xx — server/downstream failure.

## Idempotency key

Server сохраняет key, fingerprint payload, status и response/result. Concurrent одинаковые запросы должны координироваться. Тот же key с другим payload — 409. TTL выбирают по бизнес-окну retry.

Timeout клиента не означает, что server откатил операцию; idempotency позволяет безопасно повторить.

## Pagination

Offset:

- простая page number;
- дорогой большой offset;
- duplicates/missing при concurrent changes.

Cursor/keyset:

- condition `(created_at, id) < (:created, :id)`;
- стабильный unique order;
- opaque signed/encoded cursor;
- хорошо для последовательного чтения;
- сложно прыгнуть на страницу N.

## API versioning

Предпочитайте additive change, tolerant readers и deprecation. Breaking change: удаление/переименование поля, изменение meaning/type, ужесточение required input, изменение enum handling.

Версия в URI видна и проста; media/header versioning чище URI, но сложнее tooling. Главное — lifecycle, metrics использования старой версии и migration window.

## Servlet async, WebFlux и backpressure

Servlet model может освобождать request thread через async API, но code сложнее. Virtual threads дают blocking style для I/O, сохраняя downstream limits.

WebFlux использует event loop и reactive streams demand. Blocking call на event loop задерживает многие connections. `publishOn`/boundedElastic — временная изоляция, не бесконечный ресурс.

Backpressure контролирует скорость элементов в stream, но HTTP client, DB driver и broker должны поддерживать её end-to-end.

## Content negotiation и converters

`Content-Type` описывает request/response body, `Accept` — желаемый response media type. 415 — неподдерживаемый input type, 406 — нет приемлемого output representation.

Jackson mapping должен иметь явную policy unknown fields, dates/timezones, enums и numeric precision. `BigDecimal` не превращают бездумно в double.

## Критерий готовности

Вы готовы, если проходите request lifecycle, выбираете filter/interceptor/advice, проектируете idempotent POST, cursor pagination и стабильный error contract.

