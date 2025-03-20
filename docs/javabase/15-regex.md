### Регулярные выражения в Java

Регулярные выражения (регекс) — это мощный инструмент для поиска, проверки, извлечения и замены текста по заданным шаблонам. В Java поддержка регулярных выражений реализована через пакет `java.util.regex`, который включает основные классы `Pattern` и `Matcher`. В этом разделе мы рассмотрим основы регулярных выражений, их синтаксис, а также примеры использования в Java.

---

### 1. **Что такое регулярные выражения?**

**Регулярное выражение (regex)** — это последовательность символов, определяющая шаблон поиска. Регулярные выражения используются для выполнения сложных операций с текстом, таких как поиск подстрок, проверка формата данных, извлечение информации и замена частей строки.

#### Основные применения регулярных выражений:
- **Валидация данных**: Проверка корректности ввода, например, email-адресов, номеров телефонов, почтовых индексов.
- **Поиск и извлечение**: Нахождение определённых шаблонов в тексте и извлечение нужных данных.
- **Замена текста**: Изменение или удаление частей строки, соответствующих шаблону.
- **Разделение строк**: Разбиение строки на части по определённым разделителям.

---

### 2. **Пакет `java.util.regex`**

Java предоставляет поддержку регулярных выражений через пакет `java.util.regex`, который включает следующие основные классы:

- **`Pattern`**: Компилирует регулярное выражение в шаблон, который можно использовать для создания объектов `Matcher`.
- **`Matcher`**: Используется для выполнения операций сопоставления с использованием шаблона `Pattern`.
- **`PatternSyntaxException`**: Исключение, выбрасываемое при синтаксической ошибке в регулярном выражении.

#### Основные методы класса `Pattern`:
- **`compile(String regex)`**: Компилирует регулярное выражение в объект `Pattern`.
- **`matches(String regex, CharSequence input)`**: Проверяет, соответствует ли вся входная строка заданному шаблону.
- **`split(CharSequence input)`**: Разбивает строку по шаблону и возвращает массив строк.
- **`pattern()`**: Возвращает строковое представление шаблона.

#### Основные методы класса `Matcher`:
- **`matches()`**: Проверяет, соответствует ли вся строка шаблону.
- **`find()`**: Ищет следующее вхождение шаблона в строке.
- **`group()`**: Возвращает последнюю найденную подстроку.
- **`replaceAll(String replacement)`**: Заменяет все вхождения шаблона на заданную строку.
- **`replaceFirst(String replacement)`**: Заменяет первое вхождение шаблона на заданную строку.
- **`start()` и `end()`**: Возвращают начальный и конечный индексы последнего найденного совпадения.

---

### 3. **Синтаксис регулярных выражений**

Регулярные выражения состоят из обычных символов и специальных метасимволов, которые имеют особое значение. Ниже приведены основные компоненты и синтаксис регулярных выражений.

#### 3.1. **Литералы и экранирование**

