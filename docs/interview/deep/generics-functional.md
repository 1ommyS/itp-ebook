# Generics и функциональный стиль — подробный конспект

## Зачем нужны generics

Generics переносят ошибку типа из runtime в compile time и позволяют повторно использовать алгоритм без casts.

```java
interface Repository<ID, T> {
    Optional<T> findById(ID id);
    T save(T entity);
}
```

Type parameter должен иметь смысл в контракте. Если он встречается только один раз и ни с чем не связывает тип, wildcard часто проще.

## Invariance

`List<Integer>` не является `List<Number>`. Иначе в переменную `List<Number>` можно было бы добавить `Double`, нарушив исходный список Integer.

Массивы covariant и проверяют несовместимую запись в runtime, поэтому возможен `ArrayStoreException`. Generics выбрали invariance, чтобы ловить проблему при компиляции.

## Wildcards и PECS

`List<? extends Number>` — список неизвестного конкретного подтипа Number. Из него можно читать Number, но безопасно добавить только `null`.

`List<? super Integer>` — список некоторого supertype Integer. В него можно добавлять Integer, но читать только Object.

PECS работает на уровне роли параметра:

- producer для метода — `extends`;
- consumer — `super`;
- одновременно читает и пишет один точный тип — обычно `List<T>`.

Wildcard capture позволяет helper method «назвать» неизвестный тип и, например, переставить элементы списка `List<?>`.

## Bounds и generic methods

Multiple bounds записываются с классом первым:

```java
static <T extends Number & Comparable<T>> T max(T left, T right) {
    return left.compareTo(right) >= 0 ? left : right;
}
```

Type inference использует аргументы и target type. Явный type witness `<String>` нужен редко, обычно при неоднозначности сложной цепочки.

## Type erasure

После компиляции параметры типов в основном стираются до upper bound/Object, а compiler вставляет casts и bridge methods.

Следствия:

- `List<String>.class` не существует;
- нельзя `new T()` без factory;
- нельзя `new List<String>[10]`;
- нельзя `instanceof List<String>`;
- overload `process(List<String>)` и `process(List<Integer>)` конфликтует по erasure;
- static поле не может быть отдельным для каждого T.

Heap pollution возникает, когда переменная parameterized type ссылается на объект несовместимого типа, обычно из-за raw type, unchecked cast или generic varargs.

`@SafeVarargs` обещает, что метод не выполняет небезопасные операции с generic varargs; это ответственность автора, не отключение реального риска.

## Functional interfaces и lambda

Функциональный интерфейс имеет один abstract method; методы Object и default methods не увеличивают это число. `@FunctionalInterface` даёт compile-time проверку намерения.

Lambda не создаёт отдельный lexical `this`: `this` относится к окружающему объекту. Anonymous class имеет собственный `this`.

Захваченная локальная переменная должна быть effectively final, потому что локальная переменная живёт в stack frame, а lambda может пережить вызов метода. Захватывается значение; для ссылки всё ещё можно менять сам объект.

## Stream pipeline подробно

Stream не хранит данные. Он хранит описание вычисления над source. Intermediate operations ленивы: до terminal operation элементы не проходят pipeline.

Pipeline обычно обрабатывает элемент вертикально: source → filter → map для одного элемента, затем следующий. Stateful operations вроде `sorted` или `distinct` могут потребовать буфер.

```java
Map<Status, BigDecimal> totalByStatus = orders.stream()
        .filter(order -> order.createdAt().isAfter(from))
        .collect(Collectors.groupingBy(
                Order::status,
                Collectors.reducing(
                        BigDecimal.ZERO,
                        Order::total,
                        BigDecimal::add)));
```

## `map`, `flatMap`, reduce и collect

`map` сохраняет количество уровней контейнера: `Stream<Order> → Stream<List<Item>>`. `flatMap` превращает каждый order в stream items и объединяет: `Stream<Item>`.

`reduce` сворачивает immutable value ассоциативной операцией. Mutable reduction через один общий ArrayList в parallel stream опасен; для неё есть `collect` с supplier, accumulator и combiner.

Identity reduce должна быть нейтральной, а accumulator/combiner — совместимыми и ассоциативными, иначе parallel result отличается от sequential.

## Short-circuit и порядок

`findFirst` сохраняет encounter order и может быть дороже параллельно. `findAny` разрешает любой элемент. `anyMatch`, `allMatch`, `noneMatch`, `limit` могут остановить pipeline раньше.

Side effects нарушают reasoning. Не изменяйте внешнюю collection из `forEach` parallel stream. Если нужен результат — собирайте collector.

## Parallel streams

Полезны, если:

- источник легко split (array лучше linked structure);
- операция CPU-bound;
- элементов достаточно много;
- работа на элемент существенна;
- нет общего mutable state;
- common pool не конфликтует с нагрузкой приложения.

Проверка выполняется benchmark, а не количеством ядер. Blocking I/O в common ForkJoinPool может исчерпать workers.

## Optional

Optional — return type для отсутствующего результата. Не используйте как field JPA entity, аргумент каждого метода или serializable DTO без причины.

Различия:

- `map` оборачивает non-null result, null становится empty;
- `flatMap` нужен, если mapper уже возвращает Optional;
- `orElse` eager;
- `orElseGet` lazy;
- `or` лениво выбирает другой Optional;
- `stream` превращает Optional в поток 0..1.

Optional не должен быть `null`: это двойная модель отсутствия.

## Типичные вопросы

1. Почему `List<? extends Number>` нельзя пополнить Integer?
2. Что такое bridge method?
3. Почему mutable reduction ломается parallel?
4. Чем lambda отличается от anonymous class относительно `this`?
5. Когда `Optional<List<T>>` хуже пустого списка?

## Критерий готовности

Вы готовы, если можете написать `copy` по PECS, объяснить erasure restrictions, построить collector с grouping, доказать associativity reduction и найти side effect, несовместимый с parallel execution.

