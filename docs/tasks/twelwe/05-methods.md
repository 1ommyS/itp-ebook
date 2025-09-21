# Теория, которую нужно знать перед выполнением ДЗ

## 1) Что такое метод в Java

* **Метод** — именованный блок кода, который можно вызывать многократно.
* Позволяет cпрятать логику, **переиспользовать** её и **тестировать** изолированно.
* Синтаксис:

  ```java
  [модификаторы] [static (или его нет)] [возвращаемый_тип] имяМетода(параметры) [throws ...] {
      // тело
      return значение; // если тип не void
  }
  ```

  Пример:

  ```java
  public static int sum(int a, int b) {
      return a + b;
  }
  ```

## 2) Сигнатура метода и перегрузка

* **Сигнатура** = имя метода + список типов параметров (порядок важен, имена параметров — нет).
* **Перегрузка (overload)** — несколько методов с одним именем, но разной сигнатурой:

  ```java
  int sum(int a, int b) { ... }
  double sum(double a, double b) { ... }
  int sum(int a, int b, int c) { ... }
  ```

## 3) Возвращаемые типы и `void`

* Если метод **что-то вычисляет**, обычно он **должен возвращать результат** (не делать “принт и забыли”).
* `void` — когда нужен **эффект** (печать, запись в БД, изменение состояния), но **нет значения** для возврата.
* Можно возвратить **ссылочный тип** (например, `String`, `List<Integer>`), **примитив** (`int`, `double`), или *
  *собственный класс**.

## 4) Параметры: значение vs. ссылки

* В Java всегда **передача по значению**.
* Примитивы внутри метода **не влияют** на оригинал.
* Объекты внутри метода **могут быть изменены**, если вы меняете их состояние:

  ```java
  void addItem(List<Integer> list, int x) { list.add(x); } // изменит внешний список
  ```

## 5) Область видимости (scope) и `return`

* Переменные, объявленные внутри метода/блока, **не видны** снаружи.
* `return` завершает метод. В непустых методах (не `void`) **должен** быть `return` на всех ветках.

## 6) Именование

* Имя должно **отвечать на вопрос “что делает?”**: `calculateDiscount`, `isPrime`, `readFile`.

## 7) Методы работы со строками (краткий обзор)

* `length()`, `charAt(i)`, `substring(begin, end)`, `indexOf`, `lastIndexOf`, `startsWith`, `endsWith`.
* `toLowerCase()`, `toUpperCase()`, `trim()`, `strip()`.
* `replace(old, new)`, `replaceAll(regex, repl)`, `split(regex)`.
* Сравнение: `equals`, `equalsIgnoreCase`, `compareTo`.
* Сборка строк: **StringBuilder** в циклах, чтобы не плодить объекты:

  ```java
  String joinWithComma(String[] items) {
      StringBuilder sb = new StringBuilder();
      for (int i = 0; i < items.length; i++) {
          if (i > 0) sb.append(", ");
          sb.append(items[i]);
      }
      return sb.toString();
  }
  ```
## 8) Пример правильной композиции

```java
public class PasswordUtils {
    public static boolean isLongEnough(String s) {
        return s != null && s.length() >= 8;
    }

    public static boolean hasDigit(String s) {
        if (s == null) return false;
        for (char c : s.toCharArray()) if (Character.isDigit(c)) return true;
        return false;
    }

    public static boolean hasLetter(String s) {
        if (s == null) return false;
        for (char c : s.toCharArray()) if (Character.isLetter(c)) return true;
        return false;
    }

    public static boolean isStrongPassword(String s) {
        // Композиция: метод использует другие методы
        return isLongEnough(s) && hasDigit(s) && hasLetter(s);
    }
}
```

* Отдельные маленькие проверки можно **легко тестировать** и **сочетать**.

---

# Практика 1: 16 задач на собственные методы 

### Задача 1: Чётность числа

* **Цель:** написать простой булев метод.
* **Сигнатура:** `public static boolean isEven(int n)`
* **Описание:** вернуть `true`, если `n` чётное.
* **Критерий:** работает для отрицательных и больших чисел.
* **Подсказка:** `n % 2 == 0`.

### Задача 2: Максимум из трёх

* **Сигнатура:** `public static int max3(int a, int b, int c)`
* **Описание:** вернуть наибольшее из трёх.
* **Критерий:** без использования массивов/коллекций.
* **Подсказка:** вложенные `Math.max`.

### Задача 3: Модуль числа (перегрузка)

* **Сигнатуры:**
  `public static int abs(int x)`
  `public static double abs(double x)`
* **Описание:** вернуть модуль.
* **Критерий:** корректно для `-0.0`? (для простоты можно оставить как `0.0`).

### Задача 4: Безопасное деление

* **Сигнатура:** `public static int safeDivide(int a, int b)`
* **Описание:** бросить `IllegalArgumentException`, если `b == 0`. Иначе вернуть `a / b`.
* **Критерий:** корректные исключения.

### Задача 5: Сумма массива

* **Сигнатура:** `public static int sum(int[] arr)`
* **Описание:** сумма элементов, `null` → `0` (или бросить исключение — выбери и задокументируй).
* **Критерий:** пустой массив → `0`.

### Задача 6: Подсчёт положительных

* **Сигнатура:** `public static int countPositive(int[] arr)`
* **Описание:** количество `> 0`.
* **Критерий:** устойчивость к `null` (опиши поведение).

