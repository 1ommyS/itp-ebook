# System design

## Каркас ответа

Собеседование по архитектуре оценивает ход решений, а не угадывание «правильной схемы».

1. Уточнить функциональные сценарии и то, что не входит в scope.
2. Согласовать non-functional requirements: latency, availability, durability, consistency, geography, security.
3. Оценить масштаб.
4. Определить API и главные data models.
5. Нарисовать минимальный end-to-end поток.
6. Найти bottlenecks и single points of failure.
7. Углубиться в 1–2 критических trade-offs.
8. Обсудить деградацию, наблюдаемость и evolution.

## Быстрые оценки

Начинайте с входных допущений и порядка величин.

```text
average RPS = requests per day / 86 400
peak RPS    = average RPS × peak factor
storage/day = writes/day × average record size × replication factor
bandwidth   = RPS × average payload size
```

Отделяйте average от peak, read от write, payload от metadata, raw storage от replication/index overhead. Точность до процента не нужна; важна проверяемая логика.

## Вертикальное и горизонтальное масштабирование

Вертикальное масштабирование проще, но имеет предел и увеличивает blast radius. Горизонтальное требует stateless compute или явного управления state, балансировки и согласованности.

Сначала устраните неэффективный запрос/алгоритм, затем масштабируйте: десять instances не исправляют полный scan на каждый request.

## Load balancing

L4 направляет соединения по transport information, L7 понимает HTTP и может маршрутизировать по host/path/header. Алгоритмы: round robin, least connections, weighted и consistent hashing.

Health check должен исключать непригодный instance, connection draining — завершать in-flight requests при deploy. Sticky sessions упрощают локальный state, но ухудшают баланс и failover; предпочтительнее внешний session store или stateless token при подходящем контракте.

## Cache

- Cache-aside: приложение читает source при miss.
- Read-through: cache сам загружает.
- Write-through: запись синхронно идёт через cache в storage.
- Write-behind: cache подтверждает раньше storage; выше риск потери и сложность.

Обсудите key, TTL, invalidation, stampede, hot key, consistency и максимальный размер. «Добавим Redis» — начало, а не завершение дизайна.

## SQL или NoSQL

Выбирайте по access patterns и guarantees:

- relational DB — joins, constraints, transactions и гибкие запросы;
- key-value — простой доступ по key и масштабирование;
- document — агрегаты с изменяемой schema;
- wide-column — большие распределённые записи по partition key;
- search engine — inverted index и relevance;
- object storage — большие blobs.

В одной системе могут быть разные stores, но каждый добавляет replication, backup, security и on-call cost.

## Replication и failover

Replication повышает доступность и read capacity, но вводит lag и конфликт ролей. Определите sync/async, read routing, promotion, split-brain prevention, RPO и RTO.

- RPO: сколько данных допустимо потерять.
- RTO: сколько времени допустимо восстанавливаться.

Backup не равен replica: ошибочное удаление быстро реплицируется, поэтому нужны независимые backups и проверка restore.

## Sharding

Shard key должен распределять нагрузку и поддерживать основные запросы. Риски: hot shard, cross-shard joins/transactions, resharding и глобальные уникальные constraints.

Range sharding удобен для диапазонов, но может создать hot tail. Hash sharding распределяет равномернее, но теряет локальность диапазона. Consistent hashing уменьшает перемещение keys при изменении nodes, но сам не решает replication и hot keys.

## Queue или log

Традиционная queue ориентирована на распределение задач и удаление/скрытие обработанного сообщения. Distributed log хранит ordered partitions и независимые offsets consumers, удобен для replay и множества подписчиков.

Обсудите ordering, delivery semantics, retention, backpressure, poison messages и consumer lag.

## Consistency

Strong consistency упрощает reasoning, но может увеличить latency/снизить availability при partition. Eventual consistency требует versioning, conflict policy и понятного пользовательского состояния.

Часто разные данные имеют разные требования: баланс и лимит требуют строгой проверки, каталог может допустить небольшую задержку.

## Rate limiting

Token bucket допускает burst до capacity и пополняет tokens с постоянной скоростью. Leaky bucket выравнивает выход. Fixed window прост, но имеет всплеск на границе; sliding window точнее, но дороже.

Определите identity (user/API key/IP), distributed coordination, fail-open/fail-closed, response 429, headers и отдельные лимиты для дорогих операций.

## API Gateway и service discovery

Gateway завершает внешний TLS, authentication, routing, quotas и protocol adaptation. Не превращайте его в общий бизнес-сервис: это создаёт coupling и bottleneck.

Service discovery сопоставляет logical service с instances. Client-side discovery выбирает endpoint в клиенте; server-side — через proxy/load balancer. Нужны health, TTL/lease, removal stale instances и безопасный rollout.

## Кейс 1: URL shortener

API: создать короткий URL, перенаправить, получить статистику. Ключевой путь чтения должен быть быстрым.

- ID generator или случайный code с проверкой collision;
- primary store `code → target, owner, expiry`;
- cache для hot codes;
- async analytics через log;
- защита от abuse и unsafe URLs;
- TTL/cleanup и redirect semantics 301/302 по требованиям.

Уточните custom aliases, объём ссылок, read/write ratio и требуемую консистентность после создания.

## Кейс 2: Notification service

Принимает команды, хранит intent, выбирает channel/template, публикует задания workers. Нужны user preferences, rate limits, provider failover, retry с jitter, idempotency, DLQ и status tracking.

Exactly-once доставка email/SMS обычно недостижима на стороне внешнего provider; проектируйте at-least-once с deduplication там, где поддерживается, и честным статусом.

## Кейс 3: Order service

Order владеет состоянием заказа, а payment/inventory — своими данными. Local transaction + outbox публикует изменение, saga координирует шаги, optimistic version защищает переходы состояния.

Сформулируйте state machine (`NEW → RESERVED → PAID → CONFIRMED` и ветви отмены), повтор команд, позднее событие и ручное восстановление.

## Чек-лист завершения дизайна

- Все требования связаны с компонентом или решением.
- Есть числа для peak load и storage.
- У каждой stateful части названы replication и backup.
- У внешнего вызова есть timeout и retry policy.
- У повторяемой команды есть idempotency.
- Описаны overload и degraded mode.
- Названы SLI, alert и trace path.
- Показаны два главных trade-offs, а не только плюсы.

