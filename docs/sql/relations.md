## 1. 1:1 (один к одному)

### 1.1. Определение

Каждый экземпляр сущности A соотносится не более чем с одним экземпляром сущности B, и наоборот.

### 1.2. Примеры

* Пользователь ↔ Профиль (Profile)
* Расширенный документ ↔ Метаданные (DocumentMeta)

### 1.3. Реализация в PostgreSQL

1. **Разделённые таблицы с общим PK**

   ```sql
   CREATE TABLE users (
     id SERIAL PRIMARY KEY,
     username TEXT NOT NULL UNIQUE
   );

   CREATE TABLE user_profiles (
     user_id INT PRIMARY KEY,           -- точно один профиль на пользователя
     full_name TEXT,
     birth_date DATE,
     CONSTRAINT fk_user
       FOREIGN KEY (user_id) 
       REFERENCES users(id)
       ON DELETE CASCADE
   );
   ```

   Здесь `user_profiles.user_id` одновременно PK и FK → жёсткая связь 1:1.

2. **Уникальный FK во вспомогательной таблице**

   ```sql
   CREATE TABLE passports (
     id SERIAL PRIMARY KEY,
     number TEXT NOT NULL UNIQUE,
     user_id INT UNIQUE,                -- уникальный внешний ключ
     CONSTRAINT fk_user_passport FOREIGN KEY (user_id)
       REFERENCES users(id)
   );
   ```

### 1.4. Плюсы и минусы

* **Плюсы:**

    * Логическая разбивка «часто используемые» / «редко используемые» поля.
    * Изоляция чувствительных данных (например, документы и их шифрованные ключи).
* **Минусы:**

    * При каждом JOIN – дополнительный I/O.
    * Если связь всегда нужна, бывает проще держать все поля в одной таблице.

---

## 2. 1\:N (один ко многим)

### 2.1. Определение

Каждому экземпляру сущности A может соответствовать любое количество экземпляров B, но каждому B — не более одного A.

### 2.2. Примеры

* Автор ↔ Книги
* Категория ↔ Товары

### 2.3. Реализация в PostgreSQL

```sql
CREATE TABLE authors (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE books (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  author_id INT NOT NULL,
  CONSTRAINT fk_book_author FOREIGN KEY (author_id)
    REFERENCES authors(id)
    ON DELETE RESTRICT
);
```

* Здесь `books.author_id` — внешний ключ, указывающий на одного автора, а у автора может быть множество книг.

### 2.4. Плюсы и минусы

* **Плюсы:**

    * Простота: в большинстве ER-моделей связь «один-ко-многим» — основная.
    * Эффективные индексы: B-Tree по `author_id` ускоряет поиск всех книг автора.
* **Минусы:**

    * Нет прямой ссылки из родителя на список детей: для извлечения списка всегда нужен JOIN.
    * При массовом удалении родителя нужно заботиться о поведении (`CASCADE` vs `RESTRICT`).

---

## 3. N\:M (многие ко многим)

### 3.1. Определение

Каждому экземпляру A может соответствовать множество B, и каждому B — множество A.

### 3.2. Примеры

* Студенты ↔ Курсы
* Поставщики ↔ Продукты

### 3.3. Реализация в PostgreSQL через «связующую таблицу»

```sql
CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE courses (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL
);

CREATE TABLE student_courses (
  student_id INT NOT NULL,
  course_id  INT NOT NULL,
  enrolled_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (student_id, course_id),   -- составной PK
  CONSTRAINT fk_sc_student FOREIGN KEY (student_id)
    REFERENCES students(id) ON DELETE CASCADE,
  CONSTRAINT fk_sc_course FOREIGN KEY (course_id)
    REFERENCES courses(id) ON DELETE CASCADE
);
```

* **JOIN**:

  ```sql
  SELECT s.name, c.title
  FROM students s
  JOIN student_courses sc ON sc.student_id = s.id
  JOIN courses c         ON sc.course_id  = c.id;
  ```

