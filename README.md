# Pizza Sales Analysis SQL Project

---

## Project Overview

- **Project Titles:** Pizza Sales Analysis  
- **Level:** Beginner  

This project is designed to demonstrate SQL skilss and techniques typically used by data analyst to explore, clean, and analyze pizza sales data.
This project involves setting up a pizza sales databse,performing exploratory data analysis (EDA), and answering specific business questions through 
queries. 

--- 

## Objectives

- Analyze the distribution of orders across different pizza categories and sizes.
- Apply advanced SQL techniques (joins, CTEs, and window functions) to generate actionable business insights.
- Identify top-performing and low-performing pizzas, sizes, and categories based on quantity and revenue.
- Analyze overall business performance by calculating total orders, total revenue, and key sales metrics.

--- 

## Dataset

The data for this project is sourced from the Maven dataset:

- **Dataset Link:** [Pizza Dataset](https://mavenanalytics.io/data-playground/pizza-place-sales)

--- 

## Schema

```sql
create table orders (
order_id int not null,
order_date date not null ,
order_time time not null,
primary key(order_id));

create table order_details (
order_details_id int not null,
order_id int not null ,
pizza_id text not null,    
quantity int not null ,
primary key(order_details_id));
```
---

## Business Problems and Solutions

 ## 1. Retrieve the total number of orders placed.

 ```sql
SELECT 
    COUNT(*) AS Total_orders;
```
**Objective:** Determine the distribution of content types on Netflix.  

## 2. Calculate the total revenue generated from pizza sales.

 ```sql
SELECT 
    round(sum(o.quantity*p.price),2)  AS Total_revenue
FROM
    order_details o
	JOIN
    pizzas p ON o.pizza_id = o.pizza_id;
 ```
**Objective:** Evaluate total business income. 

##  3. Identify the highest-priced pizza.

 ```sql
  SELECT 
    pt.name AS Pizza, p.price AS Price
FROM
    pizza_types pt
       left  JOIN
    pizzas p ON p.pizza_type_id = pt.pizza_type_id
    order by Price desc
    limit 1;
 ```

**Objective:** Identify premium-priced products.

##  4. Identify the most common pizza size ordered.

 ```sql
  SELECT 
    p.size AS Most_Common_Size,
    COUNT(o.order_details_id) AS Total_orders
FROM
    pizzas p
        JOIN
    order_details o ON o.pizza_id = p.pizza_id
GROUP BY p.size
ORDER BY Total_orders DESC
LIMIT 1;
 ```

**Objective:** To determine which pizza size is ordered most frequently so production and inventory can be planned accordingly. 

## 5. List the top 5 most ordered pizza types along with their quantities
   
```sql
 Select pt.name As Pizza , count(o.order_details_id) as Total_orders 
from order_details o 
join pizzas p
on p.pizza_id= o.pizza_id
join pizza_types pt 
on pt.pizza_type_id= p.pizza_type_id
group by Pizza 
order by Total_orders desc
limit 5;
 ```

**Objective:** To identify the five most popular pizza types based on order frequency and focus on high-demand products.  

 ## 6. How does total revenue change month-wise?-
```sql
SELECT 
    EXTRACT(MONTH FROM o.order_date) AS month,
    SUM(od.quantity * p.price) AS revenue
FROM orders o
JOIN order_details od 
    ON o.order_id = od.order_id
JOIN pizzas p 
    ON od.pizza_id = p.pizza_id
GROUP BY month
ORDER BY month;
 ```

**Objective:** To analyze how revenue changes each month and detect seasonal or time-based sales patterns.  

## 7. Rank pizzas based on revenue.
```sql
SELECT 
    pt.name,
    SUM(od.quantity * p.price) AS revenue,
    RANK() OVER (ORDER BY SUM(od.quantity * p.price) DESC) AS rank_no
FROM order_details od
JOIN pizzas p 
    ON od.pizza_id = p.pizza_id
JOIN pizza_types pt 
    ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name;
 ```

**Objective:**  To rank pizzas by total revenue in order to compare their financial contribution to the business.

## 8. Join the necessary tables to find the total quantity of each pizza category ordered.
```sql
Select pt.category as Pizza_Category , sum(o.quantity) as Total_Quantity 
from order_details o 
join pizzas p
on p.pizza_id= o.pizza_id
join pizza_types pt 
on pt.pizza_type_id= p.pizza_type_id
group by Pizza_Category
order by Total_Quantity desc;
 ```

**Objective:** To calculate how many pizzas are sold in each category so I can evaluate category-level demand.

## 9. Determine the distribution of orders by hour of the day.
```sql

Select hour(order_time) as Hour_of_day , count(order_id) as Total_orders
from orders 
group by Hour_of_day
order by Total_orders desc;
 ```

**Objective:** Identify peak ordering hours.  

## 10. Join relevant tables to find the category-wise distribution of pizzas.
```sql
SELECT 
    category, COUNT(name) AS Pizza_distribution
FROM
    pizza_types
GROUP BY category; 
 ```

**Objective:** o understand how many pizza types exist in each category and evaluate menu balance.

## 11. Group the orders by date and calculate the average number of pizzas ordered per day.

```sql
SELECT 
    o.order_date, SUM(od.Quantity) AS Total_pizzas
FROM
    orders o
        JOIN
    order_details od ON od.order_id = o.order_id
GROUP BY order_date
ORDER BY Total_pizzas DESC;
 ```

**Objective:** To calculate how many pizzas are ordered each day in order to analyze daily sales volume.

## 12. Find the total revenue generated by each pizza and display only those pizzas whose revenue is more than 5000.

```sql
WITH pizza_revenue AS (
    SELECT 
        pt.name AS pizza_name,
        SUM(od.quantity * p.price) AS total_revenue
    FROM order_details od
    JOIN pizzas p 
        ON od.pizza_id = p.pizza_id
    JOIN pizza_types pt 
        ON p.pizza_type_id = pt.pizza_type_id
    GROUP BY pt.name
)

SELECT 
    pizza_name,
    total_revenue
FROM pizza_revenue
WHERE total_revenue > 5000
ORDER BY total_revenue DESC;
 ```

**Objective:** To identify pizzas that generate significant revenue and prioritize them in business strategy.  

## 13. Determine the top 3 most ordered pizza types based on revenue.
```sql
SELECT 
    pizza_types.name AS Pizza,
    SUM(order_details.quantity * pizzas.price) AS Total_revenue
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id
        JOIN
    pizza_types ON pizza_types.pizza_type_id = pizzas.pizza_type_id
GROUP BY Pizza
ORDER BY Total_revenue DESC;
 ```

**Objective:** To find the most profitable pizza types based on total revenue for marketing and promotion purposes.  


## 14. Calculate the percentage contribution of each pizza type to total revenue.

```sql
SELECT 
    pt.name AS pizza_type,
    ROUND(
        SUM(od.quantity * p.price) 
        / (SELECT SUM(od2.quantity * p2.price)
           FROM order_details od2
           JOIN pizzas p2 ON od2.pizza_id = p2.pizza_id
          ) * 100, 
        2
    ) AS revenue_percentage
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY revenue_percentage DESC;
 ```

**Objective:** To measure how much each pizza type contributes to total revenue and assess product importance.  

## 15. Analyze the cumulative revenue generated over time.

```sql

SELECT
    order_date,
    ROUND(
        SUM(daily_revenue) OVER (ORDER BY order_date),
        2
    ) AS cumulative_revenue
FROM (
    SELECT 
        o.order_date,
        SUM(od.quantity * p.price) AS daily_revenue
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    GROUP BY o.order_date
) t
ORDER BY order_date;
 ```

**Objective:** To track how total revenue grows over time and evaluate long-term business performance.  

## 16. Determine the top 3 most ordered pizza types based on revenue for each pizza category.
```sql
SELECT
    category,
    pizza_type,
    revenue
FROM (
    SELECT
        pt.category,
        pt.name AS pizza_type,
        SUM(od.quantity * p.price) AS revenue,
        DENSE_RANK() OVER (
            PARTITION BY pt.category
            ORDER BY SUM(od.quantity * p.price) DESC
        ) AS rank_in_category
    FROM order_details od
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
    GROUP BY pt.category, pt.name
) ranked_pizzas
WHERE rank_in_category <= 3
ORDER BY category, revenue DESC;
 ```

**Objective:**  To identify the highest-performing pizzas within each category to optimize product placement.  

## 17. Find total orders per day and show only days with more than 100 orders.

```sql
WITH daily_orders AS (
    SELECT 
        order_date,
        COUNT(DISTINCT order_id) AS total_orders
    FROM orders
    GROUP BY order_date
)

SELECT *
FROM daily_orders
WHERE total_orders > 100
ORDER BY total_orders DESC;
 ```

**Objective:** To detect unusually busy days in order to improve operational planning and resource allocation.

## 18. Rank months based on total revenue.

```sql
SELECT 
    EXTRACT(MONTH FROM o.order_date) AS month,
    SUM(od.quantity * p.price) AS revenue,
    RANK() OVER (ORDER BY SUM(od.quantity * p.price) DESC) AS month_rank
FROM orders o
JOIN order_details od 
    ON o.order_id = od.order_id
JOIN pizzas p 
    ON od.pizza_id = p.pizza_id
GROUP BY month;
 ```

**Objective:** To rank months based on revenue so strong and weak sales periods can be compared.

---


## Findings and Conclusion

- **Sales Distribution:** The dataset reveals variouspizza sales based on distinct pizza types, sizes, and categories, depicting the various preferences of customers.
- **Top-Performing:** Analysis of order frequency and revenue indicates which pizza is most popular and which pizza generates the most revenue.
- **Time-Based Insights:** Monthly and hourly trends facilitate the identification of sales peaks and seasonal sales fluctuations.
- **Category and Size Analysis:** In addition, certain categories and pizza size tend to generate large sales in terms of quantity and money, implying customers are buying these types of pizzas.


  This analysis will give a comprehensive picture of how well pizzas are selling, which can be used to make decisions.





  
