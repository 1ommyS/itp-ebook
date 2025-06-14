
## 1. Feature-Sliced Design (FSD)

> **Идея:** группировать весь код (UI, сервисы, репозитории) не по слоям (controller/service/repo), а по **фичам** – автономным функциональным блокам.

### 1.1. Структура пакетов

```
src/main/java/com/example/app
├── features
│   ├── user
│   │   ├── api
│   │   │   ├── UserController.java
│   │   │   └── dto/UserDto.java
│   │   ├── service
│   │   │   └── UserService.java
│   │   └── repository
│   │       └── UserRepository.java
│   └── order
│       ├── api
│       │   └── OrderController.java
│       ├── service
│       │   └── OrderService.java
│       └── repository
│           └── OrderRepository.java
└── shared      // общие утилиты, константы, конвертеры
    ├── dto
    ├── config
    └── util
```

### 1.2. Пример кода

```java
// features/user/api/UserController.java
@RestController
@RequestMapping("/api/users")
public class UserController {
  private final UserService service;
  public UserController(UserService service) { this.service = service; }

  @GetMapping("/{id}")
  public ResponseEntity<UserDto> get(@PathVariable Long id) {
    return service.findById(id)
      .map(ResponseEntity::ok)
      .orElse(ResponseEntity.notFound().build());
  }
}
```

```java
// features/user/service/UserService.java
@Service
public class UserService {
  private final UserRepository repo;
  public UserService(UserRepository repo) { this.repo = repo; }

  public Optional<UserDto> findById(Long id) {
    return repo.findById(id)
      .map(u -> new UserDto(u.getId(), u.getName(), u.getEmail()));
  }
}
```

```java
// features/user/repository/UserRepository.java
public interface UserRepository extends JpaRepository<User, Long> { }
```

### 1.3. Плюсы и минусы FSD

* **Плюсы:** фичи автономны, легко выделять в модули, понятнее локализация изменений.
* **Минусы:** дублирование DTO/моделей в разных фичах, сложнее переиспользовать общее.

---

## 2. Domain-Driven Design (DDD)

> **Идея:** фокус на **домене** (бизнес-модели). Чёткое разделение на слои: **Domain**, **Application**, **Infrastructure**, **Interface**.

### 2.1. Структура пакетов

```
src/main/java/com/example/app
├── domain
│   ├── model       // сущности, VO, репозитории (интерфейсы)
│   │   ├── User.java
│   │   └── UserRepository.java
│   ├── service     // доменные сервисы
│   │   └── UserValidationService.java
│   └── exception
├── application
│   ├── dto         // входные/выходные DTO
│   ├── service     // use-case сервисы
│   │   └── UserRegistrationUseCase.java
│   └── port
│       ├── in      // входные порты (интерфейсы)
│       │   └── RegisterUserCommand.java
│       └── out     // выходные порты (интерфейсы репозиториев)
│           └── SaveUserPort.java
├── infrastructure
│   └── persistence
│       └── JpaUserRepository.java   // implements SaveUserPort & UserRepository
└── interface
    └── rest
        └── UserController.java
```

### 2.2. Пример кода

```java
// domain/model/User.java
@Entity
public class User {
  @Id @GeneratedValue private Long id;
  @Column(nullable=false) private String email;
  // геттеры/сеттеры, бизнес-методы внутри модели
}

// domain/model/UserRepository.java
public interface UserRepository {
  Optional<User> findByEmail(String email);
}

// application/port/out/SaveUserPort.java
public interface SaveUserPort {
  User save(User user);
}

// application/service/UserRegistrationUseCase.java
@Service
public class UserRegistrationUseCase {
  private final SaveUserPort savePort;
  private final UserRepository userRepo;
  public UserRegistrationUseCase(SaveUserPort savePort, UserRepository userRepo) {
    this.savePort = savePort; this.userRepo = userRepo;
  }
  public UserDto register(RegisterUserCommand cmd) {
    if (userRepo.findByEmail(cmd.getEmail()).isPresent()) {
      throw new BusinessException("Email уже занят");
    }
    User u = new User(cmd.getEmail(), cmd.getPassword());
    User saved = savePort.save(u);
    return new UserDto(saved.getId(), saved.getEmail());
  }
}

// infrastructure/persistence/JpaUserRepository.java
@Repository
public class JpaUserRepository implements UserRepository, SaveUserPort {
  private final SpringUserCrud repo; // extends JpaRepository<User, Long>
  public JpaUserRepository(SpringUserCrud repo) { this.repo = repo; }
  public Optional<User> findByEmail(String email) { return repo.findByEmail(email); }
  public User save(User u) { return repo.save(u); }
}
```