### 3.4. Плюсы и минусы

* **Плюсы:**

    * Гибкость: можно добавлять атрибуты связи (`enrolled_at`).
    * Явно нормализованная схема (третья нормальная форма).
* **Минусы:**

    * Дополнительная таблица → больше JOIN.
    * При большом числе связей (миллионы) — рост размеров связующей таблицы.

---

## 4. Опциональные связи: 0:1 и 0\:N

### 4.1. 0:1 (ноль или один)

– похоже на 1:1, но допускать отсутствие связи:

```sql
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  ceo_of_company_id INT UNIQUE,  -- у CEO может не быть своей компании
  CONSTRAINT fk_ceo FOREIGN KEY (ceo_of_company_id)
    REFERENCES companies(id)
);
```

### 4.2. 0\:N (ноль или много)

– стандартный 1\:N, но допускающий отсутствие детей: просто `FOREIGN KEY` без `NOT NULL`:

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  customer_id INT,                 -- заказ может быть без привязанного клиента
  CONSTRAINT fk_order_customer FOREIGN KEY (customer_id)
    REFERENCES customers(id)
);
```

---

## 5. Самоссылки (рекурсивные связи)

### 5.1. Пример: иерархия сотрудников

```sql
CREATE TABLE employees (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  manager_id INT,
  CONSTRAINT fk_manager FOREIGN KEY (manager_id)
    REFERENCES employees(id)
);
```

* Позволяет строить древовидные структуры: каждый сотрудник может иметь «управляющего» в той же таблице.

### 5.2. Запрос с рекурсивным CTE

```sql
WITH RECURSIVE subordinates AS (
  SELECT id, name, manager_id
  FROM employees
  WHERE id = 1            -- топовый менеджер
  UNION ALL
  SELECT e.id, e.name, e.manager_id
  FROM employees e
  JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates;
```

---

## 6. Полиморфные связи

Иногда одну стороннюю таблицу хотят связать с разными сущностями. В PostgreSQL это реализуют через:

1. **Две колонки + CHECK Constraint**

   ```sql
   CREATE TABLE comments (
     id SERIAL PRIMARY KEY,
     target_type TEXT NOT NULL,        -- 'article' или 'product'
     target_id   INT  NOT NULL,
     content TEXT NOT NULL,
     CONSTRAINT chk_target_type CHECK (target_type IN ('article','product'))
   );
   ```

   – но без FK, целостность приходится проверять на уровне приложения или триггеров.

2. **Отдельный столбец-ссылка для каждого типа**

   ```sql
   CREATE TABLE comments (
     id SERIAL PRIMARY KEY,
     article_id INT REFERENCES articles(id),
     product_id INT REFERENCES products(id),
     CHECK (
       (article_id IS NOT NULL)::int +
       (product_id IS NOT NULL)::int = 1
     )
   );
   ```

3. **UUID-ссылка + single sequence + универсальный PK** (через `REFERENCES any_table(id)` нельзя — требует PL/pgSQL проверки).

---

## 7. Выбор правильной кардинальности

1. **Определяйте связи по предметной области** — если у пользователей может быть не более одной учётной записи, это 1:1; если множество заказов — 1\:N.
2. **Нормализуйте схему**:

    * Избегайте избыточности (N\:M с атрибутами → отдельная связующая таблица).
    * Учитывайте частоту JOIN: если связь всегда нужна, можно вложить поля напрямую.
3. **Контроль целостности**:

    * Всегда ставьте `FOREIGN KEY`.
    * Прописывайте `ON DELETE`/`ON UPDATE` (CASCADE, RESTRICT).
4. **Производительность**:

    * Индексируйте внешние ключи (`book.author_id` → `CREATE INDEX ON books(author_id)`).
    * Для N\:M — индексируйте обе колонки связующей таблицы.
