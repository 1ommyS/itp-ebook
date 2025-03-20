### Stream API в Java

Stream API, введенный в Java 8, предоставляет мощный способ обработки коллекций и других источников данных в декларативном стиле. Он позволяет выполнять сложные операции обработки данных, такие как фильтрация, сортировка, преобразование и агрегация, с использованием цепочек операций. В этом разделе мы рассмотрим основные концепции Stream API, его основные операции, а также особенности параллельной обработки с использованием `.parallelStream()` и `.stream().parallel()`. Также мы обсудим класс `Stream.of` и класс `Optional`.

---

### 1. **Что такое Stream API?**

**Stream** — это последовательность элементов, поддерживающая различные виды операций над ними. Основная идея Stream API заключается в том, чтобы предоставить высокоуровневые абстракции для обработки данных, делая код более читаемым и выразительным.

#### Основные характеристики Stream API:
- **Декларативный стиль**: Описываете *что* нужно сделать, а не *как*.
- **Ленивые вычисления**: Операции над потоками выполняются только при необходимости.
- **Поддержка параллельной обработки**: Легко выполнять операции в многопоточном режиме.
- **Неизменяемость**: Потоки не хранят состояние и не изменяют исходные данные.

---

### 2. **Создание потоков (Streams)**

Потоки можно создавать из различных источников данных, таких как коллекции, массивы или генераторы.

#### Примеры создания потоков:

1. **Создание потока из коллекции:**

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

public class StreamCreationExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Анна", "Борис", "Виктор", "Галина", "Дмитрий");

        // Создание потока с использованием метода stream()
        Stream<String> streamFromList = names.stream();

        // Создание потока с использованием метода parallelStream()
        Stream<String> parallelStreamFromList = names.parallelStream();
    }
}
```

2. **Создание потока из массива:**

```java
public class StreamFromArrayExample {
    public static void main(String[] args) {
        String[] fruits = {"Apple", "Banana", "Orange", "Grape"};

        // Создание потока с использованием Arrays.stream()
        Stream<String> streamFromArray = Arrays.stream(fruits);
    }
}
```

3. **Создание потока с использованием `Stream.of`:**

```java
public class StreamOfExample {
    public static void main(String[] args) {
        // Создание потока из отдельных элементов
        Stream<String> stream = Stream.of("One", "Two", "Three", "Four");

        stream.forEach(System.out::println);
    }
}
```

**`Stream.of`** позволяет создавать потоки из отдельных элементов или из массива.

---

### 3. **Основные операции Stream API**

Stream API делит операции на **промежуточные (intermediate)** и **терминальные (terminal)**.

- **Промежуточные операции**: Возвращают новый поток и могут быть объединены в цепочку. Они выполняются лениво.
- **Терминальные операции**: Завершают обработку потока и возвращают результат или выполняют побочные действия.

#### Примеры основных операций:

1. **Фильтрация (`filter`)**: Отбирает элементы, соответствующие заданному условию.

    ```java
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

    List<Integer> evenNumbers = numbers.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());

    System.out.println("Четные числа: " + evenNumbers); // Выведет: Четные числа: [2, 4, 6]
    ```

2. **Преобразование (`map`)**: Применяет функцию к каждому элементу потока.

    ```java
    List<String> names = Arrays.asList("Анна", "Борис", "Виктор");

    List<Integer> nameLengths = names.stream()
            .map(String::length)
            .collect(Collectors.toList());

    System.out.println("Длины имен: " + nameLengths); // Выведет: Длины имен: [4, 5, 6]
    ```

3. **Сортировка (`sorted`)**: Сортирует элементы потока.

    ```java
    List<String> names = Arrays.asList("Анна", "Борис", "Виктор", "Галина");

    List<String> sortedNames = names.stream()
            .sorted()
            .collect(Collectors.toList());

    System.out.println("Отсортированные имена: " + sortedNames);
    // Выведет: Отсортированные имена: [Анна, Борис, Виктор, Галина]
    ```

4. **Агрегация (`reduce`)**: Сводит элементы потока к одному значению.

    ```java
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

    int sum = numbers.stream()
            .reduce(0, Integer::sum);

    System.out.println("Сумма чисел: " + sum); // Выведет: Сумма чисел: 15
    ```

5. **Сбор результатов (`collect`)**: Собирать элементы потока в коллекции или другие структуры данных.

    ```java
    List<String> names = Arrays.asList("Анна", "Борис", "Виктор");

    Set<String> uniqueNames = names.stream()
            .collect(Collectors.toSet());

    System.out.println("Уникальные имена: " + uniqueNames);
    // Выведет: Уникальные имена: [Анна, Борис, Виктор]
    ```

**Примеры создания потоков:**
1. **Из коллекции:**
    ```java
    List<String> list = Arrays.asList("А", "Б", "В");
    Stream<String> stream = list.stream();
    ```

2. **Из массива:**
    ```java
    String[] array = {"А", "Б", "В"};
    Stream<String> stream = Arrays.stream(array);
    ```

3. **Используя `Stream.of`:**
    ```java
    Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
    ```

4. **Бесконечные потоки:**
    ```java
    Stream<Double> randoms = Stream.generate(Math::random).limit(5);
    Stream<Integer> integers = Stream.iterate(0, n -> n + 2).limit(5);
    ```

### Основные операции над потоками

#### Промежуточные операции
Промежуточные операции возвращают новый поток, позволяя выстраивать цепочки вызовов.

**Основные промежуточные операции:**
- `filter(Predicate<? super T> predicate)` — фильтрация элементов по условию.
- `map(Function<? super T, ? extends R> mapper)` — преобразование элементов.
- `flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)` — преобразование элементов в потоки и их объединение.
- `distinct()` — удаление дубликатов.
- `sorted()` — сортировка по естественному порядку.
- `sorted(Comparator<? super T> comparator)` — сортировка по заданному компаратору.
- `limit(long maxSize)` — ограничение количества элементов.
- `skip(long n)` — пропуск первых `n` элементов.
- `peek(Consumer<? super T> action)` — выполнение действия над каждым элементом без изменения потока.
- `unordered()` — возвращение потока без определенного порядка.

**Примеры:**
```java
List<String> names = Arrays.asList("Анна", "Борис", "Виктор", "Анна");
List<String> uniqueSortedNames = names.stream()
                                      .filter(name -> name.length() > 3)
                                      .distinct()
                                      .sorted()
                                      .collect(Collectors.toList());
