# Hevo_Assignment2
ASSIGNMENT2
Assignment 2 focuses on transforming raw ingested data into structured, analytics-ready datasets using Hevo Models and Snowflake SQL.

##The objective was to:
Reuse the existing PostgreSQL → Hevo → Snowflake pipeline
Load new raw data
Create clean staging models
Build a final denormalized analytics model
Validate transformations in Snowflake
This assignment demonstrates data cleaning, standardization, relational joins, and business level aggregation.

##Architecture Layering
Raw Layer (Ingested from PostgreSQL)
->Clean Layer (Staging Models)
->Final Business Model (Analytics Ready)

1️.Raw Data Layer
The following raw tables were ingested into Snowflake via Hevo:
CUSTOMERS
ORDERS
PRODUCTS
FEEDBACK
ORDER_EVENTS (generated from transformer)

These tables represent operational source data.

2️. Clean Staging Models
To ensure modularity and maintainability, clean staging models were created before building the final business model.
customers_clean Model
Purpose:
Standardize and validate customer data.
Query:
SELECT
id AS customer_id,
TRIM(first_name) AS first_name,
TRIM(last_name) AS last_name,
LOWER(email) AS email,
username,
created_at
FROM RAW.CUSTOMERS
WHERE id IS NOT NULL;

Transformations Applied:
Renamed id → customer_id
Removed leading spaces
Standardized email format
Filtered invalid rows

#products_clean Model
Purpose:
Ensure product pricing and structure is consistent.

Query:
SELECT
id AS product_id,
TRIM(product_name) AS product_name,
category,
price::NUMBER(10,2) AS price
FROM RAW.PRODUCTS
WHERE price > 0;

Transformations Applied:
Renamed primary key
Standardized product name
Ensured numeric precision for price
Filtered invalid pricing records

##orders_clean Model
#Purpose:
Prepare order-level data for analytics and joins.
Query:
SELECT
id AS order_id,
customer_id,
product_id,
quantity,
amount,
LOWER(status) AS status,
order_date
FROM RAW.ORDERS
WHERE customer_id IS NOT NULL
AND product_id IS NOT NULL;

Transformations Applied:
Standardized status values

Removed incomplete records

Prepared for relational joins

3.Final Business Model

Objective
#Create a denormalized dataset combining:
Customer details
Product information
Order data
Transaction metrics
Final Model Query

SELECT
c.customer_id,
c.first_name,
c.last_name,
c.email,
p.product_name,
p.category,
o.order_id,
o.quantity,
o.amount,
o.status,
o.order_date
FROM customers_clean c
JOIN orders_clean o
ON c.customer_id = o.customer_id
JOIN products_clean p
ON o.product_id = p.product_id;

Optional Aggregation Model (Customer-Level Metrics)

SELECT
c.customer_id,
COUNT(o.order_id) AS total_orders,
SUM(o.amount) AS total_revenue
FROM customers_clean c
JOIN orders_clean o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id;

4. Validation in Snowflake
Record validation:
SELECT COUNT(*) FROM FINAL_MODEL;

Data inspection:
SELECT * FROM FINAL_MODEL LIMIT 10;

#Checks performed:
Row count logical and consistent
No duplicate inflation due to joins
Aggregated metrics accurate
No unexpected null values

#Design Principles Followed
Separation of raw and clean layers
Modular transformation design
Standardized schema naming
Explicit filtering and validation
Business-ready denormalized model
End-to-end verification

##Outcome
#This assignment demonstrates:
Data modeling best practices
Structured transformation layering
Warehouse-level validation
Business-oriented dataset design

#The final output is analytics ready and supports:
Revenue analysis
Customer-level metrics
Product performance insights
Order tracking

##Conclusion
Through Assignment 2, raw operational data was transformed into clean, structured, and analytics ready datasets using a layered modeling approach.



This reflects real-world data engineering practices where data quality, modularity, and maintainability are critical.

