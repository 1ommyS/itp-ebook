## Под капотом: как работает многопоточность в Java

При работе с многопоточностью в Java ключевую роль играют механизмы операционной системы и саму JVM (Java Virtual Machine). Когда вы создаёте поток, JVM запрашивает у операционной системы ресурсы (например, выделение стека для нового потока) и планирует выполнение потоков с учётом приоритетов и квантов времени. Под капотом синхронизация осуществляется с помощью специальных примитивов, предоставляемых операционной системой (например, мьютексов и семафоров), которые интегрированы в механизм «мониторов» в JVM.

**Мониторы и блокировки:**

- **Монитор:** Каждый объект в Java имеет связанный с ним монитор. Когда вы используете ключевое слово `synchronized`, поток пытается захватить монитор соответствующего объекта. Если монитор уже занят другим потоком, выполнение блокируется до его освобождения.
- **Интринсивные (встроенные) локи:** Управляются JVM, что позволяет автоматически освобождать блокировку при выходе из синхронизированного блока или метода.
- **Явные локи:** (например, `ReentrantLock`, `ReadWriteLock` и `StampedLock`) предоставляют расширенные возможности управления, позволяя, например, использовать таймауты, не блокировать поток навсегда и осуществлять неблокирующий вызов.

Под капотом JVM использует низкоуровневые инструкции процессора для реализации атомарных операций (например, CAS — compare-and-swap), что делает возможными неблокирующие алгоритмы и эффективную работу многопоточных программ.

---

## Статья: Многопоточность в Java

Многопоточность (Concurrency) — это способность программы выполнять несколько потоков одновременно, что позволяет эффективнее использовать ресурсы процессора и улучшать производительность приложений. В Java многопоточность реализована на основе класса `Thread` и интерфейса `Runnable`, а также через более высокоуровневые конструкции из пакета `java.util.concurrent`. В этой статье рассмотрены основные концепции многопоточности, виды локов, мониторы, семафоры, потокобезопасные коллекции, `ExecutorService`, проблемы многопоточности (такие как дедлок и ливелок) и способы борьбы с ними.

---

### Содержание

