# Redis — подробный конспект

## Execution model

Redis выполняет большинство commands последовательно в main execution thread, поэтому отдельная команда atomic относительно других. Network I/O и background persistence могут использовать дополнительные threads/processes.

Одна медленная O(n) команда блокирует другие. Complexity команды и размер collection критичны; `KEYS *` в production заменяют incremental `SCAN`, но SCAN даёт weak iteration и duplicates.

## Data structures

String — bytes до лимита, counters через INCR. Hash — fields object. List — ordered sequence, но random access дорог. Set — membership/operations множеств. Sorted set — member + score, range/rank.

Streams дают append log, consumer groups и pending entries, но не являются Kafka replacement по retention/partitioning ecosystem.

Выбирайте ключи с namespace и bounded cardinality: `app:env:entity:id`. Не храните огромный aggregate одним value, если маленькое изменение переписывает всё.

## TTL

Expiry хранится отдельно. Passive expiration удаляет ключ при доступе, active sampling очищает часть истёкших. Команда может изменить TTL semantics; проверяйте `SET` options (`EX/PX/NX/XX/KEEPTTL`).

Добавляйте jitter, чтобы миллионы keys не истекли одновременно.

## Cache-aside race

Read miss → DB → SET. Между DB read и SET другой поток может update DB и invalidate, затем первый положит старое значение. Варианты: versioned value, delayed double delete, short TTL, write-through/CDC invalidation, accept bounded staleness.

Cache не должен быть единственной копией данных, если loss неприемлем.

## Stampede

После hot key expiry множество callers идут в DB. Защиты:

- mutex/single-flight на key;
- logical expiry + stale-while-revalidate;
- proactive refresh;
- jitter;
- bounded fallback/concurrency.

Lock сам может стать hot; waiting clients должны иметь timeout и fallback.

## Eviction

При `maxmemory` policy выбирает candidates: noeviction, allkeys/volatile LRU/LFU/random/TTL variants. Approximated LRU/LFU экономит memory/CPU.

Если persistence/replication buffers и fragmentation растут, RSS может превышать logical used memory. Планируйте headroom.

## Persistence

RDB fork делает point-in-time snapshot; возможна потеря после snapshot. AOF журналирует write commands, fsync always/everysec/no меняет durability/latency. Rewrite compact AOF.

Fork copy-on-write при интенсивных writes увеличивает memory. Тестируйте restore и измеряйте fork latency.

## Replication и failover

Replication асинхронна. Replica может читать stale. Sentinel наблюдает primary, достигает quorum и выполняет failover; clients должны обновить route.

Split brain/partition может принять writes у старого primary до fencing/network policy, затем они потеряются при reconnection. Redis не становится strongly consistent DB автоматически.

## Cluster

16384 hash slots распределены nodes. Key hash tag `{user:1}` заставляет related keys попасть в один slot для multi-key operations. Resharding переносит slots; clients обрабатывают MOVED/ASK.

Hot key не решается равномерным slots: один key обслуживает один primary. Нужны local cache, sharded counters или изменение модели.

## Distributed lock

Acquire: `SET resource token NX PX ttl`. Release — Lua script: удалить только если token совпадает. Renewal требует ownership и stop policy.

Проблема lease: клиент замер после expiry, новый получил lock, старый продолжил. Fencing counter передаётся защищаемому resource; resource принимает только возрастающий token.

## Критерий готовности

Вы готовы, если выбираете structure, объясняете TTL/eviction/persistence, находите cache-aside race и проектируете lock с token/fencing, не обещая абсолютную consistency.

