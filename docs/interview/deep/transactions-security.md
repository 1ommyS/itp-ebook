# Spring Transactions и Security — подробный конспект

## Transaction interceptor

Proxy читает transaction metadata, выбирает `PlatformTransactionManager`, открывает/присоединяет transaction, вызывает target и завершает commit/rollback. JDBC manager привязывает Connection к thread; JPA manager — EntityManager/persistence context.

Transaction context обычно thread-bound. Передача работы в `@Async`, новый thread или reactive chain не переносит его автоматически.

## Propagation

`REQUIRED` объединяет вызовы в одну physical transaction. Внутренний метод может отметить её rollback-only; внешний попытается commit и получит `UnexpectedRollbackException`.

`REQUIRES_NEW` приостанавливает внешнюю и берёт отдельный connection. При маленьком pool вложенные новые transactions способны вызвать pool starvation. Независимый commit создаёт эффект даже при rollback внешней.

`NESTED` использует savepoint, если manager/resource поддерживает. Rollback к savepoint не равен отдельной transaction и не освобождает locks до внешнего завершения.

## Isolation mapping

Spring isolation передаётся resource/driver. Реальные гарантии определяет DB. Не обещайте поведение по названию annotation без знания PostgreSQL/MySQL и access pattern.

Transaction boundary ставят на application use case. Слишком широкая держит connection/locks во время HTTP; слишком узкая разрывает invariant между read и write.

## Rollback rules

Default rollback для `RuntimeException`/`Error`. Checked exception обычно commit, если не задан `rollbackFor`. Это framework convention, а не свойство БД.

Пойманное исключение не доходит до interceptor. Если операция должна откатиться, пробросьте подходящую ошибку или явно пометьте rollback-only — лучше не связывать domain с Spring API без необходимости.

## `readOnly`

Hint может переключить flush mode, connection read-only и routing. Он не заменяет права. Надёжная read-only защита — read-only transaction/user/replica с осознанной lag policy.

## SecurityFilterChain

Запрос проходит filters: context persistence, CORS/CSRF, authentication mechanisms, anonymous, authorization, exception translation. Порядок критичен.

AuthenticationManager передаёт token providers. После успеха `Authentication` с principal/authorities попадает в SecurityContext.

## Authentication vs authorization

Authentication проверяет identity/credentials. Authorization применяет policy к identity, action и resource. Роли часто coarse-grained, permission/ownership — fine-grained.

Проверка `hasRole('ADMIN')` на URL не защищает object-level access. Метод `getOrder(id)` обязан проверить tenant/owner или query сразу ограничивает выборку.

## Passwords и sessions

Пароли хранят adaptive salted hash (bcrypt/Argon2/PBKDF2), не encryption. Cost обновляется со временем; после успешного login можно rehash.

Session ID — bearer secret, cookie должна быть Secure, HttpOnly и подходящий SameSite. Session fixation предотвращается сменой ID после login. Logout/revoke требует server-side state.

## JWT

JWS signature защищает целостность, не конфиденциальность. Проверки: allowlist algorithm, signature key, `iss`, `aud`, `exp`, `nbf`, clock skew и required claims.

Key rotation использует `kid` и период перекрытия. Нельзя доверять algorithm из token без server policy. Long-lived access token усложняет revoke; короткий TTL ограничивает окно.

## OAuth 2.0 / OIDC

Roles: resource owner, client, authorization server, resource server. Authorization Code + PKCE защищает public clients. Client Credentials — machine identity.

OAuth scope описывает делегированный доступ, не обязательно application role. OIDC добавляет ID Token/UserInfo и правила аутентификации.

Refresh token хранится защищённо, rotation обнаруживает reuse. Browser app часто выигрывает от BFF/session cookie, чтобы token не был доступен JavaScript.

## CSRF и CORS

CSRF возможен, когда browser автоматически прикладывает credential. Synchronizer token/double-submit и SameSite снижают риск. Stateless bearer token в Authorization header не добавляется cross-site автоматически, но XSS остаётся угрозой.

CORS preflight OPTIONS проверяет method/headers/origin. Server всё равно выполняет authentication/authorization; CORS ограничивает чтение browser script, не curl/attacker server.

## Критерий готовности

Вы готовы, если рисуете proxy transaction, объясняете rollback-only/REQUIRES_NEW, проходите SecurityFilterChain и проектируете JWT/OAuth flow с CSRF/CORS без смешения понятий.

