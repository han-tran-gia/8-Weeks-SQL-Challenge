## Data Cleaning & Transformation

### 🔨 Table: customer_orders
<img width="650" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/92fad897-5ad8-41c1-ad31-0d31dc4e2d1f">

#### Observations & Ways forward:
- Observations:
  - `exclusions`: this column contains missing and inconsistent data: null values, blank spaces, "null"
  - `extras`: this column also contains missing and inconsistent data: null values, blank spcaes, "null"
- Ways foward:
  - Create a temporary table since I want to keep the original table as is
  - Convert 'null' in these 2 columns to blank space" " so that later I can change the data type easily

```
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions = 'null' THEN ''
	  ELSE exclusions END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras = 'null' THEN ''
	  ELSE extras END AS extras,
	order_time
FROM pizza_runner.customer_orders;
```

#### customer_orders_temp table: This table will be used for all queries later
<img width="1063" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/bf60914e-b825-4e33-a9b1-075bd362eee9">

***

### 🔨 Table: runner_orders
<img width="1037" alt="image" src="https://user-images.githubusercontent.com/81607668/129472585-badae450-52d2-442e-9d50-e4d0d8fce83a.png">

#### Observations & Ways forward:
- Observations:
  - `pickup_time`: Contains null value and wrong data type: varchar instead of timestamp
  - `distance`: Contains null value, inconsistenct data: 20km, 10, 23.4 km and wrong data type: varchar instead of numeric
  - `duration`: Contains null value, inconsistenct data: 32 minutes, 20 mins, 40, 25mins, 15 minute and wrong data type: varchar instead of integer
  - `cancellation`: Contains missing value, "null" value
- Ways foward:
  - Create a temporary table since I want to keep the original table as is
  - `pickup_time`: Convert "null" value into blank space " "
  - `distance`: remove "km" and convert "null" value into blank space " "
  - `duration`: remove "minutes", "mins", "minute" and convert "null" value into blank space " "
  - `cancellation`: convert "null" values into blank space " "
  - Changing data type will be executed later when joining customers_orders_temp and runner_orders_temp together

```
CREATE TEMP TABLE runner_orders_temp AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN ''
	  ELSE pickup_time
  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN ''
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
   END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN ''
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	END AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ''
	  ELSE cancellation
	END AS cancellation
FROM pizza_runner.runner_orders;
```

#### runner_orders_temp table: This table will be used for all queries later
<img width="807" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/631db521-673a-436c-acb8-60d5915aea06">

***

### 🔨 Join table: customer_orders_temp & runner_orders_temp
Since there are several questions that need to join the tables, I will: 
  - Create a CTE named `orders` and merge `customer_orders_temp` and `runner_orders_temp` tables to get all the information from 2 tables
  - Convert the mentioned 3 columns - `pickup_time`, `distance`, `duration`
  - Remove orders that have been cancelled by both restaurant and customer

```
WITH orders AS
(SELECT order_id, customer_id, pizza_id, exclusions, extras, order_time, runner_id
		,CAST(pickup_time AS timestamp)
        ,CAST(distance	AS NUMERIC) 
        ,CAST(duration AS INT)
        ,cancellation
FROM pizza_runner.customer_orders
LEFT JOIN runner_orders_temp
	USING (order_id)
WHERE distance != '')
```
#### orders table
<img width="909" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/202afbb5-b03c-4f5f-b640-47ddebc0cd4f">

***

### 🔨 Table: pizza_recipes
<img width="190" alt="image" src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/d269db15-3991-42e8-941d-5aaca106ac7d">