### Задача 7: Вар-аргс сумма

* **Сигнатура:** `public static long sum(long... nums)`
* **Описание:** сумма произвольного количества аргументов.
* **Критерий:** `sum()` без аргументов → `0`.

### Задача 8: Палиндром для числа

* **Сигнатура:** `public static boolean isNumericPalindrome(int n)`
* **Описание:** число читается одинаково слева направо и справа налево (без строк).
* **Критерий:** корректно для отрицательных (реши, как трактовать; задокументируй).

### Задача 9: НОД (алгоритм Евклида)

* **Сигнатура:** `public static int gcd(int a, int b)`
* **Описание:** наибольший общий делитель, корректно для отрицательных.
* **Критерий:** `gcd(a, 0) == abs(a)`.

### Задача 10: Простота числа

* **Сигнатура:** `public static boolean isPrime(int n)`
* **Описание:** проверить простоту.
* **Критерий:** корректно и эффективно до `√n`.

### Задача 11: Композиция — сумма простых в диапазоне

* **Сигнатура:** `public static int sumPrimes(int from, int to)`
* **Описание:** использовать **isPrime** из задачи 10.
* **Критерий:** границы включительно; если `from > to` — поменять местами или вернуть `0` (задокументируй).

### Задача 12: Нормализация массива

* **Сигнатуры:**
  `public static int max(int[] arr)`
  `public static double[] normalize(int[] arr)`
* **Описание:** `normalize`: поделить каждый элемент на `max(arr)` и вернуть `double[]`.
* **Критерий:** `max == 0` — не делим на ноль: опиши поведение (например, вернуть нули).

### Задача 13: Функция “сигнум”

* **Сигнатура:** `public static int sign(int x)`
* **Описание:** `-1`, `0` или `1`.
* **Критерий:** если число > 0, то вернуть 1, если = 0, то 0, если < 0, то -1

### Задача 14: Валидатор пароля (композиция)

* **Сигнатуры:**
  `public static boolean hasDigit(String s)`
  `public static boolean hasLetter(String s)`
  `public static boolean isLongEnough(String s, int minLen)`
  `public static boolean isStrongPassword(String s)`
* **Описание:** `isStrongPassword` = композиция трёх первых.
* **Критерий:** `null`-безопасность.

### Задача 15: Уникальные элементы

* **Сигнатура:** `public static int countUnique(int[] arr)`
* **Описание:** вернуть количество уникальных значений. 

### Задача 16: Нормированная проверка e-mail (композиция строковых помощников)

* **Сигнатуры:**
  `public static boolean hasExactlyOne(String s, char c)`
  `public static boolean hasAtLeast(String s, char c, int count)`
  `public static boolean isValidEmailBasic(String s)`
* **Описание:** `isValidEmailBasic` использует **haveExactlyOne('@')**, проверяет наличие точки в правой части, без
  пробелов.
* **Критерий:** `null` → `false`, “a\@b” допустимо ли? реши и задокументируй.


# Практика 2: 8 задач на методы работы со строками

### Строка-1: Инвертировать строку

* **Сигнатура:** `public static String reverse(String s)`
* **Описание:** вернуть перевёрнутую строку.
* **Подсказка:** `StringBuilder(s).reverse().toString()` или ручной цикл.

### Строка-2: Подсчёт слов

* **Сигнатура:** `public static int countWords(String s)`
* **Описание:** слова разделены одним или несколькими пробелами/табами/переводами строк.
* **Критерий:** `trim` + `split("\\s+")`, пустая строка → 0.

### Строка-3: Удалить повторы пробелов

* **Сигнатура:** `public static String squeezeSpaces(String s)`
* **Описание:** заменить последовательности пробелов на один пробел.
* **Критерий:** `"a   b \t  c"` → `"a b c"`.
* **Подсказка:** `s.replaceAll("\\s+", " ").trim()`.

### Строка-4: Проверка префикса/суффикса без учёта регистра

* **Сигнатура:** `public static boolean startsWithIgnoreCase(String s, String prefix)`
* **Описание:** `true`, если `s` начинается с `prefix` (ignoring case).
* **Критерий:** `null`-безопасность.

### Строка-5: Самая длинная подстрока без пробелов

* **Сигнатура:** `public static String longestToken(String s)`
* **Описание:** вернуть самый длинный “токен” между пробелами.
* **Критерий:** при равенстве — первый.

### Строка-6: Подсчитать вхождения подстроки

* **Сигнатура:** `public static int countOccurrences(String s, String sub)`
* **Описание:** количество **неперекрывающихся** вхождений `sub` в `s`.
* **Подсказка:** итерируйтесь `indexOf(sub, fromIndex)`.

### Строка-7: Только буквы и цифры

* **Сигнатура:** `public static boolean isAlnum(String s)`
* **Описание:** `true`, если каждый символ — `Character.isLetterOrDigit`.
* **Критерий:** пустая строка → `false` или `true` — выберите и задокументируйте.

### Строка-8: КамелКейс → снейк\_кейс (простая версия)

* **Сигнатура:** `public static String camelToSnake(String s)`
* **Описание:** `"myVariableName"` → `"my_variable_name"`.
* **Подсказка:** проход по символам, перед заглавной буквой вставить `_` и понизить регистр.
