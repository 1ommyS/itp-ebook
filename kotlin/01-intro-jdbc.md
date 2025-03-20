
---

## 1. Null Safety (Безопасность от null)
**Файл:** `NullSafety.kt`

Kotlin по умолчанию не допускает значения `null` для переменных, что значительно снижает вероятность возникновения `NullPointerException`.

```kotlin
fun main() {
    // Невозможно присвоить null переменной типа String (не допускается)
    val nonNullable: String = "Привет, Kotlin"
    // nonNullable = null // Ошибка компиляции

    // Объявляем переменную, допускающую null
    var nullable: String? = "Привет, Kotlin!"
    nullable = null

    // Безопасный вызов: если nullable равен null, возвращается null вместо NPE
    println("Длина строки: ${nullable?.length}")

    // Оператор Элвиса (Elvis): если значение null, то используется значение по умолчанию
    val length = nullable?.length ?: 0
    println("Длина строки (через Elvis): $length")
}
```

---

## 2. Data Classes (Классы данных)
**Файл:** `DataClassExample.kt`

Data-классы автоматически генерируют такие методы, как `equals()`, `hashCode()`, `copy()` и `toString()`, что существенно сокращает шаблонный код при работе с моделями данных.

```kotlin
data class User(val name: String, val age: Int)

fun main() {
    val user1 = User("Alice", 30)
    val user2 = User("Alice", 30)

    // Автоматически сгенерированные методы equals(), hashCode() и toString()
    println("Пользователи равны? ${user1 == user2}")

    // Метод copy() для создания нового объекта с изменёнными полями
    val user3 = user1.copy(age = 31)
    println("Пользователь 3: $user3")
}
```

---

## 3. Extension Functions (Расширяющие функции)
**Файл:** `ExtensionFunctions.kt`

Расширяющие функции позволяют добавлять новые методы в существующие классы без наследования или изменения их исходного кода.

```kotlin
// Определяем функцию-расширение для класса String
fun String.lastChar(): Char = this.get(this.length - 1)

fun main() {
    val message = "Kotlin"
    println("Последний символ строки '$message': ${message.lastChar()}")
}
```

---

## 4. Higher-Order Functions и Lambdas (Функции высшего порядка и лямбда-выражения)
**Файл:** `HigherOrderFunctions.kt`

Kotlin позволяет передавать функции как параметры и возвращать их из других функций. Лямбда-выражения делают код более компактным и функциональным.

```kotlin
// Функция, принимающая другую функцию как аргумент
fun operateOnNumbers(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

fun main() {
    val sum = operateOnNumbers(3, 4) { x, y -> x + y }
    println("Сумма: $sum")

    val product = operateOnNumbers(3, 4) { x, y -> x * y }
    println("Произведение: $product")
}
```

---

## 5. Свойства вместо Геттеров/Сеттеров
**Файл:** `Properties.kt`

В Kotlin можно объявлять свойства прямо внутри класса, что упрощает доступ к данным и позволяет автоматически генерировать геттеры и сеттеры.

```kotlin
class Person(private var _age: Int) {
    // Свойство age с кастомными геттером и сеттером
    var age: Int
        get() = _age
        set(value) {
            if (value >= 0) _age = value
        }
}

fun main() {
    val person = Person(25)
    println("Возраст: ${person.age}")
    person.age = 30
    println("Обновлённый возраст: ${person.age}")
}
```

---

## 6. Smart Casts (Умное приведение типов)
**Файл:** `SmartCasts.kt`

После проверки типа компилятор автоматически приводит объект к нужному типу, избавляя от явного приведения.

```kotlin
open class Animal(val name: String)

class Dog(name: String) : Animal(name) {
    fun bark() {
        println("$name: Гав!")
    }
}

fun main() {
    val animal: Animal = Dog("Бадди")
    // После проверки типа, переменная автоматически приводится к типу Dog
    if (animal is Dog) {
        animal.bark()
    }
}
```

---

## 7. Параметры по умолчанию и Именованные аргументы
**Файл:** `DefaultNamedParameters.kt`

Значения по умолчанию для параметров и именованные аргументы делают вызов функций более понятным и уменьшают вероятность ошибок.

```kotlin
fun greet(name: String = "Гость", greeting: String = "Привет") {
    println("$greeting, $name!")
}

fun main() {
    greet()
    greet(name = "Алиса")
    greet(greeting = "Здравствуйте", name = "Боб")
}
```

---

## 8. Companion Objects (Компаньон-объекты)
**Файл:** `CompanionObjects.kt`

Компаньон-объекты позволяют объявлять статические методы и поля внутри класса, что соответствует понятию `static` в Java, но с более гибкой реализацией.

```kotlin
class MathUtil {
    companion object {
        fun square(x: Int) = x * x
    }
}

fun main() {
    println("Квадрат числа 5: ${MathUtil.square(5)}")
}
```

