# PIZZA_DELIVERY
The entrepreneur behind this innovative venture recognized that relying solely on pizza wouldn't be enough to secure seed funding for expanding the new Pizza Empire. To enhance the business concept, they devised a brilliant strategy to emulate the success of Uber. Thus, Pizza Runner was born.

The project commenced by enlisting "runners" who would be responsible for delivering fresh pizza, with operations based at the Pizza Runner Headquarters, affectionately known as the project leader's residence. Furthermore, substantial financial resources were invested in hiring freelance developers to create a mobile application capable of handling customer orders seamlessly.

Here we clean the data and apply calculations so we can better direct runners and optimise Pizza Runner’s operations.  All datasets exist within the pizza_runner database schema.
## Available Data
### Table 1: runners
The runners table shows the registration_date for each new runner

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/b18d02c0-6692-4eb3-b7d1-c3642a181000)

### Table 2: customer_orders
Customer pizza orders are captured in the customer_orders table with 1 row for each individual pizza that is part of the order.

The pizza_id relates to the type of pizza which was ordered whilst the exclusions are the ingredient_id values which should be removed from the pizza and the extras are the ingredient_id values which need to be added to the pizza.

Note that customers can order multiple pizzas in a single order with varying exclusions and extras values even if the pizza is the same type!

The exclusions and extras columns is cleaned up before using them in your queries.  


![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/4107cf21-8e95-4a68-9917-a03aba6ec0cc)

## Table 3: runner_orders

After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer.

The pickup_time is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The distance and duration fields are related to how far and long the runner had to travel to deliver the order to the respective customer.

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/97423b77-bdbe-4e1b-a0d1-26ee7eb9e6ab)

## Table 4: pizza_names
At the moment - Pizza Runner only has 2 pizzas available the Meat Lovers or Vegetarian!


![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/526ea7e0-cd59-48de-9707-460e8c08a47f)

## Table 5: pizza_recipes
Each pizza_id has a standard set of toppings which are used as part of the pizza recipe.

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/da19472d-2d31-4636-8ecc-75f3724cdab3)

## Table 6: pizza_toppings
This table contains all of the topping_name values with their corresponding topping_id value

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/4e6a144e-0a00-4b6a-a760-eddcdb277c1c)

## 📚 Table of Contents
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions](#case-study-questions)
- [RUNNER AND CUSTOMER EXPERIENCE](#RUNNER-AND-CUSTOMER-EXPERIENCE)

## Entity Relationship Diagram

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/bab9688c-cd90-4d48-ae7f-449a61deac82)

## Case Study Questions
## A. Pizza Metrics

### 1. How many pizzas were ordered?

```sql

SELECT
count(*) AS  Total_pizzas_ordered
FROM  pizza_runner.customer_orders

```
![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/bc921382-d3d3-49ca-ae05-4d811274fd36)

### 2. How many unique customer orders were made?
```sql
SELECT
count(distinct order_id) AS Unique_customer_orders
FROM pizza_runner.customer_orders

```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/379867b1-13df-4ea2-b32c-9b0db10b6e12)

### 3.How many successful orders were delivered by each runner?

```sql
SELECT
runner_id, COUNT(runner_id) AS Successful_orders
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL 
GROUP BY runner_id

```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/56f45ebb-08ee-4dc4-9796-1da2431b3440)

### 4.How many of each type of pizza was delivered?

```sql
WITH cte AS 
(SELECT
pizza_id , order_id
FROM pizza_runner.customer_orders)

SELECT
pizza_name, COUNT(pizza_name) AS Total_pizzas
FROM cte AS a JOIN pizza_runner.pizza_names AS b
ON a.pizza_id = b.pizza_id
JOIN pizza_runner.runner_orders AS c 
ON a.order_id = c.order_id
WHERE cancellation is null
GROUP BY pizza_name

```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/3a95a7ab-4d0f-41f0-89a1-e53c8d8984ed)

