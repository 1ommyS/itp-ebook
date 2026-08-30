# Collections — подробный конспект

## Иерархия и контракты

`Collection` представляет группу элементов. Главные подтипы: `List` — порядок и дубликаты, `Set` — уникальность, `Queue/Deque` — политика извлечения. `Map` не наследует `Collection`, потому что хранит пары key/value.

Контракт интерфейса важнее реализации. Например, `List#get` существует у `LinkedList`, но это не означает эффективный random access.

## ArrayList

Внутри — массив ссылок и logical size. Добавление в конец:

1. проверяет capacity;
2. при необходимости создаёт больший массив и копирует элементы;
3. записывает значение и увеличивает size.

Одна операция resize стоит O(n), но редкие расширения распределяются по множеству `add`, поэтому амортизированная стоимость добавления O(1). Вставка/удаление в середине сдвигает хвост через array copy — O(n).

`remove(int)` и `remove(Object)` перегружены. Для `List<Integer>` вызов `remove(1)` удалит элемент по индексу, а `remove(Integer.valueOf(1))` — значение.

## LinkedList

Каждый узел хранит значение и ссылки на соседей. Доступ по индексу требует прохода от ближайшего конца. Даже последовательный обход часто проигрывает массиву из-за allocation и cache misses.

LinkedList оправдан, когда код уже имеет iterator/узел в нужной позиции и часто вставляет рядом, либо нужен `Deque`, но обычно `ArrayDeque` компактнее и быстрее.

## HashMap: путь операции

1. `hashCode` преобразуется дополнительным spread-функционалом.
2. Маска capacity выбирает bucket.
3. Если bucket пуст, создаётся node.
4. Иначе сравниваются hash и `equals`.
5. Коллизии хранятся цепочкой или tree bin.
6. После превышения threshold выполняется resize.

Capacity обычно степень двойки, поэтому индекс вычисляется маской. Load factor балансирует память и число коллизий. Treeification защищает худший случай длинной цепочки, но конкретные пороги — implementation detail JDK.

`null` key допустим у `HashMap` и попадает в определённый bucket. `ConcurrentHashMap` запрещает `null`, чтобы `get == null` однозначно означал отсутствие mapping.

## HashSet и TreeMap

`HashSet` использует элементы как keys внутренней map. Поэтому требования к immutability и equality те же.

`TreeMap` — ordered map на balanced search tree. Операции O(log n). Порядок задаёт `Comparable` или `Comparator`. Если comparator возвращает `0` для разных по `equals` ключей, map считает их одним ключом.

Range views (`subMap`, `headMap`) связаны с исходной map: изменение view меняет оригинал и наоборот.

## Iterator и fail-fast

Enhanced for использует iterator. Structural modification меняет размер/структуру, а не просто значение существующего элемента. Iterator сравнивает expected modification count с текущим и может бросить `ConcurrentModificationException`.

Это best-effort диагностика. Нельзя строить thread safety на том, что исключение «обязательно появится».

Безопасные варианты удаления:

```java
for (Iterator<Order> it = orders.iterator(); it.hasNext();) {
    if (it.next().expired()) it.remove();
}
```

или `removeIf`, если predicate не меняет ту же коллекцию побочно.

## Queue и Deque

У Queue пары методов различаются политикой ошибки:

| Операция | Exception | Special value |
|---|---|---|
| Вставка | `add` | `offer` |
| Чтение головы | `element` | `peek` |
| Извлечение | `remove` | `poll` |

`ArrayDeque` применяют как queue и stack (`addLast/removeFirst`, `push/pop`). Legacy `Stack` синхронизирован и наследует `Vector`; для локального stack лучше Deque.

`PriorityQueue` гарантирует только head по comparator. Итерация не выдаёт элементы полностью отсортированными.

## Comparable и Comparator

Natural ordering хранится в типе. Внешний comparator позволяет несколько порядков, null policy и composition.

Не пишите `return a - b`: возможен integer overflow. Используйте `Integer.compare(a, b)`.

Comparator должен быть antisymmetric и transitive; иначе sort может дать неверный результат или сообщить о нарушении contract.

## ConcurrentHashMap

Современная реализация использует CAS для пустых buckets и синхронизацию на отдельных bins при конфликте. Чтения в основном не требуют общей блокировки. Resize выполняется кооперативно.

Compound operation `if (!map.containsKey(k)) map.put(k, v)` имеет race. Используйте atomic map methods:

```java
cache.computeIfAbsent(key, this::load);
```

Mapping function должна быть короткой и не выполнять рекурсивное обновление того же key. Для expensive I/O `computeIfAbsent` может удерживать внутреннюю координацию; иногда лучше хранить `CompletableFuture<V>` и управлять single-flight явно.

Iterators concurrent collections weakly consistent: не бросают CME и могут видеть часть concurrent updates без единого snapshot.

## Complexity с оговорками

| Структура | Поиск | Добавление | Удаление | Порядок |
|---|---:|---:|---:|---|
| ArrayList | O(n), index O(1) | конец amortized O(1) | O(n) | insertion |
| HashMap/Set | average O(1) | average O(1) | average O(1) | нет |
| TreeMap/Set | O(log n) | O(log n) | O(log n) | sorted |
| PriorityQueue | O(n), head O(1) | O(log n) | head O(log n) | только priority head |

Big O не показывает allocation, locality, boxing и contention. Для production-выбора измеряйте representative workload.

## Типичные вопросы

1. Почему resize HashMap требует перераспределения?
2. Что сломает mutable key?
3. Почему `ConcurrentHashMap` не делает изменяемое value thread-safe?
4. Когда `CopyOnWriteArrayList` уместен?
5. Чем weakly consistent iterator отличается от snapshot?

## Критерий готовности

Вы готовы, если можете выбрать коллекцию по операциям, объяснить HashMap от hash до bucket, написать comparator без overflow и заменить race `containsKey + put` атомарной операцией.

