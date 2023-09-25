# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Table Of Contents
- Problem Statement
- Entity Relationship Diagram
- Questions & Answers
- Insights & Recommendation

***

## Problem Statement

Danny intends to utilize the data to address some straightforward inquiries regarding his clientele. He is particularly interested in their visitation trends, expenditure levels, and their preferred menu items. His objective is to leverage these insights to make informed decisions regarding the potential expansion of the current customer loyalty program. Furthermore, he requires assistance in creating basic datasets that his team can readily examine without the necessity of employing SQL.

***

## Entity Relationship Diagram
![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)


## Question & Answers
- For these 8 case studies, I used [DB Fiddle - PostgreSQL13](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) to write and execute the queries, where you can also find the datasets for this case studies. Feel free to join me by clicking to the link above.

- Since there are several questions that need to join the tables, I will create a CTE and join the sales table with menu tables first for later use
```
WITH join_sales_menu AS
  (SELECT *
  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.menu
		  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id)
```

**1. What is the total amount each customer spent at the restaurant?**
```
SELECT customer_id AS customer
		,SUM(price) AS total_spend
FROM join_sales_menu
GROUP BY customer_id
ORDER BY customer
```


