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

CREATE TRIGGER trg_after_row_change_sales
AFTER INSERT OR UPDATE OR DELETE
ON pract_functions.sales
FOR EACH ROW
EXECUTE FUNCTION compute_sales();
```
