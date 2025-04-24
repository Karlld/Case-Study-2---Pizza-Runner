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
|----------|-------------------|
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

| pizzas_ordered | hour     |
|----------------|----------|
|   1 |             11  |
|  3	       |        13 |
|  3	  |             18 | 
|  1	    |           19 |
|  3	    |           21 |
|  3	    |           23 |



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


**B. Runner and Customer Experience**

How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
WITH week_reg AS (SELECT runner_id, 
				  CASE WHEN registration_date BETWEEN '2021-01-01' AND '2021-01-07' THEN 'week 1'
                       WHEN registration_date BETWEEN '2021-01-08' AND '2021-01-14' THEN 'week 2'
			 ELSE 'week 3' 
			 END AS week_registered
			 FROM runners
			 GROUP BY week_registered, runner_id)
      SELECT week_registered, COUNT(week_registered) AS total_registered
		      FROM week_reg
			 	 GROUP BY week_registered
				 ORDER BY week_registered;

```

| week_registered |	total_registered |
|-----------------|----------------------|
| week 1  |	                 2 |
| week 2  |	                 1 |
| week 3  |	                 1 |

What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
WITH journey_time AS (SELECT r.runner_id, 
	                      (r.pickup_time - order_time) AS journey_time
                                   FROM runner_orders AS r
                                   JOIN customer_orders AS c ON r.order_id = c.order_id)
SELECT runner_id, AVG(journey_time) AS average_delivery_time
FROM journey_time
GROUP BY runner_id  
ORDER BY runner_id;

```

|  runner_id  |  average_delivery_time |
|-------------|------------------------|
| 1	       |       00:15:40.666667 |
| 2	       |       00:23:43.2 |
| 3	       |       00:10:28 |



Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
SELECT number_of_pizzas, AVG(preparation_time) AS Average_prep_time_mins FROM 
         (SELECT c.order_id, 
		  COUNT(c.pizza_id) AS number_of_pizzas, 
		  (r.pickup_time-c.order_time) AS preparation_time
						FROM customer_orders c
						 JOIN runner_orders r ON r.order_id = c.order_id
						  WHERE r.cancellation ISNULL
						 GROUP BY c.order_id,r.pickup_time,c.order_time
						 ORDER BY c.order_id) 
		  GROUP BY number_of_pizzas;

```
		  
| number_of_pizzas   |	average_prep_time_mins  |
|--------------------|--------------------------|
 |    3	             |               00:29:17  |
 |    2	             |               00:18:22.5 |
 |    1	             |               00:12:21.4 |
 

It can be seen that there is relationship between the number of pizzas and how long it takes to prepare.




What was the average distance travelled for each customer?

```sql

SELECT c.customer_id,
       CEIL(AVG(r.distance_km)) AS Average_distance
          FROM customer_orders c 
          JOIN runner_orders r ON c.order_id = r.order_id
WHERE r.cancellation ISNULL						   
GROUP BY c.customer_id
ORDER BY customer_id;

```

| customer_id  | average_distance |
|--------------|------------------|
| 101	   |               20  |
| 102	    |              17  |
| 103 	   |               24  |
| 104	   |               10 |
| 105	   |               25 |

What was the difference between the longest and shortest delivery times for all orders?

```sql
SELECT (MAX(duration_mins)-MIN(duration_mins)) AS Duration_differance 
       FROM runner_orders;

```

| duration_differance |
|---------------------|
|       30  |

What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
SELECT runner_id, 
CEIL((distance_km*1000)/(duration_mins*60)) AS "average_speed_m/s"
FROM runner_orders
WHERE cancellation ISNULL
ORDER BY runner_id;
```

|  runner_id   |   average_speed_m/s  |
|--------------|----------------------|
|    1	       |            11   |
|    1	        |           13   |
|    1	       |            12   |
|    1	       |            17   |
|    2	       |            17   |
|    2	       |            26   |
|    2	       |            10   |
|    3	       |            12   |


Runner 1 has the most reliable average speed, varying between 11m/s and 17m/s.
Runner 2 has a large difference in their average speeds, varying between 10m/s and 26m/s.
Runner 3 has only one successful delivery, so we cannot identify any trends.



What is the successful delivery percentage for each runner?

```sql
SELECT runner_id, 
       COUNT(duration_mins)*100/COUNT(order_id) AS Percentage_delivered 
FROM runner_orders
GROUP BY runner_id
ORDER BY runner_id;

```

|  runner_id  |  percentage_delivered  |
|-------------|------------------------|
|   1	  |               100  |
|   2	  |                75 | 
|   3	  |                50 |


