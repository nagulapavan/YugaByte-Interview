# YugaByte-Interview

This repo contain Docker commands to install 3 node cluster and set of YB commands to create database and load data. Also 5 SQLs to answer the queries
# Docker commands to install 3 node YB cluster
Here are all the Docker commands that worked for your 3-node YugabyteDB cluster:
bash# Add Docker to PATH
export PATH="$PATH:/Applications/Docker.app/Contents/Resources/bin"

# Create network
docker network create yugabyte-network

# Start Node 1 (foreground - keep this terminal open)
docker run --name yugabyte-node1 \
  --network yugabyte-network \
  -p 15433:15433 -p 5434:5433 \
  yugabytedb/yugabyte:2025.2.3.0-b149 \
  bin/yugabyted start \
  --advertise_address=yugabyte-node1 \
  --cloud_location=aws.us-east-2.us-east-2a \
  --background=false

# Start Node 2 (new terminal)
docker run -d --name yugabyte-node2 \
  --network yugabyte-network \
  yugabytedb/yugabyte:2025.2.3.0-b149 \
  bin/yugabyted start \
  --advertise_address=yugabyte-node2 \
  --join=yugabyte-node1 \
  --cloud_location=aws.us-east-2.us-east-2b \
  --background=false

# Start Node 3 (new terminal)
docker run -d --name yugabyte-node3 \
  --network yugabyte-network \
  yugabytedb/yugabyte:2025.2.3.0-b149 \
  bin/yugabyted start \
  --advertise_address=yugabyte-node3 \
  --join=yugabyte-node1 \
  --cloud_location=aws.us-east-2.us-east-2c \
  --background=false

# Verify cluster - should show 3 ALIVE masters
docker exec -it yugabyte-node1 \
  bin/yb-admin --master_addresses=yugabyte-node1:7100 list_all_masters

# Connect via YSQL
docker exec -it yugabyte-node1 bin/ysqlsh -h yugabyte-node1 -U yugabyte

# SQLs
## 1. Sales reps from Seattle or Redmond:
SELECT first_name, last_name
FROM employees
WHERE city IN ('Seattle', 'Redmond')
AND title = 'Sales Representative';
## 2. Total units ordered for product ID 3:
SELECT SUM(quantity) AS total_units
FROM order_details
WHERE product_id = 3;
## 3. City name and number of employees in each city:
SELECT city, COUNT(*) AS num_employees
FROM employees
GROUP BY city
ORDER BY num_employees DESC;
## 4. Companies that placed orders in 1997:
SELECT DISTINCT c.company_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE EXTRACT(YEAR FROM o.order_date) = 1997
ORDER BY c.company_name;
## 5. All orders made by employees:
SELECT e.first_name, e.last_name, o.order_id, o.order_date, o.ship_country
FROM employees e
JOIN orders o ON e.employee_id = o.employee_id
ORDER BY e.last_name, o.order_date;