System.out.println(uniqueSortedNames); // Вывод: [Анна, Борис, Виктор]
```

#### Терминальные операции
Терминальные операции завершают цепочку обработки и возвращают результат или выполняют побочный эффект.

**Основные терминальные операции:**
- `forEach(Consumer<? super T> action)` — выполнение действия над каждым элементом.
- `forEachOrdered(Consumer<? super T> action)` — выполнение действия над каждым элементом в порядке потока.
- `toArray()` — преобразование потока в массив.
- `toArray(IntFunction<A[]> generator)` — преобразование потока в массив с использованием генератора.
- `reduce(T identity, BinaryOperator<T> accumulator)` — свертка потока в одно значение с начальным значением.
- `reduce(BinaryOperator<T> accumulator)` — свертка потока без начального значения.
- `<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)` — свертка потока с преобразованием типа.
- `<R, A> R collect(Collector<? super T, A, R> collector)` — сбор элементов потока в коллекцию или другое результирующее представление.
- `min(Comparator<? super T> comparator)` — нахождение минимального элемента.
- `max(Comparator<? super T> comparator)` — нахождение максимального элемента.
- `count()` — подсчет количества элементов.
- `anyMatch(Predicate<? super T> predicate)` — проверка, соответствует ли хотя бы один элемент условию.
- `allMatch(Predicate<? super T> predicate)` — проверка, соответствуют ли все элементы условию.
- `noneMatch(Predicate<? super T> predicate)` — проверка, что ни один элемент не соответствует условию.
- `findFirst()` — нахождение первого элемента.
- `findAny()` — нахождение любого элемента.

**Примеры:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Подсчет количества четных чисел
long count = numbers.stream()
                    .filter(n -> n % 2 == 0)
                    .count();
System.out.println(count); // Вывод: 2

// Нахождение суммы всех чисел
int sum = numbers.stream()
                 .reduce(0, (a, b) -> a + b);
System.out.println(sum); // Вывод: 15
```

### Параллельные потоки
Параллельные потоки позволяют выполнять операции над данными одновременно на нескольких потоках, что может значительно ускорить обработку больших объемов данных.

**Создание параллельного потока:**
```java
List<String> list = Arrays.asList("А", "Б", "В");
Stream<String> parallelStream = list.parallelStream();
```

**Преобразование в параллельный поток:**
```java
Stream<String> stream = list.stream();
Stream<String> parallelStream = stream.parallel();
```

**Преимущества:**
- **Ускорение обработки:** Использование нескольких ядер процессора для параллельной обработки.

**Недостатки:**
- **Сложность отладки:** Параллельные операции могут приводить к трудноуловимым ошибкам.
- **Неподходящие задачи:** Не все задачи выигрывают от параллелизма; некоторые могут замедлиться из-за накладных расходов.