- **Литералы**: Символы, которые соответствуют самим себе. Например, `a` соответствует символу `'a'`.
- **Экранирование**: Некоторые символы имеют специальное значение и должны быть экранированы обратным слэшем (`\`), если нужно использовать их как обычные символы. Например, `\.` соответствует символу точки `'.'`.

#### 3.2. **Метасимволы**

| **Метасимвол** | **Описание**                                     |
|----------------|--------------------------------------------------|
| `.`            | Любой одиночный символ, кроме перевода строки     |
| `^`            | Начало строки                                    |
| `$`            | Конец строки                                      |
| `*`            | 0 или более повторений предыдущего элемента      |
| `+`            | 1 или более повторений предыдущего элемента      |
| `?`            | 0 или 1 повторение предыдущего элемента или    |
| `{n}`          | Ровно `n` повторений                             |
| `{n,}`         | `n` или более повторений                         |
| `{n,m}`        | От `n` до `m` повторений                         |
| `[]`           | Класс символов: любой символ из набора            |
| `[^]`          | Отрицание класса символов                        |
| `|`            | Логическое "ИЛИ" между выражениями                |
| `()`           | Группировка подвыражений                          |
| `\d`           | Цифра (эквивалентно `[0-9]`)                      |
| `\D`           | Не цифра (эквивалентно `[^0-9]`)                  |
| `\w`           | Слово-символ (буква, цифра или `_`)              |
| `\W`           | Не слово-символ                                   |
| `\s`           | Пробельный символ (пробел, табуляция и т.д.)      |
| `\S`           | Не пробельный символ                              |
| `\b`           | Граница слова                                     |
| `\B`           | Не граница слова                                  |
| `(?i)`         | Режим нечувствительности к регистру               |

#### 3.3. **Квантификаторы**

Квантификаторы определяют, сколько раз элемент может повторяться.

| **Квантификатор** | **Пример** | **Описание**                                     |
|-------------------|------------|--------------------------------------------------|
| `*`               | `a*`       | 0 или более `'a'`                                 |
| `+`               | `a+`       | 1 или более `'a'`                                 |
| `?`               | `a?`       | 0 или 1 `'a'`                                     |
| `{n}`             | `a{3}`     | Ровно 3 `'a'`                                     |
| `{n,}`            | `a{2,}`    | 2 или более `'a'`                                 |
| `{n,m}`           | `a{1,3}`   | От 1 до 3 `'a'`                                   |

#### 3.4. **Классы символов**

Классы символов позволяют определить набор допустимых символов в определённой позиции.

- `[abc]`: Любой из символов `'a'`, `'b'`, `'c'`.
- `[a-z]`: Любая строчная латинская буква.
- `[A-Z]`: Любая заглавная латинская буква.
- `[0-9]`: Любая цифра.
- `[^a-z]`: Любой символ, кроме строчных латинских букв.

#### 3.5. **Группировка и захват**

- `(abc)`: Группа символов `'abc'`. Позволяет применять к группе квантификаторы или извлекать подстроки.
- `(?:abc)`: Негруппирующаяся группа. Группирует символы без захвата подстроки.
- `(a|b)`: Логическое "ИЛИ" между `'a'` и `'b'`.

---

### 4. **Примеры использования регулярных выражений в Java**

#### 4.1. **Валидация email-адреса**

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class EmailValidator {
    private static final String EMAIL_REGEX = "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$";
    private static final Pattern EMAIL_PATTERN = Pattern.compile(EMAIL_REGEX);

    public static boolean isValidEmail(String email) {
        Matcher matcher = EMAIL_PATTERN.matcher(email);
        return matcher.matches();
    }

    public static void main(String[] args) {
        String[] emails = { "user@example.com", "user.name@domain.co", "user#domain.com", "user@.com" };
        for (String email : emails) {
            System.out.println(email + " is valid? " + isValidEmail(email));
        }
    }
}
```

**Вывод:**
```
user@example.com is valid? true
user.name@domain.co is valid? true
user#domain.com is valid? false
user@.com is valid? false
```

#### 4.2. **Извлечение номеров телефонов из текста**

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class PhoneNumberExtractor {
    private static final String PHONE_REGEX = "\\b\\d{3}[-.]?\\d{3}[-.]?\\d{4}\\b";
    private static final Pattern PHONE_PATTERN = Pattern.compile(PHONE_REGEX);

    public static void extractPhoneNumbers(String text) {
        Matcher matcher = PHONE_PATTERN.matcher(text);
        while (matcher.find()) {
            System.out.println("Найден номер телефона: " + matcher.group());
        }
    }

    public static void main(String[] args) {
        String text = "Свяжитесь с нами по телефонам: 123-456-7890, 9876543210 или 555.666.7777.";
        extractPhoneNumbers(text);
    }
}
```

**Вывод:**
```
Найден номер телефона: 123-456-7890
Найден номер телефона: 9876543210
Найден номер телефона: 555.666.7777
```

#### 4.3. **Замена всех пробельных символов на один пробел**

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class WhitespaceNormalizer {
    private static final String WHITESPACE_REGEX = "\\s+";
    private static final Pattern WHITESPACE_PATTERN = Pattern.compile(WHITESPACE_REGEX);

    public static String normalizeWhitespace(String input) {
        Matcher matcher = WHITESPACE_PATTERN.matcher(input);
        return matcher.replaceAll(" ");
    }

    public static void main(String[] args) {
        String text = "This   is a\t\ttext with\nvarious    whitespace characters.";
        String normalized = normalizeWhitespace(text);
        System.out.println("Нормализованный текст: " + normalized);
    }
}
```

**Вывод:**
```
Нормализованный текст: This is a text with various whitespace characters.
```

#### 4.4. **Разбиение строки по запятым с возможными пробелами**

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;
import java.util.Arrays;

public class StringSplitter {
    private static final String SPLIT_REGEX = "\\s*,\\s*";

    public static void main(String[] args) {
        String input = "apple, banana,  orange ,grape,melon";
        String[] fruits = input.split(SPLIT_REGEX);
        System.out.println("Фрукты: " + Arrays.toString(fruits));
    }
}
```

**Вывод:**
```
Фрукты: [apple, banana, orange, grape, melon]
```

---

### 5. **Работа с классами `Pattern` и `Matcher`**

#### 5.1. **Компиляция шаблона и создание `Matcher`**

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class PatternMatcherExample {
    public static void main(String[] args) {
        String regex = "\\d{3}-\\d{2}-\\d{4}"; // Шаблон для SSN (например, 123-45-6789)
        String text = "Мой SSN: 123-45-6789, а друг имеет 987-65-4321.";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        while (matcher.find()) {
            System.out.println("Найден SSN: " + matcher.group());
        }
    }
}
```

**Вывод:**
```
Найден SSN: 123-45-6789
Найден SSN: 987-65-4321
```

#### 5.2. **Использование флагов для модификации поведения шаблона**

Флаги позволяют изменять поведение регулярных выражений, например, делать их нечувствительными к регистру.

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class FlagExample {
    public static void main(String[] args) {
        String regex = "java";
        String text = "Java is a programming language. I love JAVA and JaVa.";

        // Создание шаблона с флагом CASE_INSENSITIVE
        Pattern pattern = Pattern.compile(regex, Pattern.CASE_INSENSITIVE);
        Matcher matcher = pattern.matcher(text);

        while (matcher.find()) {
            System.out.println("Найдено совпадение: " + matcher.group());
        }
    }
}
```

**Вывод:**
```
Найдено совпадение: Java
Найдено совпадение: JAVA
Найдено совпадение: JaVa
```

#### 5.3. **Группировка и извлечение подгрупп**

Регулярные выражения позволяют группировать части шаблона и извлекать их отдельно.

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class GroupingExample {
    public static void main(String[] args) {
        String regex = "(\\d{4})-(\\d{2})-(\\d{2})"; // Шаблон даты YYYY-MM-DD
        String text = "Сегодня 2025-01-14, а вчера была 2025-01-13.";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        while (matcher.find()) {
            String year = matcher.group(1);
            String month = matcher.group(2);
            String day = matcher.group(3);
            System.out.println("Год: " + year + ", Месяц: " + month + ", День: " + day);
        }
    }
}
```

**Вывод:**
```
Год: 2025, Месяц: 01, День: 14
Год: 2025, Месяц: 01, День: 13
```

---

### 6. **Класс `Matcher` и его методы**

Класс `Matcher` предоставляет различные методы для работы с найденными совпадениями.

#### 6.1. **Поиск всех совпадений**

Использование метода `find()` позволяет последовательно искать все совпадения в строке.

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class FindAllMatches {
    public static void main(String[] args) {
        String regex = "\\b\\w{5}\\b"; // Слова ровно из 5 букв
        String text = "Hello world, this is a regex example with several words.";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);

        while (matcher.find()) {
            System.out.println("Найдено слово: " + matcher.group());
        }
    }
}
```

**Вывод:**
```
Найдено слово: Hello
Найдено слово: world
Найдено слово: regex
```

#### 6.2. **Проверка наличия совпадения**

Метод `matches()` проверяет, соответствует ли вся строка шаблону.

```java
import java.util.regex.Pattern;

