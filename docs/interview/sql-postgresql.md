# SQL и PostgreSQL

## Логический порядок запроса

Для объяснения alias, aggregation и window functions полезно помнить логический порядок:

`FROM/JOIN → WHERE → GROUP BY → HAVING → window functions → SELECT → ORDER BY → LIMIT`

Физический план может выполнять операции иначе, сохраняя результат.

## JOIN, aggregation и `NULL`

- `INNER JOIN` оставляет совпавшие строки.
- `LEFT JOIN` сохраняет все строки слева и дополняет отсутствующие значения `NULL`.
- `RIGHT/FULL JOIN` нужны реже; часто запрос проще после перестановки таблиц.
- Фильтр по правой таблице в `WHERE` после `LEFT JOIN` может фактически превратить его в inner join; условие сохранения строк помещают в `ON`.

`NULL` означает неизвестное/отсутствующее значение. Сравнение через `=` не работает; используйте `IS NULL`, `IS DISTINCT FROM` и осознанный `COALESCE`. `COUNT(column)` не считает `NULL`, `COUNT(*)` считает строки.

`WHERE` фильтрует строки до группировки, `HAVING` — группы после неё.

```sql
SELECT c.id,
       COUNT(o.id) AS orders,
       SUM(o.amount) AS revenue
FROM customer c
LEFT JOIN orders o
  ON o.customer_id = c.id
 AND o.created_at >= :from_date
GROUP BY c.id
HAVING COUNT(o.id) >= 3;
```

## Subquery, CTE и window functions

Correlated subquery зависит от текущей строки внешнего запроса и может выполняться многократно, хотя optimizer иногда преобразует его. CTE улучшает читаемость, recursive CTE проходит иерархии и графы.

Window function считает значение по окну, не схлопывая строки:

```sql
SELECT order_id,
       customer_id,
       amount,
       ROW_NUMBER() OVER (
           PARTITION BY customer_id
           ORDER BY created_at DESC
       ) AS recency,
       LAG(amount) OVER (
           PARTITION BY customer_id
           ORDER BY created_at
       ) AS previous_amount
FROM orders;
```

`ROW_NUMBER` всегда уникален внутри partition, `RANK` оставляет пропуски после равных мест, `DENSE_RANK` — нет.

## Индексы

| Тип | Когда применять |
|---|---|
| B-tree | равенство, диапазоны, сортировка, большинство OLTP-запросов |
| GIN | составные значения: full text, arrays, JSONB |
| GiST | геометрия, ranges и operator classes |
| BRIN | очень большие физически коррелированные таблицы |

Composite B-tree эффективен, когда ведущие колонки соответствуют predicates и порядку запроса. Правило «самая селективная первой» недостаточно: учитывайте equality/range, `ORDER BY`, другие запросы и стоимость записи.

`INCLUDE` добавляет покрывающие данные в leaf entries без участия в поисковом ключе. Index-only scan возможен, когда нужны только данные индекса и visibility map подтверждает видимость страниц.

Индекс не бесплатен: занимает место, увеличивает WAL и стоимость DML. Проверяйте реальный workload.

## Как читать `EXPLAIN ANALYZE`

Читайте снизу вверх:

1. Найдите узел с большим `actual time × loops` и большим числом строк.
2. Сравните `rows` estimate с actual rows — ошибка оценки влияет на выбор join/scan.
3. Проверьте способ доступа: sequential/index/bitmap scan.
4. Посмотрите filters и `Rows Removed by Filter`.
5. Проверьте sort/hash на выход за memory и повторные loops.
6. Сравнивайте после прогрева и на репрезентативных данных.

`EXPLAIN ANALYZE` действительно выполняет запрос. Для изменяющего запроса используйте безопасную транзакцию или тестовую среду.

## MVCC и `VACUUM`

PostgreSQL хранит версии строк. Snapshot определяет, какие версии видит транзакция; читатели обычно не блокируют писателей и наоборот. Update создаёт новую tuple version, delete помечает старую недоступной.

`VACUUM` освобождает место мёртвых tuples для повторного использования и поддерживает visibility information. Autovacuum также предотвращает transaction ID wraparound. Долгая транзакция удерживает старый snapshot и мешает очистке.

## ACID и уровни изоляции

- **Atomicity:** транзакция применяется целиком или не применяется.
- **Consistency:** ограничения и бизнес-инварианты переходят из допустимого состояния в допустимое.
- **Isolation:** конкурентный результат соответствует гарантиям уровня.
- **Durability:** подтверждённые изменения переживают сбой в пределах настроек хранения.

| Уровень PostgreSQL | Главное ожидание |
|---|---|
| Read Committed | каждый statement получает актуальный snapshot |
| Repeatable Read | один snapshot на транзакцию; возможна serialization failure при конфликте |
| Serializable | результат эквивалентен некоторому последовательному выполнению; нужен retry |

Изоляция не заменяет ограничения. Уникальность защищайте `UNIQUE`, ссылки — `FOREIGN KEY`, допустимость — `CHECK`, а не только предварительным `SELECT`.

## Locks и deadlocks

Row locks защищают изменяемые строки, table locks — операции над таблицей/схемой. Deadlock возникает при циклическом ожидании; PostgreSQL обнаруживает цикл и отменяет одну транзакцию.

Снижайте риск единым порядком обновления, короткими транзакциями и отсутствием сетевых вызовов внутри них. Приложение должно уметь повторить операцию после deadlock/serialization failure, если команда идемпотентна.

## Partitioning, replication и pool

Partitioning помогает pruning, обслуживанию и управлению жизненным циклом больших таблиц. Оно не является автоматическим ускорителем каждого запроса и усложняет ключи, индексы и миграции.

Streaming replication передаёт WAL на replicas. Асинхронная replica может отставать, поэтому read-after-write не гарантирован при чтении с неё. Failover требует решения, кто становится primary, как предотвращается split brain и как клиенты обновляют маршрут.

Connection pool ограничивает дорогие соединения. Максимум HikariCP нельзя выбирать как «чем больше, тем лучше»: учитывайте лимит БД, число instances, длительность запросов и ожидающее время. Backpressure лучше бесконтрольного роста connections.

## Вопросы для самопроверки

1. Почему фильтр в `WHERE` меняет смысл `LEFT JOIN`?
2. Когда composite index `(status, created_at)` поможет сортировке?
3. Что означает большая разница estimated/actual rows?
4. Почему долгая idle transaction опасна для MVCC?
5. Как обеспечить retry serializable transaction без двойного эффекта?

