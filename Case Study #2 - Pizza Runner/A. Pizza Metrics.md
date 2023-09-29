# 🍕 Case Study #2 - Pizza Runner

## 🍝 Solution - A. Pizza Metrics

### 1. How many pizzas were ordered?
```
SELECT COUNT(order_id) as pizza_ordered
FROM customer_orders_temp
```

#### Answer:
<img width="111" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/304f67a6-a3b1-4346-8ffb-6e51dc9e4d64">

There are 14 pizzas that were ordered

***

### 2. How many unique customer orders were made?
```
SELECT COUNT(DISTINCT order_id) as pizza_ordered
FROM customer_orders_temp
```

#### Explanation:
- The question is asking `unique customer order` which means if a customer ordered 2 pizzas in 1 order and the table is showing 2 records with the same order_id, we just count it as 1

#### Answer:
<img width="110" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/975e9b1a-de44-4b24-a53b-550261f17b05">

There are 10 unique customer orders.

***

### 3. How many successful orders were delivered by each runner?
```
SELECT runner_id
		,COUNT (pickup_time) AS successful_order_count
FROM runner_orders_temp
WHERE pickup_time != ''
GROUP BY runner_id
ORDER BY runner_id
```

#### Explanation:
- From the runner_orders_table: 2 records which are cancelled didn't have any information about pickup_time, duration and distance -> Using **WHERE** clause with criteria `pickup_time != '' to exclude these cancelled orders, the remaining records are successful delivered orders.

#### Answer:
<img width="427" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/3b686e18-88ba-4b9c-8283-c8626fc5e16c">

- Runner 1: has 4 successful delivered orders
- Runner 2: has 3 successful delivered orders
- Runner 3: has 1 successful delivered order

***

### 4. How many of each type of pizza was delivered?
```
SELECT pizza_name AS pizza
		,COUNT(order_id) AS pizza_count
FROM orders
JOIN pizza_runner.pizza_names
  USING (pizza_id)
GROUP BY pizza_name
ORDER BY pizza
```

#### Explanation:
- The question is asking how many pizza was delivered which means we need to exclude the cancelled order because they haven't delivered yet -> I will use orders table instead of runner_orders_temp table
- **JOIN**: To get the pizza names, I will join orders table with pizza_names table

#### Answer:
<img width="479" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/6ddfe1d1-e7a9-47d3-84af-d76aa2761a09">


- Meatlovers: 9 delivered pizzas
- Vegetarian: 3 delivered pizzas

***

### 5. How many Vegetarian and Meatlovers were ordered by each customer?**
```
,pizza_info AS
	(SELECT *
	FROM customer_orders_temp
	LEFT JOIN pizza_runner.pizza_names
		USING (pizza_id))

SELECT customer_id AS customer
		,pizza_name
		,COUNT (pizza_name) AS pizza_count
FROM pizza_info
GROUP BY customer_id, pizza_name
ORDER BY customer, pizza_name
```

#### Explanation:
- The question is asking the number of pizzas were ordered and didn't mention about delivery status, therefore, I will include both cancelled order and using `customer_orders_temp` table

#### Answer:
<img width="680" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/251af9bd-d5ba-49b1-ab60-2dab1be5d5f1">

- Customer 101 ordered 2 Meatlovers pizzas.
- Customer 102 ordered 2 Meatlovers pizzas and 2 Vegetarian pizzas.
- Customer 103 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 104 ordered 3 Meatlovers pizzas.
- Customer 105 ordered 1 Vegetarian pizza.

***

### 6. What was the maximum number of pizzas delivered in a single order?
```
SELECT order_id AS order
		,COUNT(pizza_id) AS max_pizza
FROM orders
GROUP BY order_id
ORDER BY max_pizza DESC
LIMIT 1
```

#### Answer:
<img width="469" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/b6032890-af22-4913-9111-333428d40aaa">

- Maximum number of pizza delivered in a single order is 3 pizzas.

***

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```
SELECT customer_id AS customer
		,SUM(CASE
        			WHEN exclusions = '' AND extras = '' THEN 1
            		ELSE 0 END) 
         AS no_change_pizza
        ,SUM(CASE
        			WHEN exclusions != '' OR extras != '' THEN 1
            		ELSE 0 END)
         AS at_least_1change
FROM orders
GROUP BY customer_id
ORDER BY customer_id
```
#### Explanation:
- For `delivered pizza` -> I will use orders table which has been exclude cancelled orders
- For `at least 1 change`: which means we need to count both situations: customers add extras or exclude any toppings from the pizza
- **CASE WHEN**: I used 2 CASE WHEN to calculate the number of no change and at least 1 change pizza
  - If `exclusions` and `extras` columns are blank, which means no toppings are added or excluded from the pizza, 1 will be added and later sum all of that to calculate no_change_pizza count
  - If these columns are NOT blank, which means at least 1 topping will be added or excluded from the pizza, 1 will be added and later sum all of that to calculate at_least_1change count

#### Answer:
<img width="703" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/7a5de81e-a148-41ce-a4d3-e4cf5ba8c201">

There are 6 delivered pizzas had at least 1 change and 6 delivered pizzas had no change:
  - Customer 101 and 102: there are 2 and 3 no change pizzas were delivered to these 2 customers, respectively
  - Customer 103 and 105: there are 3 and 1 delivered pizzas that have at least 1 change, respectively
  - Customer 104: received 1 no change pizza and 2 pizzas that have at least 1 change

***

### 8. How many pizzas were delivered that had both exclusions and extras?
```
SELECT SUM(CASE
        		WHEN exclusions != '' AND extras != '' THEN 1
            ELSE 0 END) 
         AS both_excl_extra_pizza_count
FROM orders
```
#### Explanation:
- The question is asking the number of pizza were delivered so I will use orders table which has been excluded cancelled orders

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/25fbbe1c-09d5-4124-9e6c-80ac18efacc7)

There is only 1 pizza was delivered that had both exclusions and extras

***

### 9. What was the total volume of pizzas ordered for each hour of the day?
```
SELECT EXTRACT(HOUR FROM order_time) AS hour_time
	,COUNT(pizza_id) AS pizza_count
FROM customer_orders_temp
GROUP BY EXTRACT(HOUR FROM order_time)
ORDER BY hour_time
```

#### Explanation:
- The question is asking the number of pizza were ordered so I will use customer_orders_temp table which included all orders regardless cancel or not

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/ead265f7-6917-4bc1-89b3-bb7b6059afde)

- At 11AM and 19PM: There was 1 pizza was ordered
- At 13PM, 18PM,21PM and 23PM: There were 3 pizzas were ordered

***
### 10. What was the volume of orders for each day of the week?
```
SELECT TO_CHAR(order_time, 'DAY') AS day_of_week
	,COUNT(order_id) AS order_count
FROM customer_orders_temp
GROUP BY TO_CHAR(order_time, 'DAY')
ORDER BY order_count
```
#### Explanation:
- **TO_CHAR**: Get the name of the day of the week
- The question is asking the number of pizza were ordered so I will use customer_orders_temp table which included all orders regardless cancel or not

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/212821ac-afdd-42c4-8505-89ff0156f94f)

- On Wednesday and Saturday, the restaurant received the highest number of orders: 5
- Thursday: the restaurant received 3 orders
- Friday: the restaurant received 1 order
