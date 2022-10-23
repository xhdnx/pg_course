## создайте новый кластер PostgresSQL 
>hdn@denispg:~$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log

## применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
Отредактировал параметры файла postgres.conf со следующими параметрами
max_connections = 40
shared_buffers = 1GB
effective_cache_size = 3GB
maintenance_work_mem = 512MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 500
random_page_cost = 4
effective_io_concurrency = 2
work_mem = 6553kB
min_wal_size = 4GB
max_wal_size = 16GB 

## выполнить pgbench -i postgres
Запустил инициализацию pgbench. Создает необходимые таблицы в бд postgres

## запустить pgbench -c8 -P 10 -T 600 -U postgres postgres
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 319024
latency average = 15.045 ms
latency stddev = 11.390 ms
initial connection time = 14.514 ms
tps = 531.699263 (without initial connection time)

Были получены следующие средние значения при указанной выше конфигурации сервера, без настроенного autovacuum

## настроить autovacuum максимально эффективно
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
duration: 600 s
number of transactions actually processed: 365087
latency average = 13.147 ms
latency stddev = 9.105 ms
initial connection time = 28.468 ms
tps = 608.485229 (without initial connection time)

## построить график по получившимся значениям
![alt text](images/6_1.png)

конфигурация autovacuum
![alt text](images/6_2.png)

