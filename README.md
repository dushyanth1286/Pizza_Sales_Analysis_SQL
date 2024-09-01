1. Identify the most common pizza size ordered.
SELECT 
    p.size, COUNT(o.Order_details_id) max_size_ordered
FROM
    pizzas p
        JOIN
    order_details o ON p.pizza_id = o.pizza_id
GROUP BY 1
ORDER BY COUNT(o.Order_details_id) DESC
LIMIT 1;
-----------------------------------------------------------------------------------------------------------------------------------------------
2. Calculate the total revenue generated from pizza sales.
SELECT 
    ROUND(SUM(p.price * o.quantity), 2) Total_revenue
FROM
    pizzas p
        JOIN
    order_details o ON p.pizza_id = o.pizza_id;
-----------------------------------------------------------------------------------------------------------------------------------------------
3. Identify the highest-priced pizza.
SELECT 
    pt.name, p.price
FROM
    pizzas p
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
ORDER BY p.price DESC
LIMIT 1;
-----------------------------------------------------------------------------------------------------------------------------------------------
4. List the top 5 most ordered pizza types along with their quantities.
SELECT 
    pt.name, SUM(od.quantity) total_quantity
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON od.Pizza_id = p.pizza_id
GROUP BY pt.name
ORDER BY SUM(od.quantity) DESC
LIMIT 5;
-----------------------------------------------------------------------------------------------------------------------------------------------
5. Find the total quantity of each pizza category ordered.
SELECT 
    pt.category, SUM(od.quantity) total_quantity
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON od.Pizza_id = p.pizza_id
GROUP BY pt.category
ORDER BY SUM(od.quantity) DESC;
-----------------------------------------------------------------------------------------------------------------------------------------------
6. Determine the distribution of orders by hour of the day.
SELECT 
    HOUR(order_time) hour_day, COUNT(order_id) order_count
FROM
    orders
GROUP BY HOUR(order_time)
ORDER BY COUNT(order_id) DESC;
-----------------------------------------------------------------------------------------------------------------------------------------------
7. Group the orders by date and calculate the average number of pizzas ordered per day.
SELECT ROUND(AVG(sum_orders), 2) AS avg_orders
FROM (
    SELECT o.Order_date, SUM(od.quantity) AS sum_orders
    FROM 
        orders o 
    JOIN 
        order_details od ON o.Order_id = od.Order_id
    GROUP BY  
        o.Order_date
) ord;
-----------------------------------------------------------------------------------------------------------------------------------------------
8. Determine the top 3 most ordered pizza types based on revenue.
SELECT pt.name, SUM(od.quantity * p.price) rev
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.Pizza_type_id = p.Pizza_type_id
        JOIN
    order_details od ON od.Pizza_id = p.Pizza_id
GROUP BY pt.name
ORDER BY SUM(od.quantity * p.price) DESC
LIMIT 3;
-----------------------------------------------------------------------------------------------------------------------------------------------
10.  Calculate the percentage contribution of each pizza type to total revenue.
WITH cte AS (
    SELECT pt.category, SUM(od.quantity * p.price) AS rev
    FROM 
        pizza_types pt
    JOIN 
        pizzas p ON pt.pizza_type_id = p.pizza_type_id
    JOIN 
        order_details od ON p.pizza_id = od.pizza_id
    GROUP BY pt.category
)
SELECT category, 
    ROUND(rev / (SELECT SUM(rev) FROM cte) * 100.0, 2) AS percent_cont
FROM 
    cte
ORDER BY percent_cont DESC;
-----------------------------------------------------------------------------------------------------------------------------------------------
11. Analyze the cumulative revenue generated over time.
WITH cte AS (
    SELECT o.order_date,
        SUM(od.quantity * p.price) AS sum_rev
    FROM 
        pizzas p
    JOIN 
        order_details od ON od.Pizza_id = p.Pizza_id
    JOIN 
        orders o ON od.Order_id = o.Order_id
    GROUP BY 
        o.order_date
)
SELECT order_date, 
    ROUND(SUM(sum_rev) OVER (ORDER BY order_date), 0) AS rev_gen
FROM 
    cte
ORDER BY rev_gen DESC;
-----------------------------------------------------------------------------------------------------------------------------------------------
12. WITH cte AS (
    SELECT pt.category, pt.name, ROUND(SUM(od.quantity * p.price), 0) AS revenue
    FROM 
        pizza_types pt
    JOIN 
        pizzas p ON pt.Pizza_type_id = p.Pizza_type_id
    JOIN 
        order_details od ON od.Pizza_id = p.Pizza_id
    GROUP BY pt.category, pt.name
    ORDER BY SUM(od.quantity * p.price) DESC
),
cte2 AS (
    SELECT category, name, revenue, DENSE_RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
    FROM 
        cte
)
SELECT 
    category, name, revenue
FROM 
    cte2
WHERE rn <= 3;
