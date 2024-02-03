* развернуть виртуальную машину любым удобным способом - сделано
* поставить на неё PostgreSQL 15 любым способом - сделано
* настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины - сделано
* нагрузить кластер через утилиту через утилиту pgbench - сделано
* написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему - сделано

Характеристики ВМ:
```
OS : Ubuntu 22
RAM: 4 GB
CPUs num: 2
Data Storage: ssd 10 GB
```
Чтобы смотреть в перспективе вызовем pgbench на дефолтных настройках
```
pgbench -c8 -P 6 -T 60 -U konstantin postgres

pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 331.2 tps, lat 24.071 ms stddev 19.845, 0 failed
progress: 12.0 s, 433.7 tps, lat 18.442 ms stddev 14.244, 0 failed
progress: 18.0 s, 315.5 tps, lat 25.172 ms stddev 25.040, 0 failed
progress: 24.0 s, 222.8 tps, lat 36.110 ms stddev 26.192, 0 failed
progress: 30.0 s, 181.7 tps, lat 43.996 ms stddev 29.842, 0 failed
progress: 36.0 s, 189.8 tps, lat 42.040 ms stddev 19.398, 0 failed
progress: 42.0 s, 199.2 tps, lat 40.296 ms stddev 19.400, 0 failed
progress: 48.0 s, 304.0 tps, lat 26.343 ms stddev 18.445, 0 failed
progress: 54.0 s, 562.3 tps, lat 14.232 ms stddev 10.787, 0 failed
progress: 60.0 s, 303.0 tps, lat 26.334 ms stddev 25.765, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 18267
number of failed transactions: 0 (0.000%)
latency average = 26.275 ms
latency stddev = 22.363 ms
initial connection time = 14.909 ms
tps = 304.409361 (without initial connection time)
```
Теперь поменяем несколько следующие настройки
```
# уменьшим количество одновременных конектов, для теста нам не нужно много
max_connections = 50
# выделим больше памяти для кэширования
shared_buffers = 2GB - 
effective_cache_size = 2GB
# ихбавимся от сброса данных с WAL на диск
max_wal_size = 4GB
checkpoint_timeout = 1d
```
Запустим pgbench на новых настройках
```
pgbench -c8 -P 6 -T 60 -U konstantin postgres

pgbench (15.5 (Ubuntu 15.5-1.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 422.8 tps, lat 18.815 ms stddev 14.677, 0 failed
progress: 12.0 s, 490.8 tps, lat 16.335 ms stddev 12.144, 0 failed
progress: 18.0 s, 322.3 tps, lat 24.743 ms stddev 26.904, 0 failed
progress: 24.0 s, 403.8 tps, lat 19.864 ms stddev 16.492, 0 failed
progress: 30.0 s, 702.0 tps, lat 11.347 ms stddev 9.306, 0 failed
progress: 36.0 s, 209.5 tps, lat 38.228 ms stddev 21.628, 0 failed
progress: 42.0 s, 522.5 tps, lat 15.359 ms stddev 12.306, 0 failed
progress: 48.0 s, 359.5 tps, lat 22.213 ms stddev 21.039, 0 failed
progress: 54.0 s, 648.5 tps, lat 12.352 ms stddev 8.887, 0 failed
progress: 60.0 s, 599.8 tps, lat 13.328 ms stddev 10.421, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 28098
number of failed transactions: 0 (0.000%)
latency average = 17.086 ms
latency stddev = 15.975 ms
initial connection time = 15.350 ms
tps = 468.142189 (without initial connection time)
```
Как видно латенси уменьшилось, а tps в свою очередь возросло
