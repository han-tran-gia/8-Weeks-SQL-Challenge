# 🍕 Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## 📚 Table Of Contents
- [Problem Statement](#problem_statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- Questions & Answers
  - [Data Cleaning and Transformation](#-data-cleaning--transformation)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)

***

## Problem Statement
Danny was initially excited about his idea to expand his Pizza Empire but knew he needed more than just pizza to secure seed funding. 
So, he came up with the idea to create Pizza Runner, a service that would deliver fresh pizzas rom Pizza Runner Headquarters (otherwise known as Danny’s house) using a mobile app. 
He recruited delivery "runners" and invested in building the app, even maxing out his credit card to make it happen.

***

## Entity Relationship Diagram
![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

**TABLE 1: runners**
The runners table shows the registration_date for each new runner

![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/13b46a3c-c87a-4619-8613-4192d47d6492)

**TABLE 2: customer_orders**
Customer pizza orders are captured in the customer_orders table with 1 row for each individual pizza that is part of the order.

The pizza_id relates to the type of pizza which was ordered whilst the exclusions are the ingredient_id values which should be removed from the pizza and the extras are the ingredient_id values which need to be added to the pizza.

Note that customers can order multiple pizzas in a single order with varying exclusions and extras values even if the pizza is the same type!

![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/57b3eedc-f8c7-4ba8-b341-77cb40d8fd76)

**TABLE 3: runner_orders**
After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer.

The pickup_time is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The distance and duration fields are related to how far and long the runner had to travel to deliver the order to the respective customer.

![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/a30a3ef8-5c4f-488a-a23f-4c10a90f7f11)

**TABLE 4: pizza_names**
At the moment - Pizza Runner only has 2 pizzas available the Meat Lovers or Vegetarian!

![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/2d058b02-73b5-47f0-b9c6-526145ea7aaa)

**TABLE 5: pizza_recipes**
Each pizza_id has a standard set of toppings which are used as part of the pizza recipe.

![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/37cb2dd4-7563-446b-a39e-9baf9a58f04f)

**TABLE 6: pizza_toppings**
This table contains all of the topping_name values with their corresponding topping_id value

![image](https://github.com/han-tran-gia/8-weeks-sql-challenge/assets/144699083/db3add80-a7ba-467a-837c-4ce375a9c8d4)


Click [here](https://github.com/han-tran-gia/8-weeks-sql-challenge/blob/main/Case%20Study%20%232%20-%20Pizza%20Runner/Data%20Cleaning%20%26%20Transformation.md) for solution to Data Cleaning and Transformation
