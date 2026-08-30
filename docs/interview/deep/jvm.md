# JVM — подробный конспект

## От исходника до выполнения

`javac` компилирует source в class files с bytecode, constant pool и metadata. JVM загружает классы, проверяет безопасность bytecode, связывает символические ссылки и выполняет код интерпретатором/JIT.

Bytecode — инструкция абстрактной stack machine. Это обеспечивает переносимость class file между совместимыми JVM, но конкретный machine code зависит от процессора и профиля запуска.

## Class loading

### Loading

ClassLoader находит bytes и вызывает `defineClass`. Результат — объект `Class<?>` и внутренние metadata. Identity класса равна паре `(binary name, defining ClassLoader)`.

### Linking

- Verification проверяет формат, типы operand stack и допустимость инструкций.
- Preparation выделяет static fields и задаёт default values.
- Resolution заменяет symbolic references прямыми ссылками; часть может выполняться лениво.

### Initialization

Выполняет `<clinit>`: явные присваивания static fields и static blocks в порядке source. Инициализация синхронизирована: один поток выполняет её, остальные ждут. Ошибка может оставить класс в erroneous state для этого loader.

Parent delegation сначала предлагает загрузку parent. Bootstrap загружает базовые Java classes, platform — platform modules, application — classpath/module path приложения. Custom loaders нужны plugin systems, application servers и isolation.

## Runtime data areas

### Heap

Общий для потоков, содержит объекты/массивы. Collector может перемещать объекты; ссылка — не обязательно физический адрес. Young/old — практическая организация generational collectors, а не требование JVM spec.

### Stack

Каждый поток имеет stack frames. Frame содержит local variables, operand stack и данные возврата/constant pool reference. Глубокая рекурсия вызывает `StackOverflowError`; слишком маленький stack снижает допустимую глубину, слишком большой уменьшает число возможных platform threads.

### Metaspace

Хранит class metadata в native memory. Он может расти при утечке ClassLoader: loader удерживает загруженные classes, а его удерживает thread local, static registry или незавершённый thread.

## Reachability и GC roots

Collector начинает от roots и проходит граф ссылок. Типичные roots: локальные/operand references активных frames, static fields, JNI handles, active threads и внутренние structures JVM.

Memory leak в managed runtime — достижимые, но бизнесу уже не нужные объекты. Примеры: cache без eviction, listener без unregister, ThreadLocal в pool, queue быстрее consumer.

Reference types:

- strong — обычная ссылка;
- soft — очистка под memory pressure, непредсказуема для cache policy;
- weak — не удерживает объект от collection;
- phantom — post-mortem coordination через ReferenceQueue, `get()` всегда null.

## Generational hypothesis

Большинство объектов умирает молодыми. Young collection часто и быстро освобождает eden/survivors. Выжившие объекты могут продвигаться в old. Большие/долго живущие allocation и retained graph влияют на old pressure.

Allocation обычно очень быстро благодаря thread-local allocation buffer. Главная цена — не сам `new`, а объём, lifetime, copying и частота collection.

## G1

G1 делит heap на equal-sized regions. Young collection эвакуирует живые объекты из выбранных regions. Concurrent marking оценивает live data old regions; mixed collections добавляют выгодные old regions.

Remembered sets отслеживают межрегиональные ссылки. Humongous objects занимают несколько regions и могут создавать fragmentation/pressure.

Pause target — цель, не SLA. Если allocation rate, remembered-set work или live set слишком велики, collector не обязан уложиться.

## ZGC

ZGC выполняет marking и relocation в основном конкурентно, используя load barriers и metadata в ссылках/таблицах. Он рассчитан на низкие pauses при больших heaps. Concurrent работа потребляет CPU; throughput и memory overhead сравнивают нагрузочным тестом.

## JIT и профилирование

Hot methods обнаруживаются counters/profile. Tiered compilation сочетает быстрый старт и более агрессивную оптимизацию. Оптимизации: inlining, devirtualization, loop optimizations, constant folding, dead-code elimination.

Оптимизация speculative: JVM предполагает наблюдаемые типы. При нарушении предположения выполняется deoptimization и возврат к менее оптимизированному коду.

### Escape analysis

Если объект не escape method/thread, возможны scalar replacement и lock elimination. Нельзя полагаться на это как на контракт и писать некорректный synchronized code «потому что JIT уберёт lock».

## Warmup и benchmark

Первые итерации включают loading, initialization, cache fill и compilation. Наивный `System.nanoTime` benchmark страдает dead-code elimination, constant folding, OS noise и недостатком forks.

JMH использует warmup/measurement iterations, forks, Blackhole и правильную state scope. Benchmark должен повторять реальные размеры данных и contention.

## Диагностика

### Высокий CPU

1. Снять несколько thread dumps.
2. Сопоставить OS thread ID с Java thread.
3. Найти повторяющийся runnable stack.
4. Подтвердить profiler/JFR, а не делать вывод по одному snapshot.

### Hang/deadlock

`jstack`/`jcmd Thread.print` показывает states и owned/waiting locks. Deadlock detector выводит cycle, но application-level ожидание через queues/futures может не быть monitor deadlock.

### Memory growth

Смотрите heap usage после full/concurrent cycles, allocation rate и class count. Heap dump анализируют dominator tree и retained size. Shallow size объекта не показывает удерживаемый граф.

### Инструменты

- `jcmd VM.version`, `VM.flags`, `GC.heap_info`, `Thread.print`;
- JFR для CPU, allocation, locks, file/socket I/O, GC;
- `jmap` и heap dump при контролируемых условиях;
- `javap -c -v` для class file/bytecode;
- Native Memory Tracking для off-heap/metaspace/thread stacks.

## Вопросы

1. Почему class с одинаковым именем может быть несовместим сам с собой?
2. Чем shallow size отличается от retained size?
3. Почему большой heap не всегда уменьшает latency?
4. Что вызывает deoptimization?
5. Как отличить heap leak от native memory growth?

## Критерий готовности

Вы готовы, если можете пройти этапы loading/linking/initialization, нарисовать roots и heap graph, объяснить G1/ZGC без рекламных лозунгов и составить план диагностики CPU, hang и memory leak.

