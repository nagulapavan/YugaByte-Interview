# Installation of 3-node YugaByte cluster

### Create network
    docker network create yugabyte-network

### Start Node 1 
    docker run --name yugabyte-node1 \
      --network yugabyte-network \
      -p 15433:15433 -p 5434:5433 \
      yugabytedb/yugabyte:2025.2.3.0-b149 \
      bin/yugabyted start \
      --advertise_address=yugabyte-node1 \
      --cloud_location=aws.us-east-2.us-east-2a \
      --background=false

### Start Node 2 
    docker run -d --name yugabyte-node2 \
      --network yugabyte-network \
      yugabytedb/yugabyte:2025.2.3.0-b149 \
      bin/yugabyted start \
      --advertise_address=yugabyte-node2 \
      --join=yugabyte-node1 \
      --cloud_location=aws.us-east-2.us-east-2b \
      --background=false

### Start Node 3 
    docker run -d --name yugabyte-node3 \
      --network yugabyte-network \
      yugabytedb/yugabyte:2025.2.3.0-b149 \
      bin/yugabyted start \
      --advertise_address=yugabyte-node3 \
      --join=yugabyte-node1 \
      --cloud_location=aws.us-east-2.us-east-2c \
      --background=false
    
### Verify cluster - shows 3 ALIVE masters
    docker exec -it yugabyte-node1 \
      bin/yb-admin --master_addresses=yugabyte-node1:7100 list_all_masters
#### Results
    Master UUID                      	RPC Host/Port        	State    	Role 	Broadcast Host/Port 
    b2b6c4f7ec694e9c961ab25091f38f3f 	yugabyte-node1:7100  	ALIVE    	LEADER 	yugabyte-node1:7100 
    de987047f23a4217b2c5b8409c33cd75 	yugabyte-node2:7100  	ALIVE    	FOLLOWER 	yugabyte-node2:7100 
    acb67f99df754505aa4ec8390cf5ef5c 	yugabyte-node3:7100  	ALIVE    	FOLLOWER 	yugabyte-node3:7100 
### Connect via YSQL
    docker exec -it yugabyte-node1 bin/ysqlsh -h yugabyte-node1 -U yugabyte

# Create DB Objects and load data
### Create the Northwind database
    To create the Northwind database, run the following CREATE DATABASE statement.
    CREATE DATABASE northwind;
    Confirm that you have the Northwind database by listing the databases on your cluster.
    yugabyte=# \l
    Connect to the Northwind database.
    yugabyte=# \c northwind
### Build the tables and objects
    To build the tables and database objects, execute the northwind_ddl.sql SQL script.
    northwind=# \i share/northwind_ddl.sql
    You can verify that all 14 tables have been created by running the \d command.
    northwind=# \d

                List of relations
     Schema |          Name          | Type  | Owner
    --------+------------------------+-------+-------
     public | categories             | table | admin
     public | customer_customer_demo | table | admin
     public | customer_demographics  | table | admin
     public | customers              | table | admin
     public | employee_territories   | table | admin
     public | employees              | table | admin
     public | order_details          | table | admin
     public | orders                 | table | admin
     public | products               | table | admin
     public | region                 | table | admin
     public | shippers               | table | admin
     public | suppliers              | table | admin
     public | territories            | table | admin
     public | us_states              | table | admin
    (14 rows)
### Load the sample data
    To load the northwind database with sample data, run the \i command to execute commands in the northwind_data.sql file.

    northwind=# \i share/northwind_data.sql

    To verify that you have some data to work with, you can run a simple SELECT statement to pull data from the customers table.

    northwind=# SELECT * FROM customers LIMIT 2;

     customer_id |       company_name        | contact_name |    contact_title    |      address       |   city    | region | postal_code | country |     phone     |     fax
    -------------+---------------------------+--------------+---------------------+--------------------+-----------+--------+-------------+---------+---------------+-------------
     FAMIA       | Familia Arquibaldo        | Aria Cruz    | Marketing Assistant | Rua Orós, 92       | Sao Paulo | SP     | 05442-030   | Brazil  | (11) 555-9857 |
     VINET       | Vins et alcools Chevalier | Paul Henriot | Accounting Manager  | 59 rue de l'Abbaye | Reims     |        | 51100       | France  | 26.47.15.10   | 26.47.15.11
    (2 rows)

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
