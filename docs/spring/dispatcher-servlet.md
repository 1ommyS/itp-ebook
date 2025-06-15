
## 1. Front-Controller: роль DispatcherServlet

В основе Spring MVC лежит шаблон **Front Controller**. Вся входящая HTTP-активность направляется в единый точка — `DispatcherServlet`:

```
client → DispatcherServlet → (HandlerMapping → Controller → ViewResolver → View) → response
```

* **DispatcherServlet** — это `javax.servlet.http.HttpServlet`, который регистрируется в контейнере (Tomcat и т. д.) и прослушивает определённый URL-паттерн (обычно `"/"`).
* Он **централизованно** обрабатывает все запросы вашего веб-приложения.

---

## 2. Регистрация DispatcherServlet

### 2.1. Через `web.xml`

```xml
<servlet>
  <servlet-name>dispatcher</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/dispatcher-servlet.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>dispatcher</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

* **`servlet-name`** влияет на имя Spring-контекста: по умолчанию он ищет `classpath:/dispatcher-servlet.xml` или `dispatcher-servlet.properties`.

### 2.2. Без `web.xml` (Java-конфигурация)

```java
public class WebAppInitializer implements WebApplicationInitializer {
  @Override
  public void onStartup(ServletContext sc) {
    AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
    ctx.register(WebConfig.class);
    ServletRegistration.Dynamic dispatcher = sc.addServlet("dispatcher", new DispatcherServlet(ctx));
    dispatcher.setLoadOnStartup(1);
    dispatcher.addMapping("/");
  }
}
```

### 2.3. Spring Boot

При старте Boot автоматически регистрирует `DispatcherServlet` на `"/"` и создаёт `ApplicationContext`, если в classpath есть Spring MVC.

---

## 3. Инициализация и жизненный цикл

1. **Инициализация** (`init()`):

    * Создаёт (или получает) `WebApplicationContext`.
    * Подхватывает все бины: `HandlerMapping`, `HandlerAdapter`, `ViewResolver`, `HandlerExceptionResolver`, `MultipartResolver`, `LocaleResolver`, `ThemeResolver` и т. д.
2. **Destruction** (`destroy()`): чистит ресурсы при остановке.

---

## 4. Пайплайн обработки запроса

Главный метод в `DispatcherServlet` — `doDispatch(HttpServletRequest, HttpServletResponse)`. Упрощённо:

```text
doDispatch(req, resp):
  1. Запросить Locale и Theme (через LocaleResolver/ThemeResolver).
  2. Найти подходящий HandlerExecutionChain через все HandlerMapping’и.
  3. Преобразовать multipart (если есть) через MultipartResolver.
  4. Применить перед-обработчики (preHandle) всех зарегистрированных HandlerInterceptor’ов.
  5. Вызвать Controller через соответствующий HandlerAdapter.
     → получить ModelAndView
  6. Применить postHandle у Interceptor’ов.
  7. Разрешить View через ViewResolver → получить View. - ЕСЛИ SPRING MVC
  8. Отрендерить View: передать в него Model, Request, Response.  - ЕСЛИ SPRING MVC
  9. Применить afterCompletion у Interceptor’ов.
 10. Обработать исключения через HandlerExceptionResolver.
```

### 4.1. HandlerMapping

Spring ищет **какой контроллер** (или другой обработчик) должен отвечать на URI и HTTP-метод:

* Стандартные реализации:

    * `RequestMappingHandlerMapping` — сканирует `@RequestMapping`, `@GetMapping` и т. д.
    * `BeanNameUrlHandlerMapping`, `SimpleUrlHandlerMapping` — по строчным паттернам.
* Первый, кто найдёт совпадение, возвращает `HandlerExecutionChain` (контроллер + список Interceptor’ов).

### 4.2. HandlerAdapter

Независимо от типа контроллера (`@Controller`, `HttpRequestHandler`, `WebFlux`), `DispatcherServlet` вызывает **HandlerAdapter**, умеющий его «подключить»:

* `RequestMappingHandlerAdapter` — для аннотированных контроллеров, преобразует аргументы метода (через аргументы резолверы) и обрабатывает возвращаемые значения (`@ResponseBody`, `ModelAndView`).
* `SimpleControllerHandlerAdapter`, `HttpRequestHandlerAdapter` и т. д.

### 4.3. HandlerInterceptor

Позволяют перехватывать и дополнять обработку:

```java
public class LoggingInterceptor implements HandlerInterceptor {
  @Override
  public boolean preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler) { … }
  @Override
  public void postHandle(HttpServletRequest req, HttpServletResponse resp,
                         Object handler, ModelAndView mav) { … }
  @Override
  public void afterCompletion(HttpServletRequest req, HttpServletResponse resp,
                              Object handler, Exception ex) { … }
}
```

Регистрируются в конфигурации MVC:

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new LoggingInterceptor())
            .addPathPatterns("/api/**");
}
```


## 5. Обработка ошибок

* Если в `doDispatch` бросается исключение, Spring пробует каждое `HandlerExceptionResolver`:

    * `ExceptionHandlerExceptionResolver` — для `@ExceptionHandler` в контроллерах.
    * `ResponseStatusExceptionResolver` — для `@ResponseStatus`.
    * `DefaultHandlerExceptionResolver` — обрабатывает стандартные Servlet-исключения.
* Через `@ControllerAdvice` можно централизованно ловить ошибки и превращать их в `ResponseEntity`.

---

## 6. Загрузка файлов (multipart)

* При наличии `MultipartResolver` (в Spring Boot — `StandardServletMultipartResolver`) `DispatcherServlet` оборачивает запрос в `MultipartHttpServletRequest`, и вы можете в контроллерах получать `MultipartFile`.

---

## 7. Настройка и расширение

1. **Переопределить** бины `HandlerMapping` или `HandlerAdapter`, если нужна своя логика маршрутизации или вызова.
2. **Унаследовать** `DispatcherServlet`, переопределить `doDispatch` или `doService` для особых кейсов.
3. **Добавить** собственные `ViewResolver`-ы, `HandlerInterceptor`-ы, `HandlerExceptionResolver`.
4. **Параметры**:

    * `throwExceptionIfNoHandlerFound=true` позволит выбрасывать исключение, если нет подходящего контроллера (иначе отдаётся 404 встроенный).
    * `cleanupAfterInclude` для управления очисткой атрибутов при `RequestDispatcher.include`.

---
