Воспользуемся (демонстрационной БД)[https://postgrespro.ru/education/demodb]. [Cхема данных](https://postgrespro.ru/docs/postgrespro/9.6/apjs02).

### Индекс к какой-либо из таблиц БД
Создадим индекс 
```sql
create index bookings_book_date_idx on bookings.bookings (book_date);
```
Выполним запрос
```sql
explain (analyze)
select * from bookings b 
where '2017-07-01' <= book_date and book_date <= '2017-07-31'
order by book_date desc 
```
Результат
```
Index Scan Backward using bookings_book_date_idx on bookings b  (cost=0.42..19520.09 rows=166196 width=21) (actual time=0.025..64.672 rows=166285 loops=1)
  Index Cond: ((book_date >= '2017-07-01 00:00:00+03'::timestamp with time zone) AND (book_date <= '2017-07-31 00:00:00+03'::timestamp with time zone))
Planning Time: 0.081 ms
Execution Time: 69.968 ms
```
Видно, что индекс использовался как при поиске, так и при сортировке
### Индекс для полнотекстового поиска
Добавим столбец для полнотекстового поиска и заполним его
```sql
alter table tickets add column passenger_name_lexeme tsvector;

update tickets
set passenger_name_lexeme = to_tsvector(passenger_name);
```
Добавим индекс
```sql
create index tickets_passenger_name_lexeme_idx on tickets using GIN(passenger_name_lexeme);
```
Выполним запрос
```sql
explain (analyze)
select *
from tickets
where passenger_name_lexeme @@ to_tsquery('sidorova');
```
Результат
```
Bitmap Heap Scan on tickets  (cost=46.36..11968.85 rows=3886 width=140) (actual time=0.989..4.232 rows=3924 loops=1)
  Recheck Cond: (passenger_name_lexeme @@ to_tsquery('sidorova'::text))
  Heap Blocks: exact=3512
  ->  Bitmap Index Scan on tickets_passenger_name_lexeme_idx  (cost=0.00..45.39 rows=3886 width=0) (actual time=0.523..0.523 rows=3924 loops=1)
        Index Cond: (passenger_name_lexeme @@ to_tsquery('sidorova'::text))
Planning Time: 0.117 ms
Execution Time: 4.397 ms
```
### Индекс на часть таблицы или индекс на поле с функцией
Добавим индекс
```sql
create index ticket_flights_amount_100000_idx on ticket_flights(amount) where amount >= 100000;
```
Выполним запрос
```sql
explain (analyze)
select * 
from ticket_flights tf 
where amount >= 100000;
```
Результат
```
Bitmap Heap Scan on ticket_flights tf  (cost=515.82..21370.24 rows=31835 width=32) (actual time=1.520..5.396 rows=31115 loops=1)
  Recheck Cond: (amount >= '100000'::numeric)
  Heap Blocks: exact=956
  ->  Bitmap Index Scan on ticket_flights_amount_100000_idx  (cost=0.00..507.86 rows=31835 width=0) (actual time=1.396..1.396 rows=31115 loops=1)
Planning Time: 0.274 ms
Execution Time: 6.332 ms
```
### Индекс на несколько полей
Добавим индекс
```sql
create index flights_airports_idx on flights(departure_airport, arrival_airport);
```
Выполним запрос
```sql
explain (analyze)
select * 
from flights f 
where departure_airport = 'DME' and arrival_airport = 'LED';
```
Результат
```
Bitmap Heap Scan on flights f  (cost=8.04..657.15 rows=366 width=63) (actual time=0.034..0.078 rows=484 loops=1)
  Recheck Cond: ((departure_airport = 'DME'::bpchar) AND (arrival_airport = 'LED'::bpchar))
  Heap Blocks: exact=7
  ->  Bitmap Index Scan on flights_airports_idx  (cost=0.00..7.95 rows=366 width=0) (actual time=0.025..0.026 rows=484 loops=1)
        Index Cond: ((departure_airport = 'DME'::bpchar) AND (arrival_airport = 'LED'::bpchar))
Planning Time: 0.080 ms
Execution Time: 0.108 ms
```