### Коллекторы
**Коллекторы** (`Collectors`) — это специальные объекты, которые определяют, как элементы потока будут собраны в итоговую структуру данных или преобразованы.

**Основные коллекторы:**
- `toList()` — сбор в список.
- `toSet()` — сбор в множество.
- `toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends V> valueMapper)` — сбор в карту.
- `joining()` — объединение строк.
- `joining(CharSequence delimiter)` — объединение строк с разделителем.
- `joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)` — объединение строк с разделителем и обрамлением.
- `averagingInt(ToIntFunction<? super T> mapper)` — вычисление среднего значения для целочисленных элементов.
- `averagingDouble(ToDoubleFunction<? super T> mapper)` — вычисление среднего значения для чисел с плавающей точкой.
- `summarizingInt(ToIntFunction<? super T> mapper)` — получение статистики (сумма, среднее, максимум, минимум и т.д.) для целочисленных элементов.
- `groupingBy(Function<? super T, ? extends K> classifier)` — группировка элементов по ключу.
- `groupingBy(Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream)` — группировка с дополнительной обработкой.
- `partitioningBy(Predicate<? super T> predicate)` — разделение элементов на две группы по условию.
- `counting()` — подсчет количества элементов.
- `maxBy(Comparator<? super T> comparator)` — нахождение максимального элемента по компаратору.
- `minBy(Comparator<? super T> comparator)` — нахождение минимального элемента по компаратору.

**Примеры использования:**
```java
List<String> fruits = Arrays.asList("яблоко", "банан", "киви", "яблоко");

// Сбор в множество
Set<String> fruitSet = fruits.stream()
                             .collect(Collectors.toSet());
System.out.println(fruitSet); // Вывод: [яблоко, банан, киви]

// Группировка по длине строки
Map<Integer, List<String>> groupedByLength = fruits.stream()
    .collect(Collectors.groupingBy(String::length));
System.out.println(groupedByLength); // Вывод: {6=[банан, яблоко, яблоко], 4=[киви]}

// Объединение строк с разделителем
String concatenated = fruits.stream()
                            .collect(Collectors.joining(", "));
System.out.println(concatenated); // Вывод: яблоко, банан, киви, яблоко
```

### Основные методы Stream API

#### Методы интерфейса `Stream<T>`
**Промежуточные операции:**
- `filter(Predicate<? super T> predicate)`
- `map(Function<? super T, ? extends R> mapper)`
- `flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)`
- `distinct()`
- `sorted()`
- `sorted(Comparator<? super T> comparator)`
- `limit(long maxSize)`
- `skip(long n)`
- `peek(Consumer<? super T> action)`
- `unordered()`

**Терминальные операции:**
- `forEach(Consumer<? super T> action)`
- `forEachOrdered(Consumer<? super T> action)`
- `toArray()`
- `toArray(IntFunction<A[]> generator)`
- `reduce(T identity, BinaryOperator<T> accumulator)`
- `reduce(BinaryOperator<T> accumulator)`
- `<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)`
- `<R, A> R collect(Collector<? super T, A, R> collector)`
- `min(Comparator<? super T> comparator)`
- `max(Comparator<? super T> comparator)`
- `count()`
- `anyMatch(Predicate<? super T> predicate)`
- `allMatch(Predicate<? super T> predicate)`
- `noneMatch(Predicate<? super T> predicate)`
- `findFirst()`
- `findAny()`

**Примеры использования методов:**

1. **Фильтрация и сбор в список:**
    ```java
    List<String> names = Arrays.asList("Анна", "Борис", "Виктор", "Анна");
    List<String> filtered = names.stream()
                                 .filter(name -> name.startsWith("А"))
                                 .collect(Collectors.toList());
    System.out.println(filtered); // Вывод: [Анна, Анна]
    ```

2. **Преобразование и подсчет:**
    ```java
    List<String> words = Arrays.asList("apple", "banana", "cherry");
    long count = words.stream()
                      .map(String::toUpperCase)
                      .distinct()
                      .count();
    System.out.println(count); // Вывод: 3
    ```

3. **Группировка и агрегация:**
    ```java
    List<Person> people = Arrays.asList(
        new Person("Анна", 30),
        new Person("Борис", 25),
        new Person("Виктор", 30),
        new Person("Галина", 25)
    );

    Map<Integer, Long> ageCount = people.stream()
        .collect(Collectors.groupingBy(Person::getAge, Collectors.counting()));
    System.out.println(ageCount); // Вывод: {25=2, 30=2}
    ```

