# Готовые решения задач Kotlin

Ниже приведены эталонные варианты для всех обычных заданий практикума. Итоговый проект намеренно не решён: он должен объединить изученные механизмы самостоятельно. Фрагменты рассчитаны на Kotlin/JVM 2.x; coroutine-примеры требуют `kotlinx-coroutines-core` и `kotlinx-coroutines-test`.

## 1. Безопасный профиль пользователя { #kotlin-01 }

```kotlin
data class UserDto(val name: String?, val age: Int?)
data class UserProfile(val name: String, val age: Int)

sealed interface ProfileResult {
    data class Success(val profile: UserProfile) : ProfileResult
    data class Invalid(val fields: Map<String, String>) : ProfileResult
}

fun normalize(dto: UserDto?): ProfileResult {
    if (dto == null) return ProfileResult.Invalid(mapOf("body" to "required"))
    val errors = buildMap {
        if (dto.name.isNullOrBlank()) put("name", "must not be blank")
        if (dto.age == null || dto.age !in 0..150) put("age", "must be in 0..150")
    }
    if (errors.isNotEmpty()) return ProfileResult.Invalid(errors)
    val validName = dto.name?.trim() ?: error("name was validated")
    val validAge = dto.age ?: error("age was validated")
    return ProfileResult.Success(UserProfile(validName, validAge))
}
```

## 2. Конечный автомат заказа { #kotlin-02 }

```kotlin
sealed interface OrderState {
    data object Created : OrderState
    data class Paid(val paymentId: String) : OrderState
    data class Shipped(val tracking: String) : OrderState
    data class Cancelled(val reason: String) : OrderState
}

sealed interface Command {
    data class Pay(val paymentId: String) : Command
    data class Ship(val tracking: String) : Command
    data class Cancel(val reason: String) : Command
}

sealed interface Transition {
    data class Ok(val state: OrderState) : Transition
    data class Rejected(val message: String) : Transition
}

fun evolve(state: OrderState, command: Command): Transition = when (state) {
    OrderState.Created -> when (command) {
        is Command.Pay -> Transition.Ok(OrderState.Paid(command.paymentId))
        is Command.Cancel -> Transition.Ok(OrderState.Cancelled(command.reason))
        is Command.Ship -> Transition.Rejected("unpaid order")
    }
    is OrderState.Paid -> when (command) {
        is Command.Ship -> Transition.Ok(OrderState.Shipped(command.tracking))
        is Command.Cancel -> Transition.Ok(OrderState.Cancelled(command.reason))
        is Command.Pay -> if (state.paymentId == command.paymentId) Transition.Ok(state)
                          else Transition.Rejected("already paid")
    }
    is OrderState.Shipped -> Transition.Rejected("shipped order is final")
    is OrderState.Cancelled -> when (command) {
        is Command.Cancel -> Transition.Ok(state)
        else -> Transition.Rejected("cancelled order is final")
    }
}
```

## 3. Value class идентификатора { #kotlin-03 }

```kotlin
@JvmInline
value class OrderId private constructor(val value: String) {
    companion object {
        private val pattern = Regex("ORD-[0-9]{8}")
        @JvmStatic
        fun parse(raw: String): Result<OrderId> = runCatching {
            require(pattern.matches(raw)) { "invalid order id: $raw" }
            OrderId(raw)
        }
    }
    override fun toString(): String = value
}
```

## 4. Коллекции без лишних проходов { #kotlin-04 }

```kotlin
data class Event(val customerId: Long, val amount: Long, val accepted: Boolean)
data class Stats(val count: Int, val total: Long)

fun eager(events: List<Event>): Map<Long, Stats> = events
    .filter(Event::accepted)
    .groupBy(Event::customerId)
    .mapValues { (_, values) -> Stats(values.size, values.sumOf(Event::amount)) }

fun singlePass(events: Sequence<Event>): Map<Long, Stats> = buildMap {
    events.filter(Event::accepted).forEach { event ->
        val old = get(event.customerId) ?: Stats(0, 0)
        put(event.customerId, Stats(old.count + 1, old.total + event.amount))
    }
}
```

`singlePass` не создаёт списки после `groupBy`, но сложнее читается. Выбор подтверждается профилированием на реальном объёме.

## 5. Scope functions без вложенности { #kotlin-05 }

```kotlin
data class Config(var host: String = "", var port: Int = 0)

fun createConfig(env: Map<String, String>): Config {
    val host = env["HOST"]?.trim().orEmpty()
    require(host.isNotEmpty()) { "HOST is required" }
    val port = env["PORT"]?.toIntOrNull() ?: 8080
    require(port in 1..65535) { "invalid PORT" }

    return Config().apply {
        this.host = host
        this.port = port
    }
}
```

