**Data Cleaning**

```sql
ALTER TABLE customer_orders 
ALTER COLUMN exclusions TYPE VARCHAR(4) 
     using NULLIF(exclusions, '')::VARCHAR(4);

ALTER TABLE customer_orders 
ALTER COLUMN exclusions TYPE VARCHAR(4) 
      using NULLIF(exclusions, 'null')::VARCHAR(4);

ALTER TABLE customer_orders 
ALTER COLUMN extras TYPE VARCHAR(4) 
      using NULLIF(extras, '')::VARCHAR(4);

ALTER TABLE customer_orders 
ALTER COLUMN extras TYPE VARCHAR(4) 
      using NULLIF(extras, 'null')::VARCHAR(4);

ALTER TABLE customer_orders
ADD COLUMN Row_id serial primary key

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN pickup_time TYPE VARCHAR(19) 
      using NULLIF(pickup_time, 'null')::VARCHAR(19),
ALTER COLUMN distance TYPE VARCHAR(7)
      using NULLIF(distance, 'null')::VARCHAR(7),
ALTER COLUMN duration TYPE VARCHAR(10) 
      using NULLIF(duration, 'null')::VARCHAR(10);

ALTER TABLE runner_orders 
ALTER COLUMN cancellation TYPE VARCHAR(23) 
using case  
         WHEN cancellation = ‘null’ or cancellation = 'Nan' THEN null
       ELSE NULLIF(cancellation, '') 
       END::VARCHAR(23);

ALTER TABLE runner_orders
RENAME COLUMN distance to distance_km

ALTER TABLE runner_orders
RENAME COLUMN duration to duration_mins

UPDATE runner_orders
SET duration_mins = REPLACE(REPLACE(REPLACE(duration_mins, 'mins', ''), 'minutes', ''), 'minute', '')


UPDATE runner_orders
set distance_km = REPLACE(distance_km, 'km', '')

UPDATE runner_orders
set duration_mins = TRIM(duration_mins), distance_km = TRIM(distance_km)


ALTER TABLE runner_orders
ALTER COLUMN duration_mins TYPE INT
using duration_mins::INT

ALTER TABLE runner_orders
ALTER COLUMN distance_km TYPE FLOAT
using distance_km::FLOAT

```