---

## 9. Sealed Classes (Запечатлённые классы)
**Файл:** `SealedClasses.kt`

Запечатлённые классы ограничивают иерархию наследования, позволяя заранее определить набор допустимых подтипов. Это упрощает обработку состояний с помощью конструкции `when`.

```kotlin
sealed class OperationResult {
    data class Success(val data: String): OperationResult()
    data class Error(val error: String): OperationResult()
}

fun handleResult(result: OperationResult) {
    when(result) {
        is OperationResult.Success -> println("Успех: ${result.data}")
        is OperationResult.Error -> println("Ошибка: ${result.error}")
    }
}

fun main() {
    val success = OperationResult.Success("Данные загружены")
    val error = OperationResult.Error("Не удалось загрузить данные")

    handleResult(success)
    handleResult(error)
}
```

---

## 10. Coroutines (Корутины)
**Файл:** `CoroutinesExample.kt`

Корутины позволяют писать асинхронный и неблокирующий код, упрощая работу с многопоточностью.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch {
        delay(1000L)
        println("Мир!")
    }
    println("Привет")
    // Короутина задерживает выполнение на 1 секунду, после чего выводится "Мир!"
}
```

---

## 11. Type Inference и Operator Overloading (Вывод типов и перегрузка операторов)
**Файл:** `TypeInferenceAndOperators.kt`

Благодаря выводу типов, не требуется явно указывать типы переменных, а перегрузка операторов делает работу с пользовательскими классами интуитивной.

```kotlin
// Перегрузка оператора сложения для класса Vector
data class Vector(val x: Int, val y: Int) {
    operator fun plus(other: Vector) = Vector(x + other.x, y + other.y)
}

fun main() {
    val v1 = Vector(1, 2)
    val v2 = Vector(3, 4)
    val v3 = v1 + v2  // Использование перегруженного оператора
    println("v3: $v3")

    // Вывод типа определяется автоматически
    val v = 10
    println("Значение v: $v")
}
```

---

## 12. When Expression и String Templates (Выражение when и строковые шаблоны)
**Файл:** `WhenAndStringTemplates.kt`

Конструкция `when` более мощная, чем классический `switch`, а строковые шаблоны позволяют удобно вставлять переменные в строки.

```kotlin
fun describeNumber(x: Int): String {
    return when (x) {
        0 -> "Ноль"
        in 1..10 -> "Маленькое число"
        else -> "Большое число"
    }
}

fun main() {
    val number = 7
    println("Число $number: ${describeNumber(number)}")
}
```

---

## 13. Оператор Элвиса (Elvis Operator)
**Файл:** `ElvisOperator.kt`

Оператор Элвиса (`?:`) позволяет задать значение по умолчанию, если выражение возвращает `null`.

```kotlin
fun getLength(str: String?): Int {
    // Если str не null, вернёт его длину, иначе – -1 (значение по умолчанию)
    return str?.length ?: -1
}

fun main() {
    val nonNullStr: String? = "Kotlin"
    val nullStr: String? = null

    println("Длина строки '$nonNullStr': ${getLength(nonNullStr)}")
    println("Длина строки null: ${getLength(nullStr)}")
}
```

---

## 14. Отсутствие ключевого слова return (Expression Body Functions)
**Файл:** `ExpressionBody.kt`

Функции с телом-выражением позволяют не использовать ключевое слово `return`, делая код более лаконичным.

```kotlin
// Функция max определена с использованием выражения тела и не содержит ключевого слова return
fun max(a: Int, b: Int) = if (a > b) a else b

// Функция greet, возвращающая строку без явного return
fun greet(name: String) = "Привет, $name!"

fun main() {
    println("Max(10, 5) = ${max(10, 5)}")
    println(greet("Мир"))
}
```

---

## 15. Kotlin Collections API (Альтернатива Java Stream API)
**Файл:** `KotlinStreamAPI.kt`

Стандартная библиотека Kotlin предоставляет мощный набор функций для работы с коллекциями, что позволяет выполнять цепочки вызовов (фильтрация, маппинг, сортировка) в одном потоке.

```kotlin
fun main() {
    // Создаём список чисел
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    
    // Фильтрация, преобразование и сортировка в одном потоке вызовов
    val evenSquares = numbers
        .filter { it % 2 == 0 }    // Оставляем только чётные числа
        .map { it * it }           // Возводим каждое число в квадрат
        .sortedDescending()        // Сортируем в обратном порядке

    println("Квадраты чётных чисел в обратном порядке: $evenSquares")
}
```

---

## 16. Особые лямбды и лямбды с ресивером
**Файл:** `SpecialLambdas.kt`

Kotlin поддерживает различные виды лямбд, включая лямбды с неявным параметром `it`, лямбды с ресивером (для создания DSL) и trailing lambda.

```kotlin
data class Person(var name: String, var age: Int)

