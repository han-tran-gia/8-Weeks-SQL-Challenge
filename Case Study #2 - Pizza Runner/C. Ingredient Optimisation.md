# 🍕 Case Study #2 Pizza Runner

## Solution - C. Ingredient Optimisation

### 1. What are the standard ingredients for each pizza?
```
SELECT pizza_name
		,STRING_AGG(topping_name, ', ') AS toppings
FROM topping_list
INNER JOIN pizza_runner.pizza_toppings
	USING (topping_id)
INNER JOIN pizza_runner.pizza_names
	USING (pizza_id)
GROUP BY pizza_name
ORDER BY pizza_name
```

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/4ddb48c2-42e6-427d-9759-189510ea3138)

***

### 2. What was the most commonly added extra?
```
,get_data AS
  (SELECT DISTINCT order_id, topping_name, extras
  FROM 
      (SELECT order_id, customer_id, pizza_id
              ,CAST(UNNEST(STRING_TO_ARRAY(extras, ',')) AS INT)	AS extras 
      FROM customer_orders_temp) AS extras_table
  LEFT JOIN topping_list
      ON extras_table.extras = topping_list.topping_id
  LEFT JOIN pizza_runner.pizza_toppings
      USING (topping_id))

SELECT topping_name
		  ,COUNT(extras) AS extras_count
FROM get_data
GROUP BY topping_name
ORDER BY extras_count DESC
```

#### Explanation:
- **extras_table**: Create a subquery called `extra_table` to convert comma separated values to rows using `string_to_array` and `unnest` functions
- In the outer query, join 2 tables `topping_list` and `pizza_runner.pizza_toppings` to get the name of the ingredients.

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/e8dd509b-910a-4985-bf41-0a32567355af)

- Bacon is the most commonly added extra, following by cheese and chicken

***

### 3. What was the most common exclusion?
```
,exclusions_table AS
  (SELECT order_id
          ,customer_id
          ,pizza_id
          ,CAST(UNNEST(STRING_TO_ARRAY(exclusions, ',')) AS INT) AS exclusions 
          ,order_time
  FROM customer_orders_temp)

SELECT topping_name
		  ,COUNT(exclusions) AS exclusions_count
FROM exclusions_table
LEFT JOIN topping_list
	ON exclusions_table.exclusions = topping_list.topping_id
	AND exclusions_table.pizza_id = topping_list.pizza_id
LEFT JOIN pizza_runner.pizza_toppings
	USING (topping_id)
GROUP BY topping_name
ORDER BY exclusions_count
```

#### Answer:
![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/2c321928-a206-4486-ac3d-e9e53558b34d)

- Cheese is the most commonly excluded topping, following by mushrooms and BBQ sauce

***

4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

