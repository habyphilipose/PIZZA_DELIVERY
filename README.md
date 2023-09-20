# PIZZA_DELIVERY
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

## Entity Relationship Diagram

![image](https://github.com/habyphilipose/PIZZA_DELIVERY/assets/31076902/bab9688c-cd90-4d48-ae7f-449a61deac82)

