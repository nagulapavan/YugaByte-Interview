# YugaByte-Interview

This repo contain Docker commands to install 3 node cluster and set of YB commands to create database and load data. Also 5 SQLs to answer the queries

## 1. Sales reps from Seattle or Redmond:
sqlSELECT first_name, last_name
FROM employees
WHERE city IN ('Seattle', 'Redmond')
AND title = 'Sales Representative';
## 2. Total units ordered for product ID 3:
sqlSELECT SUM(quantity) AS total_units
FROM order_details
WHERE product_id = 3;
## 3. City name and number of employees in each city:
sqlSELECT city, COUNT(*) AS num_employees
FROM employees
GROUP BY city
ORDER BY num_employees DESC;
## 4. Companies that placed orders in 1997:
sqlSELECT DISTINCT c.company_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1997
ORDER BY c.company_name;
## 5. All orders made by employees:
sqlSELECT e.first_name, e.last_name, o.order_id, o.order_date, o.ship_country
FROM employees e
JOIN orders o ON e.employee_id = o.employee_id
ORDER BY e.last_name, o.order_date;
