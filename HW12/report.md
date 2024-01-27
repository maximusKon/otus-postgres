Воспользуемся [демонстрационной БД](https://postgrespro.ru/education/demodb). [Cхема данных](https://postgrespro.ru/docs/postgrespro/9.6/apjs02).

* Прямое соединение двух или более таблиц
```sql
-- получим информацию о посадочных местах бизнес-класса для каждого самолёта
select * from aircrafts_data ad join seats s on s.aircraft_code = ad.aircraft_code 
where s.fare_conditions = 'Business'
```
* Левостороннее (или правостороннее) соединение двух или более таблиц
```sql
-- для каждого аэропорта получим все прибывшие рейсы
select ad.airport_name, f.flight_no from airports_data ad left join flights f on f.arrival_airport = ad.airport_code 
where f.status = 'Arrived'
```
* Кросс соединение двух или более таблиц
```sql
-- получим сочитания кодов аэропортов и самолётов
select ap.airport_code, ac.aircraft_code from airports_data ap cross join aircrafts_data ac
```
* Полное соединение двух или более таблиц
```sql
-- получим аэропорта в которые не прилетают самолёты и полёты в неизвестные аэропорта (да, пример максимально искусственный, но лучше так и не придумал)
select ad.airport_code, f.flight_no from airports_data ad full join flights f on f.arrival_airport = ad.airport_code
where ad.airport_code is null or f.flight_no is null
```
* Запрос, в котором будут использованы разные типы соединений
```sql
-- для каждого аэропорта получим все посадочные талоны рейсов за 12 июня по МСК
select * from airports_data ad 
	left join flights f on ad.airport_code = f.departure_airport
	join ticket_flights tf on tf.flight_id = f.flight_id 
	join boarding_passes bp on bp.ticket_no = tf.ticket_no and bp.flight_id = f.flight_id 
where  '2017-06-12 00:00:00.000 +0300' < f.actual_departure and f.actual_departure < '2017-06-12 23:59:59.999 +0300'
```
