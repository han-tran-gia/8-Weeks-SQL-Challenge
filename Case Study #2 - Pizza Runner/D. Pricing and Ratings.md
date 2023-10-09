# 🍕 Case Study #2 Pizza Runner

## Solution - C. Ingredient Optimisation

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

```
SELECT SUM(CASE
        		WHEN pizza_id = 1 THEN 12
             	ELSE 10 END) AS revenue
FROM orders
```
#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/2e9cb0fc-ee97-40d1-9def-e4c7deba389d)

***

### 2. What if there was an additional $1 charge for any pizza extras?
```
CREATE TEMP TABLE extras_table AS
SELECT order_id
        ,customer_id
        ,pizza_id
        ,exclusions 
        ,UNNEST(STRING_TO_ARRAY(extras, ',')) AS extras
        ,order_time
FROM customer_orders_temp;

DELETE FROM customer_orders_temp
WHERE extras != '';

INSERT INTO customer_orders_temp
SELECT *
FROM extras_table;

SELECT (SUM(CASE
                WHEN pizza_id = 1 THEN 12
                ELSE 10 END)
        +
        SUM(CASE 
                WHEN extras IS NOT NULL THEN 1
                ELSE 0 END)) AS revenue
FROM customer_orders_temp
LEFT JOIN runner_orders_temp
        USING (order_id)
LEFT JOIN pizza_runner.pizza_names
 	USING (pizza_id)
WHERE distance != ''
```

#### Explanation:
- First, I created a temporary table called `extras_table` and use **UNNEST** and **STRING_TO_ARRAY** to split the extras into rows·
- Then, in `customer_orders_temp` table, I deleted rows which have info about extras and insert the new extra data that has been splited into multiple rows from `extras_table`
- In the main query, 2 **CASE WHEN** were executed to calculate the amount of money that Pizza Runner made:
        - 1st **CASE WHEN**: if pizza_id is 1 then the price will be 12, if not, the price will be 10, then calculate the sum.
        - 2nd **CASE WHEN**: if extras column is not blank, the price for extra toppings would be 1, if not, no additional payment required, then calculate the sum.

##### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/4283ff91-3840-458d-b4dd-19146790c648)

- Pizza Runner made $163 if each extra cost $1

***

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```
CREATE SCHEMA IF NOT EXISTS pizza_runner;

DROP TABLE IF EXISTS runners_rating;
CREATE TABLE pizza_runner.runners_rating
  ("order_id" INTEGER,
  "rating" INTEGER);
  
INSERT INTO pizza_runner.runners_rating
	("order_id", "rating")
VALUES
('1', '3'),
    ('2', '5'),
    ('3', '2'),
    ('4', '1'),
    ('5', '3'),
    ('7', '5'),
    ('8', '5'),
    ('10', '5');
```

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/25b6b3b4-73c7-42fe-b8c7-40eedf160e62)

***

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
```
SELECT customer_id, order_id, runner_id, rating, order_time, pickup_time
        ,EXTRACT(MIN FROM (pickup_time - order_time)) AS pickup_order_mins
        ,duration
        ,ROUND((distance/duration * 60),2) AS avg_speed
        ,COUNT(pizza_id) AS total_pizza_ordered
FROM orders
LEFT JOIN pizza_runner.runners_rating
        USING (order_id)
GROUP BY customer_id, order_id, runner_id, rating, order_time, pickup_time, duration, distance
ORDER BY order_id
```


