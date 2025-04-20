**Data Cleaning**

```sql
ALTER TABLE customer_orders 
ALTER COLUMN exclusions TYPE varchar(4) 
     using NULLIF(exclusions, '')::varchar(4);

ALTER TABLE customer_orders 
ALTER COLUMN exclusions TYPE varchar(4) 
      using NULLIF(exclusions, 'null')::varchar(4);

ALTER TABLE customer_orders 
ALTER COLUMN extras TYPE varchar(4) 
      using NULLIF(extras, '')::varchar(4);

ALTER TABLE customer_orders 
ALTER COLUMN extras TYPE varchar(4) 
      using NULLIF(extras, 'null')::varchar(4);

ALTER TABLE customer_orders
add column Row_id serial primary key

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN pickup_time TYPE varchar(19) 
      using NULLIF(pickup_time, 'null')::varchar(19),
ALTER COLUMN distance TYPE varchar(7)
      using NULLIF(distance, 'null')::varchar(7),
ALTER COLUMN duration TYPE varchar(10) 
      using NULLIF(duration, 'null')::varchar(10);

ALTER TABLE runner_orders 
ALTER COLUMN cancellation TYPE varchar(23) 
using case  
         when cancellation = ‘null’ or cancellation = 'Nan' then null
       else NULLIF(cancellation, '') 
       end::varchar(23);

ALTER TABLE runner_orders
rename column distance to distance_km

ALTER TABLE runner_orders
rename column duration to duration_mins

update runner_orders
SET duration_mins = REPLACE(REPLACE(REPLACE(duration_mins, 'mins', ''), 'minutes', ''), 'minute', '')


update runner_orders
set distance_km = REPLACE(distance_km, 'km', '')

update runner_orders
set duration_mins = trim(duration_mins), distance_km = trim(distance_km)


ALTER TABLE runner_orders
ALTER COLUMN duration_mins TYPE int
using duration_mins::int

ALTER TABLE runner_orders
ALTER COLUMN distance_km TYPE float
using distance_km::float

```
