* Настройте выполнение контрольной точки раз в 30 секунд. - сделано
* 10 минут c помощью утилиты pgbench подавайте нагрузку - sudo -u postgres pgbench -c2 -P 60 -T 600 -U postgres

До подачи нагрузки:
```
select * from pg_stat_bgwriter;

 checkpoints_timed | checkpoints_req | checkpoint_write_time | checkpoint_sync_time | buffers_checkpoint | buffers_clean | maxwritten_clean | buffers_backend | buffers_backend_fsync | buffers_alloc |          stats_reset
-------------------+-----------------+-----------------------+----------------------+--------------------+---------------+------------------+-----------------+-----------------------+---------------+-------------------------------
               113 |               1 |                 30829 |                   48 |               1746 |             0 |                0 |            1699 |                     0 |          2288 | 2023-11-12 11:19:03.881063+00

SELECT * FROM pg_ls_waldir() LIMIT 10;

           name           |   size   |      modification
--------------------------+----------+------------------------
 000000010000000000000003 | 16777216 | 2023-11-12 11:48:21+00
 000000010000000000000002 | 16777216 | 2023-11-12 11:49:15+00

postgres=# SELECT pg_current_wal_lsn(), pg_current_wal_insert_lsn();

 pg_current_wal_lsn | pg_current_wal_insert_lsn
--------------------+---------------------------
 0/153DC70          | 0/153DC70
```
После:
```
 checkpoints_timed | checkpoints_req | checkpoint_write_time | checkpoint_sync_time | buffers_checkpoint | buffers_clean | maxwritten_clean | buffers_backend | buffers_backend_fsync | buffers_alloc |          stats_reset
-------------------+-----------------+-----------------------+----------------------+--------------------+---------------+------------------+-----------------+-----------------------+---------------+-------------------------------
               135 |               1 |                568140 |                  546 |              39215 |             0 |                0 |            3167 |                     0 |          4047 | 2023-11-12 11:19:03.881063+00

SELECT * FROM pg_ls_waldir() LIMIT 10;

           name           |   size   |      modification
--------------------------+----------+------------------------
 00000001000000000000001A | 16777216 | 2023-11-12 12:33:21+00
 00000001000000000000001C | 16777216 | 2023-11-12 12:34:20+00
 00000001000000000000001B | 16777216 | 2023-11-12 12:33:50+00
 000000010000000000000019 | 16777216 | 2023-11-12 12:36:16+00

postgres=# SELECT pg_current_wal_lsn(), pg_current_wal_insert_lsn();

 pg_current_wal_lsn | pg_current_wal_insert_lsn
--------------------+---------------------------
 0/193AE9E0         | 0/193AE9E0
```
Также обратимся к логам:
```
2023-11-12 12:24:49.796 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:25:16.145 UTC [5906] LOG:  checkpoint complete: wrote 2039 buffers (12.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.213 s, sync=0.051 s, total=26.349 s; sync files=18, longest=0.050 s, average=0.003 s; distance=20344 kB, estimate=20344 kB
2023-11-12 12:25:19.148 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:25:46.111 UTC [5906] LOG:  checkpoint complete: wrote 1834 buffers (11.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.876 s, sync=0.028 s, total=26.963 s; sync files=15, longest=0.014 s, average=0.002 s; distance=16487 kB, estimate=19958 kB
2023-11-12 12:25:49.114 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:26:16.041 UTC [5906] LOG:  checkpoint complete: wrote 1873 buffers (11.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.867 s, sync=0.012 s, total=26.927 s; sync files=7, longest=0.012 s, average=0.002 s; distance=18988 kB, estimate=19861 kB
2023-11-12 12:26:19.044 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:26:46.195 UTC [5906] LOG:  checkpoint complete: wrote 1976 buffers (12.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.987 s, sync=0.074 s, total=27.152 s; sync files=14, longest=0.030 s, average=0.006 s; distance=20371 kB, estimate=20371 kB
2023-11-12 12:26:49.198 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:27:16.116 UTC [5906] LOG:  checkpoint complete: wrote 1858 buffers (11.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.811 s, sync=0.024 s, total=26.919 s; sync files=7, longest=0.014 s, average=0.004 s; distance=19418 kB, estimate=20276 kB
2023-11-12 12:27:19.120 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:27:46.147 UTC [5906] LOG:  checkpoint complete: wrote 1966 buffers (12.0%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.808 s, sync=0.046 s, total=27.028 s; sync files=14, longest=0.022 s, average=0.004 s; distance=17327 kB, estimate=19981 kB
2023-11-12 12:27:49.150 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:28:16.126 UTC [5906] LOG:  checkpoint complete: wrote 1542 buffers (9.4%); 0 WAL file(s) added, 0 removed, 0 recycled; write=26.896 s, sync=0.014 s, total=26.976 s; sync files=6, longest=0.008 s, average=0.003 s; distance=16061 kB, estimate=19589 kB
2023-11-12 12:28:19.129 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:28:46.098 UTC [5906] LOG:  checkpoint complete: wrote 1987 buffers (12.1%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.876 s, sync=0.019 s, total=26.969 s; sync files=14, longest=0.010 s, average=0.002 s; distance=18997 kB, estimate=19530 kB
2023-11-12 12:28:49.101 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:29:16.143 UTC [5906] LOG:  checkpoint complete: wrote 1901 buffers (11.6%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.968 s, sync=0.019 s, total=27.042 s; sync files=6, longest=0.013 s, average=0.004 s; distance=20844 kB, estimate=20844 kB
2023-11-12 12:29:19.146 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:29:46.113 UTC [5906] LOG:  checkpoint complete: wrote 1981 buffers (12.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.855 s, sync=0.053 s, total=26.968 s; sync files=14, longest=0.019 s, average=0.004 s; distance=18978 kB, estimate=20657 kB
2023-11-12 12:29:49.116 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:30:16.156 UTC [5906] LOG:  checkpoint complete: wrote 1663 buffers (10.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.965 s, sync=0.013 s, total=27.040 s; sync files=6, longest=0.013 s, average=0.003 s; distance=17177 kB, estimate=20309 kB
2023-11-12 12:30:19.157 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:30:46.043 UTC [5906] LOG:  checkpoint complete: wrote 1820 buffers (11.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.789 s, sync=0.017 s, total=26.887 s; sync files=10, longest=0.011 s, average=0.002 s; distance=15226 kB, estimate=19801 kB
2023-11-12 12:30:49.046 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:31:16.112 UTC [5906] LOG:  checkpoint complete: wrote 1860 buffers (11.4%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.987 s, sync=0.016 s, total=27.066 s; sync files=8, longest=0.016 s, average=0.002 s; distance=18185 kB, estimate=19639 kB
2023-11-12 12:31:19.115 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:31:46.076 UTC [5906] LOG:  checkpoint complete: wrote 1821 buffers (11.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.893 s, sync=0.019 s, total=26.961 s; sync files=6, longest=0.013 s, average=0.004 s; distance=19054 kB, estimate=19581 kB
2023-11-12 12:31:49.076 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:32:16.048 UTC [5906] LOG:  checkpoint complete: wrote 2035 buffers (12.4%); 0 WAL file(s) added, 0 removed, 2 recycled; write=26.888 s, sync=0.019 s, total=26.973 s; sync files=13, longest=0.011 s, average=0.002 s; distance=19724 kB, estimate=19724 kB
2023-11-12 12:32:19.051 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:32:46.135 UTC [5906] LOG:  checkpoint complete: wrote 1844 buffers (11.3%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.989 s, sync=0.024 s, total=27.084 s; sync files=6, longest=0.016 s, average=0.004 s; distance=18950 kB, estimate=19646 kB
2023-11-12 12:32:49.138 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:33:16.087 UTC [5906] LOG:  checkpoint complete: wrote 1960 buffers (12.0%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.885 s, sync=0.010 s, total=26.950 s; sync files=10, longest=0.010 s, average=0.001 s; distance=19792 kB, estimate=19792 kB
2023-11-12 12:33:19.091 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:33:46.121 UTC [5906] LOG:  checkpoint complete: wrote 1828 buffers (11.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.971 s, sync=0.013 s, total=27.031 s; sync files=6, longest=0.011 s, average=0.003 s; distance=19708 kB, estimate=19783 kB
2023-11-12 12:33:49.124 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:34:16.087 UTC [5906] LOG:  checkpoint complete: wrote 2079 buffers (12.7%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.880 s, sync=0.024 s, total=26.963 s; sync files=12, longest=0.016 s, average=0.002 s; distance=19926 kB, estimate=19926 kB
2023-11-12 12:34:19.090 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:34:46.064 UTC [5906] LOG:  checkpoint complete: wrote 1663 buffers (10.2%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.907 s, sync=0.003 s, total=26.975 s; sync files=6, longest=0.003 s, average=0.001 s; distance=17931 kB, estimate=19726 kB
2023-11-12 12:35:49.116 UTC [5906] LOG:  checkpoint starting: time
2023-11-12 12:36:16.046 UTC [5906] LOG:  checkpoint complete: wrote 344 buffers (2.1%); 0 WAL file(s) added, 0 removed, 1 recycled; write=26.881 s, sync=0.004 s, total=26.931 s; sync files=12, longest=0.004 s, average=0.001 s; distance=5297 kB, estimate=18283 kB
```
* Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку. - Было выполнено 20 контрольных точек. На каждую пришлось где-то по 20 MB данных.
* Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло? - да, всё точно по расписанию, но могли и чаще, если бы был превышен max_wal_size. Но установлен в 1 GB. Его превышения не произошло, нагрузка была недостаточно интеснсивной.
* Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
* Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?