fun main() {
    // Пример 1: Лямбда с неявным параметром it
    val numbers = listOf(1, 2, 3, 4, 5)
    numbers.forEach {
        println("Число: $it")
    }

    // Пример 2: Использование let для безопасной работы с null
    val name: String? = "Кирилл"
    name?.let {
        println("Привет, $it!")
    }

    // Пример 3: Лямбда с ресивером для создания DSL-подобного кода
    fun buildMessage(builder: StringBuilder.() -> Unit): String {
        val sb = StringBuilder()
        sb.builder()  // Вызов лямбды, где StringBuilder выступает как ресивер
        return sb.toString()
    }

    val message = buildMessage {
        append("Здравствуйте, ")
        append("мир!")
    }
    println("Сообщение: $message")

    // Пример 4: Trailing lambda при использовании функции map
    val doubled = numbers.map { it * 2 }
    println("Удвоенные числа: $doubled")
}
```

---

## 17. Inline, crossinline и noinline
**Файл:** `InlineCrossinlineNoinlineExamples.kt`

Kotlin позволяет оптимизировать вызовы функций с помощью ключевого слова `inline`. Дополнительные модификаторы:
- **non-local return:** позволяет выйти из внешней функции прямо из лямбды.
- **crossinline:** запрещает non-local return, если лямбда передаётся в другой контекст (например, в Runnable).
- **noinline:** отключает встраивание для конкретной лямбды.

```kotlin
fun main() {
    println("=== Пример non-local return в inline функции ===")
    nonLocalReturnExample() // После non-local return дальнейший код не выполняется

    println("\n=== Пример crossinline, запрещающего non-local return ===")
    crossinlineExample()

    println("\n=== Пример использования noinline параметра ===")
    noinlineExample()
}

/**
 * Функция демонстрирует возможность non-local return.
 * Если лямбда внутри inlineNonLocal вызывает return,
 * выполнение немедленно завершается в вызывающей функции.
 */
fun nonLocalReturnExample() {
    println("Начало nonLocalReturnExample")
    inlineNonLocal {
        println("Внутри inlineNonLocal: выполняется return")
        return  // non-local return – завершает выполнение nonLocalReturnExample!
    }
    // Эта строка не будет выполнена
    println("Конец nonLocalReturnExample")
}

/**
 * Inline-функция, позволяющая использовать non-local return.
 */
inline fun inlineNonLocal(block: () -> Unit) {
    println("До вызова блока в inlineNonLocal")
    block()
    println("После вызова блока в inlineNonLocal")
}

/**
 * Функция демонстрирует использование crossinline.
 * Лямбда, переданная в inlineCrossinline, не может выполнять non-local return.
 */
fun crossinlineExample() {
    println("Начало crossinlineExample")
    inlineCrossinline {
        println("Выполнение crossinline лямбды")
        // Попытка выполнить non-local return здесь вызовет ошибку компиляции:
        // return // Ошибка: non-local return из crossinline лямбды запрещён.
    }
    println("Конец crossinlineExample")
}

/**
 * Inline-функция с параметром, помеченным как crossinline.
 */
inline fun inlineCrossinline(crossinline block: () -> Unit) {
    println("До выполнения crossinline блока")
    val runnable = Runnable { block() }
    runnable.run()
    println("После выполнения crossinline блока")
}

/**
 * Функция демонстрирует использование noinline-параметра.
 */
fun noinlineExample() {
    println("Начало noinlineExample")
    inlineWithNoinline(
        inlineBlock = { println("Выполнение inline лямбды") },
        noinline noinlineBlock = { println("Выполнение noinline лямбды") }
    )
    println("Конец noinlineExample")
}

/**
 * Inline-функция с двумя лямбда-параметрами:
 * - inlineBlock будет встроена,
 * - noinlineBlock не будет встроена.
 */
inline fun inlineWithNoinline(
    inlineBlock: () -> Unit,
    noinline noinlineBlock: () -> Unit
) {
    println("Перед вызовом inlineBlock")
    inlineBlock()
    println("Перед вызовом noinlineBlock")
    noinlineBlock()
    println("После вызова noinlineBlock")
}
```

---

## 18. Работа с JDBC на Kotlin

### 18.1. Подключение к базе данных и управление соединением
**Файл:** `JDBCConnectionExample.kt`

```kotlin
import java.sql.Connection
import java.sql.DriverManager
import java.sql.SQLException

