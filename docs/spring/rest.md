## 1. Что такое REST-контроллер в Spring

* Аннотация `@RestController` сочетает в себе `@Controller` и `@ResponseBody`.
* Методы такого контроллера автоматически сериализуют возвращаемые объекты в тело HTTP-ответа (обычно JSON, через Jackson).
* Spring MVC подбирает метод контроллера по HTTP-методу (GET/POST/PUT/DELETE…) и пути (URI), сопоставляя с аннотациями `@RequestMapping`, `@GetMapping` и т. д.

---

## 2. Общее устройство класса

```kotlin
@RestController
@RequestMapping("/hello")
class HelloWorldController {
    private val logger = LoggerFactory.getLogger(HelloWorldController::class.java)
    …
}
```

1. `@RequestMapping("/hello")` на уровне класса — префикс пути ко всем методам: все эндпоинты живут под `/hello`.
2. `logger` (SLF4J) — стандартный способ писать логи: уровни `info`, `debug`, `error` и т. д.

---

## 3. DTO для обмена данными

```kotlin
import ru.itpark.lessons.restdemo.controller.dto.HelloDto
```

Обычно это Kotlin data-class:

```kotlin
data class HelloDto(
  val message: String,
  val status: Int
)
```

* Jackson автоматически конвертирует JSON ↔ `HelloDto`.
* Поля DTO называют «property» — они попадают в тело ответа или читаются из тела запроса.

---

## 4. GET `/hello/mir`

```kotlin
@GetMapping("/mir")
fun sayHello(): HelloDto =
    HelloDto(message = "Misha", status = 200)
```

* **HTTP-метод:** GET.
* **Путь:** `/hello/mir`.
* **Возвращаемый тип:** `HelloDto` → сериализуется в JSON:

  ```json
  {
    "message": "Misha",
    "status": 200
  }
  ```
* **Статус ответа:** по умолчанию 200 OK.

---

## 5. POST `/hello`

```kotlin
@PostMapping
fun createHello(@RequestBody dto: HelloDto): String? {
    logger.info("Saying hello for $dto")
    return dto?.message
}
```

1. **`@PostMapping`** без пути → обрабатывает POST на `/hello`.
2. **`@RequestBody`** заставляет Spring взять JSON-тело запроса, распарсить его в `HelloDto`.
3. **Логирование:** `logger.info("Saying hello for $dto")`.
4. **Возврат:** просто `String` (сообщение). Клиент получит текстовый ответ, а не JSON-объект.

---

## 6. PUT `/hello/update`

```kotlin
@PutMapping("/update")
fun updateHello(@RequestBody dto: HelloDto) {
    logger.info("Updating hello for $dto")
}
```

* **HTTP-метод:** PUT.
* **Путь:** `/hello/update`.
* Принимать JSON через `@RequestBody`, но метод возвращает `Unit` → пустое тело, статус 200 OK.
* Подходит для «обновления» ресурса.

---

## 7. GET `/hello/{userId}/user`

```kotlin
@GetMapping("/{userId}/user")
fun isExistsUserById(@PathVariable(name = "userId") id: Int): Boolean {
    if (id in (0..10)) return true
    return false
}
```

1. **`@PathVariable("userId")`** — извлекает сегмент пути в переменную `id`.
2. **Контроль диапазона:** возвращает `true`, если `id` от 0 до 10.
3. **Возвращаемый тип:** `Boolean` → JSON-логика: `true` или `false`.

---

## 8. GET `/hello/user?name=…&age=…`

```kotlin
@GetMapping("/user")
fun findByParams(
    @RequestParam(value = "name", required = false) name: String?,
    @RequestParam(value = "age", required = false) age: Int?,
): String = "Searching for $name and $age"
```

* **`@RequestParam`** — извлечение query-параметров: `?name=Ivan&age=30`.
* **`required = false`** — делает параметр необязательным, в случае отсутствия — `null`.
* Возвращает простую строку, например:

  ```
  Searching for Ivan and 30
  ```

---

## 9. POST `/hello/file` — загрузка файла

```kotlin
@PostMapping("/file")
fun acceptFile(@RequestParam("file") file: MultipartFile): ByteArray {
    return file.bytes
}
```

1. **`MultipartFile`** — Spring MVC умеет разбирать `multipart/form-data`.
2. **`file.bytes`** — байтовое содержимое загруженного файла.
3. **Возвращаемый тип:** `ByteArray` → клиенту приходит «голый» массив байт, Content-Type будет `application/octet-stream`.
4. **Чтобы это работало**, в приложении должен быть `MultipartResolver` (в Spring Boot включён по умолчанию).

---

## 10. Как Spring обрабатывает запрос и ответ

1. **Поиск подходящего метода** — по HTTP-методу и совпадению URI-шаблона.
2. **Разбор входных данных**:

    * `@RequestBody` → через `HttpMessageConverter` (Jackson JSON).
    * `@RequestParam` → из query или form-data.
    * `@PathVariable` → из пути.
3. **Выполнение метода**.
4. **Сериализация результата**:

    * Объект → JSON (или строка → текст).
    * HTTP-статус по умолчанию 200, можно менять через `ResponseEntity`.

---

## 11. Дополнительные возможности и «под капотом»

* **Validation** — добавить `@Valid` перед `@RequestBody dto: HelloDto` и в DTO поставить `@NotBlank`, `@Size` и т. д. Ошибки превращаются в 400 Bad Request.
* **ResponseEntity** — оборачивайте результат, если нужно возвращать кастомный статус, заголовки:

  ```kotlin
  @PostMapping
  fun createHello(@RequestBody dto: HelloDto): ResponseEntity<String> {
    return ResponseEntity.status(HttpStatus.CREATED).body(dto.message)
  }
  ```
* **ExceptionHandler** — `@ControllerAdvice` можно ловить и трансформировать исключения в понятные HTTP-ошибки.
* **Swagger/OpenAPI** — аннотации `@Operation`, `@Parameter` помогут автогенерировать документацию.
