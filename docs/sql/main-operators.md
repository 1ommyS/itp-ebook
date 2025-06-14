## 1. DDL (Data Definition Language)

DDL-операторы используются для определения и изменения структуры объектов базы данных (таблиц, индексов, схем и др.).

### 1.1. CREATE TABLE

**Назначение:** создание новой таблицы.

```sql
CREATE TABLE имя_таблицы (
    столбец1 тип_данных [CONSTRAINT условия],
    столбец2 тип_данных [NOT NULL] [DEFAULT значение],
    ...,
    PRIMARY KEY (столбец_с_ключом)
);
```

* **тип\_данных**: `INTEGER`, `VARCHAR(n)`, `TEXT`, `TIMESTAMP`, `BOOLEAN` и т.д.
* **CONSTRAINT**: `UNIQUE`, `CHECK`, `FOREIGN KEY` и т.д.
* **DEFAULT**: значение по умолчанию.

**Пример:**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,           -- автоинкрементируемый идентификатор
    username VARCHAR(50) NOT NULL,   -- логин пользователя
    email TEXT UNIQUE,               -- уникальный email
    created_at TIMESTAMP DEFAULT NOW()  -- дата создания
);
```

### 1.2. ALTER TABLE

**Назначение:** изменение структуры уже существующей таблицы.

```sql
ALTER TABLE имя_таблицы
    ADD COLUMN новый_столбец тип_данных [опции],
    DROP COLUMN имя_столбца,
    ALTER COLUMN имя_столбца TYPE новый_тип,
    RENAME COLUMN старое_имя TO новое_имя,
    RENAME TO новое_имя_таблицы;
```

**Примеры:**

```sql
-- Добавить столбец
ALTER TABLE users
    ADD COLUMN last_login TIMESTAMP;

-- Удалить столбец
ALTER TABLE users
    DROP COLUMN last_login;

-- Поменять тип столбца
ALTER TABLE users
    ALTER COLUMN username TYPE VARCHAR(100);

-- Переименовать столбец
ALTER TABLE users
    RENAME COLUMN username TO user_name;

-- Переименовать таблицу
ALTER TABLE users
    RENAME TO app_users;
```

### 1.3. DROP TABLE

**Назначение:** удаление таблицы и всех её данных.

```sql
DROP TABLE [IF EXISTS] имя_таблицы [CASCADE | RESTRICT];
```

* `IF EXISTS` предотвращает ошибку, если таблицы не существует.
* `CASCADE` удаляет зависимости (например, внешние ключи).
* `RESTRICT` (по умолчанию) запрещает удаление, если есть зависимости.

**Пример:**

```sql
DROP TABLE IF EXISTS app_users CASCADE;
```

### 1.4. TRUNCATE TABLE

**Назначение:** быстрое удаление **всех** строк из таблицы, при этом структура остаётся.

```sql
TRUNCATE TABLE имя_таблицы [RESTART IDENTITY] [CASCADE];
```

* `RESTART IDENTITY` сбрасывает последовательности (SERIAL).
* `CASCADE` применяет к зависимым таблицам.

**Пример:**

```sql
TRUNCATE TABLE logs RESTART IDENTITY;
```

### 1.5. CREATE INDEX / DROP INDEX

**Назначение:** ускорение поиска по таблице за счёт создания дополнительных структур.

```sql
CREATE INDEX имя_индекса
    ON имя_таблицы (столбец1 [ASC|DESC], столбец2 ...)
    [USING тип_метода]    -- например, btree, hash, gin, gist
    [WHERE условие];      -- частичный индекс

DROP INDEX [IF EXISTS] имя_индекса;
```

**Примеры:**

```sql
-- Простой B-tree индекс по email
CREATE INDEX idx_users_email
    ON users (email);

-- Частичный индекс только по активным записям
CREATE INDEX idx_active_users
    ON users (created_at)
    WHERE active = TRUE;

-- Удаление индекса
DROP INDEX IF EXISTS idx_active_users;
```

---

## 2. DML (Data Manipulation Language)

DML-операторы работают с данными: добавляют, изменяют, удаляют и извлекают строки.

### 2.1. SELECT

**Назначение:** извлечение данных из одной или нескольких таблиц.

```sql
SELECT [DISTINCT] выражение1 [AS псевдоним], выражение2, ...
FROM таблица1 [AS псевдоним1]
    [JOIN таблица2 ON условие]
