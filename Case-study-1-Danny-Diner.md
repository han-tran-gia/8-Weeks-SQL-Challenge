# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Table Of Contents
- [Problem Statement](#problem_statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Questions & Answers](#questions-&-answers)
- [Insights & Recommendations](#insights-&-recommendations)

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
- **GROUP BY**: Group the products by name by using `product_name` column
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
	- **GROUP BY**: Group the records by `customer_id` and `product_name` and then count the `product_name` in each group
	- **ORDER BY**: Order the resulta by `customer_id` in ascending order, if 2 records have the same `customer_id`, then order them by `pop_item_count`, which is the new column generated by **COUNT(product_name)**, in descending order
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

#### Explanation:
- **mem_date**: Create a CTE called `mem_date` to join 2 tables `dannys_diner.sales` and `dannys_diner.members` on the `customer_id`
	- Select `customer_id`, `product_id` and rank them from 1 to end by using **DENSE_RANK**. With 1 is the first order the customer made after becoming the member.
 	- **WHERE order_date >= join_date**The selected rows must meet the criteria in **WHERE** claue: order_date >= join_date, which means the order must be made **on or after** the date the customers become a member at the restaurant.
	- For example: customer A becoming a member on 2021-01-07 and he/she also made an order on that day. Therefore, I will consider he/she became the member first and then placed an ordered. 
	- **PARTITION BY**: Divide the data by `members.customer_id`
	- **ORDER BY**: Order the rows within each `dannys_diner.sales.customer_id` partition by `order_date`
- In the outer query: Join the CTE `mem_date` with `dannys_diner.menu` table on `product_id`
- **WHERE ranking = 1**: Select appropriate column that meet the criteria in the WHERE clause: ranking = 1
- **ORDER BY customer**: Order the result by `customer_id` which was renamed to `customer` in ascending order

#### Answer:
|customer|product_name|
|------|-------|
|A|curry|
|B|sushi|

- Customer A's first order after becoming a member: curry
- Customer B's first order after becoming a member: sushi
- Customer C's hasn't became the member at the restaurant yet

***

**7. Which item was purchased just before the customer became a member?**

```
WITH non_mem_date AS
  (SELECT dannys_diner.sales.customer_id AS customer
	,dannys_diner.sales.product_id
	,DENSE_RANK() OVER(PARTITION BY dannys_diner.sales.customer_id ORDER BY order_date DESC) AS ranking
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.members
    ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
  WHERE order_date < join_date)

SELECT customer, product_name
FROM non_mem_date
LEFT JOIN dannys_diner.menu
	ON non_mem_date.product_id = dannys_diner.menu.product_id
WHERE ranking = 1
ORDER BY customer
```

#### Explanation:
- **non_mem_date**: Create a CTE called `non_mem_date` to join 2 tables `dannys_diner.sales` and `dannys_diner.members` on the `customer_id`
	- Select approriate columns and rank them from 1 to end the partition by using **DENSE_RANK**. With 1 is the last order the customer made right before they bacame a member of the restaurant.
	- **WHERE order_date < join_date**The selected rows must meet the criteria in **WHERE** claue: order_date < join_date, which means the order must be made **before** the date the customers become a member at the restaurant. If the customer became the member and ordered a dish on the same date, in this case, that order will be considered as making after the customer became the member.
	- **PARTITION BY**: Divide the data by `members.customer_id`
	- **ORDER BY**: Order the rows within each `dannys_diner.sales.customer_id` partition by `order_date`
- In the outer query: Join the CTE `non_mem_date` with `dannys_diner.menu` table on `product_id`
- **WHERE ranking = 1**: Select appropriate column that meet the criteria in the WHERE clause: ranking = 1
- **ORDER BY customer**: Order the result by `customer_id` which was renamed to `customer` in ascending order

#### Answer:
|customer|product_name|
|----|-----|
|A|sushi|
|A|curry|
|B|sushi|

- Customer A ordered 2 dishes: sushi & curry right before becoming a member at the restaurant
- Customer B ordered: sushi as the last dish before becoming the member at the restaurant
- Customer C hasn't became the member, therefore, he/she will not be considered in this question

***
**8. What is the total items and amount spent for each member before they became a member?**

```
WITH non_mem_date AS
  (SELECT dannys_diner.sales.customer_id AS customer
   			,dannys_diner.sales.product_id
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.members
    ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
  WHERE order_date < join_date)

SELECT customer
	,COUNT(product_name) AS item_count
        ,SUM(price) AS total_spend
FROM non_mem_date
LEFT JOIN dannys_diner.menu
	ON non_mem_date.product_id = dannys_diner.menu.product_id
GROUP BY customer
ORDER BY customer
```

#### Explanation:
- **non_mem_date**: Create a CTE called `non_mem_date` like question #7
- In the outer query: Join 2 tables `non_mem_date` and `dannys_diner.menu` on the `product_id` same as second step in question #7
- **COUNT(product_name)**: Calculate the number of item each customer purchased and called it `item_count`
- **SUM(price)**: Calculate the total that each customer spent at the restaurant and called it `total_spend`
- **GROUP BY customer**: Group the records by `customer_id` which was renamed to `customer`
- **ORDER BY customer**: Order the results by `customer` in ascending order

#### Answer:
|customer|item_count|total_spend|
|----|----|-----|
|A|2|25|
|B|3|40|

- Before becoming a member, customer B ordered more dishes than customer A which makes the amount customer B spent is also larger than customer A
- Customer A: ordered 2 dishes and spent $25 for these
- Customer B: ordered 3 dishes and spent $40 for these

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```
SELECT customer_id AS customer
	,SUM(CASE 
		WHEN product_name = 'sushi' THEN price * 2 * 10
            	ELSE price * 10 END) AS point
FROM join_sales_menu
GROUP BY customer_id
ORDER BY customer
```

#### Explanation:
- The question giving a scenario that:
	- If customers ordered sushi: they will get 2x points = 20 points - $1 = 10 points
	- If they ordered the remaining items: they will get 10 point - $1 = 10 points
- **CASE WHEN**: From this scenario, I used CASE WHEN to perform the calculation:
	- **product_name = 'sushi'**: if the product is sushi, then the customer will get 2 * 10 = 20 points
 	- Otherwise, they will get 10 points for the remaining items
- **SUM**: calculate the total of points each customer received
- **GROUP BY**: Group the records by `customer_id`
- **ORDER BY**: Order the results by `customer` in ascending order

#### Answer:
|customer|point|
|----|----|
|A|860|
|B|940|
|C|360|

- Customer B received the highest points, higher than customer A 100 points and about triple the points of customer C
- Customer A: 860 points
- Customer B: 940 points
- Customer C: 360 points

****

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```
WITH mem_date AS
	(SELECT dannys_diner.sales.customer_id AS customer
   		,dannys_diner.sales.product_id
   		,dannys_diner.sales.order_date
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.members
		ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
	WHERE order_date >= join_date)
  
SELECT customer
	,SUM(CASE
        	WHEN ('2021-01-01'< order_date) OR (order_date + 6 < '2021-01-31') AND product_name = 'sushi' THEN price * 20
        	WHEN order_date <= order_date + 6 THEN price * 2 * 10
            	ELSE price * 10 END) AS point
FROM mem_date
LEFT JOIN dannys_diner.menu
	ON mem_date.product_id = dannys_diner.menu.product_id
GROUP BY customer
ORDER BY customer
```

#### Explantion:
- The question giving a scenario that:
	- Before customers become a member: they will get 2x points = 20 points if the order sushi and 10 points if they ordered the rest items.
	- After becoming a member:
		- 1st week after becoming a member: customers will get 2x points = 20 points for all items the purchased
		- After 1st week: they will get 2x points = 20 points if the order sushi and 10 points if they ordered the rest items like before becoming members.
- **mem_date**: Create a CTE called `mem_date` to join 2 tables `dannys_diner.sales` and `dannys_diner.members` on the `customer_id` same as question #6
- In the outer query: Join 2 tables `mem_date` and `dannys_diner.menu` on the `product_id` same as second step in question #6
- **GROUP BY customer**: Group the records by `customer`
- **CASE WHEN**: From the scenario, I used CASE WHEN to perform the calculation:
  	- **('2021-01-01'< order_date) OR (order_date + 6 < '2021-01-31') AND product_name = 'sushi'**: If the customer ordered between time range 2021-01-01 and 2021-01-31, excluding the first week after becoming a member - which means order_date to order_date + 6 AND they ordered sushi, they will get 20 points
  	- **order_date <= order_date + 6**: If they ordered in the first week after becoming a member - which means `order_date <= order_date _6` they will get 20 points for all items
  	- The remaining cases: they will get 10 points each item.
- **ORDER BY**: Order the results by `customer` in ascending order

#### Answer:
|customer|point|
|---|---|
|A|1020|
|B|680|

- In January, customer A earns 1.5 times as much as customer B
- Customer A earns: 1020 points
- Customer B earns: 680 points

****
## BONUS QUESTIONS
**11. Join All The Things**
**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

```
SELECT dannys_diner.sales.customer_id AS customer
	,dannys_diner.sales.order_date
        ,dannys_diner.menu.product_name
        ,dannys_diner.menu.price
        ,(CASE
        	WHEN order_date < join_date THEN 'N'
		WHEN order_date >= join_date THEN 'Y' 
          	ELSE 'N' END) AS member
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
	ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
LEFT JOIN dannys_diner.members
	ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
ORDER BY customer, order_date
```

#### Explanation:
- Joining 2 tables `dannys_diner.sales` & `dannys_diner.menu` on the `product_id` and then joining them with `dannys_diner.members` table on the `customer_id`
- For each line of records, if the customer became a member, write Y, if not write N
	- **CASE WHEN**: For this reason, I used CASE WHEN to perform:
		- If order_date < join_date: which means customers ordered before becoming a member, write 'N' in the `member` column
  		- If order_date >= join_date: which means customers ordered on or after becoming a member, write 'N' in the `member` column
		- The remaining cases - customer C, who hasn't became a member yet: will write 'N' in `member` column
-**ORDER BY**: Order the result by customer in ascending order, if customer is the same, order the results by order_date in ascending order

#### Answer:
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

***

**12. Rank All The Things**

**Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.**

```
WITH get_data AS
	(SELECT dannys_diner.sales.customer_id AS customer
		,dannys_diner.sales.order_date
        	,dannys_diner.menu.product_name
        	,dannys_diner.menu.price
        	,(CASE
        		WHEN order_date < join_date THEN 'N'
			WHEN order_date >= join_date THEN 'Y' 
          		ELSE 'N' END) AS member
	FROM dannys_diner.sales
	LEFT JOIN dannys_diner.menu
		ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
	LEFT JOIN dannys_diner.members
		ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
	ORDER BY customer, order_date)

SELECT *,
	(CASE
		WHEN member = 'Y' THEN
           		DENSE_RANK() OVER(PARTITION BY customer, member ORDER BY order_date)
           	ELSE null END) AS ranking
FROM get_data
```

#### Explanation:
- **get_data**: Create a CTE called `get_data` same as question #11
- **CASE WHEN**:
	- If the customers is a member - member = 'Y', I will use **DENSE_RANK** to rank the records from 1 to end, with 1 is the first order that customer made after being a member

#### Answer:
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL|
| A           | 2021-01-01 | curry        | 15    | N      | NULL|
| A           | 2021-01-07 | curry        | 15    | Y      | 1|
| A           | 2021-01-10 | ramen        | 12    | Y      | 2|
| A           | 2021-01-11 | ramen        | 12    | Y      | 3|
| A           | 2021-01-11 | ramen        | 12    | Y      | 3|
| B           | 2021-01-01 | curry        | 15    | N      | NULL|
| B           | 2021-01-02 | curry        | 15    | N      | NULL|
| B           | 2021-01-04 | sushi        | 10    | N      | NULL|
| B           | 2021-01-11 | sushi        | 10    | Y      | 1|
| B           | 2021-01-16 | ramen        | 12    | Y      | 2|
| B           | 2021-02-01 | ramen        | 12    | Y      | 3|
| C           | 2021-01-01 | ramen        | 12    | N      | NULL|
| C           | 2021-01-01 | ramen        | 12    | N      | NULL|
| C           | 2021-01-07 | ramen        | 12    | N      | NULL|

***

## Insights & Recommendation