## 6. Variance { #kotlin-06 }

```kotlin
interface Source<out T> { fun next(): T }
interface Sink<in T> { fun accept(value: T) }

open class Event
class PaymentEvent : Event()

fun pipe(source: Source<PaymentEvent>, sink: Sink<Event>) {
    sink.accept(source.next())
}

class OneSource<T>(private val value: T) : Source<T> {
    override fun next(): T = value
}
class ListSink<T> : Sink<T> {
    val values = mutableListOf<T>()
    override fun accept(value: T) { values += value }
}
```

## 7. Обёртка над Java API { #kotlin-07 }

```kotlin
interface SafeUserDirectory {
    fun name(id: Long): String?
}

class JavaDirectoryAdapter(
    private val delegate: LegacyJavaDirectory
) : SafeUserDirectory {
    override fun name(id: Long): String? = delegate.findName(id)
        ?.trim()
        ?.takeIf(String::isNotEmpty)
}

// Имитация Java API с platform type в реальном проекте.
interface LegacyJavaDirectory { fun findName(id: Long): String? }
```

## 8. Kotlin API для Java-клиента { #kotlin-08 }

```kotlin
class GreetingService @JvmOverloads constructor(
    private val prefix: String = "Hello"
) {
    @JvmOverloads
    fun greet(name: String, punctuation: String = "!"): String =
        "$prefix, $name$punctuation"
}

object Greetings {
    @JvmStatic fun defaultService(): GreetingService = GreetingService()
}
```

```java
GreetingService service = Greetings.defaultService();
String first = service.greet("Ada");
String second = service.greet("Ada", ".");
```

## 9. Checked exception для Java { #kotlin-09 }

```kotlin
import java.io.IOException
import java.nio.file.Files
import java.nio.file.Path

@Throws(IOException::class)
fun readUtf8(path: Path): String = Files.readString(path)
```

```java
try {
    String value = FileApiKt.readUtf8(path);
} catch (IOException ex) {
    // Java compiler требует обработку.
}
```

## 10. Defensive copy { #kotlin-10 }

```kotlin
class Catalog(items: Collection<String>) {
    private val snapshot = items.toList()
    fun items(): List<String> = snapshot
}

fun main() {
    val source = mutableListOf("A")
    val catalog = Catalog(source)
    source += "B"
    check(catalog.items() == listOf("A"))
}
```

## 11. Параллельный агрегатор { #kotlin-11 }

```kotlin
import kotlinx.coroutines.*

data class Dashboard(val profile: String, val orders: List<String>, val recommendations: List<String>)

suspend fun dashboard(id: Long): Dashboard = withTimeout(2_000) {
    coroutineScope {
        val profile = async { loadProfile(id) }
        val orders = async { loadOrders(id) }
        val recommendations = async {
            runCatching { loadRecommendations(id) }
                .getOrElse { error ->
                    if (error is CancellationException) throw error
                    emptyList()
                }
        }
        Dashboard(profile.await(), orders.await(), recommendations.await())
    }
}

suspend fun loadProfile(id: Long) = "profile-$id"
suspend fun loadOrders(id: Long) = listOf("order-$id")
suspend fun loadRecommendations(id: Long) = listOf("item-$id")
```

## 12. Ограничение параллелизма { #kotlin-12 }

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.sync.Semaphore
import kotlinx.coroutines.sync.withPermit

suspend fun <T, R> mapLimited(
    values: List<T>, limit: Int, transform: suspend (T) -> R
): List<R> = coroutineScope {
    require(limit > 0)
    val semaphore = Semaphore(limit)
    values.map { value -> async { semaphore.withPermit { transform(value) } } }.awaitAll()
}
```

## 13. Flow pipeline { #kotlin-13 }

```kotlin
import kotlinx.coroutines.flow.*

data class RawEvent(val id: Long, val payload: String)
data class EnrichedEvent(val id: Long, val payload: String)

fun pipeline(source: Flow<RawEvent>): Flow<List<EnrichedEvent>> = source
    .filter { it.payload.isNotBlank() }
    .map { EnrichedEvent(it.id, it.payload.trim()) }
    .buffer(32)
    .chunked(100)

