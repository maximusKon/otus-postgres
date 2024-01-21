В демонстрационной БД среднего размера от Postgres я увидел две таблицы, которые потенциально можно секционировать:
* `bookings`. по диапазонам значений дат времени бронирования (`book_date`)
* `ticket_flights`. По классу обслуживания (`fare_conditions`)

С практической точки зрения вероятно было бы полезнее секционировать `bookings`, так как вероятно наиболее часто при работе с системой нужна информация по бронированиям лишь на ближайшее время. Однако я всё же решил секционировать `ticket_flights`, так как по списку значений я ещё не секционировал и сама таблица больше. Хотя понятно, что партиции будут весьма неравномерные за счёт того, что в экономе явно летает больше человек, чем в бизнесе.

Скрипт партиционирования:
```sql
do $$
declare
	fare_conditions	constant varchar(20)[] = array['Business', 'Comfort', 'Economy'];
	fare_condition varchar(20);
begin
	
	CREATE TABLE bookings.ticket_flights_part (
		ticket_no bpchar(13) NOT NULL,
		flight_id int4 NOT NULL,
		fare_conditions varchar(10) NOT NULL,
		amount numeric(10, 2) NOT NULL,
		CONSTRAINT ticket_flights_part_amount_check CHECK ((amount >= (0)::numeric)),
		CONSTRAINT ticket_flights_part_pkey PRIMARY KEY (ticket_no, flight_id, fare_conditions)
	) partition by list (fare_conditions);

	foreach fare_condition in array fare_conditions -- добавляем секции
	loop
		RAISE NOTICE E'create table bookings.ticket_flights_%s partition', lower(fare_condition);
		execute format('create table bookings.ticket_flights_%s partition of bookings.ticket_flights_part for values in (%s);', lower(fare_condition), quote_literal(fare_condition));	
	end loop;

	foreach fare_condition in array fare_conditions -- переносим данные
	loop
		RAISE NOTICE E'insert to fare_conditions %s', lower(fare_condition);
		insert into bookings.ticket_flights_part (ticket_no, flight_id, fare_conditions, amount)
		select tf.ticket_no, tf.flight_id, tf.fare_conditions, tf.amount from ticket_flights tf where tf.fare_conditions = fare_condition;
	end loop;	

	ALTER TABLE bookings.boarding_passes DROP CONSTRAINT boarding_passes_ticket_no_fkey; -- удаляем ключ, который ссылался на старую таблицу и добавляем на новую
	
	DROP TABLE bookings.ticket_flights; -- удаляем старую таблицу

	ALTER TABLE bookings.ticket_flights_part RENAME TO ticket_flights; -- переименовываем новую таблицу в ticket_flights и добавляем внешние ключи
	ALTER TABLE bookings.ticket_flights ADD CONSTRAINT ticket_flights_flight_id_fkey FOREIGN KEY (flight_id) REFERENCES flights(flight_id);
	ALTER TABLE bookings.ticket_flights ADD CONSTRAINT ticket_flights_ticket_no_fkey FOREIGN KEY (ticket_no) REFERENCES tickets(ticket_no);

end;
$$ language plpgsql;
```