### 5. How many Vegetarian and Meatlovers were ordered by each customer?

WITH cte AS 
(SELECT 
customer_id, pizza_id
FROM pizza_runner.customer_orders)

SELECT 
a.customer_id, b.pizza_name, count(*) AS pizza_count
FROM cte AS a JOIN pizza_runner.pizza_names AS b
on a.pizza_id = b.pizza_id
GROUP BY a.customer_id, b.pizza_name
ORDER BY customer_id, pizza_name, pizza_count DESC

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/c1a5349c-41c6-4459-8cec-4198417124e7)

### 6. What was the maximum number of pizzas delivered in a single order?

```sql
WITH customer_orders1 AS 
(SELECT 
order_id
FROM pizza_runner.customer_orders),

runner_orders1 AS 
(SELECT 
order_id, cancellation
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL)

SELECT 
a.order_id, COUNT(b.order_id) as TOTAL_PIZZAS_per_ORDER
FROM customer_orders1 AS a LEFT JOIN runner_orders1 AS b
ON a.order_id = b.order_id
WHERE b.order_id IS NOT NULL
GROUP BY a.order_id
ORDER BY TOTAL_PIZZAS_per_ORDER DESC
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/e8b473b9-bf70-49d5-8ef0-f8cbc1ae2bab)

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
WITH delivered_orders AS 
(SELECT
order_id
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL)

SELECT
customer_id, 
IFNULL(SUM(CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1 ELSE 0 END),0) AS unchanged_pizza,
IFNULL(SUM((CASE WHEN exclusions IS NOT NULL THEN 1 ELSE 0 END) + (CASE WHEN extras IS NOT NULL THEN 1 ELSE 0 END)),0) AS total_changed_pizza
FROM pizza_runner.customer_orders AS a LEFT JOIN delivered_orders AS b 
ON a.order_id = b.order_id
WHERE b.order_id IS NOT NULL
GROUP BY customer_id
```
![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/6a478f95-9290-4d6a-8014-51045d71c3a8)

### 8. How many pizzas were delivered that had both exclusions and extras?

```sql
WITH delivered_orders AS 
(SELECT order_id
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL)

SELECT COUNT(*) AS pizza_with_both_exclusion_and_extras
FROM pizza_runner.customer_orders AS a LEFT JOIN delivered_orders AS b 
ON a.order_id = b.order_id
WHERE b.order_id IS NOT NULL AND exclusions IS NOT NULL AND extras IS NOT NULL

```
![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/39f52d1e-f92a-44e6-a18f-716530dfe99c)

### 9. What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
HOUR(order_time) AS hour_of_the_day, COUNT(order_id) AS Total_orders
FROM pizza_runner.customer_orders
GROUP BY HOUR(order_time)
ORDER BY hour_of_the_day
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/77301520-75db-4e9a-b987-7e898e2c7286)

### 10. What was the volume of orders for each day of the week?
```sql
SELECT 
DATE_FORMAT(order_time, '%a') AS day_of_the_week,  COUNT(order_id) AS Total_orders
FROM pizza_runner.customer_orders
GROUP BY DATE_FORMAT(order_time, '%a')
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/ccb6dbd4-b8a5-45b3-a125-813352bd4c77)

## B.	RUNNER AND CUSTOMER EXPERIENCE

### 1.	How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01) 

```sql
SELECT registration_week , COUNT(runner_id) AS Total_runners_signed_up
FROM (SELECT runner_id, DATE_FORMAT(registration_date, '%u')+1  AS registration_week  
FROM pizza_runner.runners) AS abc
GROUP BY registration_week
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/5d20cc1b-a4bb-47cb-9579-feab3f5b4d8b)

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
WITH runner_orders1 AS 
(SELECT
order_id, runner_id, pickup_time 
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL),

customer_orders1 AS
(SELECT
order_id, order_time 
FROM pizza_runner.customer_orders)


