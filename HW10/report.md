* На 1 ВМ создаем таблицы test1 для записи, test2 для запросов на чтение.
```
create table test1(id integer PRIMARY KEY, test_column varchar(40));
create table test2(id integer PRIMARY KEY, test_column varchar(40));
```
* Создаем публикацию таблицы test1 и подписываемся на публикацию таблицы test2 с ВМ №2.
```
CREATE PUBLICATION test_pub FOR TABLE test1;

CREATE SUBSCRIPTION test_sub
CONNECTION 'host=158.160.13.77 port=5432 user=postgres password=pas123 dbname=postgres'
PUBLICATION test_pub WITH (copy_data = true);
```
* На 2 ВМ создаем таблицы test2 для записи, test1 для запросов на чтение.
```
create table test1(id integer PRIMARY KEY, test_column varchar(40));
create table test2(id integer PRIMARY KEY, test_column varchar(40));
```
* Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.
```
CREATE PUBLICATION test_pub FOR TABLE test2;

CREATE SUBSCRIPTION test_sub
CONNECTION 'host=158.160.84.13 port=5432 user=postgres password=pas123 dbname=postgres'
PUBLICATION test_pub WITH (copy_data = true);
```
* 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
```
CREATE SUBSCRIPTION test_sub1
CONNECTION 'host=158.160.13.77 port=5432 user=postgres password=pas123 dbname=postgres'
PUBLICATION test_pub WITH (copy_data = true);

CREATE SUBSCRIPTION test_sub2
CONNECTION 'host=158.160.84.13 port=5432 user=postgres password=pas123 dbname=postgres'
PUBLICATION test_pub WITH (copy_data = true);
```