private fun <T> Flow<T>.chunked(size: Int): Flow<List<T>> = flow {
    val buffer = ArrayList<T>(size)
    collect { value ->
        buffer += value
        if (buffer.size == size) {
            emit(buffer.toList())
            buffer.clear()
        }
    }
    if (buffer.isNotEmpty()) emit(buffer.toList())
}
```

## 14. Retry временных ошибок { #kotlin-14 }

```kotlin
import kotlinx.coroutines.*
import kotlin.math.min
import kotlin.random.Random

class TransientFailure(message: String) : RuntimeException(message)

suspend fun <T> retryTransient(maxAttempts: Int = 4, block: suspend () -> T): T {
    require(maxAttempts > 0)
    var delayMs = 100L
    repeat(maxAttempts - 1) {
        try { return block() }
        catch (e: CancellationException) { throw e }
        catch (e: TransientFailure) {
            delay(delayMs + Random.nextLong(0, delayMs / 2 + 1))
            delayMs = min(delayMs * 2, 2_000)
        }
    }
    return block()
}
```

## 15. Защита shared state { #kotlin-15 }

```kotlin
import kotlinx.coroutines.sync.Mutex
import kotlinx.coroutines.sync.withLock
import java.util.concurrent.atomic.AtomicLong

class MutexCounter {
    private val mutex = Mutex()
    private var value = 0L
    suspend fun increment(): Long = mutex.withLock { ++value }
}

class AtomicCounter {
    private val value = AtomicLong()
    fun increment(): Long = value.incrementAndGet()
}
```

Для нескольких связанных полей нужен `Mutex`; для независимого счётчика достаточно atomic. Actor-вариант оправдан, когда состояние естественно обрабатывается как последовательность команд.

## 16. JDBC-репозиторий { #kotlin-16 }

```kotlin
import java.sql.Connection
import javax.sql.DataSource

data class DbOrder(val id: Long, val status: String)

class JdbcOrderRepository(private val dataSource: DataSource) {
    fun find(id: Long): DbOrder? = dataSource.connection.use { connection ->
        connection.prepareStatement(
            "SELECT id, status FROM orders WHERE id = ?"
        ).use { statement ->
            statement.setLong(1, id)
            statement.executeQuery().use { rows ->
                if (rows.next()) DbOrder(rows.getLong("id"), rows.getString("status")) else null
            }
        }
    }

    fun save(connection: Connection, order: DbOrder) {
        connection.prepareStatement(
            "INSERT INTO orders(id, status) VALUES (?, ?)"
        ).use { statement ->
            statement.setLong(1, order.id)
            statement.setString(2, order.status)
            statement.executeUpdate()
        }
    }
}
```

## 17. Денежный перевод { #kotlin-17 }

```kotlin
import javax.sql.DataSource

class TransferService(private val dataSource: DataSource) {
    fun transfer(key: String, from: Long, to: Long, amount: Long) {
        require(amount > 0)
        dataSource.connection.use { connection ->
            connection.autoCommit = false
            try {
                connection.prepareStatement(
                    "INSERT INTO transfer_keys(id) VALUES (?) ON CONFLICT DO NOTHING"
                ).use { s ->
                    s.setString(1, key)
                    if (s.executeUpdate() == 0) { connection.rollback(); return }
                }
                connection.prepareStatement(
                    "UPDATE accounts SET balance=balance-? WHERE id=? AND balance>=?"
                ).use { s ->
                    s.setLong(1, amount); s.setLong(2, from); s.setLong(3, amount)
                    check(s.executeUpdate() == 1) { "insufficient balance" }
                }
                connection.prepareStatement(
                    "UPDATE accounts SET balance=balance+? WHERE id=?"
                ).use { s ->
                    s.setLong(1, amount); s.setLong(2, to)
                    check(s.executeUpdate() == 1) { "recipient not found" }
                }
                connection.commit()
            } catch (e: Exception) {
                connection.rollback()
                throw e
            } finally {
                connection.autoCommit = true
            }
        }
    }
}
```

## 18. Coroutine boundary для JDBC { #kotlin-18 }

```kotlin
import kotlinx.coroutines.*
import java.util.concurrent.Executors

class SuspendingOrders(
    private val repository: JdbcOrderRepository,
    threads: Int = 8
) : AutoCloseable {
    private val executor = Executors.newFixedThreadPool(threads)
    private val dispatcher = executor.asCoroutineDispatcher()

    suspend fun find(id: Long): DbOrder? = withContext(dispatcher) {
        repository.find(id)
    }

    override fun close() {
        dispatcher.close()
        executor.shutdown()
    }
}
```

Выделенный ограниченный pool не делает JDBC неблокирующим, но изолирует блокирующую работу и задаёт предел одновременных запросов.
