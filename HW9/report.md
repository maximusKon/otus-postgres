* Создаем ВМ/докер c ПГ. - сделано
* Создаем БД, схему и в ней таблицу. - сделано
* Заполним таблицы автосгенерированными 100 записями.
```
select generate_series(1,100) as id, md5(random()::text)::char(20) as test_text;
```
* Под линукс пользователем Postgres создадим каталог для бэкапов
```
sudo su postgres
mkdir ~/backups
```
* Сделаем логический бэкап используя утилиту COPY
```
\copy test_schema.test_table to /var/lib/postgresql/backups/test_table_backuo_1.csv
```
* Восстановим в 2 таблицу данные из бэкапа.
```
create table test_schema.test_table2(id bigint, test_text char(20));
\copy test_schema.test_table2 from /var/lib/postgresql/backups/test_table_backuo_1.csv
```
* Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц
```
pg_dump -d test_bd -C -U postgres -Fc >  /var/lib/postgresql/backups/test_bd_backup_1.gz
```
* Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!
```
pg_restore -d test_bd2 -U postgres -n test_schema -t test_table2 /var/lib/postgresql/backups/test_bd_backup_1.gz
```