### 2.3. Плюсы и минусы DDD

* **Плюсы:** ясная модель, изоляция домена от технических деталей, лёгкое тестирование домена.
* **Минусы:** большая «весомость» шаблона, в простых проектах может быть избыточно.

---

## 3. Onion Architecture

> **Идея:** слоёвка в форме луковицы: ядро (domain) — внутренний слой, внешние слои зависят только от внутренних.

### 3.1. Слои и зависимости

```
[ UI / External ]
     ↑
[ Infrastructure ]  ← сюда входят адаптеры БД, веб, email
     ↑
[ Application ]     ← здесь бизнес-логика, сервисы операций
     ↑
[ Domain ]          ← ядро: сущности, интерфейсы репозиториев
```

Стрелки обозначают направление **зависимости** (внешние слои знают о внутренних, но не наоборот).

### 3.2. Пример аннотаций и пакетов

* **domain/** — чистые POJO, интерфейсы
* **application/** — `@Service`-классы, используют domain
* **infrastructure/** — `@Repository`, `@Controller`, используют application и domain
* **config/** — `@Configuration`, настраивает сканирование

```java
@Configuration
@ComponentScan({"com.example.app.infrastructure", "com.example.app.application"})
public class AppConfig { }
```

---

## 4. Hexagonal Architecture (Ports & Adapters)

> **Идея:** изолировать домен с помощью «портов» (интерфейсов) и «адаптеров» (конкретных реализаций). Каждый внешний интерфейс (REST, БД, UI) — отдельный адаптер.

### 4.1. Структура пакетов

```
src/main/java/com/example/app
├── domain
│   ├── model
│   └── port
│       ├── in      // входные порты (UseCase интерфейсы)
│       └── out     // выходные порты (репозитории, шлюзы)
├── adapter
│   ├── inbound
│   │   └── rest    // контроллеры, веб-адаптер
│   └── outbound
│       ├── persistence  // JPA, jdbc
│       └── messaging    // Kafka, Rabbit
└── config
    └── SpringConfig.java
```

### 4.2. Пример порта и адаптера

```java
// domain/port/in/UserQueryService.java
public interface UserQueryService {
  Optional<UserDto> getById(Long id);
}

// adapter/inbound/rest/UserController.java
@RestController
@AllArgsConstructor
public class UserController {
  private final UserQueryService query;
  @GetMapping("/users/{id}")
  public UserDto get(@PathVariable Long id) {
    return query.getById(id).orElseThrow(NotFound::new);
  }
}
```

```java
// adapter/outbound/persistence/UserJpaAdapter.java
@Repository
public class UserJpaAdapter implements UserQueryService, UserSavePort {
  private final UserSpringRepository repo;
  
  @Override
  public Optional<UserDto> getById(Long id) {
    return repo.findById(id).map(u -> new UserDto(u.getId(),u.getEmail()));
  }

  @Override
  public User save(User u) {
    return repo.save(u);
  }
}
```

---

## 5. Сравнение подходов

|                           | FSD                         | DDD                   | Onion                    | Hexagonal               |
| ------------------------- | --------------------------- | --------------------- | ------------------------ | ----------------------- |
| **Фокус**                 | фичи (функциональные блоки) | доменная модель       | слои по зависимости      | порты/адаптеры          |
| **Структура**             | по фичам                    | по слоям домена/прил. | луковица (центр→внешние) | ядро + inbound/outbound |
| **Гибкость тестирования** | средняя                     | высокая               | высокая                  | высокая                 |
| **Уровень формализма**    | лёгкий                      | высокий               | средний                  | средний                 |

---

## 6. Итоговые рекомендации

1. **Маленькие проекты** (CRUD-сервисы): достаточно FSD + Onion.
2. **Сложные домены** с богатой бизнес-логикой: DDD + Hexagonal (чётко вынести порты/адаптеры).
3. **Командная работа**: FSD упрощает ownership фич и параллельную разработку.
4. **Интеграции** (очереди, внешние API): реализуйте как отдельные адаптеры в hexagonal.

Комбинируйте — например, используйте FSD-структуру внутри каждого DDD-модуля, а ядро выносите по onion/hexagonal принципу. Такое сочетание даст максимальную гибкость, тестируемость и понятность архитектуры.
