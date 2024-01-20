Напишем функцию для расчёта суммы денег за проданные товары и обновления представления `good_sum_mart`:
```sql
CREATE OR REPLACE FUNCTION compute_sales()
RETURNS trigger
AS
$$
DECLARE
    data_row record;
   	new_good_sum_mart_record record;
begin
	
	CASE TG_OP
	    WHEN 'DELETE'
	        THEN data_row = OLD;
	    WHEN 'UPDATE'
	        THEN data_row = NEW; -- здесь исхожу из того, что можно лишь апдейтнуть количество проданного товара, но не саму ссылку на товар
	    WHEN 'INSERT'
	        THEN data_row = NEW;
	END CASE;
      
   	SELECT G.good_name as good_name, coalesce(sum(G.good_price * S.sales_qty), 0) as sum_sale into new_good_sum_mart_record
		FROM pract_functions.goods G
		left JOIN pract_functions.sales S ON S.good_id = G.goods_id
		where G.goods_id = data_row.good_id
		GROUP BY G.good_name;
   	
	if exists (select 1 from pract_functions.good_sum_mart where good_name = new_good_sum_mart_record.good_name) then
		if (new_good_sum_mart_record.sum_sale = 0) then 
			delete from pract_functions.good_sum_mart where good_name = new_good_sum_mart_record.good_name;
		else
			update pract_functions.good_sum_mart set sum_sale = new_good_sum_mart_record.sum_sale where good_name = new_good_sum_mart_record.good_name;
		end if;
	else
		insert into pract_functions.good_sum_mart (good_name, sum_sale) values (new_good_sum_mart_record.good_name, new_good_sum_mart_record.sum_sale);
	end if;

  RETURN data_row;
END;
$$ LANGUAGE plpgsql;
```
Добавим тригер на после вставки, апдейта и удаления для каждой строки: 
```sql
CREATE TRIGGER trg_after_row_change_sales
AFTER INSERT OR UPDATE OR DELETE
ON pract_functions.sales
FOR EACH ROW
EXECUTE FUNCTION compute_sales();
```
Примеры

Каталог товаров (`goods`):
| goods_id	| good_name 	| good_price	|
| ------------- | ------------- | ------------- |
| 1		| Спички хозайственные | 0.50	|	
| 2		| Автомобиль Ferrari FXX K	| 185000000.01	|

Добавим в `sales` следующие строки:
```sql
INSERT INTO pract_functions.sales (good_id,sales_time,sales_qty) VALUES
	 (1,'2024-01-09 20:50:32.683388+03',2),
	 (1,'2024-01-10 21:50:32.683388+03',10),
	 (2,'2024-01-09 20:50:32.683+03',2);
```

Состояние `good_sum_mart`:
| good_name	| sum_sale 	|
| ------------- | ------------- |
| Спички хозайственные	| 6.00	|
| Автомобиль Ferrari FXX K	| 370000000.02	|

Изменим количество товаров в одной из строк в `sales`:
```sql
update pract_functions.sales set sales_qty = 3 where sales_id = 2; -- строка (1,'2024-01-10 21:50:32.683388+03',10)
```

Состояние `good_sum_mart`:
| good_name	| sum_sale 	|
| ------------- | ------------- |
| Спички хозайственные	| 2.5	|
| Автомобиль Ferrari FXX K	| 370000000.02	|

Удалим строки из `sales`:
```sql
delete from pract_functions.sales where sales_id = 1; -- строка (1,'2024-01-09 20:50:32.683388+03',2)
delete from pract_functions.sales where sales_id = 3; -- строка (2,'2024-01-09 20:50:32.683+03',2)
```

Состояние `good_sum_mart`:
| good_name	| sum_sale 	|
| ------------- | ------------- |
| Спички хозайственные	| 1.5	|
