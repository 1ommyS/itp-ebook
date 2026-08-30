# JVM и многопоточность

## Карта JVM

Путь выполнения удобно держать в голове как цепочку:

`source → bytecode → class loading → runtime data areas → interpreter/JIT → machine code`

Class Loader Subsystem загружает классы, проверяет и связывает их, а затем выполняет инициализацию static state. Модель делегирования сначала спрашивает родительский loader и защищает базовые классы от случайной подмены.

| Область | Что хранится | Принадлежность |
|---|---|---|
| Heap | Объекты и массивы | Общая для потоков |
| Java stack | Frames, локальные переменные, operand stack | Отдельная для потока |
| Metaspace | Метаданные загруженных классов | Native memory процесса |
| PC register | Текущая инструкция | Отдельный для потока |
| Native stack | Вызовы native-кода | Отдельный для потока |

Локальная переменная со ссылкой может находиться в stack frame, а сам объект — в heap. Фраза «локальные объекты живут в стеке» как общее правило неверна: размещение — деталь оптимизации JVM.

## Загрузка и инициализация классов

Основные этапы:

1. **Loading** — поиск байткода и создание представления класса.
2. **Linking** — verification, preparation и resolution символических ссылок.
3. **Initialization** — выполнение class initialization method с присваиваниями static-полям и static-блоками.

Инициализация происходит при первом активном использовании. Class identity включает не только имя, но и загрузивший `ClassLoader`; одинаковый байткод из разных loaders даёт разные типы.

## Garbage Collection

Объект считается достижимым, если существует путь от GC roots: активных stack frames, static-ссылок, JNI references и внутренних roots JVM. Reference counting недостаточен из-за циклов.

### G1 и ZGC

- **G1** делит heap на regions, собирает сначала регионы с наибольшей ожидаемой отдачей и старается соблюдать цель паузы. Это хороший general-purpose collector.
- **ZGC** выполняет большую часть работы конкурентно с приложением и ориентирован на очень короткие паузы при больших heaps. Цена — дополнительные ресурсы и иные компромиссы throughput.

Выбор collector начинается с требований к latency, throughput и памяти, а подтверждается метриками и нагрузочным тестом. Название collector само по себе не лечит утечку или чрезмерную аллокацию.

## JIT, warmup и escape analysis

HotSpot сначала интерпретирует или быстро компилирует код, собирает профиль и оптимизирует горячие участки. Поэтому короткий микротест часто измеряет warmup, class loading и оптимизацию, а не steady-state.

Escape analysis позволяет понять, выходит ли объект за пределы метода или потока. JVM может устранить аллокацию, разложить объект на скалярные значения или убрать лишнюю блокировку. Это оптимизация, а не обещание разместить любой локальный объект в stack.

Для microbenchmark используйте JMH: он управляет прогревом, итерациями, dead-code elimination и fork JVM.

## Байт-код и диагностика

`javap -c` помогает увидеть инструкции вызова, boxing и compiler-generated код. В production полезны:

- `jcmd <pid> VM.flags` и `GC.heap_info` — параметры и heap;
- `jstack <pid>` — stack traces и поиск deadlock;
- `jmap`/heap dump — анализ удерживаемых объектов;
- Java Flight Recorder — профиль CPU, allocations, locks и pauses с умеренной нагрузкой.

Снимайте данные до перезапуска процесса: после рестарта причина часто исчезает вместе с доказательствами.

## Потоки и задачи

Состояния `Thread`: NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED. `BLOCKED` относится к ожиданию monitor entry; ожидание `Future.get()` обычно отображается как WAITING/TIMED_WAITING.

`Runnable` не возвращает результат и не объявляет checked exception. `Callable<V>` возвращает `V` и может выбросить exception. В прикладном коде предпочитайте управление задачами через `ExecutorService`, а не ручное создание потоков.

```java
var executor = Executors.newFixedThreadPool(8);
try {
    Future<Result> future = executor.submit(() -> calculate());
    return future.get(500, TimeUnit.MILLISECONDS);
} finally {
    executor.shutdown();
}
```

Размер pool зависит от характера нагрузки и ограничений downstream. Неограниченная очередь может скрывать перегрузку ростом latency и памяти.

## `synchronized`, `volatile` и atomics

| Механизм | Даёт | Не даёт |
|---|---|---|
| `synchronized` | Взаимное исключение, visibility, ordering | Автоматической защиты кода вне того же monitor |
| `volatile` | Видимость последней записи и порядок вокруг неё | Атомарности `count++` |
| Atomic classes | Атомарные операции через CAS | Инварианта между несколькими независимыми переменными |
| `Lock` | Явное управление, timeout, interruptible acquisition | Автоматического release без `finally` |

```java
lock.lock();
try {
    updateInvariant();
} finally {
    lock.unlock();
}
```

`ReadWriteLock` оправдан, если чтения существенно чаще и достаточно длинные. Для коротких критических секций overhead и starvation могут убрать выигрыш.

## Java Memory Model и happens-before

JMM описывает, когда запись одного потока обязана быть видна другому и какие переупорядочивания допустимы. Основные happens-before edges:

- действие до выхода из monitor — действию после последующего входа в тот же monitor;
- запись в volatile — последующему чтению этой переменной;
- действия до `Thread.start()` — действиям нового потока;
- действия потока — успешному возврату из `join()`;
- инициализация final-полей при корректном конструировании — их чтению после безопасной публикации.

Без happens-before race остаётся ошибкой, даже если «на моём компьютере работает».

### Safe publication

Надёжные варианты: static initialization, запись ссылки под lock, volatile reference, concurrent collection или передача через synchronizer/queue. Не публикуйте `this` из конструктора: другой поток может увидеть частично построенный объект.

## Deadlock, livelock и starvation

- **Deadlock:** участники циклически ждут ресурсы друг друга.
- **Livelock:** потоки активны и уступают, но полезная работа не продвигается.
- **Starvation:** поток систематически не получает CPU или lock.

Профилактика deadlock: единый порядок захвата, короткие критические секции, отсутствие внешних вызовов под lock, `tryLock` с контролируемым timeout и диагностика через thread dump.

## `CompletableFuture`

Используйте `thenApply` для синхронного преобразования результата, `thenCompose` для композиции асинхронных этапов, `thenCombine` для независимых ветвей. Всегда задавайте политику executor, timeout и обработки исключений.

```java
return loadUser(id)
        .thenCompose(user -> loadOrders(user.id()))
        .orTimeout(500, TimeUnit.MILLISECONDS)
        .exceptionally(this::fallback);
```

## Virtual threads и Fork/Join

Virtual threads позволяют иметь очень много блокирующих задач с привычным imperative-кодом. Они уменьшают стоимость ожидания, но не ускоряют CPU-bound работу и не отменяют лимиты БД, соединений и внешних API. Ограничивайте downstream отдельными semaphore/rate limits.

`ForkJoinPool` предназначен для рекурсивно делимых CPU-bound задач и использует work stealing. Blocking I/O в таком pool может задержать все задачи.

## Вопросы для самопроверки

1. Почему `volatile int count` не делает `count++` безопасным?
2. Как thread dump подтверждает deadlock?
3. Чем reachability отличается от «объект больше не нужен бизнесу»?
4. Почему benchmark без warmup вводит в заблуждение?
5. Что virtual threads меняют, а что оставляют прежним?
