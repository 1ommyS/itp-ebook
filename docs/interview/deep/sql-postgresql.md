# SQL и PostgreSQL — подробный конспект

## Реляционная модель и логика SQL

Таблица представляет relation, строка — tuple, ограничения задают допустимые состояния. SQL декларативен: вы описываете результат, optimizer выбирает физический план.

Трёхзначная логика добавляет UNKNOWN. В `WHERE` остаётся только TRUE; FALSE и UNKNOWN отбрасываются. Поэтому `NOT IN` с `NULL` может неожиданно не вернуть строки. Безопасная альтернатива — `NOT EXISTS` с явной корреляцией.

## JOIN подробно

Join condition определяет соответствие строк, filter — допустимость результата. Для outer join их расположение принципиально.

```sql
-- Сохраняет всех клиентов
SELECT c.id, o.id
FROM customer c
LEFT JOIN orders o
  ON o.customer_id = c.id
 AND o.status = 'PAID';
```

Если `o.status = 'PAID'` перенести в WHERE, строки без заказа получат UNKNOWN и исчезнут.

Физические join algorithms:

- nested loop — хорош при малом outer и indexed inner;
- hash join — equality, строит hash table одной стороны;
- merge join — отсортированные входы, equality/range variants.

## GROUP BY и HAVING

После grouping каждая select expression должна быть group key, aggregate или функционально зависеть по правилам DB. `HAVING` фильтрует группы.

Aggregate игнорирует NULL, кроме `COUNT(*)`. Для условной агрегации PostgreSQL поддерживает `FILTER`:

```sql
COUNT(*) FILTER (WHERE status = 'PAID') AS paid_count
```

## Subquery и EXISTS

Scalar subquery обязан вернуть не более одной строки. `EXISTS` проверяет сам факт и может остановиться на первом совпадении. Correlated subquery логически выполняется для каждой outer row, но optimizer может decorrelate.

Не переписывайте любой subquery в join автоматически: join может размножить строки и изменить aggregate.

## CTE

CTE улучшает читаемость и reuse. Recursive CTE имеет anchor и recursive term; обязательно условие завершения/защита от cycle.

Materialization behavior зависит от запроса и версии/указаний. Смотрите plan: CTE не гарантированно «временная таблица» и не гарантированно inline.

## Window functions

`PARTITION BY` делит строки, `ORDER BY` задаёт порядок, frame определяет диапазон для aggregate window. Default frame при ORDER BY может быть до current peer, поэтому running sum и total partition различаются.

```sql
SUM(amount) OVER (
  PARTITION BY customer_id
  ORDER BY created_at
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total
```

`ROW_NUMBER` уникален, `RANK` оставляет пропуски, `DENSE_RANK` нет. Для deterministic result добавляйте tie-breaker.

## B-tree и composite indexes

B-tree сортирует index entries. Запрос может использовать leading equality columns, затем range/order. Индекс `(tenant_id, status, created_at)` подходит для equality tenant/status и range/order created_at.

Skip scan/optimizer capabilities меняются, поэтому не опирайтесь на миф «индекс работает только по левому префиксу» без plan, но проектируйте под реальные leading predicates.

Partial index уменьшает размер для частого subset. Predicate запроса должен логически подразумевать predicate индекса; parameterization иногда мешает proof.

Expression index работает, только если expression запроса соответствует. Например `lower(email)` для case-insensitive lookup.

## GIN, GiST, BRIN

GIN хранит inverted mapping element→rows и хорош для JSONB/array/full text containment. Запись дороже, возможен pending list.

GiST — framework для tree operator classes: геометрия, ranges, nearest-neighbor. Он может быть lossy и требовать recheck.

BRIN хранит summary ranges heap pages. Мал, эффективен при correlation физического порядка и значения. Если данные перемешаны, много ranges проходят filter и экономия исчезает.

## Index-only scan

Нужные columns находятся в index, но PostgreSQL всё равно должен проверить visibility. Visibility map позволяет пропустить heap visit для all-visible pages. Поэтому vacuum и update frequency влияют на реальную пользу INCLUDE.

## EXPLAIN ANALYZE

Поля:

- cost — условная оценка optimizer, не milliseconds;
- rows/width — estimate;
- actual time — first row..all rows на loop;
- loops — число повторов;
- buffers — cache/disk pages при `BUFFERS`;
- rows removed — работа, не дошедшая до результата.

Умножайте per-loop actual rows/time по смыслу. Огромная ошибка cardinality часто первопричина плохого join/scan. Обновите statistics, проверьте correlation и зависимость columns.

## MVCC

Tuple имеет metadata видимости transaction IDs. Snapshot определяет видимые versions. UPDATE создаёт новую version; старая остаётся до vacuum. HOT update возможен, если indexed columns не меняются и на странице есть место.

Долгая transaction удерживает `xmin`, мешая reclaim. Idle in transaction опаснее просто idle connection.

## Isolation

Read Committed: snapshot на statement, два SELECT могут увидеть разные committed data.

Repeatable Read PostgreSQL использует snapshot isolation и предотвращает non-repeatable/phantom observation в обычном смысле, но write skew возможен как логическая аномалия.

Serializable SSI отслеживает dangerous dependency structures и отменяет transaction с serialization failure. Приложение повторяет всю transaction.

## Locks

`SELECT ... FOR UPDATE` блокирует строки для изменения. `FOR NO KEY UPDATE`, `FOR SHARE`, `FOR KEY SHARE` дают тоньше режимы. `NOWAIT` сразу ошибается, `SKIP LOCKED` полезен worker queue, но пропускает busy rows и не подходит для общего consistent чтения.

Advisory locks — application-defined keys; БД не знает защищаемый инвариант. Нужны стабильный key mapping и release policy.

## Partitioning

Partition key выбирают по pruning и lifecycle. Range by date удобен drop/archive периода. Слишком много partitions увеличивает planning/maintenance. Unique constraint обычно должен учитывать partition key для глобальной гарантии.

## Replication и pooling

Physical streaming replication передаёт WAL. Replica replay lag создаёт stale reads. Synchronous commit уменьшает RPO, но добавляет latency/availability dependency.

Pool size согласуют: `instances × maxPool` не должен бесконтрольно превышать DB capacity. Little's Law связывает concurrency ≈ throughput × latency. Ускорение запросов снижает нужное число connections.

## Критерий готовности

Вы готовы, если объясняете outer join/NULL, пишете window query, проектируете composite index из access pattern, читаете план по actual/estimates/buffers и различаете MVCC, isolation и locks.

