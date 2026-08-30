# REST и gRPC — подробный конспект

## REST constraints и resource model

REST — architectural style: client-server, stateless, cacheable, uniform interface, layered system. В прикладном API важнее корректная resource/HTTP semantics, чем спор о «чистоте REST».

URI идентифицирует resource, representation передаёт состояние. Команду можно моделировать subordinate resource (`/orders/{id}/cancellations`) или action, если домен так яснее.

## Idempotency и concurrency

PUT идемпотентен по intended effect, но audit timestamp/version может меняться. DELETE повторно оставляет resource отсутствующим, хотя status может стать 404.

Optimistic HTTP concurrency: server отдаёт ETag/version, client отправляет `If-Match`, mismatch → 412. Это предотвращает lost update.

## Caching

`Cache-Control`, ETag/If-None-Match и Last-Modified уменьшают traffic. Private/public/no-store имеют security смысл. Нельзя кэшировать user-specific response общим cache без Vary/key policy.

## Pagination/filtering/versioning

Cursor должен кодировать stable sort tuple и filter context. Изменение filters с тем же cursor недопустимо. Cursor делают opaque, иногда подписывают от подделки.

API evolution включает enum unknown values, additive fields, tolerant readers и deprecation telemetry. Consumer-driven contracts проверяют ожидания, но не заменяют документацию.

## Protobuf

Message fields имеют уникальные numbers — wire identity. Не меняйте type несовместимо и не переиспользуйте удалённый number; используйте `reserved`.

Scalar default не отличает «не передано» без optional/wrapper/presence semantics. `oneof` моделирует взаимоисключающие variants. Map кодируется repeated entry.

## RPC types

- unary: один request/response;
- server streaming: один request, поток responses;
- client streaming: поток requests, один response;
- bidirectional: два независимых потока.

Streaming не означает бесконечность без контроля. Нужны cancellation, backpressure/flow control, message size limits и lifecycle при half-close.

## Deadline и cancellation

Deadline — абсолютный budget, должен передаваться downstream с вычетом времени. Локальные timeout 1s на каждом из пяти hops нарушают общий 1s budget.

Когда client отменил call, server context сообщает cancellation; работа должна кооперативно остановиться. DB/HTTP clients тоже получают оставшийся timeout.

## Errors

gRPC status categories: INVALID_ARGUMENT, NOT_FOUND, ALREADY_EXISTS, FAILED_PRECONDITION, ABORTED, UNAUTHENTICATED, PERMISSION_DENIED, RESOURCE_EXHAUSTED, UNAVAILABLE, DEADLINE_EXCEEDED, INTERNAL.

Retry обычно только UNAVAILABLE/RESOURCE_EXHAUSTED с policy и idempotency; DEADLINE_EXCEEDED не доказывает отсутствие эффекта. Rich error details содержат machine-readable reason/field violations.

## Interceptors

Client/server interceptors реализуют metadata, auth, tracing, metrics, retry policy. Они не должны скрывать бизнес transaction или автоматически повторять неидемпотентную команду.

## REST vs gRPC

REST лучше для browser/public ecosystem, human debugging, HTTP caching. gRPC — typed internal contracts, generated clients и streaming. Через gateway можно дать оба, но нужно решить semantic parity и error mapping.

## Критерий готовности

Вы готовы, если проектируете resource contract, ETag/idempotency/cursor и protobuf evolution, передаёте deadline и выбираете retryable gRPC status без двойного эффекта.

