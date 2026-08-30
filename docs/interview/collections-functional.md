# Коллекции, generics и функциональный стиль

## Как выбирать коллекцию

Сначала сформулируйте требуемую операцию и гарантию: поиск по ключу, порядок, уникальность, доступ по индексу, работа с концами очереди или конкурентный доступ.

| Структура | Сильная сторона | Типичная цена |
|---|---|---|
| `ArrayList` | Доступ по индексу O(1), компактное хранение | Вставка в середину O(n), resize |
| `LinkedList` | Вставка по уже известному узлу | Поиск O(n), много объектов и плохая locality |
| `HashSet` | Уникальность и средний поиск O(1) | Нет естественного порядка |
| `HashMap` | Поиск значения по ключу в среднем O(1) | Зависит от корректного hash и распределения |
| `TreeMap` | Отсортированные ключи, диапазоны O(log n) | Дороже hash-структуры |
| `ArrayDeque` | Быстрые операции с обоих концов | Нет произвольного доступа |
| `PriorityQueue` | Быстро получить минимум/максимум | Полный порядок не гарантируется |

`LinkedList` редко выигрывает у `ArrayList`: даже при формально дешёвой вставке сначала обычно нужно O(n) пройти до позиции, а разрозненные узлы хуже используют кэш процессора.

## Как работает `HashMap`

1. Из hash ключа вычисляется индекс bucket.
2. В bucket сравнивается hash, затем ключ через `equals`.
3. При коллизии элементы остаются в одном bucket; длинная цепочка может стать деревом.
4. При превышении порога `capacity × loadFactor` таблица расширяется и элементы перераспределяются.

Следствия:

- качественный `hashCode` распределяет ключи равномерно;
- `equals` и `hashCode` должны использовать согласованный неизменяемый набор полей;
- начальная capacity полезна при заранее известном большом размере;
- детали порогов treeification — реализация JDK, а не контракт `Map`.

`HashSet` хранит элементы как ключи внутренней map. `ConcurrentHashMap` обеспечивает потокобезопасные операции без единственной глобальной блокировки и запрещает `null`, чтобы отсутствие значения не было неоднозначным.

## Итераторы и изменение коллекции

Fail-fast iterator обнаруживает структурное изменение коллекции вне собственного `remove()` и обычно выбрасывает `ConcurrentModificationException`. Это средство раннего обнаружения ошибки, а не гарантия синхронизации.

Для конкурентных сценариев выбирают структуру по семантике:

- `ConcurrentHashMap` — конкурентный доступ по ключу;
- `CopyOnWriteArrayList` — много чтений и очень мало записей;
- `BlockingQueue` — producer/consumer и backpressure;
- immutable snapshot — простая безопасная публикация состояния.

## `Comparable` и `Comparator`

`Comparable` задаёт один естественный порядок внутри типа. `Comparator` — внешнюю стратегию, поэтому их может быть несколько.

```java
Comparator<User> byScoreThenName = Comparator
        .comparingInt(User::score).reversed()
        .thenComparing(User::name);
```

Порядок должен быть транзитивным. Для sorted set/map желательно, чтобы результат сравнения `0` соответствовал логической эквивалентности, иначе элементы могут «исчезать» как дубликаты.

## Generics и type erasure

Generics дают проверку типов на этапе компиляции. После type erasure JVM в основном видит верхние границы типов и вставленные компилятором casts. Из этого следуют ограничения:

- нельзя создать `new T()` или `new T[]` без фабрики/токена типа;
- нельзя проверить `value instanceof List<String>`;
- нельзя перегрузить методы только разными generic-параметрами с одинаковым erasure;
- `List<Integer>` не является подтипом `List<Number>`.

Generic-метод объявляет параметр типа до возвращаемого значения:

```java
static <T> T first(List<? extends T> items) {
    if (items.isEmpty()) throw new NoSuchElementException();
    return items.get(0);
}
```

### PECS

**Producer Extends, Consumer Super**:

```java
static <T> void copy(List<? extends T> source, List<? super T> target) {
    target.addAll(source);
}
```

Из `? extends T` безопасно читать `T`, но нельзя добавлять конкретный `T`. В `? super T` можно добавлять `T`, а читать — только как `Object`.

## Lambda и функциональные интерфейсы

Функциональный интерфейс имеет один абстрактный метод. Основные роли:

| Интерфейс | Сигнатура | Смысл |
|---|---|---|
| `Function<T,R>` | `T → R` | преобразовать |
| `Predicate<T>` | `T → boolean` | проверить |
| `Consumer<T>` | `T → void` | выполнить побочный эффект |
| `Supplier<T>` | `() → T` | получить лениво |

Lambda захватывает effectively final локальные переменные. Не путайте лаконичный синтаксис с отсутствием состояния: захваченный изменяемый объект всё ещё может создать race condition.

## Stream pipeline

Stream описывает одноразовый pipeline:

1. source;
2. ленивые intermediate operations (`map`, `filter`, `flatMap`, `sorted`);
3. terminal operation (`toList`, `collect`, `reduce`, `findFirst`).

`map` превращает один элемент в один; `flatMap` — элемент в поток и затем выравнивает результаты.

```java
Map<String, Long> countByCity = users.stream()
        .filter(User::active)
        .collect(Collectors.groupingBy(User::city, Collectors.counting()));
```

Ленивость позволяет объединить операции и прекратить вычисление раньше для short-circuit terminal operations. Не помещайте важные side effects в `peek` и не переиспользуйте stream после terminal operation.

### `parallelStream`

Parallel stream использует разбиение задачи и общий `ForkJoinPool`. Он полезен для достаточно большой CPU-bound работы с хорошо делимыми данными и ассоциативной агрегацией. Он часто вреден для blocking I/O, маленьких наборов, stateful lambdas и приложений, где общий pool уже занят.

## `Optional`

`Optional` хорошо выражает возможное отсутствие результата метода. Это не универсальная замена `null` для полей entity, DTO и параметров.

Предпочитайте композицию:

```java
return repository.findById(id)
        .filter(User::active)
        .map(mapper::toDto)
        .orElseThrow(() -> new UserNotFoundException(id));
```

`orElse` вычисляет аргумент заранее, а `orElseGet` — лениво. Не используйте `isPresent()` + `get()`, если тот же код выражается через `map`, `flatMap`, `ifPresent` или `orElseThrow`.

## Вопросы для самопроверки

1. Почему среднее O(1) у `HashMap` не означает O(1) всегда?
2. В какой задаче `ArrayDeque` лучше `Stack` и `LinkedList`?
3. Почему `List<? extends Number>` нельзя пополнить `Integer`?
4. Чем `flatMap` отличается от `map` на примере заказов и позиций?
5. Какие измерения нужны до включения `parallelStream`?
