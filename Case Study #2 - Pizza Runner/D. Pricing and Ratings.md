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