1. [Основы многопоточности в Java](#1-основы-многопоточности-в-java)
2. [Синхронизация и мониторы](#2-синхронизация-и-мониторы)
3. [Виды локов в Java](#3-виды-локов-в-java)
   - [Интринсивные локи (synchronized)](#31-интринсивные-локи-synchronized)
   - [Явные локи (Lock интерфейс)](#32-явные-локи-lock-интерфейс)
4. [Семафоры](#4-семафоры)
5. [Потокобезопасные коллекции](#5-потокобезопасные-коллекции)
6. [ExecutorService](#6-executorservice)
7. [Проблемы многопоточности](#7-проблемы-многопоточности)
   - [Дедлок (Deadlock)](#71-дедлок-deadlock)
   - [Ливелок (Livelock)](#72-ливелок-livelock)
8. [Способы борьбы с дедлоком и ливелоком](#8-способы-борьбы-с-дедлоком-и-ливелоком)
9. [Таблица сравнения локов](#9-таблица-сравнения-локов)
10. [Заключение](#10-заключение)

---

### 1. Основы многопоточности в Java

**Поток** (Thread) — это единица выполнения, которая может работать параллельно с другими потоками. В Java существует два основных способа создания потоков:

1. **Наследование от класса `Thread`:**

    ```java
    public class MyThread extends Thread {
        @Override
        public void run() {
            // Код, выполняемый в новом потоке
            System.out.println("Поток выполняется: " + Thread.currentThread().getName());
        }
    }

    public class Main {
        public static void main(String[] args) {
            MyThread thread = new MyThread();
            thread.start(); // Запуск нового потока
        }
    }
    ```

2. **Реализация интерфейса `Runnable`:**

    ```java
    public class MyRunnable implements Runnable {
        @Override
        public void run() {
            // Код, выполняемый в новом потоке
            System.out.println("Поток выполняется: " + Thread.currentThread().getName());
        }
    }

    public class Main {
        public static void main(String[] args) {
            Thread thread = new Thread(new MyRunnable());
            thread.start(); // Запуск нового потока
        }
    }
    ```

**Важно:** Рекомендуется использовать реализацию интерфейса `Runnable` или функциональный интерфейс `Callable`, поскольку Java поддерживает множественное наследование интерфейсов, но только одиночное наследование классов.

---

### 2. Синхронизация и мониторы

В многопоточном программировании необходимо контролировать доступ к общим ресурсам, чтобы избежать состояния гонки (race conditions) и обеспечить корректность данных.

- **Синхронизация** — это механизм, позволяющий управлять доступом к общим ресурсам, гарантируя, что только один поток может выполнять определённый блок кода или метод в данный момент времени.
- **Монитор** — внутренняя структура, связанная с объектом, которая обеспечивает синхронизацию. Каждый объект в Java имеет монитор, который используется при выполнении синхронизированных методов или блоков.

**Пример синхронизированного метода:**

```java
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

**Пример синхронизированного блока:**

```java
public class Counter {
    private int count = 0;
    private final Object lock = new Object();

    public void increment() {
        synchronized (lock) {
            count++;
        }
    }

    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }
}
```

Особенности мониторов включают взаимное исключение (mutual exclusion) и возможность приостановки/возобновления работы потоков с использованием методов `wait()`, `notify()` и `notifyAll()`.

---

### 3. Виды локов в Java

Java предоставляет два основных подхода к синхронизации: интринсивные (встроенные) локи и явные (программируемые) локи.

#### 3.1. Интринсивные локи (`synchronized`)

Интринсивные локи используют ключевое слово `synchronized` и являются встроенной частью языка. При входе в синхронизированный блок или метод поток автоматически захватывает монитор объекта, а при выходе из него — освобождает его.

**Преимущества:**

- Простота использования.
- Автоматическое освобождение блокировки.

**Недостатки:**

- Меньше возможностей по сравнению с явными локами.
- Могут привести к дедлокам при неправильном использовании.

**Пример:**

```java
public class SynchronizedCounter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}
```

#### 3.2. Явные локи (`Lock` интерфейс)

Явные локи, реализуемые через классы из пакета `java.util.concurrent.locks`, дают более гибкий контроль над синхронизацией. Они позволяют использовать такие методы, как `tryLock()`, таймауты и прерывание, что полезно при решении сложных задач синхронизации.

**Основные реализации:**

- **`ReentrantLock`:** Рекурсивная блокировка, позволяющая одному потоку захватывать лок несколько раз.
- **`ReadWriteLock`:** Разделяет доступ на чтение и запись, позволяя нескольким потокам читать одновременно, но только одному писать.
- **`StampedLock`:** Введение в Java 8, позволяющее ещё эффективнее управлять блокировками, поддерживая оптимистичные чтения.

**Пример использования `ReentrantLock`:**

```java
import java.util.concurrent.locks.ReentrantLock;

public class LockCounter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public void increment() {
        lock.lock(); // Захват лока
        try {
            count++;
        } finally {
            lock.unlock(); // Освобождение лока
        }
    }

    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

---

### 4. Семафоры

**Семафор** — это синхронизирующий примитив, который ограничивает количество потоков, одновременно получающих доступ к ресурсу. В Java он реализован классом `Semaphore` из пакета `java.util.concurrent`.

**Основные методы:**

- `acquire()` — захватывает разрешение, блокируя поток, если разрешений нет.
- `release()` — освобождает разрешение.
- `availablePermits()` — возвращает количество доступных разрешений.

**Пример:**

```java
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
    private static final int MAX_PERMITS = 3;
    private final Semaphore semaphore = new Semaphore(MAX_PERMITS);

    public void accessResource() {
        try {
            semaphore.acquire(); // Захват разрешения
            System.out.println(Thread.currentThread().getName() + " получил доступ к ресурсу.");
            // Имитация работы с ресурсом
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            System.out.println(Thread.currentThread().getName() + " освобождает ресурс.");
            semaphore.release(); // Освобождение разрешения
        }
    }

    public static void main(String[] args) {
        SemaphoreExample example = new SemaphoreExample();

        Runnable task = example::accessResource;

        for (int i = 0; i < 5; i++) {
            new Thread(task, "Поток-" + (i + 1)).start();
        }
    }
}
```

---

### 5. Потокобезопасные коллекции

В многопоточном окружении важно использовать коллекции, безопасные для доступа несколькими потоками. Java предоставляет их в пакете `java.util.concurrent`.

**Основные потокобезопасные коллекции:**

- **`ConcurrentHashMap`:** Позволяет одновременное чтение и модификацию.
- **`CopyOnWriteArrayList`:** При каждом изменении создаётся новая копия массива.
- **`CopyOnWriteArraySet`:** Основана на `CopyOnWriteArrayList`.
- **`BlockingQueue`:** Интерфейс для очередей с блокирующими операциями (например, `LinkedBlockingQueue`, `ArrayBlockingQueue`).
- **`ConcurrentLinkedQueue`:** Неблокирующая очередь на основе связанного списка.

**Пример использования `ConcurrentHashMap`:**

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentHashMapExample {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        // Добавление элементов
        map.put("Apple", 3);
        map.put("Banana", 2);
        map.put("Orange", 5);

        // Параллельное обновление
        map.forEach((key, value) -> map.put(key, value + 1));

        System.out.println(map); // {Apple=4, Banana=3, Orange=6}
    }
}
```

**Пример использования `CopyOnWriteArrayList`:**

```java
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteArrayListExample {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();
        list.add("Java");
        list.add("Python");
        list.add("C++");

        // Параллельное добавление элементов
        new Thread(() -> list.add("JavaScript")).start();
        new Thread(() -> list.add("Go")).start();

        // Итерация без ConcurrentModificationException
        list.forEach(System.out::println);
    }
}
```

---

### 6. ExecutorService

**`ExecutorService`** — высокоуровневый интерфейс для управления пулом потоков, позволяющий отправлять задачи на выполнение без необходимости явного создания и управления потоками.

**Основные методы:**

- `submit(Runnable task)` — отправка задачи на выполнение.
- `submit(Callable<T> task)` — отправка задачи с возвращением результата через `Future`.
- `shutdown()` — инициирует корректное завершение работы.
- `shutdownNow()` — пытается немедленно завершить выполнение задач.

**Пример использования:**

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ExecutorServiceExample {
    public static void main(String[] args) {
        // Создание пула из 3 потоков
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // Отправка задач на выполнение
        for (int i = 1; i <= 5; i++) {
            int taskId = i;
            executor.submit(() -> {
                System.out.println("Выполнение задачи " + taskId + " в " + Thread.currentThread().getName());
                try {
                    Thread.sleep(2000); // Имитация работы
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Задача " + taskId + " завершена в " + Thread.currentThread().getName());
            });
        }

        // Корректное завершение ExecutorService
        executor.shutdown();
        try {
            if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
        }
    }
}
```

---

### 7. Проблемы многопоточности

При разработке многопоточных приложений могут возникать следующие проблемы:

#### 7.1. Дедлок (Deadlock)

Дедлок возникает, когда два или более потоков блокируют друг друга, ожидая освобождения ресурсов, занятых другим потоком. Это может произойти, если потоки захватывают ресурсы в разном порядке.

**Пример дедлока:**

```java
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void methodA() {
        synchronized (lock1) {
            System.out.println("Поток " + Thread.currentThread().getName() + " захватил lock1");
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            synchronized (lock2) {
                System.out.println("Поток " + Thread.currentThread().getName() + " захватил lock2");
            }
        }
    }

    public void methodB() {
        synchronized (lock2) {
            System.out.println("Поток " + Thread.currentThread().getName() + " захватил lock2");
            try { Thread.sleep(100); } catch (InterruptedException e) { }
            synchronized (lock1) {
                System.out.println("Поток " + Thread.currentThread().getName() + " захватил lock1");
            }
        }
    }

    public static void main(String[] args) {
        DeadlockExample example = new DeadlockExample();

        Thread t1 = new Thread(example::methodA, "Поток-1");
        Thread t2 = new Thread(example::methodB, "Поток-2");

        t1.start();
        t2.start();
    }
}
```

*Вывод (пример):*
```
Поток Поток-1 захватил lock1
Поток Поток-2 захватил lock2
```
— оба потока ждут освобождения второго локатора.

#### 7.2. Ливелок (Livelock)

Ливелок возникает, когда потоки активно изменяют своё состояние в попытке избежать дедлока, но ни один поток не делает реального прогресса.

**Пример ливелока:**

```java
public class LivelockExample {
    private boolean isTaker = false;

    public synchronized void takeTurn(LivelockExample other) {
        while (other.isTaker) {
            System.out.println(Thread.currentThread().getName() + " ждет своей очереди.");
            try { wait(); } catch (InterruptedException e) { }
        }
        isTaker = true;
        System.out.println(Thread.currentThread().getName() + " берет свою очередь.");
        isTaker = false;
        notifyAll();
    }

    public static void main(String[] args) {
        LivelockExample obj1 = new LivelockExample();
        LivelockExample obj2 = new LivelockExample();

        Runnable task1 = () -> {
            while (true) {
                obj1.takeTurn(obj2);
            }
        };

        Runnable task2 = () -> {
            while (true) {
                obj2.takeTurn(obj1);
            }
        };

        new Thread(task1, "Поток-1").start();
        new Thread(task2, "Поток-2").start();
    }
}
```

*Вывод (пример):*
```
Поток-1 берет свою очередь.
Поток-2 берет свою очередь.
Поток-1 берет свою очередь.
Поток-2 берет свою очередь.
...
```

---

### 8. Способы борьбы с дедлоком и ливелоком

#### 8.1. Избежание дедлока

1. **Упорядочивание захвата ресурсов:**  
   Все потоки должны захватывать ресурсы в одном и том же порядке.

    ```java
    public class DeadlockPrevention {
        private final Object lock1 = new Object();
        private final Object lock2 = new Object();

        public void methodA() {
            synchronized (lock1) {
                synchronized (lock2) {
                    // Критическая секция
                }
            }
        }

        public void methodB() {
            synchronized (lock1) { // Захват ресурсов в одном порядке
                synchronized (lock2) {
                    // Критическая секция
                }
            }
        }
    }
    ```

2. **Использование таймаутов:**  
   Применение методов вроде `tryLock(timeout, unit)` для избежания бесконечного ожидания.

    ```java
    import java.util.concurrent.locks.ReentrantLock;
    import java.util.concurrent.TimeUnit;

    public class TimeoutLock {
        private final ReentrantLock lock1 = new ReentrantLock();
        private final ReentrantLock lock2 = new ReentrantLock();

        public void safeMethod() {
            try {
                if (lock1.tryLock(1, TimeUnit.SECONDS)) {
                    try {
                        if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                            try {
                                // Критическая секция
                            } finally {
                                lock2.unlock();
                            }
                        }
                    } finally {
                        lock1.unlock();
                    }
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    ```

3. **Использование одного общего лока:**  
   Вместо нескольких локов можно использовать один общий для связанных ресурсов.

#### 8.2. Избежание ливелока

1. **Стабильное поведение потоков:**  
   Избегать чрезмерно агрессивного изменения состояния при попытке захватить ресурс.

2. **Ограничение количества попыток:**  
   Использование счетчиков или задержек для предотвращения бесконечного цикла попыток.

3. **Установка приоритетов:**  
   Присвоение потокам разных приоритетов для упорядочивания доступа к ресурсам.

**Пример предотвращения ливелока с ограничением количества попыток:**

```java
public class LivelockPrevention {
    private boolean isTaker = false;

    public synchronized void takeTurn(LivelockPrevention other) {
        // Если оба потока активны, выйти, чтобы избежать бесконечного ожидания
        if (!isTaker && !other.isTaker) {
            isTaker = true;
            System.out.println(Thread.currentThread().getName() + " берет свою очередь.");
            isTaker = false;
            notifyAll();
        } else {
            System.out.println(Thread.currentThread().getName() + " ждет своей очереди.");
            try { wait(100); } catch (InterruptedException e) { }
        }
    }

    public static void main(String[] args) {
        LivelockPrevention obj1 = new LivelockPrevention();
        LivelockPrevention obj2 = new LivelockPrevention();

        Runnable task1 = () -> {
            for (int i = 0; i < 5; i++) {
                obj1.takeTurn(obj2);
            }
        };

        Runnable task2 = () -> {
            for (int i = 0; i < 5; i++) {
                obj2.takeTurn(obj1);
            }
        };

        new Thread(task1, "Поток-1").start();
        new Thread(task2, "Поток-2").start();
    }
}
```

---

### 9. Таблица сравнения локов

Ниже приведена таблица, которая поможет сравнить основные характеристики различных типов локов в Java:

| **Тип локов**         | **Механизм**                    | **Основные возможности**                                                   | **Преимущества**                                         | **Недостатки**                                           |
|-----------------------|---------------------------------|---------------------------------------------------------------------------|----------------------------------------------------------|----------------------------------------------------------|
| `synchronized`        | Интринсивный (встроенный в JVM) | Автоматическое захватывание и освобождение монитора, упрощённый синтаксис    | Простота, отсутствие явного освобождения                 | Ограниченная функциональность, невозможность таймаута     |
| `ReentrantLock`       | Явный (Lock API)                | Рекурсивность, возможность прерывания, таймаут на захват (tryLock)          | Гибкость, расширенные возможности управления             | Необходимость явного освобождения (unlock)                |
| `ReadWriteLock`       | Явный (Lock API)                | Разделение доступа: многопоточное чтение и эксклюзивная запись              | Повышение производительности при частых операциях чтения  | Сложность реализации, не всегда подходит для записи       |
| `StampedLock`         | Явный (Lock API, Java 8+)       | Оптимистичные чтения, режимы эксклюзивного и совместного доступа             | Более высокая производительность, оптимизация чтения      | Более сложный API, потенциальные сложности при использовании |

---

### 10. Заключение

Многопоточность является важной составляющей разработки эффективных приложений на Java. Понимание того, как работает синхронизация «под капотом», помогает выбрать оптимальный инструмент для конкретной задачи. Использование встроенных механизмов (ключевое слово `synchronized`) подходит для простых случаев, а явные локи, такие как `ReentrantLock`, `ReadWriteLock` и `StampedLock`, позволяют решать более сложные проблемы синхронизации, предоставляя расширенные возможности управления блокировками. При этом важно избегать распространённых проблем, таких как дедлок и ливелок, используя проверенные подходы и инструменты диагностики (Thread Dumps, Java VisualVM).

Эффективное применение потокобезопасных коллекций и `ExecutorService` позволяет создавать масштабируемые и надежные многопоточные приложения, а таблица сравнения локов помогает выбрать правильное решение для конкретных условий.

---

Данный материал должен помочь в глубоких знаниях многопоточности в Java и применении лучших практик для создания стабильных и эффективных приложений.