4. **Нахождение максимального значения:**
    ```java
    List<Integer> numbers = Arrays.asList(10, 20, 30, 40, 50);
    Optional<Integer> max = numbers.stream()
                                   .max(Integer::compareTo);
    max.ifPresent(System.out::println); // Вывод: 50
    ```
---

### 4. **Параллельные потоки: `.parallelStream()` vs `.stream().parallel()`**

Параллельные потоки позволяют выполнять операции над потоками данных в многопоточном режиме, что может повысить производительность при обработке больших объемов данных.

#### Разница между `.parallelStream()` и `.stream().parallel()`:

- **`.parallelStream()`**: Создает параллельный поток непосредственно из коллекции.
- **`.stream().parallel()`**: Сначала создает последовательный поток, а затем преобразует его в параллельный.

Хотя оба подхода приводят к созданию параллельного потока, использование `.parallelStream()` является более прямым и предпочтительным способом создания параллельного потока из коллекции.

#### Пример использования `.parallelStream()`:

```java
import java.util.Arrays;
import java.util.List;

public class ParallelStreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        int sum = numbers.parallelStream()
                .filter(n -> n % 2 == 0)
                .mapToInt(Integer::intValue)
                .sum();

        System.out.println("Сумма четных чисел: " + sum); // Выведет: Сумма четных чисел: 30
    }
}
```

#### Пример использования `.stream().parallel()`:

```java
import java.util.Arrays;
import java.util.List;

public class StreamParallelExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        int sum = numbers.stream()
                .parallel()
                .filter(n -> n % 2 == 0)
                .mapToInt(Integer::intValue)
                .sum();

        System.out.println("Сумма четных чисел: " + sum); // Выведет: Сумма четных чисел: 30
    }
}
```

**Важно:** Оба метода создают параллельный поток, однако `.parallelStream()` предпочтительнее для создания параллельных потоков из коллекций, так как это более очевидный и читаемый подход.

#### Когда использовать параллельные потоки?

- **Большие объемы данных**: При обработке больших коллекций параллельные потоки могут значительно повысить производительность.
- **Многоядерные процессоры**: На многоядерных системах параллельные потоки могут эффективно использовать все доступные ядра.
- **Независимые операции**: Когда операции над элементами потока не зависят друг от друга и могут выполняться независимо.

**Предостережения:**
- **Потокобезопасность**: Убедитесь, что используемые операции потокобезопасны.
- **Перегрузка системы**: Параллельные потоки могут привести к избыточной нагрузке на систему, если неправильно использовать.
- **Порядок операций**: Параллельные потоки могут изменять порядок выполнения операций, что может влиять на результаты.

---

### 5. **Класс `Stream.of`**

**`Stream.of`** — это статический метод класса `Stream`, который позволяет создавать поток из заданных элементов.

#### Примеры использования `Stream.of`:

1. **Создание потока из отдельных элементов:**

    ```java
    public class StreamOfExample {
        public static void main(String[] args) {
            Stream<String> stream = Stream.of("Java", "Python", "C++", "JavaScript");

            stream.forEach(System.out::println);
        }
    }
    ```

   **Вывод:**
    ```
    Java
    Python
    C++
    JavaScript
    ```

2. **Создание потока из массива:**

    ```java
    public class StreamOfArrayExample {
        public static void main(String[] args) {
            String[] languages = {"Java", "Python", "C++", "JavaScript"};

            Stream<String> stream = Stream.of(languages);

            stream.forEach(System.out::println);
        }
    }
    ```

   **Вывод:**
    ```
    Java
    Python
    C++
    JavaScript
    ```

---

### 6. **Класс `Optional`**

**`Optional`** — это контейнерный класс, введенный в Java 8, который может содержать или не содержать значение. Он используется для предотвращения ошибок `NullPointerException` и для явного обозначения того, что значение может отсутствовать.

#### Основные методы `Optional`:

- **`of(T value)`**: Создает `Optional`, содержащий заданное ненулевое значение.
- **`ofNullable(T value)`**: Создает `Optional`, который может содержать заданное значение или быть пустым.
- **`isPresent()`**: Проверяет, содержит ли `Optional` значение.
- **`get()`**: Возвращает значение, если оно присутствует, иначе выбрасывает `NoSuchElementException`.
- **`ifPresent(Consumer<? super T> action)`**: Выполняет действие, если значение присутствует.
- **`orElse(T other)`**: Возвращает значение, если оно присутствует, иначе возвращает другое заданное значение.
- **`orElseGet(Supplier<? extends T> other)`**: Возвращает значение, если оно присутствует, иначе генерирует значение с помощью поставщика.
- **`orElseThrow(Supplier<? extends X> exceptionSupplier)`**: Возвращает значение, если оно присутствует, иначе выбрасывает исключение, созданное поставщиком.

