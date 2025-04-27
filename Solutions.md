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


**C. Ingredient Optimisation**

For the following questions it was necessary to split the numerical values in the toppings column from our pizza_recipes table.

This was achieved using python with pandas, running the following functions;

import pandas as pd

pizza = pd.read_csv(pizza_recipes.csv')

revised_recipe = pizza.assign(toppings=pizza[‘toppings’].str.split(','))

result:

|       | pizza_id  |            toppings              |
|-------|-----------|----------------------------------|
|  0    |     1     | [1,  2,  3,  4,  5,  6,  8,  10] |
|  1    |     2     |       [4,  6,  7,  9,  11,  12]  |



revised_recipe_new =revised_recipe.explode(‘toppings’)

result:

|        | pizza_id   | toppings   |
|--------|------------|------------|
| 0     |    1   |     1 |
| 0     |    1   |     2 |
| 0     |    1   |     3 |
| 0     |    1   |     4 |
| 0     |    1   |     5 |
| 0     |    1   |     6 |
| 0     |    1   |     8 |
| 0     |    1   |    10 |
| 1     |    2   |     4 |
| 1     |    2   |     6 |
| 1     |    2   |     7 |
| 1     |    2   |     9 |
| 1     |    2   |    11 |
| 1     |    2   |    12 |

Now that we have our toppings split we can export as CSV and import into PostgreSQL.

reivised_recipe_new.to_csv(‘pizza_recipes_ver2.csv’, index=False)

Here the index column is removed as it is exported to prevent issues importing it into PostgreSQL.

Now back in PostgreSQL

```sql
CREATE TABLE pizza_recipes_revised(pizza_id int, toppings text);

COPY pizza_recipes_revised (pizza_id, toppings)
FROM 'pizza_recipe_revised.csv'
DELIMITER ','
HEADER CSV;
```

1.What are the standard ingredients for each pizza?

```sql
SELECT n.pizza_name,
       STRING_AGG(t.topping_name, ', ') as Toppings
    FROM pizza_recipes_revised p
    JOIN pizza_toppings t ON p.toppings = t.topping_id
    JOIN pizza_names n ON p.pizza_id = n.pizza_id
GROUP BY n.pizza_name
ORDER BY n.pizza_name
```

|  pizza_name    |                         toppings                                      |
|----------------|-----------------------------------------------------------------------|
| Meatlovers     | BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms, Beef |
| Vegetarian     | Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes            |
					

2.What was the most commonly added extra?

```sql
WITH trimmed AS 
   (WITH extras_unnested AS
            ( SELECT order_id, 
              UNNEST(string_to_array(extras, ',')) AS extras_unnested 
                FROM customer_orders)
                            SELECT CAST(TRIM(extras_unnested) AS int) AS trimmed
                            FROM extras_unnested) 
SELECT p.topping_name, 
           COUNT(trimmed) AS extras
    FROM trimmed AS t
    JOIN pizza_toppings AS p ON t.trimmed = p.topping_id
  GROUP BY p.topping_name
  ORDER BY extras desc
  LIMIT 1;
```

| topping_name   |  extras       |
|----------------|---------------|
|    Bacon       |       4       |

3.What was the most common exclusion?

```sql
WITH trimmed AS 
    (WITH exclusions_unnested AS
           (SELECT order_id, 
            UNNEST(string_to_array(exclusions, ',')) AS exclusions_unnested
               FROM customer_orders)
                   SELECT CAST(TRIM(exclusions_unnested) AS int) AS trimmed
                         FROM exclusions_unnested) 
SELECT p.topping_name, 
           COUNT(trimmed) AS exclusions
     FROM trimmed AS t
     JOIN pizza_toppings AS p ON t.trimmed = p.topping_id
     GROUP BY p.topping_name
     ORDER BY exclusions desc
     LIMIT 1;
```
|  topping_name |     exclusions    |
|---------------|-------------------|
|    Cheese	|           4       |


4.Generate an order item for each record in the customers_orders table in the format of one of the following:
 
 Meat Lovers
 Meat Lovers - Exclude Beef
 Meat Lovers - Extra Bacon
 Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

Two temporary tables were used form this question to split the exclusions and extras from the customer_orders table into separate row entries.

```sql
CREATE TEMPORARY TABLE split_extras AS
SELECT c.row_id, CAST(TRIM(e.extras) AS INT) AS extra_id
FROM customer_orders c 
LEFT JOIN UNNEST(string_to_array(c.extras, ',')) AS e(extras) ON true;

CREATE TEMPORARY TABLE split_exclusions AS
SELECT c.row_id, CAST(TRIM(e.exclusions) AS INT) AS exclusion_id
FROM customer_orders c 
LEFT JOIN UNNEST(string_to_array(c.exclusions, ',')) AS e(exclusions) ON true;


WITH extras_sub AS (select e.row_id, 
					'Extra ' || STRING_AGG(t.topping_name, ', ') AS pizza_mod 
                    FROM split_extras e 
					JOIN pizza_toppings t ON e.extra_id = t.topping_id
					GROUP BY e.row_id),
exclusions_sub AS (select e.row_id, 
					'Exclude ' || STRING_AGG(t.topping_name, ', ') AS pizza_mod 
                    FROM split_exclusions e 
					JOIN pizza_toppings t ON e.exclusion_id = t.topping_id
					GROUP BY e.row_id),
Union_sub AS (SELECT * FROM extras_sub UNION SELECT * FROM exclusions_sub)

SELECT c.order_id, c.customer_id, c.order_time, 
       CONCAT_WS(' - ', p.pizza_name, STRING_AGG(u.pizza_mod, ', ')) AS pizza_info
	   FROM customer_orders c 
	   LEFT JOIN Union_sub u ON c.row_id = u.row_id
	   JOIN pizza_names p ON c.pizza_id = p.pizza_id 
GROUP BY c.row_id, c.order_id, c.customer_id, c.pizza_id,
         c.order_id, p.pizza_name
ORDER BY order_id 
```					
					
|  order_id  | customer_id  |  order_time	          |        pizza_info                                             |
|------------|--------------|-----------------------------|---------------------------------------------------------------|
| 1	     |      10      |  2020-01-01 18:05:02        | Meatlovers |
| 2	     |      101	    |  2020-01-01 19:00:52        | Meatlovers |
| 3	     |      10      |  2020-01-02 23:51:23        | Meatlovers | 
| 3	     |      102	    |  2020-01-02 23:51:23        | Vegetarian |
| 4	     |      103	    |  2020-01-04 13:23:46        | Meatlovers - Exclude Cheese  |
| 4	     |      103     |  2020-01-04 13:23:46        | Vegetarian - Exclude Cheese  |
| 4	     |      103	    |  2020-01-04 13:23:46        | Meatlovers - Exclude Cheese  |
| 5	     |      104	    |  2020-01-08 21:00:29	  | Meatlovers - Extra Bacon    |
| 6	     |      101	    |  2020-01-08 21:03:13        | Vegetarian  |
| 7	     |      105	    |  2020-01-08 21:20:29        | Vegetarian - Extra Bacon  |
| 8	     |      102	    |  2020-01-09 23:54:33        | Meatlovers   |
| 9	     |      103     |  2020-01-10 11:22:59        | Meatlovers - Exclude Cheese, Extra Bacon, Chicken   |
| 10	     |      104	    |  2020-01-11 18:34:49	  | Meatlovers  |
| 10	     |      104     |  2020-01-11 18:34:49	  | Meatlovers - Extra Bacon, Cheese, Exclude BBQ Sauce, Mushrooms |


5.Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami”

```sql
WITH toppings AS (SELECT c.row_id, c.order_id, c.customer_id, 
       c.pizza_id, c.order_time, p.pizza_name,
	   CASE WHEN t.topping_id IN 
	   (SELECT ext.extra_id FROM split_extras ext  
		WHERE ext.row_id = c.row_id) THEN '2x ' || t.topping_name
		ELSE t.topping_name
		END AS toppings
		FROM customer_orders c 
		JOIN pizza_names p ON c.pizza_id = p.pizza_id
		JOIN pizza_recipes_revised AS pr on c.pizza_id = pr.pizza_id
		JOIN pizza_toppings t ON pr.toppings = t.topping_id
        LEFT JOIN split_exclusions se ON se.row_id = c.row_id 
        AND se.exclusion_id = t.topping_id
    WHERE 
        se.exclusion_id ISNULL)
SELECT t.order_id, t.customer_id, t.pizza_id, t.order_time, 
       CONCAT(t.pizza_name, ': ', STRING_AGG(t.toppings, ', 'order by t.toppings)) AS Pizza_recipe
	   FROM toppings t 
	   JOIN runner_orders r ON t.order_id = r.order_id
	   WHERE cancellation ISNULL
	   GROUP BY t.order_id, t.customer_id, t.pizza_id, t.order_time, t.row_id, t.pizza_name
	   ORDER BY t.order_id;
```

| order_id  |	 customer_id |	pizza_id  |	order_time   |                        pizza_recipe                                   |
|-----------|----------------|------------|------------------|-----------------------------------------------------------------------|
| 1  |	101  |	1   | 	2020-01-01 18:05:02  |   Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
| 2  |	101  |	1   |	2020-01-01 19:00:52  |   Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
| 3  |	102  |	2   |	2020-01-02 23:51:23  |   Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes  |
| 3  |	102  |	1   |	2020-01-02 23:51:23  |   Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
| 4  |	103  |	2   |	2020-01-04 13:23:46  |   Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes   |
| 4  |	103  |	1   |	2020-01-04 13:23:46  |   Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami  |
| 4  |	103  |	1   |	2020-01-04 13:23:46  |   Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami  |
| 5  |	104  |	1   |	2020-01-08 21:00:29  |   Meatlovers: 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
| 7  |	105  |	2   |	2020-01-08 21:20:29  |   Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes  |
| 8  |	102  |	1   |	2020-01-09 23:54:33  |   Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
| 10 |	104  |	1   |	2020-01-11 18:34:49  |   Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami  |
| 10 |	104  |	1   |	2020-01-11 18:34:49  |   Meatlovers: 2x Bacon, 2x Cheese, Beef, Chicken, Pepperoni, Salami   |

6.What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

```sql
WITH topping_count AS (SELECT c.row_id, c.order_id, c.customer_id, 
       c.pizza_id, c.order_time, p.pizza_name, t.topping_name,
	   CASE WHEN t.topping_id IN 
	   (SELECT ext.extra_id FROM split_extras ext  
		WHERE ext.row_id = c.row_id) THEN 2
		ELSE 1
		END AS toppings
		FROM customer_orders c 
		JOIN pizza_names p ON c.pizza_id = p.pizza_id
		JOIN pizza_recipes_revised AS pr on c.pizza_id = pr.pizza_id
		JOIN pizza_toppings t ON pr.toppings = t.topping_id
        LEFT JOIN split_exclusions se ON se.row_id = c.row_id 
        AND se.exclusion_id = t.topping_id
    WHERE se.exclusion_id is null)
	
	SELECT t.topping_name, SUM(t.toppings) AS total_toppings_used
	FROM topping_count t
	JOIN runner_orders r ON t.order_id = r.order_id
	WHERE r.cancellation ISNULL
	GROUP BY topping_name
	ORDER BY total_toppings_used DESC
```

| topping_name  |   total_toppings_used  |
|---------------|------------------------|
| Bacon   |     	11  |
| Mushrooms |	11  |
| Cheese |	10 |
| Pepperoni  |	9  |
| Salami  |	9  |
| Chicken  |	9  |
| Beef  |	9  |
| BBQ Sauce  |	8 | 
| Tomatoes  |	3  |
| Onions    |	3  |
| Peppers   |	3  |
|Tomato Sauce  |	3   |

**D. Pricing and Ratings**

1.If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```sql
SELECT SUM(pizza_price) AS "total_sales_$" FROM
	                  (SELECT c.order_id, c.pizza_id,
                      CASE WHEN c.pizza_id = 1 THEN 12
	                  ELSE 10
		              END AS pizza_price
					FROM customer_orders c
					  JOIN runner_orders r ON r.order_id = c.order_id
		            WHERE cancellation ISNULL)
```	   

| total_sales_$ |
|---------------|
|      138   |

2.What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra

```sql
CREATE TEMPORARY TABLE split_extras AS
SELECT c.row_id, CAST(TRIM(e.extras) AS INT) AS extra_id
FROM customer_orders c 
LEFT JOIN UNNEST(STRING_TO_ARRAY(c.extras, ',')) AS e(extras) ON true;

WITH total_pizzas AS (SELECT SUM(pizza_price) AS total_pizzas FROM
	                  (SELECT c.order_id, c.pizza_id,
                      CASE WHEN c.pizza_id = 1 THEN 12
	                  ELSE 10
		              END AS pizza_price
					FROM customer_orders c
					  JOIN runner_orders r ON r.order_id = c.order_id
		            WHERE cancellation ISNULL)),
		   
total_extras AS (SELECT SUM(extra_cost) AS total_extras FROM
	                  (SELECT CASE WHEN e.extra_id = 4 THEN 2
					   WHEN e.extra_id ISNULL THEN 0
	                   ELSE 1
		               END AS extra_cost
					FROM split_extras e
					   JOIN customer_orders c ON c.row_id = e.row_id
					 JOIN runner_orders r ON r.order_id = c.order_id
		            WHERE r.cancellation ISNULL))
					
       SELECT total_pizzas+total_extras AS total_sales
               FROM total_pizzas, total_extras 

```

|  total_sales  |
|---------------|
|     143      |


3.The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

```sql

CREATE TABLE runner_ratings 
(order_id INT, rating INT, check (rating between 1 AND 5))

							 
INSERT INTO runner_ratings (order_id, rating)
values
  (1,4),(2,2),(3,4),(4,2),(5,2),(7,1),(8,5),(10,3);

SELECT * FROM runner_ratings;

```

|  order_id  |   rating  |
|------------|-----------|
| 1	|          4  |
| 2	|            2 |
| 3	|            4 |
| 4	|           2 |
| 5	|            2 |
| 7	|            1 |
| 8	|           5 |
| 10	|           3 |

							 
4.Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas

```sql
WITH average_speed_ms AS (SELECT order_id, 
CEIL((distance_km*1000)/(duration_mins*60)) AS average_speed_ms
FROM runner_orders 
WHERE cancellation ISNULL
ORDER BY runner_id)

SELECT c.customer_id, c.order_id, r.runner_id,
        s.rating, c.order_time, r.pickup_time, (r.pickup_time-c.order_time)AS prep_time,
		r.duration_mins, a.average_speed_ms, count(c.pizza_id)AS total_pizza
		FROM customer_orders c 
		JOIN runner_orders r ON r.order_id = c.order_id 
		JOIN runner_ratings s ON s.order_id = c.order_id
		JOIN average_speed_ms a ON a.order_id = c.order_id
		GROUP BY c.order_id, c.customer_id, 
		r.runner_id, s.rating, c.order_time, r.pickup_time,
		r.duration_mins, a.average_speed_ms;
```

| customer_id |	order_id |  runner_id | rating | order_time | pickup_time | prep_time | duration_mins | average_speed_ms | total_pizza |
|-------------|----------|------------|--------|------------|-------------|-----------|---------------|------------------|-------------|
| 101 |	1  |	1 |	4 |	2020-01-01 18:05:02 |	2020-01-01 18:15:34 |	00:10:32 |	32 |	11 |	1 |
| 101 |	2  |	1 |	2 |	2020-01-01 19:00:52 |	2020-01-01 19:10:54 |	00:10:02 |	27 |	13 |	1 |
| 102 |	3  |	1 |	4 |	2020-01-02 23:51:23 |	2020-01-03 00:12:37 |	00:21:14 |	20 | 	12 |	2 | 
| 103 |	4  |	2 |	2 |	2020-01-04 13:23:46 |	2020-01-04 13:53:03 |	00:29:17 |	40 |	10 | 	3 |
| 104 |	5  |	3 |	2 |	2020-01-08 21:00:29 |	2020-01-08 21:10:57 |	00:10:28 |	15 |	12 |	1 |
| 105 |	7 |	2 |	1 |     2020-01-08 21:20:29 |	2020-01-08 21:30:45 |	00:10:16 |	25 |	17 |	1 |
| 102 |	8  |	2 |	5 |	 2020-01-09 23:54:33 |	2020-01-10 00:15:02 | 	00:20:29 |	15 | 	26 |	1 |
| 104 |	10 |	1 |	3 |	2020-01-11 18:34:49 |  2020-01-11 18:50:20 |	00:15:31 |	10 |	17 |	2 |




5.If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

```sql
CREATE TEMPORARY TABLE runner_fees 
AS select runner_id, order_id,
(distance_km*0.3) AS runner_fee 
FROM runner_orders

WITH total_sales AS (select sum(pizza_price) AS total_sales FROM
                       (SELECT c.order_id, c.pizza_id,
                         CASE WHEN c.pizza_id = 1 THEN 12
	                          ELSE 10
		                      END AS pizza_price
					     FROM customer_orders c
				        JOIN runner_orders r ON r.order_id = c.order_id
				          WHERE cancellation ISNULL) AS prices),
		
	total_fees AS (SELECT SUM(runner_fee) AS total_fees FROM runner_fees)

				 SELECT (total_sales-total_fees) AS Total_profit
				 FROM total_sales, total_fees
```

| total_profit |
|--------------|
|   94.44  |

E. Bonus Questions

If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

Adding extra pizzas to the current range would not massively impact the existing data design, with only the pizza_names and pizza_recipes tables needing extra information added.

```sql
IMSERT INTO pizza_names (pizza_id, pizza_name)
VALUES (3, ‘Supreme')

SELECT * FROM pizza_names;

```

| pizza_id   |   pizza_name  |
|------------|---------------|
| 1	|        Meatlovers  |
| 2	|        Vegetarian |
| 3	|        Supreme  |

```sql
INSERT INTO pizza_recipes (pizza_id, toppings)
VALUES (3,’1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12')

SELECT * FROM pizza_recipes;

```	

| pizza_id  |    	toppings       |
|-----------|--------------------------|
|  1	 | 1, 2, 3, 4, 5, 6, 8, 10  |
| 2	 | 4, 6, 7, 9, 11, 12 |
| 3	| 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12  |

Also for any further queries on toppings in the future, the revised table is also updated.	

```sql
INSERT INTO pizza_recipes_revised (pizza_id, toppings)
VALUES (3,1),(3,2),(3,3),(3,4),(3,5),(3,6),
       (3,7),(3,8),(3,9),(3,10),(3,11),(3,12);

SELECT * FROM pizza_recipes_revised;

```


| pizza_id  |	toppings  |
|-----------|-------------|
| 1 	|              1 |
| 1	 |              2 |
| 1	 |              3 |
| 1	 |             4 |
| 1 	 |             5 |
| 1	 |              6 |
| 1	 |              8 |
| 1      |            10 |
| 2	 |             4 |
| 2	 |             6 |
| 2	 |           7 |
| 2	 |           9 |
| 2	 |          11 | 
| 2	 |          12 |
| 3	 |           1 |
| 3	 |           2 |
| 3	 |           3 |
| 3	 |           4 |
| 3	 |           5 |
| 3	 |            6 |
| 3	 |            7 |
| 3	 |            8 |
| 3	 |            9 |
| 3	 |           10 |
| 3	 |           11 |
| 3	 |           12 |






				



