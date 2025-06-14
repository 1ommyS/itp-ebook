## 1. Общая модель Master–Slave

1. **Master (ведущий)**
   – принимает все операции записи (команды изменения состояния), отвечает за консистентность.
   – синхронизирует состояние с ведомыми (slave).
2. **Slave (ведомый)**
   – получает данные или команды от master.
   – может обслуживать только чтение или выполнять вспомогательные задачи.
3. **Ключевые цели**

    * **Отказоустойчивость**: при падении ведущего один из slaves может быть «продвинут» в ведущие.
    * **Горизонтальное масштабирование чтения**: чтение с ведомых разгружает master.
    * **Распределение задач**: master распределяет работу по slave.
    * **Резервное копирование**: slave служит «горячим» резервом.

---

## 2. Репликация в базах данных

### 2.1. Топологии

| Топология                        | Описание                                                                                                                   |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Master → 1 Slave**             | самый простой: master пишет в одну копию.                                                                                  |
| **Master → N Slaves (звезда)**   | master дублирует изменения сразу в N ведомых; хорошо для масштабирования чтения.                                           |
| **Каскадная (cascading)**        | master → A → B → C; снижает нагрузку на master при большом числе ведомых, но увеличивает задержку у дальних звеньев.       |
| **Кольцевая (circular)**         | каждый узел slave для предыдущего и master для следующего — обеспечивает непрерывность потока, но высокий риск конфликтов. |
| **Multi-source (многоисточник)** | slave принимает логи сразу от нескольких master; редко в open-source, чаще в коммерческих СУБД.                            |

### 2.2. Синхронность репликации

* **Асинхронная**: master не ждёт, пока slave применит изменения → минимальная задержка записи, но возможна потеря последних транзакций.
* **Синхронная**: master ждёт подтверждения от одного/нескольких slave → отсутствие потерь, но высокая latency.
* **Полусинхронная**: ждёт подтверждения хотя бы одного из настроенных синхронных slave.

### 2.3. Физическая vs логическая

| Тип репликации     | Физическая (streaming)              | Логическая                          |
| ------------------ | ----------------------------------- | ----------------------------------- |
| **Что копируется** | WAL-логи целиком                    | Изменения на уровне таблиц/строк    |
| **Гибкость**       | Низкая                              | Высокая: можно фильтровать таблицы  |
| **Применение**     | «горячий» standby, быстрый failover | ETL, миграции, частичная репликация |

### 2.4. Примеры в популярных СУБД

#### PostgreSQL

1. **Настройка мастера (`postgresql.conf`):**

   ```conf
   wal_level = replica
   max_wal_senders = 10
   wal_keep_size = '1GB'
   synchronous_commit = on        # для синхронной реплики
   synchronous_standby_names = '1 (replica1, replica2)'
   ```
2. **Создание слейва:**

   ```bash
   pg_basebackup -h master.host -D /var/lib/postgresql/14/main -U replicator -Fp -Xs -P
   ```
3. **`recovery.conf` (Postgres < 12) или `standby.signal` + `postgresql.auto.conf`:**

   ```conf
   primary_conninfo = 'host=master.host port=5432 user=replicator password=secret'
   primary_slot_name = 'slot1'         # для логической реплики
   ```
4. **Logical Replication (Postgres >= 10):**

   ```sql
   -- На мастере
   CREATE PUBLICATION pub_all FOR ALL TABLES;
   -- На слейве
   CREATE SUBSCRIPTION sub_all
     CONNECTION 'host=master.host port=5432 user=replicator dbname=mydb'
     PUBLICATION pub_all;
   ```

#### MongoDB

1. **Replica Set с одним Primary и N Secondaries:**

   ```js
   // replset-init.js
   rs.initiate({
     _id: 'rs0',
     members: [
       { _id: 0, host: 'mongo1:27017' },
       { _id: 1, host: 'mongo2:27017' },
       { _id: 2, host: 'mongo3:27017' }
     ]
   });
   ```
2. **Чтение с вторичных (Secondary Reads):**

   ```js
   db.getMongo().setReadPref('secondaryPreferred');
   ```

### 2.5. Failover и Orchestration

* **Patroni** (PostgreSQL) — контролирует DCS (etcd/Consul/ZooKeeper), автоматизированный failover.
* **repmgr** — утилита для управления репликами и промоушена.
* **MHA** (MySQL) — менеджер High Availability.
* **Orchestrator** — визуализация топологии MySQL, автоматический failover.

### 2.6. Мониторинг и метрики

* **Задержка репликации (lag):**

    * PostgreSQL: `pg_stat_replication` → `write_lag`, `flush_lag`, `replay_lag`.
    * MySQL: `Seconds_Behind_Master`.
* **WAL/Relay log retention**: следить, чтобы old logs не удалялись до копирования.
* **Нагрузка на master** vs **slave**: QPS, IOPS, CPU, network.

---

## 3. Системы очередей и событий

| Система      | Master / Leader                       | Slave / Follower          | Особенности                                          |
| ------------ | ------------------------------------- | ------------------------- | ---------------------------------------------------- |
| **RabbitMQ** | кафка очереди (mirrored queue master) | зеркальные копии (slaves) | при падении master один из слейвов становится master |
| **Kafka**    | leader партиции                       | in-sync replicas (ISRs)   | `min.insync.replicas`, ack=all для надёжности        |
| **ActiveMQ** | broker-master                         | network of brokers-миры   | master-slave сети с shared storage или replication   |

## 8. Эволюция: Multi-Master и Consensus

1. **Multi-Master (active-active)**
   – каждая нода может принимать записи.
   – требуется разрешение конфликтов (CRDT, last-write-wins).
   – примеры: MySQL Group Replication, Galera, CouchDB.
2. **Quorum-based (Paxos, Raft)**
   – запись фиксируется при подтверждении кворума узлов.
   – обеспечивает сильную согласованность.

---
