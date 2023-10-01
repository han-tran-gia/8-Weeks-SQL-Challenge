# 🍕 Case Study #2 Pizza Runner

## Solution - B. Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```
WITH weeks AS
  (SELECT generate_series (DATE '2021-01-01'
                        	,(SELECT MAX(registration_date) 
                          	FROM pizza_runner.runners)
                        	,'1 week') AS week_start_date)
SELECT week_start_date
		,COUNT(*) AS runner_count
FROM pizza_runner.runners AS r
LEFT JOIN weeks AS w
  ON r.registration_date >= w.week_start_date
  AND r.registration_date < (w.week_start_date + '1 week')
GROUP BY week_start_date
ORDER BY week_start_date
```

#### Explanation:
- First, I created a series of week starting from `2021-01-01` until the last day in the data set and put it in a CTE called `weeks`
- In the main query, I joined 2 tables: `pizza_runner.runners` and `weeks` with 2 conditions:
	1. **r.registration_date >= w.week_start_date**: The register date must be > = the first date of the week
	2. **r.registration_date < (w.week_start_date + '1 week')**: the register date must be < the first date of next week
-> This will put the runners into each week bins

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/4f06707f-55f1-4ad5-b862-eeb3d18ac984)

- Week of 2021-01-01: There were 2 runners signed up
- Week of 2021-01-08: 1 runner signed up
- Week of 2021-01-15: 1 runner signed up

***

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```
,diff_time AS
  (SELECT runner_id, order_id
          ,EXTRACT('minute' 
                       FROM (CAST(
                              pickup_time AS timestamp))-order_time)
          AS arrive_time
  FROM orders
  GROUP BY runner_id, order_id, order_time, pickup_time)
  
SELECT runner_id 
	,ROUND(AVG(arrive_time)) AS avg_arrive_time
        ,COUNT(order_id) AS order_count
FROM diff_time
GROUP BY runner_id
ORDER BY runner_id
```
#### Explanation:
- In this question, besides the average moving time, I also want to know how many orders each runners delivered comparing to the time they get to the Head Quarter.

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/02a6a4eb-ffc2-4339-8955-9f25b669bed4)

- Runner 1: It took 14 mins in average to get to the Head Quarter for delivering 4 orders
- Runner 2: It took 20 mins in average to get to the Head Quarter for delivering 3 orders
- Runner 3: It took 10 mins to get to the Head Quarter for delivering 1 order

***
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```
,prepare_time AS
  (SELECT order_id
          ,COUNT(order_id) AS pizza_count
	  ,EXTRACT('minute' FROM (pickup_time) - order_time) AS time_for_prepare
  FROM orders
  GROUP BY order_id, order_time, pickup_time
  ORDER BY order_id)
  
SELECT pizza_count
	,AVG(time_for_prepare) AS avg_prepare_time
FROM prepare_time
GROUP BY pizza_count
ORDER BY pizza_count
```

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/81607c29-db95-4811-936b-3c77e8aa2941)

- The more pizzas in the order, the faster prepare time is. 
- 1 pizza: It took 12 mins to prepare 1 pizza
- 2 pizzas: It took 18 mins to prepare 2 pizzas, which means 9 mins each
- 3 pizzas: It took 29 mins to prepare 3 pizzas, which means 9.5 mins each

***
### 4. What was the average distance travelled for each customer?
```
SELECT customer_id
	,ROUND(AVG(distance),2) AS avg_distance
FROM orders
GROUP BY customer_id
ORDER BY customer_id
```
#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/b97328bb-ad59-445e-9d9f-7d21cbedcecf)

- Customer 104 is nearest to the Head Quarter, next is customer 102, 101, 103 and farest is customer 105
- Customer 101: 20m far from Head Quarter
- Customer 102: 16.73m
- Customer 103: 23.4m
- Customer 104: 10m
- Customer 105: 25m

***
### 5. What was the difference between the longest and shortest delivery times for all orders?
```
SELECT MAX(duration) AS max_duration
        ,MIN(duration) AS min_duration
		,MAX(duration) - MIN(duration) AS diff_delivery_time
FROM orders
```

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/b454f093-dc3e-4845-b3f3-64147e663045)

The longest delivery time is 40 mins and shortest is 10 mins. 

***
### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```
SELECT runner_id, order_id
	,COUNT(pizza_id) AS pizza_count
        ,distance
        ,duration AS duration_mins
	,ROUND(distance/duration * 60,2) AS avg_speed
FROM orders
GROUP BY runner_id, order_id, distance, duration
ORDER BY runner_id
```

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/65518a3f-16ae-4bcf-a3bc-1d4793d114ef)

(I kept the duration in the table as minutes since it will be 0 if change to hour.)
(Average speed = = Distance in km / Duration in hour)

- Runner 1’s average speed runs from 37.5km/h to 60km/h.
- Runner 2’s average speed runs from 35.1km/h to 93.6km/h. The fluctuation in the speed of this runner (300% fluctuation) should be investigated more since he can affect to the delivery performance
- Runner 3’s average speed is 40km/h.

***
### 7. What is the successful delivery percentage for each runner?
```
,count_data AS
(SELECT runner_id
	,COUNT(*) AS count_all
        ,SUM(CASE WHEN distance = '' THEN 0 ELSE 1 END) AS count_successful
FROM runner_orders_temp
GROUP BY runner_id)

SELECT runner_id
	,(100 * count_successful/count_all) AS successful_rate
FROM count_data
ORDER BY runner_id
```

#### Explanation: 
- **count_data**: Create a CTE called `count_data` to calculate the total number of deliveries by using **COUNT(*)** and calculate the successful/not cancel deliveries by summing all the record that have distance recorded
- Main query: calculate the `successful_rate` by dividing successful deliveries to all deliveries

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/e4f7af5b-719c-415f-a590-b96e4a7d8cdc)

- Runner 1 has 100% successful rate
- Runner 2 has 75% successful rate
- Runner 3 has 50% successful rate
- The data set is lacking the records about cancelation made by runners or customers with the reasons related to runners and we can't conclude the runners' performance or thier successful rate since they aren't the one who affect the cancellation rate.

Click [here](https://github.com/han-tran-gia/8-weeks-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20Customer%20Experience.md) for solution to C. Ingredient Optimisation
