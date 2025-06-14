### 1. Партиционирование в PostgreSQL

PostgreSQL поддерживает три базовых метода декларативного партиционирования:

* **RANGE** — диапазоны значений (например, даты) без пересечений.
* **LIST** — явный список значений для каждой партиции.
* **HASH** — разбиение по хеш-значению, заданному модулю и остатку.

```sql
CREATE TABLE measurement (
    city_id   INT    NOT NULL,
    logdate   DATE   NOT NULL,
    peaktemp  INT,
    unitsales INT
) PARTITION BY RANGE (logdate);
```

При этом сама таблица `measurement` — виртуальная, хранилище есть только у её партиций, которые создаются так:

```sql
CREATE TABLE measurement_2024m01 PARTITION OF measurement
  FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

Любая вставка в `measurement` автоматически перенаправляется в нужную партицию по значению `logdate`, а изменение ключа — перемещает строку в другую партицию ([postgresql.org][1]).

---

### 2. Индексы на партиционированных таблицах

1. **Декларативный (разделённый) индекс**
   При выполнении

   ```sql
   CREATE INDEX ON measurement (logdate);
   ```

   PostgreSQL автоматически создаёт «виртуальный» индекс на таблице-родителе и отдельные физические индексы на каждой существующей партиции, а также на вновь прикрепляемых партициях ([postgresql.org][1]).

2. **Ограничения и CONCURRENTLY**

    * Нельзя сразу создать индекс `CONCURRENTLY` на родителе — он блокирует всё дерево.
    * Хитрость:

      ```sql
      CREATE INDEX ON ONLY measurement (unitsales);         -- создаёт невалидный родительский индекс
      CREATE INDEX CONCURRENTLY idx_202401 ON measurement_2024m01 (unitsales);
      ALTER INDEX measurement_unitsales_idx
        ATTACH PARTITION idx_202401;
      ```

      После прикрепления всех партиций родительский индекс автоматически станет валидным ([postgresql.org][1]).

3. **Уникальные и первичные ключи**
   Декларативные `PRIMARY KEY` и `UNIQUE` на разделённой таблице создают локальные уникальные индексы на каждой партиции. При этом **обязательное** условие: столбцы ограничения должны включать все поля партиционного ключа ([postgresql.org][1]).

4. **Глобальные индексы**
   В PostgreSQL **нет** поддержки единого (глобального) индекса, охватывающего все партиции. Каждый индекс по-прежнему локален для своей партиции, а «родительский» лишь агрегирует метаданные ([stackoverflow.com][2]).

---

### 3. «Мерж» (объединение) индексов при выполнении запросов

В PostgreSQL нет отдельной команды `MERGE INDEX`, но планировщик умеет **объединять результаты нескольких индексных сканов** через bitmap-структуры:

1. **Bitmap Index Scan + Bitmap Heap Scan**

    * Для каждого условия (например, `col1 = 5`) делается `Bitmap Index Scan`, строящий в памяти bitmap подходящих строк.
    * Далее `Bitmap Heap Scan` по этому bitmap читает страницы и возвращает реальные строки ([pganalyze.com][3]).

2. **BitmapAnd / BitmapOr**

    * При `WHERE a = 1 AND b = 2` план может сделать два `Bitmap Index Scan` и **пересечь** их битовые карты (`BitmapAnd`), чтобы получить строки, соответствующие обоим условиям.
    * При `WHERE x = 10 OR x = 20` — **объединить** результаты (`BitmapOr`).
    * После комбинирования битовых карт выполняется единичный `Bitmap Heap Scan` ([postgresql.org][4]).

3. **Когда применять**

    * **Плюсы:** эффективно при небольшом числе подходящих строк (I/O сгруппировано по страницам).
    * **Минусы:** теряется физический порядок строк — при наличии `ORDER BY` нужен дополнительный `Sort`.
    * **Совет:** если сочетание условий часто повторяется, стоит создать **многоколоночный** индекс `(a, b)` вместо двух отдельных, чтобы дать планировщику прямой `Index Scan` без bitmap-сборки ([knock.app][5], [percona.com][6]).

---

#### Резюме

* **Партиционирование** упрощает поддержание больших таблиц за счёт разбивки на подтаблицы по ключу.
* **Индексы** на разделённой таблице — это динамический «родительский» индекс, отражающий локальные индексы на каждой партиции.
* **Глобальных индексов** нет, но PostgreSQL объединяет (merge) локальные индексные сканы на уровне плана через bitmap-операции.
* Для **часто комбинируемых условий** лучше использовать `multicolumn index`, а для редких — полагаться на механизм `BitmapAnd`/`BitmapOr`.

Если нужны примеры конкретных планов `EXPLAIN` или помощь в оптимизации под вашу схему — спрашивайте!

[1]: https://www.postgresql.org/docs/current/ddl-partitioning.html "PostgreSQL: Documentation: 17: 5.12. Table Partitioning"
[2]: https://stackoverflow.com/questions/62909113/postgresql-index-on-partition-table?utm_source=chatgpt.com "Postgresql : Index on partition table - Stack Overflow"
[3]: https://pganalyze.com/docs/explain/scan-nodes/bitmap-index-scan?utm_source=chatgpt.com "EXPLAIN - Bitmap Index Scan - pganalyze"
[4]: https://www.postgresql.org/docs/current/indexes-bitmap-scans.html?utm_source=chatgpt.com "Documentation: 17: 11.5. Combining Multiple Indexes - PostgreSQL"
[5]: https://knock.app/blog/postgres-index-findings?utm_source=chatgpt.com "Six findings we rely on when managing PostgreSQL indexes - Knock"
[6]: https://www.percona.com/blog/one-index-three-different-postgresql-scan-types-bitmap-index-and-index-only/?utm_source=chatgpt.com "One Index, Three Different PostgreSQL Scan Types - Percona"