#### Примеры использования `Optional`:

1. **Создание и проверка значения:**

    ```java
    import java.util.Optional;

    public class OptionalExample {
        public static void main(String[] args) {
            Optional<String> optionalWithValue = Optional.of("Hello, World!");
            Optional<String> optionalEmpty = Optional.empty();

            System.out.println("optionalWithValue is present: " + optionalWithValue.isPresent()); // true
            System.out.println("optionalEmpty is present: " + optionalEmpty.isPresent());       // false
        }
    }
    ```

2. **Использование `ifPresent`:**

    ```java
    import java.util.Optional;

    public class IfPresentExample {
        public static void main(String[] args) {
            Optional<String> optional = Optional.of("Java");

            optional.ifPresent(name -> System.out.println("Привет, " + name + "!")); // Выведет: Привет, Java!
        }
    }
    ```

3. **Использование `orElse`:**

    ```java
    import java.util.Optional;

    public class OrElseExample {
        public static void main(String[] args) {
            Optional<String> optionalWithValue = Optional.of("Hello");
            Optional<String> optionalEmpty = Optional.empty();

            String greeting1 = optionalWithValue.orElse("Default Greeting");
            String greeting2 = optionalEmpty.orElse("Default Greeting");

            System.out.println(greeting1); // Выведет: Hello
            System.out.println(greeting2); // Выведет: Default Greeting
        }
    }
    ```

4. **Использование `orElseThrow`:**

    ```java
    import java.util.Optional;

    public class OrElseThrowExample {
        public static void main(String[] args) {
            Optional<String> optional = Optional.empty();

            try {
                String value = optional.orElseThrow(() -> new IllegalArgumentException("Значение отсутствует"));
            } catch (IllegalArgumentException e) {
                System.out.println(e.getMessage()); // Выведет: Значение отсутствует
            }
        }
    }
    ```

#### Применение `Optional` с Stream API:

```java
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

public class OptionalStreamExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Анна", "Борис", "Виктор", "Галина", "Дмитрий");

        Optional<String> firstLongName = names.stream()
                .filter(name -> name.length() > 5)
                .findFirst();

        firstLongName.ifPresent(name -> System.out.println("Первое длинное имя: " + name));
        // Выведет: Первое длинное имя: Виктор
    }
}
```

---

### 7. **Сравнение `.parallelStream()` и `.stream().parallel()`**

Хотя оба метода создают параллельные потоки, существует тонкая разница в их использовании и предпочтениях.

| **Особенность**                 | **`.parallelStream()`**                               | **`.stream().parallel()`**                               |
|----------------------------------|--------------------------------------------------------|----------------------------------------------------------|
| **Создание параллельного потока** | Создает параллельный поток непосредственно из коллекции | Создает последовательный поток, затем переключает его на параллельный |
| **Читаемость кода**               | Более лаконичный и очевидный способ                   | Менее очевиден, так как разделение на две операции       |
| **Производительность**            | Аналогична, так как оба метода используют одинаковые механизмы для параллельной обработки | Аналогична                                              |
| **Предпочтительность**            | Предпочтительнее для создания параллельных потоков из коллекций | Менее предпочтительно из-за менее явного синтаксиса      |

**Вывод:** Используйте `.parallelStream()` для создания параллельных потоков из коллекций, так как это более прямой и понятный подход.

---

### 8. **Преимущества использования Stream API**

- **Упрощение кода**: Делает код более компактным и читаемым.
- **Функциональный стиль**: Позволяет применять функциональные концепции, такие как чистые функции и композиция.
- **Параллельная обработка**: Легко выполнять операции в многопоточном режиме без явного управления потоками.
- **Ленивые вычисления**: Оптимизируют производительность, выполняя операции только при необходимости.
- **Мощные возможности обработки данных**: Предоставляют широкий набор операций для фильтрации, трансформации, агрегации и других операций над данными.

---

### 9. **Когда использовать Stream API?**

- **Обработка коллекций**: Когда нужно выполнить сложные операции над коллекциями, такие как фильтрация, преобразование или агрегация.
- **Работа с большими данными**: При необходимости эффективной обработки больших объемов данных.
- **Параллельная обработка**: Когда требуется ускорить обработку данных за счет использования нескольких ядер процессора.
- **Упрощение кода**: Когда нужно сделать код более декларативным и читаемым, избегая вложенных циклов и условий.

---