public class ExactMatchExample {
    public static void main(String[] args) {
        String regex = "\\d{3}-\\d{2}-\\d{4}";
        String validSSN = "123-45-6789";
        String invalidSSN = "123-456-789";

        System.out.println(validSSN + " is valid? " + Pattern.matches(regex, validSSN)); // true
        System.out.println(invalidSSN + " is valid? " + Pattern.matches(regex, invalidSSN)); // false
    }
}
```

**Вывод:**
```
123-45-6789 is valid? true
123-456-789 is valid? false
```

#### 6.3. **Замена совпадений**

Методы `replaceAll()` и `replaceFirst()` позволяют заменять найденные совпадения на заданные строки.

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class ReplacementExample {
    public static void main(String[] args) {
        String regex = "\\d{3}-\\d{2}-\\d{4}";
        String replacement = "***-**-****";
        String text = "My SSN is 123-45-6789 and your SSN is 987-65-4321.";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(text);
        String replacedText = matcher.replaceAll(replacement);

        System.out.println("Замененный текст: " + replacedText);
    }
}
```

**Вывод:**
```
Замененный текст: My SSN is ***-**-**** and your SSN is ***-**-****.
```

---

### 7. **Практические примеры использования регулярных выражений**

#### 7.1. **Валидация номера телефона**

