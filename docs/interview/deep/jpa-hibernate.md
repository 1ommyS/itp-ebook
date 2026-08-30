# JPA и Hibernate — подробный конспект

## Entity identity

Entity имеет продолжительную identity и lifecycle. Value object определяется значениями и обычно immutable. Не моделируйте всё entity: address/money/date range часто value objects.

JPA entity требует доступного constructor для provider, stable mapping и осторожного equality. Proxy subclass может ломать `getClass()` equality; стратегия зависит от ID lifecycle и provider.

## Entity states

- transient — не связан с context;
- managed — tracked context;
- detached — раньше managed, context закрыт/cleared;
- removed — помечен на delete.

`persist` делает new entity managed. `merge` не «прикрепляет тот же объект»: он копирует state в managed instance и возвращает его. Продолжать менять исходный detached object — частая ошибка.

## Persistence context

Identity map гарантирует один managed instance на DB identity внутри context. Повторный find может не выполнить SQL. Это не общий cache приложения.

Write-behind накапливает changes до flush. Flush может случиться перед commit, query или явно. SQL order может отличаться от order вызовов из-за batching/dependencies.

## Dirty checking

Provider хранит snapshot или enhanced tracking и сравнивает поля. Managed object update без `save` попадёт в SQL при flush.

Большой context увеличивает memory и checking cost. Для batch: flush+clear порциями, JDBC batch settings и контроль generated IDs.

## Lazy loading и proxies

Lazy association возвращает proxy/collection wrapper и загружается при доступе. Вне открытого context возникает LazyInitializationException. Решение — не «держать transaction в controller», а спроектировать fetch plan и DTO внутри use case.

`EAGER` — требование загрузить к моменту доступа, но SQL strategy не гарантирована; может вызвать отдельные queries и N+1.

## N+1

Диагностика: SQL count/trace, statistics и representative endpoint. Решения:

- fetch join;
- entity graph;
- DTO projection;
- batch size;
- two-step query для pagination + to-many.

Fetch join нескольких bag collections может дать Cartesian product/ошибку. Pagination с collection fetch join часто выполняется в memory или некорректно; сначала page IDs/roots, затем fetch associations.

## Cascade

Cascade управляет JPA operations, не FK. В aggregate root дочерний lifecycle может каскадировать persist/remove. Для shared reference remove cascade опасен.

`orphanRemoval` удаляет child, исключённый из owned collection/reference. Обе стороны bidirectional association поддерживайте helper methods, иначе in-memory graph расходится.

## Optimistic locking

Version входит в update predicate:

```sql
UPDATE account
SET balance=?, version=version+1
WHERE id=? AND version=?
```

Affected rows 0 → conflict. Retry только целого use case с повторным чтением и проверкой команды; blind retry может перезаписать пользовательское решение.

## Pessimistic locking

DB row lock удерживается до transaction end. Нужны lock timeout, единый order и короткий transaction. Lock не защищает от изменения данными системой, которая не использует ту же БД/row.

## L1, L2 и query cache

L1 обязателен и scoped context. L2 shared provider cache должен соответствовать consistency/invalidations. Read-mostly reference data — хороший кандидат; часто меняемый balance — плохой.

Query cache хранит result identifiers и зависит от timestamps/regions; invalidation может сделать hit rate низким.

## JPQL

JPQL использует entity names/properties и navigates associations. Explicit join обычно яснее implicit navigation. Parameter binding защищает от injection и позволяет plan reuse.

Bulk update/delete обходят managed state и callbacks; context после них stale. Очистите/синхронизируйте context.

## Mapping pitfalls

- `@ManyToOne` по умолчанию EAGER — часто задают LAZY осознанно;
- enum ordinal ломается при перестановке — string безопаснее по readability;
- `BigDecimal` precision/scale должны совпадать с БД;
- `LocalDateTime` не содержит zone/instant;
- bidirectional serialization создаёт recursion;
- schema generation не заменяет migration tool в production.

## Критерий готовности

Вы готовы, если объясняете merge, flush и dirty checking, находите N+1, проектируете fetch plan для pagination и выбираете optimistic/pessimistic lock по конфликтам.

