# Concurrency — подробный конспект

## Базовые проблемы

Конкурентность создаёт три разных класса требований:

- atomicity — операция наблюдается целиком;
- visibility — поток видит запись другого;
- ordering — compiler/CPU/JIT не меняют наблюдаемый порядок за разрешённые границы.

Race condition — результат зависит от недетерминированного interleaving. Data race — конфликтующие обращения к памяти без happens-before, хотя race condition может существовать и на более высоком бизнес-уровне.

## Thread lifecycle

- NEW — `start` не вызван;
- RUNNABLE — готов или исполняется, JVM объединяет OS ready/running;
- BLOCKED — ждёт вход в synchronized monitor;
- WAITING — ждёт без срока (`wait`, `join`, park);
- TIMED_WAITING — ждёт с timeout;
- TERMINATED — завершён.

Нельзя вызвать `start()` дважды. Вызов `run()` напрямую — обычный метод в текущем потоке.

## Runnable, Callable, Future

Callable возвращает result и бросает checked exception. Future представляет pending result, но `get` блокирует. Cancellation — cooperative: interrupt не убивает поток, а выставляет flag/вызывает `InterruptedException` в interruptible wait.

Правило обработки interruption: либо завершить/пробросить `InterruptedException`, либо восстановить flag через `Thread.currentThread().interrupt()`. Проглатывание лишает вышестоящий код возможности остановить работу.

## ExecutorService и pools

Pool отделяет submission от execution. Важны:

- число workers;
- bounded/unbounded queue;
- rejection policy;
- naming и uncaught exception handling;
- graceful shutdown;
- метрики active, queued, completed, rejected.

Unbounded queue не создаёт backpressure и превращает перегрузку в рост latency/heap. Caller-runs policy может естественно замедлить producer, но выполнение в неожиданном caller thread влияет на latency и context.

CPU-bound pool близок к числу available processors с учётом других процессов. I/O-bound может быть больше, но downstream capacity остаётся пределом.

## `synchronized`

Monitor связан с объектом; static synchronized использует `Class` object. Вход даёт mutual exclusion, выход publishes writes следующему входу в тот же monitor.

Lock object должен быть private и неизменяемой ссылкой. Синхронизация на interned string, boxed integer или публичном объекте позволяет чужому коду разделить lock.

`wait` атомарно освобождает monitor и ждёт; возвращается после reacquire. Проверять условие нужно в `while`, потому что возможны spurious wakeups и другой поток может изменить состояние до reacquire.

## `volatile`

Запись volatile happens-before последующему чтению того же поля. Это полезно для stop flag, immutable snapshot reference и state machine с одной переменной.

`count++` — read-modify-write из нескольких шагов, поэтому volatile не предотвращает lost update. Для связанного инварианта двух полей одна volatile-переменная тоже недостаточна; публикуйте immutable holder одной ссылкой или используйте lock.

## CAS и atomics

CAS меняет значение, только если оно равно ожидаемому. При конфликте операция повторяется. Это lock-free building block, но не «бесплатная синхронизация»: высокая конкуренция создаёт retries/cache-line traffic.

ABA: значение A изменилось на B и обратно A, CAS не замечает промежуточное изменение. Решение — version stamp (`AtomicStampedReference`) или модель, где ABA безопасно.

`LongAdder` распределяет contention по cells. Хорош для metrics counters, но `sum` не является atomic snapshot относительно одновременных updates.

## ReentrantLock и ReadWriteLock

ReentrantLock даёт `tryLock`, timeout, interruptible lock и несколько Conditions. Всегда unlock в `finally`.

Fair lock уменьшает starvation, но снижает throughput. ReadWriteLock полезен при длинных чтениях и редких записях; upgrade read→write может deadlock, если не предусмотрен API.

`StampedLock` поддерживает optimistic read, но не reentrant; результат optimistic read обязателен к `validate`, а работа с stamp требует аккуратного `finally`.

## Java Memory Model

Happens-before — отношение видимости и порядка, не реальная временная шкала. Оно транзитивно.

Главные правила:

- program order внутри потока;
- monitor unlock → последующий lock;
- volatile write → последующий read;
- действия до `start` → действия нового thread;
- действия thread → успешный `join`;
- static initialization → использование класса;
- transitivity.

Final field semantics дают особую гарантию после корректного construction, если `this` не escape constructor.

## Safe publication

Варианты: static initializer, volatile reference, lock, concurrent collection, Future/queue. Публикация mutable object делает видимым состояние на момент публикации, но последующие изменения всё равно требуют синхронизации.

Double-checked locking требует `volatile` instance, иначе другой поток может увидеть ссылку до завершения construction из-за reorder.

## Deadlock, livelock, starvation

Условия Coffman: mutual exclusion, hold-and-wait, no preemption, circular wait. Нарушьте хотя бы одно. На практике — общий порядок locks, acquire all-or-release, timeout и отсутствие внешнего вызова под lock.

Livelock — участники реагируют друг на друга, но не продвигаются; random backoff/jitter помогает. Starvation — отсутствие справедливого доступа; причины: priority, unfair lock, бесконечный read traffic.

## CompletableFuture

- `thenApply`: `T → U`;
- `thenCompose`: `T → CompletionStage<U>` без вложенности;
- `thenCombine`: объединить независимые stages;
- `exceptionally`: заменить ошибку значением;
- `handle`: увидеть result/error;
- `whenComplete`: side effect, не преобразование.

Async-варианты используют заданный executor или common pool. Без suffix continuation может выполниться в thread, завершившем предыдущий stage.

Timeout future не обязательно отменяет underlying I/O. Нужен timeout самого HTTP/DB client и cancellation propagation.

## Virtual threads

Virtual thread монтируется на carrier platform thread при выполнении и размонтируется на поддерживаемом blocking. Это делает thread-per-request практичным для I/O-bound workloads.

Ограничения:

- CPU не становится больше;
- connection pool/downstream quota остаются ограниченными;
- ThreadLocal на миллионы threads может быть дорогим;
- долгий synchronized/native critical section может pin carrier в некоторых сценариях;
- нужен structured lifecycle задач, а не бесконечное создание без контроля.

## ForkJoinPool

Каждый worker имеет deque, idle workers steal с другого конца. RecursiveTask делит задачу до threshold. Слишком мелкие tasks дают overhead, слишком крупные — плохой parallelism.

Managed blocking или отдельный executor нужен для blocking операций; ForkJoin оптимизирован для CPU work.

## Вопросы

1. Почему visibility не равна atomicity?
2. Что делает `while` вокруг `wait` обязательным?
3. Где CAS проигрывает lock?
4. Почему timeout CompletableFuture не гарантирует отмену HTTP-запроса?
5. Какие ресурсы ограничивают virtual-thread приложение?

## Критерий готовности

Вы готовы, если можете доказать happens-before, исправить lost update тремя способами, диагностировать deadlock по dump и спроектировать bounded executor/virtual-thread решение с backpressure.