Предположим, что номер телефона должен иметь формат `(XXX) XXX-XXXX` или `XXX-XXX-XXXX`.

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class PhoneNumberValidator {
    private static final String PHONE_REGEX = "^(\\(\\d{3}\\)\\s?|\\d{3}-)\\d{3}-\\d{4}$";
    private static final Pattern PHONE_PATTERN = Pattern.compile(PHONE_REGEX);

    public static boolean isValidPhoneNumber(String phone) {
        Matcher matcher = PHONE_PATTERN.matcher(phone);
        return matcher.matches();
    }

    public static void main(String[] args) {
        String[] phones = { "(123) 456-7890", "123-456-7890", "1234567890", "(123)456-7890" };
        for (String phone : phones) {
            System.out.println(phone + " is valid? " + isValidPhoneNumber(phone));
        }
    }
}
```

**Вывод:**
```
(123) 456-7890 is valid? true
123-456-7890 is valid? true
1234567890 is valid? false
(123)456-7890 is valid? false
```

#### 7.2. **Извлечение URL из текста**

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class URLExtractor {
    private static final String URL_REGEX = "\\bhttps?://[\\w.-]+(?:\\.[\\w.-]+)+[/\\w .-]*\\b";
    private static final Pattern URL_PATTERN = Pattern.compile(URL_REGEX, Pattern.CASE_INSENSITIVE);

    public static void extractURLs(String text) {
        Matcher matcher = URL_PATTERN.matcher(text);
        while (matcher.find()) {
            System.out.println("Найден URL: " + matcher.group());
        }
    }

    public static void main(String[] args) {
        String text = "Посетите наш сайт по адресу https://www.example.com или http://sub.domain.org/page.";
        extractURLs(text);
    }
}
```

**Вывод:**
```
Найден URL: https://www.example.com
Найден URL: http://sub.domain.org/page
```

---

### 8. **Оптимизация и советы по использованию регулярных выражений**

- **Избегайте избыточности**: Старайтесь использовать минимально необходимый шаблон для выполнения задачи.
- **Используйте предкомпиляцию шаблонов**: Компиляция регулярного выражения с помощью `Pattern.compile()` позволяет использовать один шаблон многократно без повторной компиляции, что повышает производительность.
- **Применяйте ленивые квантификаторы**: В случаях, когда необходимо минимально возможное совпадение, используйте ленивые квантификаторы (`*?`, `+?`).
- **Экранируйте специальные символы**: Если необходимо искать специальные символы (например, точку, скобки), используйте обратный слэш для экранирования (`\.`).
- **Используйте комментарии и именованные группы**: Для повышения читабельности сложных регулярных выражений можно использовать комментарии и именованные группы (начиная с Java 7).

#### Пример именованных групп:

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class NamedGroupExample {
    public static void main(String[] args) {
        String regex = "(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})";
        String date = "2025-01-14";

        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(date);

        if (matcher.matches()) {
            String year = matcher.group("year");
            String month = matcher.group("month");
            String day = matcher.group("day");
            System.out.println("Год: " + year + ", Месяц: " + month + ", День: " + day);
        }
    }
}
```

**Вывод:**
```
Год: 2025, Месяц: 01, День: 14
```

---

### 9. **Преимущества и недостатки использования регулярных выражений**

#### Преимущества:
- **Гибкость**: Позволяют описывать сложные шаблоны поиска и обработки текста.
- **Компактность**: Могут заменить многострочный код на одно строку.
- **Широкая поддержка**: Регулярные выражения поддерживаются в большинстве языков программирования и инструментов.

#### Недостатки:
- **Сложность**: Сложные регулярные выражения могут быть трудно читаемыми и поддерживаемыми.
- **Производительность**: Неправильно составленные регулярные выражения могут негативно влиять на производительность приложения.
- **Отсутствие структурированности**: Регулярные выражения не всегда подходят для парсинга сложных структурированных данных (например, JSON, XML).

---

### 10. **Заключение**

Регулярные выражения — это мощный инструмент для работы с текстом в Java, предоставляющий гибкие возможности для поиска, проверки, извлечения и замены данных. Понимание основ синтаксиса регулярных выражений, умение применять классы `Pattern` и `Matcher`, а также знание практических примеров использования поможет вам эффективно решать задачи, связанные с обработкой текста. Однако важно помнить о потенциальной сложности и внимательно подходить к созданию регулярных выражений, чтобы избежать ошибок и проблем с производительностью.

---

### Дополнительные ресурсы

- **Документация Java по регулярным выражениям**: [Java Platform SE Documentation - java.util.regex](https://docs.oracle.com/javase/8/docs/api/java/util/regex/package-summary.html)
- **Онлайн-тестировщики регулярных выражений**:
    - [regex101](https://regex101.com/)
    - [RegExr](https://regexr.com/)
- **Книги и статьи**:
    - "Mastering Regular Expressions" — Jeffrey E.F. Friedl
    - Статьи на сайте [Baeldung](https://www.baeldung.com/java-regex)