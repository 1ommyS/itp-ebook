# Algorithms — подробный конспект

## Анализ сложности

Time complexity считают относительно размера входа и выбранной elementary operation. Удаляют константы и младшие члены: `3n² + 5n + 10` → O(n²). Но при инженерном выборе константы, cache locality и allocations снова важны.

Различайте worst, average, best и amortized complexity. Dynamic array `add` иногда O(n), но последовательность n добавлений O(n), значит amortized O(1).

Space complexity включает auxiliary memory, а не всегда вход. Рекурсивный DFS использует O(height) stack; на вырожденном дереве это O(n).

## Two pointers

Подтипы:

- навстречу: pair sum в sorted array, palindrome;
- одинаковое направление: remove duplicates, compact array;
- fast/slow: cycle detection, middle linked list;
- два массива: merge sorted sequences.

Инвариант для pair sum: если сумма мала, все пары с текущим left и ещё меньшим right тоже малы, поэтому left можно увеличить.

```java
static boolean hasPair(int[] sorted, int target) {
    int l = 0, r = sorted.length - 1;
    while (l < r) {
        long sum = (long) sorted[l] + sorted[r];
        if (sum == target) return true;
        if (sum < target) l++; else r--;
    }
    return false;
}
```

Long защищает от overflow сложения int.

## Sliding window

Fixed window поддерживает агрегат длины k: добавить справа, удалить слева. Variable window работает, когда условие монотонно при расширении/сжатии, например сумма положительных чисел.

Для произвольных отрицательных значений условие суммы может перестать быть монотонным, и стандартное окно не работает; нужны prefix sums/deque/map.

## Frequency map

Hash map хранит count, first index или last seen. Выберите минимально нужную информацию. Для маленького фиксированного alphabet массив быстрее и проще map.

Two Sum за O(n): до записи текущего числа ищите complement, чтобы не использовать один элемент дважды.

## Monotonic stack

Хранит indices кандидатов в возрастающем/убывающем порядке. При нарушении порядка элементы выталкиваются и получают ответ. Каждый index push/pop один раз → O(n).

Задачи: next greater element, daily temperatures, largest rectangle, stock span. Для duplicate values заранее решите strict/non-strict comparison (`<` или `<=`).

## Trees

Traversal:

- preorder — обработать до детей, сериализация/копирование;
- inorder BST — sorted order;
- postorder — результат зависит от детей, удаление/height;
- BFS — уровни и shortest edge distance.

Recursive DFS прост, iterative DFS контролирует stack и избегает stack overflow. Complexity O(n) time, O(height) DFS space, O(width) BFS space.

## Graphs

Представления: adjacency list O(V+E) memory, matrix O(V²) и O(1) проверка ребра. Directed/undirected влияет на добавление edges.

Visited помечают при enqueue/push, если повторные добавления не нужны. Для поиска cycle в directed graph используют colors: white/gray/black; edge к gray означает back edge.

Topological sort возможен только для DAG: DFS postorder или Kahn indegrees. Если обработано меньше V, есть cycle.

## Dijkstra

Хранит best known distance и priority queue пар `(distance, vertex)`. Java PriorityQueue не умеет decrease-key удобно, поэтому добавляют новую запись, а устаревшую пропускают.

Отрицательное ребро ломает greedy proof. Для negative edges применяют Bellman–Ford; для all-pairs — Floyd–Warshall или повторные searches в зависимости от density.

## Sorting

QuickSort: partition относительно pivot, хорошая locality, average O(n log n), worst O(n²). Random/median pivot снижает вероятность worst case.

MergeSort: стабильный O(n log n), требует O(n) для array, хорошо подходит linked lists/external sorting. Stability важна при последовательных сортировках по нескольким ключам.

Counting/radix sorts обходят comparison lower bound при ограниченном key domain.

## Binary search

Главный приём — монотонный predicate. Полуинтервал `[lo, hi)` уменьшает число специальных случаев. Для answer search выбирают `mid`, проверяют feasible и сохраняют половину, где лежит граница.

Ошибки: overflow `(lo+hi)/2`, бесконечный цикл из-за неверного обновления, смешение inclusive/exclusive bounds, несортированный input.

## Heap

Binary heap — complete tree в array: parent `(i-1)/2`, children `2i+1`, `2i+2`. Insert sift-up, extract sift-down.

Top K smallest можно хранить max-heap size K: новый меньший заменяет root. Complexity O(n log k), memory O(k).

## Trie

Каждое ребро — символ/часть ключа. Lookup O(length), независимо от числа keys, но memory может быть большим. Compressed radix trie объединяет цепочки одного ребёнка.

Для Unicode «символ» может быть code point, не `char`; требования normalization/case folding определяются предметной областью.

## Как доказывать решение

Назовите invariant, initialization, maintenance и termination. Затем объясните, почему после termination invariant даёт ответ. Это сильнее фразы «видно, что работает».

## Критерий готовности

Вы готовы, если распознаёте паттерн по constraints, пишете binary search без путаницы границ, объясняете O(n) monotonic stack и выбираете BFS/Dijkstra с доказательством допустимости.