WHERE условие
GROUP BY выражения
HAVING условие_для_групп
ORDER BY выражения [ASC|DESC]
LIMIT количество [OFFSET смещение];
```

**Объяснение ключевых частей:**

* `DISTINCT` — убрать дубликаты.
* `JOIN` — объединение таблиц (`INNER`, `LEFT`, `RIGHT`, `FULL`).
* `GROUP BY` / `HAVING` — агрегация.
* `ORDER BY` — сортировка.
* `LIMIT` / `OFFSET` — пагинация.

**Примеры:**

```sql
-- Простой запрос
SELECT id, username, created_at
FROM users
WHERE active = TRUE
ORDER BY created_at DESC
LIMIT 10;

-- С агрегатами: количество пользователей по дням
SELECT DATE(created_at) AS day, COUNT(*) AS cnt
FROM users
GROUP BY day
HAVING COUNT(*) > 5
ORDER BY day;
```

### 2.2. INSERT

**Назначение:** добавление строк в таблицу.

```sql
INSERT INTO имя_таблицы (столбец1, столбец2, ...)
VALUES (знач1, знач2, ...),  -- можно несколько строк
       (знач1_2, знач2_2, ...),
... 
[RETURNING *|список_столбцов];
```

* `RETURNING` позволяет сразу получить вставленные значения (например, сгенерированные `SERIAL`).

**Примеры:**

```sql
-- Одна строка
INSERT INTO users (username, email)
VALUES ('ivan', 'ivan@example.com')
RETURNING id, created_at;

-- Несколько строк
INSERT INTO users (username, email)
VALUES 
    ('maria', 'maria@example.com'),
    ('sergey', 'sergey@example.com');
```

### 2.3. UPDATE

**Назначение:** изменение данных в существующих строках.

```sql
UPDATE имя_таблицы
SET столбец1 = выражение1,
    столбец2 = выражение2,
    ...
[FROM другая_таблица]      -- для обновления по JOIN
WHERE условие
[RETURNING *|список_столбцов];
```

**Примеры:**

```sql
-- Обновить email пользователя по id
UPDATE users
SET email = 'new@example.com'
WHERE id = 123;

-- Обновить поле в зависимости от другой таблицы
UPDATE orders o
SET status = 'processed'
FROM users u
WHERE o.user_id = u.id
  AND u.active = TRUE
RETURNING o.id, o.status;
```

### 2.4. DELETE

**Назначение:** удаление строк из таблицы.

```sql
DELETE FROM имя_таблицы
WHERE условие
[RETURNING *|список_столбцов];
```

**Пример:**

```sql
-- Удалить все неактивные записи старше года
DELETE FROM users
WHERE active = FALSE
  AND created_at < NOW() - INTERVAL '1 year';
```

### 2.5. UPSERT (INSERT ... ON CONFLICT)

**Назначение:** вставка строки или обновление при конфликте по уникальному ограничению.

```sql
INSERT INTO имя_таблицы (столбец1, столбец2, ...)
VALUES (знач1, знач2, ...)
ON CONFLICT (столбец_с_ограничением)
DO UPDATE SET
    столбец2 = EXCLUDED.столбец2,   -- EXCLUDED — значения из попытки вставки
    ...
[WHERE условие];
```

**Примеры:**

```sql
-- Если пользователь с таким email уже есть, обновить username
INSERT INTO users (username, email)
VALUES ('kolya', 'kolya@example.com')
ON CONFLICT (email)
DO UPDATE SET
    username = EXCLUDED.username
RETURNING id, username;
```

---

### Полезные советы и примечания

* Всегда используйте **транзакции** (`BEGIN` / `COMMIT` / `ROLLBACK`) при выполнении нескольких связанных DML-операций, чтобы сохранить целостность данных.
* Для массовой загрузки данных применяйте `COPY` (не DML) — это отдельная высокопроизводительная команда.
* Пользуйтесь **`EXPLAIN ANALYZE`** для оптимизации запросов (анализ плана выполнения).
* Не забывайте про **индексы**: слишком много индексов замедляет вставку/обновление, слишком мало — замедляет выборку.
* При миграциях схем используйте инструменты вроде **Liquibase** или **Flyway**, чтобы версионировать DDL.
* DDL не работает в транзакциях