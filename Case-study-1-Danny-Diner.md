# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Table Of Contents
- Problem Statement
- Entity Relationship Diagram
- Questions & Answers
- Insights & Recommendation

***

## Problem Statement

Danny intends to utilize the data to address some straightforward inquiries regarding his clientele. He is particularly interested in their visitation trends, expenditure levels, and their preferred menu items. 

His objective is to leverage these insights to make informed decisions regarding the potential expansion of the current customer loyalty program. Furthermore, he requires assistance in creating basic datasets that his team can readily examine without the necessity of employing SQL.

***

## Entity Relationship Diagram
![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question & Answers
- For these 8 case studies, I used [DB Fiddle - PostgreSQL13](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) to write and execute the queries, where you can also find the datasets for this case studies. Feel free to join me by clicking to the link above.

- Since there are several questions that need to join the tables, I will create a CTE named `join_sales_menu` and merge `dannys_diner.sales` and `dannys_diner.menu` tables to get the information of `customer_id`, `order_date`, `product_name` and `price` first for later use.

```
WITH join_sales_menu AS
	(SELECT customer_id, order_date, dannys_diner.sales.product_id, product_name, price
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.menu
		ON dannys_diner.sales.product_id = dannys_diner.menu.product_id)
```

### join_sales_menu table:

<img src="https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/fb14f03a-8a17-4837-ae5c-c956d12c1d30"  alt="image" width="956">

***

**1. What is the total amount each customer spent at the restaurant?**

```
SELECT customer_id AS customer
	,SUM(price) AS total_spend
FROM join_sales_menu
GROUP BY customer_id
ORDER BY customer
```

#### Explanation:
- **GROUP BY**: Group the aggregated result by `customer_id`
- **SUM**: Calculate the total amount that each customer spent

#### Answer:
|customer |total_spend|
|----------|------------|
|A|76|
|B|74|
|C|36|

- Customer A spent the most out of 3 customers, followed by customer B, with both of them spending twice as much as customer C.
- Customer A spent $76
- Customer B spent $74
- Customer C spent $36

***

**2. How many days has each customer visited the restaurant?**

```
SELECT customer_id AS customer
	,COUNT(DISTINCT order_date) AS visit_days
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer
```

#### Explanation:
- **COUNT (DISTINCT order_date)**: Count unique date that each customer visit the restaurant
  - Reason: I used DISTINCT in order to prevent the possibility of duplicate dates in the dataset but the question is asking the number of day each customer comes to the restaurant.
  - For example, customer A was recorded visiting the restaurant 2 times on 2021-01-01 and made purchases of 2 products respectively.
 
#### Answer:
| customer | visit_days |
|----------|------------|
|    A     |     4      |
|    B	   |     6      |
|    C     |     2      |

- Customer B visited the restaurant the most and then is customer A, final is customer B
- Customer A: visited 4 days
- Customer B: visited 6 days
- Customer C: visited 2 days

***

**3. What was the first item from the menu purchased by each customer?**

```
,item_order AS
	(SELECT customer_id AS customer
			,product_name AS item_purchased
	        ,order_date AS date
	        ,DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS item_order
	FROM join_sales_menu)

SELECT DISTINCT customer, item_purchased
FROM item_order
WHERE item_order = 1
ORDER BY customer, item_purchased
```

#### Explanation:
- **item_order**: Create another CTE called `item_order` to rank the item from 1 to end by using `DENSE_RANK()`.
	- **DENSE_RANK**: Rank the product bought from 1. With 1 is the first dish the customer ordered
   	- **PARTITION BY**: Divide the result set into partitions based on `customer_id` column and reset the ranking within each partition.
   	- **ORDER BY**: order the rows within each partition by `order_date` in ascending order
- **SELECT DISTINCT** & **WHERE item_order = 1**: Select unique `customer_id` and `product_name`, which were renamed to `customer` and `item_purchased` in the above CTE, which meet the criteria in the WHERE clause: item_order = 1

  #### Answer:
| customer| item_purchased |
|---------|----------------|
|   A     |      curry     |
|   A     |      sushi     |
|   B     |      curry     |
|   C     |      ramen     |

- Customer A visited the restaurant the first time on 2021-01-01 and ordered 2 dishes which are curry and sushi. Since since the `order_date` does not have a timestamp, we can't know which dish was ordered first when he/she coming to the restaurant. Therefore, I keep both the dishes as the first items ordered
- Customer A ordered 2 dishes as the first items: curry & sushi
- Customer B's first item ordered is curry
- Customer C's first item ordered is ramen

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```  
SELECT product_name AS item
	,COUNT(product_name) AS item_count
FROM join_sales_menu
GROUP BY product_name
ORDER BY item_count DESC
LIMIT 1
```

#### Explanation:
- **GROUP BY**: Group the product by name by using `product_name` column
- **COUNT**: Count the number of products were purchased by using `product_name` and renamed to `item_count` by using AS clause
- **ORDER BY**: Order the result on the `item_count` column in descending order and just take the 1st result - highest number of purchased items - appears by using **LIMIT**

#### Answer:
|item|item_count|
|---|-------------|
|ramen|8|

- Ramen is the most purchased dish with 8 times.

***

**5. Which item was the most popular for each customer?**
```
,get_data AS
(SELECT customer_id AS customer
	,product_name AS item
        ,COUNT(product_name) AS pop_item_count
        ,DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS ranking
FROM join_sales_menu
GROUP BY customer_id, product_name
ORDER BY customer_id, pop_item_count DESC)

SELECT customer, item, pop_item_count
FROM get_data
WHERE ranking = 1
```

#### Explanation:
 - **get_data**: Create a CTE called `get_data` to rank the item from 1 to end with the most order item ranked 1
 	- **DENSE_RANK**: Rank the product from 1 to end. With 1 is the product that was ordered the most by the customer
   	- **PARTITION BY**: Divide the result set into partitions based on `customer_id` column and reset the ranking within each partition.
   	- **ORDER BY COUNT(product_name) DESC**: Order the rows within each partition by the number of products - `COUNT(product_name)` in descending order
	- **GROUP BY**: Group the result by `customer_id` and `product_name` and then count the `product_name` in each group
	- **ORDER BY**: Order the result by `customer_id` in ascending order, if 2 records have the same `customer_id`, then order them by `pop_item_count`, which is the new column generated by **COUNT(product_name)**, in descending order
- **WHERE ranking = 1**: Select the columns from the get_data CTE which meet the criteria of the WHERE clause: ranking = 1

#### Answer:
|customer|item|pop_item_count|
|-----|------|-----|
|A|ramen|3|
|B|ramen|2|
|B|curry|2|
|B|sushi|2|
|C|ramen|3|

- Customer A & C' favored item is ramen with 3 order times for both
- Customer B enjoyed all the food on the item. Each item was ordered 2 times

***

**6. Which item was purchased first by the customer after they became a member?**

```
WITH mem_date AS
  (SELECT dannys_diner.sales.customer_id AS customer
   			,dannys_diner.sales.product_id
   			,DENSE_RANK() OVER(PARTITION BY dannys_diner.sales.customer_id ORDER BY order_date) AS ranking
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.members
    ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
  WHERE order_date >= join_date)

SELECT customer, product_name
FROM mem_date
LEFT JOIN dannys_diner.menu
	ON mem_date.product_id = dannys_diner.menu.product_id
WHERE ranking = 1
ORDER BY customer
```

