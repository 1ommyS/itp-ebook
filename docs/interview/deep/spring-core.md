# Spring Core — подробный конспект

## Контейнер и definitions

Spring сначала собирает `BeanDefinition`: class/factory method, scope, qualifiers, lazy, lifecycle metadata. Затем `BeanFactoryPostProcessor` может менять definitions до создания beans. `BeanPostProcessor` работает уже с instances.

Это различие объясняет, почему configuration/property processing должно происходить до instantiation, а proxy creation — после создания target.

## `BeanFactory` и `ApplicationContext`

BeanFactory — базовый registry/factory. ApplicationContext добавляет events, resources, message source, environment и автоматическую регистрацию важных post-processors. В обычном приложении используют context.

Singleton beans по умолчанию pre-instantiated при refresh. Lazy откладывает создание, но переносит ошибку конфигурации на runtime и ухудшает предсказуемость startup.

## Component scanning и Java config

Stereotypes (`@Component`, `@Service`, `@Repository`, `@Controller`) технически регистрируют bean, но также сообщают роль. `@Repository` участвует в translation persistence exceptions.

`@Bean` нужен для стороннего класса, явной assembly и сложного выбора implementation. Full configuration semantics обеспечивают корректные межbean calls; современный код лучше использует параметры factory method вместо прямых вызовов методов конфигурации.

## Dependency resolution

Контейнер выбирает кандидатов по типу, затем qualifier/primary/name и additional rules. Коллекция `List<Handler>` внедряет все handlers, порядок задают `@Order`/`Ordered`.

`Optional<T>` или `ObjectProvider<T>` выражают optional/lazy dependency, но optional business dependency часто означает неясный design. Null field injection ухудшает invariants.

## Constructor injection

Плюсы: immutable references, fail-fast missing dependency, простой unit test, ясный public construction contract. Один constructor не требует `@Autowired`.

Circular constructor dependency не создаётся — это полезный сигнал. Разорвите цикл выделением третьей ответственности, domain event или изменением direction.

## Bean scopes

- singleton — один instance в context;
- prototype — новый при каждом `getBean`/resolution;
- request/session/application/websocket — web scopes;
- custom scope — специальный lifecycle.

Singleton должен быть thread-safe или stateless. Spring не синхронизирует методы bean автоматически.

Scoped proxy позволяет singleton хранить proxy, который разрешает текущий request bean. Альтернатива — `ObjectProvider` и явный lookup.

## Lifecycle подробно

1. constructor/factory method;
2. dependency population;
3. Aware callbacks;
4. `postProcessBeforeInitialization`;
5. `@PostConstruct`, `InitializingBean`, custom init;
6. `postProcessAfterInitialization` — возможен proxy;
7. runtime;
8. `@PreDestroy`, DisposableBean, custom destroy.

BeanPostProcessor сам создаётся рано; зависимости BPP требуют осторожности, иначе часть beans не проходит все processors.

## AOP terminology

- aspect — модуль cross-cutting concern;
- join point — перехватываемая точка (в Spring AOP method execution);
- pointcut — условие выбора;
- advice — код before/after/around;
- target — исходный bean;
- proxy — объект, через который приходит вызов;
- weaving — связывание aspect и target.

Around advice обязан вызвать `proceed`, если хочет продолжить chain, и корректно передать exception/result.

## JDK proxy и class proxy

JDK proxy реализует interfaces и вызов target происходит через InvocationHandler. Class proxy создаёт subclass и переопределяет методы; final class/method ограничивает interception.

В обоих случаях internal `this.method()` не проходит через proxy. Proxy identity/type может влиять на casts и annotations lookup.

## Self-invocation

```java
public void checkout() {
    reserve(); // прямой вызов target, advice не участвует
}

@Transactional
public void reserve() { ... }
```

Лучшее исправление — сделать transactional operation отдельной application boundary в другом bean. `AopContext.currentProxy()` и self injection связывают бизнес-код с инфраструктурой.

## Events

Обычный listener синхронный. `@Async` меняет thread/transaction context и требует executor/error policy. `@TransactionalEventListener` привязывает фазу к transaction, но событие всё ещё in-memory и потеряется при crash. Для durable integration — outbox.

## Profiles и Environment

Property sources имеют precedence. Один key может прийти из command line, environment, config files, test overrides. Диагностика должна показывать origin безопасно, не раскрывая secret.

Profile выбирает набор beans/config, но `if prod` внутри бизнес-кода — smell. Предпочитайте разные adapters одного interface.

## Критерий готовности

Вы готовы, если можете от definition пройти до proxied bean, объяснить порядок processors/lifecycle, разрешить несколько candidates и показать, почему self-invocation ломает AOP.

