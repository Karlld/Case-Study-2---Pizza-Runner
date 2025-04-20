**Data Cleaning**

```sql
alter table customer_orders 
alter column exclusions type varchar(4) 
     using nullif(exclusions, '')::varchar(4);

alter table customer_orders 
alter column exclusions type varchar(4) 
      using nullif(exclusions, 'null')::varchar(4);

alter table customer_orders 
alter column extras type varchar(4) 
      using nullif(extras, '')::varchar(4);

alter table customer_orders 
alter column extras type varchar(4) 
      using nullif(extras, 'null')::varchar(4);

alter table customer_orders
add column Row_id serial primary key

alter table pizza_runner.runner_orders
alter column pickup_time type varchar(19) 
      using nullif(pickup_time, 'null')::varchar(19),
alter column distance type varchar(7)
      using nullif(distance, 'null')::varchar(7),
alter column duration type varchar(10) 
      using nullif(duration, 'null')::varchar(10);

alter table runner_orders 
alter column cancellation type varchar(23) 
using case  
         when cancellation = ‘null’ or cancellation = 'Nan' then null
       else nullif(cancellation, '') 
       end::varchar(23);

alter table runner_orders
rename column distance to distance_km

alter table runner_orders
rename column duration to duration_mins

update runner_orders
SET duration_mins = replace(replace(replace(duration_mins, 'mins', ''), 'minutes', ''), 'minute', '')


update runner_orders
set distance_km = replace(distance_km, 'km', '')

update runner_orders
set duration_mins = trim(duration_mins), distance_km = trim(distance_km)


alter table runner_orders
alter column duration_mins type int
using duration_mins::int

alter table runner_orders
alter column distance_km type float
using distance_km::float

```
