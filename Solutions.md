**Data Cleaning**

```sql
ALTER TABLE customer_orders 
ALTER COLUMN exclusions TYPE VARCHAR(4) 
     using NULLIF(exclusions, '')::VARCHAR(4);

ALTER TABLE customer_orders 
ALTER COLUMN exclusions TYPE VARCHAR(4) 
      using NULLIF(exclusions, 'NULL')::VARCHAR(4);

ALTER TABLE customer_orders 
ALTER COLUMN extras TYPE VARCHAR(4) 
      using NULLIF(extras, '')::VARCHAR(4);

ALTER TABLE customer_orders 
ALTER COLUMN extras TYPE VARCHAR(4) 
      using NULLIF(extras, 'NULL')::VARCHAR(4);

ALTER TABLE customer_orders
ADD COLUMN Row_id serial primary key

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN pickup_time TYPE VARCHAR(19) 
      using NULLIF(pickup_time, 'NULL')::VARCHAR(19),
ALTER COLUMN distance TYPE VARCHAR(7)
      using NULLIF(distance, 'NULL')::VARCHAR(7),
ALTER COLUMN duration TYPE VARCHAR(10) 
      using NULLIF(duration, 'NULL')::VARCHAR(10);

ALTER TABLE runner_orders 
ALTER COLUMN cancellation TYPE VARCHAR(23) 
using case  
         WHEN cancellation = ‘NULL’ OR cancellation = 'Nan' THEN NULL
       ELSE NULLIF(cancellation, '') 
       END::VARCHAR(23);

ALTER TABLE runner_orders
RENAME COLUMN distance to distance_km

ALTER TABLE runner_orders
RENAME COLUMN duration to duration_mins

UPDATE runner_orders
SET duration_mins = REPLACE(REPLACE(REPLACE(duration_mins, 'mins', ''), 'minutes', ''), 'minute', '')


UPDATE runner_orders
SET distance_km = REPLACE(distance_km, 'km', '')

UPDATE runner_orders
SET duration_mins = TRIM(duration_mins), distance_km = TRIM(distance_km)


ALTER TABLE runner_orders
ALTER COLUMN duration_mins TYPE INT
using duration_mins::INT

ALTER TABLE runner_orders
ALTER COLUMN distance_km TYPE FLOAT
using distance_km::FLOAT

```
**A. Pizza Metrics**

How many pizzas were ordered?

```SQL
SELECT COUNT(pizza_id) AS total_pizza_order 
FROM customer_orders;
```

| total_pizza_ordered |
|---------------------|
|       14            |



How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT(order_id)) AS total_orders 
       FROM customer_orders 
```
| total_orders |
|--------------|
|  10  |



How many successful orders were delivered by each runner?

```sql
SELECT runner_id, count(order_id)
FROM runner_orders
WHERE cancellation ISNULL 
GROUP BY runner_id
ORDER BY runner_id;
```

| runner_id | total_orders |
|-----------|--------------|
| 1	 |            4  |
| 2	 |           3   |
| 3	 |           1   |




How many of each type of pizza was delivered?

```sql
SELECT p.pizza_name, COUNT(c.pizza_id) AS Pizzas_sold
					FROM customer_orders c
					JOIN runner_orders r ON c.order_id = r.order_id
					JOIN pizza_names p ON c.pizza_id = p.pizza_id
					WHERE r.cancellation isnull
                    GROUP BY p.pizza_name;

```
| pizza_name | pizzas_sold |
|------------|-------------|					
| Meatlovers |	9 |
| Vegetarian |	3  |



How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT c.customer_id, COUNT(CASE WHEN p.pizza_name = 'Meatlovers' THEN 1 
						  ELSE NULL END) AS Meatlovers,
					COUNT(CASE WHEN p.pizza_name = 'Vegetarian' THEN 1 
						 ELSE NULL END) AS Vegetarian
						 
				   FROM customer_orders c
					JOIN runner_orders r ON c.order_id = r.order_id
					JOIN pizza_names p ON c.pizza_id = p.pizza_id
					WHERE r.cancellation isnull
					GROUP BY c.customer_id
					ORDER BY customer_id; 

```
| customer_id  |  meatlovers  | vegetarian |
|--------------|--------------|------------|
| 101	|              2	|                     0 |
| 102	|              2	|                     1 |
| 103	|              2	|                     1 |
| 104	|              3	|                     0 |
| 105	|              0	|                     1 |


					
What was the maximum number of pizzas delivered in a single order?

```sql
SELECT c.order_id, COUNT(c.pizza_id) AS Max_pizzas_del  
				   FROM customer_orders c
					JOIN runner_orders r ON c.order_id = r.order_id
				
			    WHERE r.cancellation isnull
					GROUP BY c.order_id
					ORDER BY max_pizzas_del DESC
					LIMIT 1;
```
					
| order_id |	max_pizzas_del |
| 4	  |                 3  |

For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT c.customer_id, COUNT(CASE WHEN c.extras NOTNULL OR c.exclusions notnull THEN 1 
						  ELSE NULL END) as Pizza_changes,
					COUNT(CASE WHEN c.extras ISNULL AND c.exclusions ISNULL THEN 1 
						  ELSE NULL END) AS No_changes   
		 FROM customer_orders c
		 JOIN runner_orders r ON c.order_id = r.order_id
		 WHERE r.cancellation ISNULL
		 GROUP BY c.customer_id
		 ORDER BY c.customer_id; 
		
```
| customer_id | pizza_changes  |  no_changes  |
|-------------|----------------|--------------|
| 101 |	              0 |	                            2 |
| 102 |	              0 |	                            3 |
| 103 |	              3 |	                            0 |
| 104 |	              1 |	                            2 |
| 105 |	              0 |	                            1 |



How many pizzas were delivered that had both exclusions and extras?

```sql
WITH extra_exclus AS (SELECT COUNT(CASE WHEN c.extras NOTNULL AND c.exclusions NOTNULL THEN 1 
						  ELSE NULL END) AS Both_extras_and_exclusions
					  FROM customer_orders c
		 JOIN runner_orders r ON c.order_id = r.order_id
		 WHERE r.cancellation ISNULL 
          GROUP BY c.order_id)
		  SELECT SUM(Both_extras_and_exclusions) AS Both_extras_and_exclusions
		  FROM extra_exclus;
```

| both_extras_and_exclusions |
|----------------------------|
  |    1  |
 

What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT COUNT(pizza_id) AS pizzas_ordered, 
       DATE_PART('hour',order_time) AS hour
FROM customer_orders
GROUP BY hour 
ORDER BY hour;

```

| pizzas_ordered | hour |
|----------------|------|
    |   1	      |                 11  |
     |  3	       |               13 |
     |  3	       |                18 | 
     |  1	       |                19 |
     |  3	       |                21 |
     |  3	       |                23 |



What was the volume of orders for each day of the week?

```sql
SELECT COUNT(pizza_id) AS pizzas_ordered, 
       TO_CHAR(order_time, 'day') AS day
FROM customer_orders
GROUP BY day 
ORDER BY day;
```
		
| pizzas_ordered |   day |
|----------------|-------|
| 1	  |               friday   |
| 5	  |             saturday |
| 3	  |             thursday |
| 5	  |          wednesday| 

