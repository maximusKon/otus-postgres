* Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB - сделано
* Установить на него PostgreSQL 15 с дефолтными настройками - сделано
* Создать БД для тестов: выполнить pgbench -i postgres - сразу не вышло, ругался, что пользака нет. Решил командой `CREATE USER konstantin;grant all on SCHEMA public to konstantin;`. Странно правда, что пришлось давать гранты отдельно на паблик, но без этого откидывало на этапе создания таблиц.
* Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres - сделано. Правда воспользовался своим пользаком, раз уже все гранты дал.
```
pgbench (15.4 (Ubuntu 15.4-2.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 327.2 tps, lat 24.351 ms stddev 23.015, 0 failed
progress: 12.0 s, 441.7 tps, lat 18.121 ms stddev 13.585, 0 failed
progress: 18.0 s, 384.3 tps, lat 20.759 ms stddev 19.064, 0 failed
progress: 24.0 s, 332.8 tps, lat 24.003 ms stddev 15.817, 0 failed
progress: 30.0 s, 472.7 tps, lat 16.979 ms stddev 14.914, 0 failed
progress: 36.0 s, 305.0 tps, lat 26.224 ms stddev 22.805, 0 failed
progress: 42.0 s, 492.2 tps, lat 16.267 ms stddev 11.453, 0 failed
progress: 48.0 s, 586.8 tps, lat 13.632 ms stddev 8.532, 0 failed
progress: 54.0 s, 494.8 tps, lat 16.161 ms stddev 11.429, 0 failed
progress: 60.0 s, 548.2 tps, lat 14.587 ms stddev 13.020, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 26322
number of failed transactions: 0 (0.000%)
latency average = 18.235 ms
latency stddev = 15.664 ms
initial connection time = 17.087 ms
tps = 438.571637 (without initial connection time)
```
* Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла - сделано
* Протестировать заново - сделано
```
pgbench (15.4 (Ubuntu 15.4-2.pgdg22.04+1))
starting vacuum...end.
progress: 6.0 s, 282.2 tps, lat 27.898 ms stddev 19.568, 0 failed
progress: 12.0 s, 298.8 tps, lat 27.035 ms stddev 30.008, 0 failed
progress: 18.0 s, 338.3 tps, lat 23.640 ms stddev 19.049, 0 failed
progress: 24.0 s, 464.3 tps, lat 17.253 ms stddev 10.886, 0 failed
progress: 30.0 s, 354.5 tps, lat 22.575 ms stddev 15.888, 0 failed
progress: 36.0 s, 434.2 tps, lat 18.189 ms stddev 11.673, 0 failed
progress: 42.0 s, 313.3 tps, lat 25.869 ms stddev 24.534, 0 failed
progress: 48.0 s, 411.0 tps, lat 19.464 ms stddev 14.300, 0 failed
progress: 54.0 s, 559.3 tps, lat 14.299 ms stddev 8.694, 0 failed
progress: 60.0 s, 344.2 tps, lat 23.235 ms stddev 17.445, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 22809
number of failed transactions: 0 (0.000%)
latency average = 21.041 ms
latency stddev = 17.738 ms
initial connection time = 17.784 ms
tps = 380.128491 (without initial connection time)
```
* Что изменилось и почему? - возрало латенси, соответственно tps уменьшился. Это странно, ведь по факту мы оптимизируем наши настройки под мощности виртуалки. 
* Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк - сделано
* Посмотреть размер файла с таблицей - сделано `SELECT pg_size_pretty(pg_table_size('test'));` 34 MB
* 5 раз обновить все строчки и добавить к каждой строчке любой символ - сделано
* Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум - 5000000 строк
* 5 раз обновить все строчки и добавить к каждой строчке любой символ - сделано
* Посмотреть размер файла с таблицей - 253 MB
* Отключить Автовакуум на конкретной таблице - сделано
* 10 раз обновить все строчки и добавить к каждой строчке любой символ - сделано
* Посмотреть размер файла с таблицей - 555 MB
* Объясните полученный результат - автовакуум не уменьшает размер файла, поэтому по размерам нет особой разницы, что он включен, что нет. Но если запустить vacuum full, то размертаблицы уменьшится в 10 раз. 
* Не забудьте включить автовакуум) - сделано

Процедура для обновления строк:
```
create or replace PROCEDURE update_data(times int) AS $$
DECLARE
  i integer := 0;
BEGIN
LOOP
   	update test set t1 = t1 || substr(md5(random()::text), 1, 1);
  	i := i + 1;
   	EXIT WHEN i = times;
END LOOP;
END;
$$ LANGUAGE plpgsql;
```
