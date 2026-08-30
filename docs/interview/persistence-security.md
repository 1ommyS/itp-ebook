# JPA, транзакции и Security

## Состояния entity и persistence context

Entity бывает transient, managed, detached или removed. Persistence context — identity map и unit of work: внутри него одной строке БД обычно соответствует один managed object.

Dirty checking сравнивает состояние managed entity и при flush формирует SQL update. Flush синхронизирует context с БД, но не обязательно commit transaction.

```java
@Transactional
public void rename(long id, String name) {
    User user = repository.findById(id).orElseThrow();
    user.renameTo(name); // save не обязателен для managed entity
}
```

Не превращайте этот механизм в неявную магию: граница транзакции должна быть видна на application service, а entity — защищать свои инварианты.

## Fetching и N+1

N+1 возникает, когда один запрос загружает список родителей, а доступ к каждой связи запускает дополнительный запрос. Решения зависят от use case:

- fetch join для одного конкретного запроса;
- entity graph для декларативного fetch plan;
- projection/DTO query, когда нужен read model;
- batch fetching для контролируемой загрузки группами.

Глобально ставить всё EAGER — не решение: появляются лишние joins, Cartesian explosion и нестабильная производительность. `LAZY` — обещание отложенной загрузки, но фактическое поведение зависит от типа связи и инструмента enhancement/proxy.

## Cascade и `orphanRemoval`

Cascade распространяет операцию persistence lifecycle от aggregate root к связанному объекту. `orphanRemoval` удаляет дочернюю entity, когда она исключена из отношения владельца. Это не одно и то же, что database `ON DELETE CASCADE`.

Применяйте cascade внутри настоящей aggregate ownership. `CascadeType.ALL` на many-to-many часто опасен: удаление одного участника не должно удалять другого.

## Locking

Optimistic lock через `@Version` обнаруживает конфликт при update и хорошо подходит, когда конфликты редки. После `OptimisticLockException` операция целиком повторяется на свежих данных или конфликт возвращается пользователю.

Pessimistic lock запрашивает lock в БД и подходит для короткой критической секции с вероятным конфликтом. Он увеличивает ожидание и риск deadlock; внешний HTTP-вызов под таким lock особенно опасен.

## Cache и JPQL

First-level cache принадлежит persistence context и всегда участвует в identity semantics. Second-level cache общий для нескольких sessions и полезен для редко изменяемых данных при ясной политике invalidation. Query cache хранит набор идентификаторов/результатов и чувствителен к изменениям зависимых таблиц.

JPQL оперирует entities и их полями, а не таблицами и колонками. Для сложного аналитического запроса native SQL или отдельный read model может быть проще и быстрее.

## Как работает `@Transactional`

Обычно proxy открывает/присоединяет transaction перед методом, вызывает target, затем commit или rollback. Следствия:

- self-invocation может обойти proxy;
- private/final methods могут не подходить выбранной proxy-модели;
- по умолчанию rollback выполняется для unchecked exception, а checked требует явной политики;
- исключение нельзя бездумно проглотить внутри метода и ожидать автоматический rollback.

### Propagation

| Режим | Смысл |
|---|---|
| REQUIRED | присоединиться к текущей или создать новую |
| REQUIRES_NEW | приостановить внешнюю и создать отдельную |
| SUPPORTS | использовать текущую, если есть |
| MANDATORY | требовать существующую |
| NOT_SUPPORTED | выполнить без transaction |
| NEVER | ошибка, если transaction существует |
| NESTED | savepoint внутри поддерживаемой transaction model |

`REQUIRES_NEW` не делает действие «надёжным» автоматически: внешняя операция может откатиться, а внутренняя — зафиксироваться. Это осознанное нарушение общей atomicity.

`readOnly=true` — hint для framework/driver/provider и сигнал намерения, а не универсальная защита от записи. Реальную защиту задают права БД и архитектура.

## Authentication и authorization

Authentication отвечает «кто пользователь?», authorization — «что ему разрешено?». SecurityFilterChain выполняет filters: извлечение credentials, authentication, сохранение security context, обработку exception и проверку доступа.

Правила должны быть deny-by-default. Проверка только на уровне URL недостаточна, если один endpoint выполняет операции над объектами разных владельцев; добавляйте method/domain authorization.

## JWT

JWT — подписанный self-contained token, а не шифрование по умолчанию. Проверяйте signature, issuer, audience, expiration и допустимый algorithm. Не помещайте секреты в payload.

Access token обычно короткоживущий. Refresh token имеет отдельный lifecycle, хранение, rotation и revoke strategy. Компрометация bearer token означает возможность использовать его до истечения/отзыва.

## OAuth 2.0

OAuth 2.0 делегирует доступ клиента к resource server. Для пользовательского входа поверх него применяется OpenID Connect. В browser/mobile сценариях authorization code + PKCE защищает перехват кода; client credentials используется для machine-to-machine без пользователя.

Не называйте OAuth «протоколом логина» без уточнения OIDC и не реализуйте собственный authorization server без веской причины.

## CSRF и CORS

- **CSRF** заставляет browser отправить аутентифицированный запрос к сайту-жертве. Особенно актуально для credentials, добавляемых browser автоматически, например session cookie. Защита: CSRF token, SameSite cookie и корректная семантика методов.
- **CORS** — browser policy, определяющая, может ли script одного origin читать ответ другого. Это не механизм аутентификации и не защита server-to-server API.

Разрешение `*` вместе с credentials несовместимо с безопасной allowlist-моделью. Настраивайте конкретные origins, methods и headers.

## Вопросы для самопроверки

1. Чем flush отличается от commit?
2. Почему Open Session in View может скрыть N+1?
3. Когда optimistic lock лучше pessimistic?
4. Почему `REQUIRES_NEW` меняет atomicity use case?
5. Чем OIDC дополняет OAuth 2.0?

