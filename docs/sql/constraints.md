## 1. NOT NULL

**Описание:**
Гарантирует, что в столбце не будет храниться `NULL`.

**Синтаксис:**

```sql
CREATE TABLE имя_таблицы (
    столбец1 тип_данных NOT NULL,
    …
);
```

или

```sql
ALTER TABLE имя_таблицы
    ALTER COLUMN столбец1 SET NOT NULL;
```

**Пример:**

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,       -- имя обязательно к заполнению
    price NUMERIC NOT NULL
);
```

**Советы:**

* Используйте для полей, которые необходимы всегда — имя пользователя, дата создания и т. п.
* Перед установкой `NOT NULL` на существующий столбец убедитесь, что в нём нет `NULL` (иначе будет ошибка).

---

## 2. UNIQUE

**Описание:**
Гарантирует, что все значения в столбце (или сочетании столбцов) уникальны.

**Синтаксис:**

```sql
CREATE TABLE имя_таблицы (
    столбец1 тип_данных UNIQUE,
    …
);
```

или

```sql
ALTER TABLE имя_таблицы
    ADD CONSTRAINT имя_уникальности UNIQUE (столбец1 [, столбец2, …]);
```

**Пример:**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,   -- два одинаковых email не зарегистрировать
    username VARCHAR(50)
);

-- составной UNIQUE
ALTER TABLE orders
    ADD CONSTRAINT uq_order UNIQUE (user_id, product_id);
```

**Советы:**

* Для составного `UNIQUE` важно давать читаемое имя (`uq_...`).
* PostgreSQL автоматически создаёт индекс для ограничения `UNIQUE`, поэтому дополнительных действий не требуется.

---

## 3. PRIMARY KEY

**Описание:**
Комбинирует в себе `UNIQUE` + `NOT NULL`; однозначно идентифицирует строку в таблице.

**Синтаксис:**

```sql
CREATE TABLE имя_таблицы (
    столбец1 тип_данных,
    столбец2 тип_данных,
    CONSTRAINT имя_pk PRIMARY KEY (столбец1 [, столбец2, …])
);
```

или кратко:

```sql
CREATE TABLE имя_таблицы (
    id SERIAL PRIMARY KEY,
    …
);
```

**Пример:**

```sql
CREATE TABLE customers (
    customer_id UUID PRIMARY KEY,
    name TEXT NOT NULL
);
```

**Советы:**

* Используйте `SERIAL`/`BIGSERIAL` или `UUID` для автогенерации.
* Составной ключ (`(col1, col2)`) замедляет ссылки и индексы — рассматривайте суррогатные ключи (идентификаторы).

---

## 4. FOREIGN KEY

**Описание:**
Обеспечивает ссылочную целостность: значение в одном столбце (или группе столбцов) должно существовать в другом, “родительском” столбце.

**Синтаксис:**

```sql
CREATE TABLE дочерняя_таблица (
    fk_col тип_данных,
    CONSTRAINT имя_fk FOREIGN KEY (fk_col)
      REFERENCES родительская_таблица (pk_col)
      [ON DELETE действие]
      [ON UPDATE действие]
);
```

Где `действие` может быть:

* `CASCADE` — при удалении/обновлении родителя действие распространяется на детей.
* `SET NULL` — при удалении/обновлении родителя в детских строках ставится `NULL`.
* `RESTRICT` (по умолчанию) — не позволит удалить/обновить родителя, если есть дети.
* `NO ACTION` — аналог `RESTRICT`.

**Пример:**

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    CONSTRAINT fk_order_user FOREIGN KEY (user_id)
      REFERENCES users (id)
      ON DELETE CASCADE
      ON UPDATE CASCADE
);
```

**Советы:**

* Выбирайте `ON DELETE/UPDATE` внимательно: `CASCADE` удобно, но может непреднамеренно снести данные.
* Для повышения производительности внешние ключи иногда временно отключают при загрузке больших объёмов; не забудьте включить обратно.

---

## 5. CHECK

**Описание:**
Позволяет задать произвольное логическое условие, которое каждое значение или группа значений должны удовлетворять.

**Синтаксис:**

```sql
CREATE TABLE имя_таблицы (
    столбец тип_данных,
    CONSTRAINT имя_check CHECK (условие)
);
```

или inline:

```sql
столбец тип_данных CHECK (условие)
```

**Пример:**

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    salary NUMERIC CHECK (salary >= 0),                    -- зарплата не отрицательная
    hire_date DATE NOT NULL CHECK (hire_date <= CURRENT_DATE)
);

-- проверка на диапазон
ALTER TABLE products
    ADD CONSTRAINT chk_price_range CHECK (price BETWEEN 0 AND 1000);
```

**Советы:**

* Условия могут обращаться к любым столбцам той же строки: `CHECK (start_date < end_date)`.
* `CHECK` выполняется и при `INSERT`, и при `UPDATE`.

---

## 6. EXCLUDE (исключающие)

**Описание:**
Гибкое расширение `UNIQUE`: запрещает комбинации столбцов с учётом операторов сравнения (не только `=`). Используется, например, для гарантии отсутствия перекрывающихся временных интервалов.

**Синтаксис:**

```sql
CREATE TABLE имя_таблицы (
    …,
    EXCLUDE USING индекс_тип (
        столбец1 WITH оператор1,
        столбец2 WITH оператор2,
        …
    )
    [WHERE условие]
);
```

* `индекс_тип`: обычно `GIST` или `SPGIST`.
* `оператор`: `=`, `<>`, `&&` (пересечение диапазонов), `<@`, `@>`, и т.д.

**Пример (некросекающиеся интервалы):**

```sql
CREATE TABLE room_booking (
    room_id INT,
    period TSRANGE,
    EXCLUDE USING GIST (
        room_id WITH =,            -- одинаковая комната
        period  WITH &&           -- перекрытие интервалов
    )
);
```

**Советы:**

* Поддерживает большие объёмы данных с `GIST`/`SPGIST` индексами.
* Часто применяется для планировщиков и расписаний.

---

## 7. DOMAIN-ограничения

**Описание:**
Позволяют создавать собственные типы данных с встроенными ограничениями (`NOT NULL`, `CHECK`, `DEFAULT`).

**Синтаксис:**

```sql
CREATE DOMAIN имя_домена AS базовый_тип
    [NOT NULL]
    [DEFAULT выражение]
    [CONSTRAINT имя_check CHECK (условие)];
```

**Пример:**

```sql
CREATE DOMAIN email_addr AS TEXT
    NOT NULL
    CONSTRAINT valid_email CHECK (VALUE ~* '^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$');

CREATE TABLE contacts (
    id SERIAL PRIMARY KEY,
    contact_email email_addr    -- тип с валидацией
);
```