SELECT
runner_id, CONCAT(ROUND(AVG(TIMESTAMPDIFF(minute, order_time, pickup_time)),2), " minutes")
AS time_taken_to_reach_in_minutes
FROM runner_orders1 AS a LEFT JOIN customer_orders1 AS b 
ON a.order_id = b.order_id
GROUP BY runner_id

```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/1f032e2e-6eee-44ff-8c53-2cda5979964b)

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
WITH customer_orders1 AS 
(SELECT 
order_id, pizza_id, order_time 
FROM pizza_runner.customer_orders),

runner_orders1 AS 
(SELECT 
order_id, pickup_time
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL)

SELECT 
pizza_id, CONCAT(ROUND(AVG(TIMESTAMPDIFF(MINUTE, order_time, pickup_time)),2), " minutes") 
AS avg_time_to_prepare,
COUNT(a.order_id) AS total_pizza_ordered
FROM customer_orders1 AS a JOIN runner_orders1 AS b 
ON a.order_id = b.order_id
GROUP BY pizza_id
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/dc11bac1-5b58-4744-b0c5-72376607bddc)

### 4. Is there any relationship between the number of pizzas ordered by each customer and the distance from his home to pizza shop?

```sql
WITH customer_orders1 AS
(SELECT
order_id,customer_id, pizza_id, order_time 
FROM pizza_runner.customer_orders),

runner_orders1 AS
(SELECT
order_id, distance
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL)

SELECT
customer_id, COUNT(customer_id) AS total_pizzas_ordered, AVG(distance) AS Distance_in_between
FROM customer_orders1 AS a JOIN runner_orders1 AS b 
ON a.order_id = b.order_id
GROUP BY customer_id
ORDER BY Distance_in_between

```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/f306e7aa-38c9-49eb-bb8b-439d6382ab3a)

### 5. What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
WITH customer_orders1 AS
(SELECT
order_id, customer_id 
FROM pizza_runner.customer_orders),

runner_orders1 AS
(SELECT
order_id, runner_id, distance, duration, cancellation
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL)

SELECT
runner_id, customer_id, distance, ((duration*1.0)/60) AS duration_in_hours, ROUND((distance/((duration*1.0)/60)),2) AS speed
FROM customer_orders1 AS a JOIN runner_orders1 AS b 
ON a.order_id = b.order_id
ORDER BY runner_id, speed

```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/1eb91091-fc5f-4e35-9a57-bc7cb312fb64)

### 6. What is the successful delivery percentage for each runner?

```sql
WITH cte AS
(SELECT
runner_id, cancellation
FROM pizza_runner.runner_orders
ORDER BY runner_id)

SELECT
runner_id, ROUND((successful_delivery/(successful_delivery + failed_delivery))*100.0,2) AS total_delivery
FROM (SELECT runner_id,
IFNULL(SUM(CASE WHEN cancellation IS NULL THEN 1  END),0) AS successful_delivery,
IFNULL(SUM(CASE WHEN cancellation IS NOT NULL THEN 1 END),0) AS failed_delivery
FROM cte
GROUP BY runner_id) AS abc

```
![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/b9c21fbb-53c4-4c56-851f-406d63746fa8)

## C. Ingredient Optimisation

### 1. What are the standard ingredients for each pizza?