fun main() {
    val url = "jdbc:mysql://localhost:3306/mydatabase?useSSL=false&serverTimezone=UTC"
    val user = "dbuser"
    val password = "dbpassword"

    try {
        DriverManager.getConnection(url, user, password).use { connection ->
            println("Соединение установлено успешно!")
            // Здесь можно выполнять операции с базой данных
        }
    } catch (e: SQLException) {
        println("Ошибка при подключении к базе данных: ${e.message}")
        e.printStackTrace()
    }
}
```

### 18.2. Выполнение SQL-запросов с использованием PreparedStatement
**Файл:** `JDBCPreparedStatementExample.kt`

```kotlin
import java.sql.DriverManager
import java.sql.SQLException

fun main() {
    val url = "jdbc:mysql://localhost:3306/mydatabase?useSSL=false&serverTimezone=UTC"
    val user = "dbuser"
    val password = "dbpassword"
    val query = "SELECT id, name, email FROM users WHERE id = ?"

    try {
        DriverManager.getConnection(url, user, password).use { connection ->
            connection.prepareStatement(query).use { pstmt ->
                pstmt.setInt(1, 1) // ищем пользователя с id = 1
                pstmt.executeQuery().use { rs ->
                    while (rs.next()) {
                        val id = rs.getInt("id")
                        val name = rs.getString("name")
                        val email = rs.getString("email")
                        println("User ID: $id, Name: $name, Email: $email")
                    }
                }
            }
        }
    } catch (e: SQLException) {
        println("Ошибка выполнения запроса: ${e.message}")
        e.printStackTrace()
    }
}
```

### 18.3. Управление транзакциями (commit и rollback)
**Файл:** `JDBCTransactionExample.kt`

```kotlin
import java.sql.DriverManager
import java.sql.SQLException

fun main() {
    val url = "jdbc:mysql://localhost:3306/mydatabase?useSSL=false&serverTimezone=UTC"
    val user = "dbuser"
    val password = "dbpassword"

    try {
        DriverManager.getConnection(url, user, password).use { connection ->
            connection.autoCommit = false

            try {
                val insertUserSQL = "INSERT INTO users (name, email) VALUES (?, ?)"
                val updateAccountSQL = "UPDATE accounts SET balance = balance - ? WHERE user_id = ?"

                connection.prepareStatement(insertUserSQL).use { insertStmt ->
                    insertStmt.setString(1, "Иван Иванов")
                    insertStmt.setString(2, "ivan@example.com")
                    insertStmt.executeUpdate()
                }

                connection.prepareStatement(updateAccountSQL).use { updateStmt ->
                    updateStmt.setDouble(1, 100.0)
                    updateStmt.setInt(2, 1)
                    updateStmt.executeUpdate()
                }

                connection.commit()
                println("Транзакция успешно завершена.")
            } catch (e: SQLException) {
                connection.rollback()
                println("Ошибка выполнения транзакции. Откат выполнен: ${e.message}")
                e.printStackTrace()
            } finally {
                connection.autoCommit = true
            }
        }
    } catch (e: SQLException) {
        println("Ошибка подключения: ${e.message}")
        e.printStackTrace()
    }
}
```

## 19. Gradle KTS зависимости для проекта
**Файл:** `build.gradle.kts`

```kotlin
plugins {
    // Плагин для компиляции Kotlin под JVM
    kotlin("jvm") version "1.8.10" // укажите актуальную версию Kotlin
    // Плагин для создания исполняемого приложения
    application
}

repositories {
    mavenCentral()
}

dependencies {
    // Стандартная библиотека Kotlin
    implementation(kotlin("stdlib"))
    
    // JDBC-драйвер для Postgres
    implementation("org.postgresql:postgresql:42.7.5")


    // Тестовые зависимости
    testImplementation("org.jetbrains.kotlin:kotlin-test")
    testImplementation("org.junit.jupiter:junit-jupiter:5.9.2")
}

application {
    // Укажите главный класс приложения
    mainClass.set("com.example.MainKt")
}

tasks.test {
    useJUnitPlatform()
}
```

---

# Заключение

Как видно из приведённых примеров, Kotlin обладает множеством преимуществ:

- **Лаконичность и читаемость:** Меньше шаблонного кода, компактный синтаксис и мощные возможности (data-классы, extension functions, строковые шаблоны).
- **Безопасность:** Встроенная поддержка null safety и умное приведение типов уменьшают вероятность ошибок.
- **Функциональные возможности:** Функции высшего порядка, лямбды, inline-функции и DSL-подход упрощают разработку.
- **Удобство работы с асинхронностью:** Корутины позволяют писать неблокирующий код проще, чем с использованием потоков.
- **Интеграция с Java:** Возможность использовать существующие библиотеки и технологии, такие как JDBC, при этом сохраняя все преимущества современного языка.

Эта коллекция файлов и подробных объяснений поможет вам глубже понять, почему Kotlin становится выбором многих разработчиков для создания чистого, безопасного и выразительного кода.

```