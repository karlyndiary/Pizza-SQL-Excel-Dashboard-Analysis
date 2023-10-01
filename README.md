# Pizza SQL & Excel Analysis

![pizza cover](https://user-images.githubusercontent.com/116041695/234172419-28b8f5ed-425d-477f-9586-44c2a86d456e.jpg)

## Table of Content

* [Background](#background)
* [Data Summary](#data-summary)
* [ER Diagram](#er-diagram)
* [Queries and Insights](#queries-and-insights)
* [Excel Dashboard](#excel-dashboard)

## Background
For the Maven Pizza Challenge, you’ll be playing the role of a BI Consultant hired by Plato's Pizza, a Greek-inspired pizza place in New Jersey. You've been hired to help the restaurant use data to improve operations, and just received the following note:

Welcome aboard, we're glad you're here to help!

Things are going OK here at Plato's, but there's room for improvement. We've been collecting transactional data for the past year, but really haven't been able to put it to good use. Hoping you can analyze the data and put together a report to help us find opportunities to drive more sales and work more efficiently.

Thanks in advance,

Mario Maven (Manager, Plato's Pizza)

## Data Summary

The dataset is found on [Maven Analytics](https://www.mavenanalytics.io/data-playground) under the name Pizza Place Sales with a total of 4 tables with 48,620 records and 12 fields.

- Order Details

  The table has the order_details_id which is the primary key of the table along with the order_id, pizza_id as the foreign key of the orders and pizzas table and last, we have the quantity column of each type of pizza.

- Orders

  This table includes the order_id which is the primary key, and the date and time of each order.

- Pizza Types

  We have the pizza_type_id as the primary key, along with each pizza's name, category and ingredients.

- Pizzas

  The pizzas table has the pizza_id as the primary key, and the pizza_type_id as the foreign key from the pizza types table, it also includes the size and price of the pizzas.

## ER Diagram

![Pizza ER Diagram](https://user-images.githubusercontent.com/116041695/234453942-3df6eb5c-52cb-4386-a385-bec29cd8e060.png)

## Queries and Insights

### 1. What is the average number of customers per day?
```
SELECT count(order_id)/count(distinct date) as average_customers_per_day
FROM pizza.dbo.orders;
```
The average daily customer traffic is 59.

### 2. How many pizzas are typically in an order?
```
SELECT COUNT(order_details_id)/COUNT(distinct order_id) as avg_no_of_pizzas_per_order
FROM pizza.dbo.order_details;
```
On average, customers order 2 pizzas per order.

### 3. Do we have any bestsellers?
```
SELECT pt.name AS pizza_name, ROUND(SUM(p.price * od.quantity),2) AS revenue
FROM pizza.dbo.order_details od 
JOIN pizza.dbo.pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza.dbo.pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY revenue DESC; 
```
Based on sales, the three most popular pizzas are The Thai Chicken Pizza, The Barbecue Chicken Pizza, and The California Chicken Pizza.

### 4. How many pizzas are we making during peak hour?
```
SELECT DATEPART(HOUR, time) AS hours, 
count(distinct(o.order_id)) as total_orders, 
count(quantity) as total_quantity 
FROM pizza.dbo.orders o 
JOIN pizza.dbo.order_details od 
ON o.order_id = od.order_id
GROUP BY DATEPART(HOUR, time) 
ORDER BY total_orders DESC;
```
At 12 PM, the restaurant received 2520 orders, which consisted of 6543 pizzas in total.

### 5. Are there any peak days?
```
SELECT count(order_id) as orders, weekday
FROM pizza.dbo.orders
GROUP BY weekday 
ORDER BY orders DESC;
```
Friday, Thursday, and Saturday are the peak days.

### 6. Which month experienced the highest number of orders?
```
SELECT count(order_id) as orders, month
FROM pizza.dbo.orders
GROUP BY month 
ORDER BY orders DESC;
```
July had the highest number of orders of 1935.

### 7. What is the total revenue from 2015?
```
SELECT ROUND(SUM(p.price * od.quantity),2) AS Revenue
FROM  pizza.dbo.pizzas p
JOIN pizza.dbo.order_details od
ON p.pizza_id = od.pizza_id;
```
Total revenue for the year 2015 is $817860.05.

### 8. Which month had the highest revenue?
```
SELECT o.month, ROUND(SUM(p.price),2) AS total_price
FROM pizza.dbo.order_details od
JOIN pizza.dbo.orders o ON od.order_id = o.order_id
JOIN pizza.dbo.pizzas p ON od.pizza_id = p.pizza_id
GROUP BY o.month
ORDER BY total_price DESC;
```
July had the highest revenue of $71027.45.

### 9. What are top-selling pizza sizes?
```
SELECT p.size as size, SUM(od.quantity) AS quantity
FROM pizza.dbo.pizza_types pt
JOIN pizza.dbo.pizzas p ON pt.pizza_type_id = p.pizza_type_id
JOIN pizza.dbo.order_details od ON p.pizza_id = od.pizza_id
GROUP BY size
ORDER BY quantity DESC;
```
The large size pizza received the most orders, totaling 18,956.

### 10. What pizza categories are ordered?
```
SELECT pt.category, SUM(od.quantity) AS quantity
FROM pizza.dbo.pizza_types pt
JOIN pizza.dbo.pizzas p ON pt.pizza_type_id = p.pizza_type_id
JOIN pizza.dbo.order_details od ON p.pizza_id = od.pizza_id
GROUP BY pt.category
ORDER BY quantity DESC;
```
There are 4 different categories consisting of Classic, Supreme, Veggie and Chicken.

### 11. What is the average order value?
```
SELECT ROUND(SUM(od.quantity * p.price) / COUNT(DISTINCT o.order_id),2) AS avg_order_value
FROM pizza.dbo.orders o
JOIN pizza.dbo.order_details od ON o.order_id = od.order_id
JOIN pizza.dbo.pizzas p ON od.pizza_id = p.pizza_id;
```
Average order value is $38.31.

### 12. What is the average price per pizza?
```
SELECT ROUND(AVG(price),2) AS average_price_per_pizza
FROM pizza.dbo.pizzas;
```
Average price per pizza is $16.44.

### 13. How many orders in total?
```
SELECT count(order_id) AS no_of_orders
FROM pizza.dbo.orders;
```
Total number of orders is 21350.

### 14. What is the quantity of pizzas sold?
```
SELECT count(quantity) as quantity
FROM pizza.dbo.order_details;
```
The total number of pizzas sold is 48,620.

### 15. How many different pizza varieties are on the menu?
```
SELECT count(*) AS pizza_varieties 
FROM pizza.dbo.pizza_types;
```
There are a total of 32 pizza varieties on the menu.

### 16. Which ingredients are the most popularly ordered?
```
SELECT DISTINCT(TRIM(value)) AS ingredient, COUNT(TRIM(value)) AS count
FROM pizza.dbo.pizza_types pt
JOIN pizza.dbo.pizzas p ON pt.pizza_type_id = p.pizza_type_id
JOIN pizza.dbo.order_details od ON p.pizza_id = od.pizza_id
CROSS APPLY STRING_SPLIT(pt.ingredients, ',') AS ingredients_split
GROUP BY TRIM(value)
ORDER BY count DESC;
```
The most popularly ordered ingredients are Garlic, Tomatoes, Red Onions, Red Peppers, and Mozzarella Cheese.

### 17. What is the distribution of order quantities by time of day?
```
SELECT
    CASE
        WHEN o.time BETWEEN '00:00:00' AND '11:59:59' THEN 'Morning'
        WHEN o.time BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
        WHEN o.time BETWEEN '18:00:00' AND '23:59:59' THEN 'Evening'
   	    ELSE 'Night'
    END AS time_of_day,
    SUM(od.quantity) AS total_quantity
FROM
    pizza.dbo.orders o
JOIN
    pizza.dbo.order_details od
ON o.order_id = od.order_id
GROUP BY
    CASE
        WHEN o.time BETWEEN '00:00:00' AND '11:59:59' THEN 'Morning'
        WHEN o.time BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
        WHEN o.time BETWEEN '18:00:00' AND '23:59:59' THEN 'Evening'
   	    ELSE 'Night'
    END
ORDER BY
    time_of_day;
```
The quantity of orders is highest in the afternoon with 29,468, followed by the evening with 17,356, and the morning with 2,750.

## Excel Dashboard
![Pizza Dashboard](https://user-images.githubusercontent.com/116041695/235843398-097bdeb8-22f9-4e8d-8a60-882037f5f182.png)