```sql
WITH pizza_recipes1 AS 
(SELECT 
pizza_id, SUBSTRING_INDEX(SUBSTRING_INDEX(toppings, ',', n.digit+1), ',', -1) AS toppings
FROM pizza_runner.pizza_recipes
INNER JOIN 
(SELECT 0 digit UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
UNION ALL SELECT 6 UNION ALL SELECT 7) n
  ON LENGTH(REPLACE(toppings, ',' , '')) <= LENGTH(toppings)-n.digit
ORDER BY
  pizza_id,
  n.digit)


SELECT 
pizza_id, GROUP_CONCAT(topping_name) standard_ingredients
FROM (SELECT pizza_id, topping_name
FROM pizza_recipes1 AS a JOIN pizza_runner.pizza_toppings as b 
ON a.toppings = b.topping_id 
ORDER BY pizza_id) AS abc
GROUP BY pizza_id
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/31c2b9da-eab9-4993-a41f-2447f23fd038)

### 2. What was the most commonly added extra?

```sql
WITH extras1 AS
(SELECT
SUBSTRING_INDEX(SUBSTRING_INDEX(extras, ',', n.digit+1), ',',-1) AS extras
FROM pizza_runner.customer_orders
INNER JOIN 
(SELECT 0 digit UNION ALL SELECT 1 UNION ALL SELECT 2) n
 ON LENGTH(REPLACE(extras, ',' , '')) <= LENGTH(extras)-n.digit)
 
SELECT
a.extras, b.topping_name, COUNT(*) AS no_of_times_added
FROM extras1 AS a join pizza_runner.pizza_toppings AS b 
ON a.extras = b.topping_id
GROUP BY a.extras, b.topping_name
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/4242d5fd-155b-4620-8274-8298a19150be)

### 3. What was the most common exclusion?

```sql
WITH exclusions1 AS
(SELECT
SUBSTRING_INDEX(SUBSTRING_INDEX(exclusions, ',', n.digit+1), ',',-1) AS exclusions
FROM pizza_runner.customer_orders
INNER JOIN 
(SELECT 0 digit UNION ALL SELECT 1 UNION ALL SELECT 2) n
 ON LENGTH(REPLACE(exclusions, ',' , '')) <= LENGTH(exclusions)-n.digit)

SELECT
a.exclusions, b.topping_name, COUNT(*) AS no_of_excluded
FROM exclusions1 AS a join pizza_runner.pizza_toppings AS b 
ON a.exclusions = b.topping_id
GROUP BY a.exclusions, b.topping_name
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/3f7ec3ce-b3f3-457d-bc2d-e5c516ef49c2)

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
### a. Meat Lovers
### b. Meat Lovers - Exclude Beef
### c. Meat Lovers - Extra Bacon
### d. Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```sql
SELECT
	tco.order_id,
  tco.pizza_id,
  pn.pizza_name,
  tco.exclusions,
  tco.extras,
  CASE
 WHEN tco.pizza_id = 1 AND tco.exclusions IS NULL AND tco.extras IS NULL THEN 'Meat Lovers'
 WHEN tco.pizza_id = 2 AND tco.exclusions IS NULL AND tco.extras IS NULL THEN 'Vegetarian'
 WHEN tco.pizza_id = 1 AND tco.exclusions = '4' AND tco.extras IS NULL THEN 'Meat Lovers - Exclude Cheese'
 WHEN tco.pizza_id = 2 AND tco.exclusions = '4' AND tco.extras IS NULL THEN 'Vegetarian - Exclude Cheese'
 WHEN tco.pizza_id = 1 AND tco.exclusions IS NULL AND tco.extras = '1' THEN 'Meat Lovers - Extra Bacon'
 WHEN tco.pizza_id = 2 AND tco.exclusions IS NULL AND tco.extras = '1' THEN 'Vegetarian - Extra Bacon'
 WHEN tco.pizza_id = 1 AND tco.exclusions = '4' AND tco.extras = '1, 5' THEN 'Meat Lovers - Exclude Cheese - Extra Bacon and Chicken'
 WHEN tco.pizza_id = 1 AND tco.exclusions = '2, 6' AND tco.extras = '1, 4' THEN 'Meat Lovers - Exclude BBQ Sauce and Mushroom - Extra Bacon and Cheese'
END AS order_item
FROM pizza_runner.customer_orders AS tco
JOIN pizza_runner.pizza_names AS pn ON tco.pizza_id = pn.pizza_id;
```

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/499d2cd6-7ff5-40ff-8fc1-d0474f00392d)
