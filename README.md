# Data Warehouse Modeling (Star Schema Example)
## Overview
This project demonstrates a simple star-schema data warehouse model for sales analytics using dummy data.
It defines dimension tables and a fact table, then showcases example analytics queries.
## Tech Stack
- SQL (PostgreSQL-style)
- Dimensional modeling (star schema)
## How It Works
- `dim_customer` stores customer attributes.
- `dim_date` stores date attributes.
- `fact_sales` stores metrics such as `sales_amount` and foreign keys to the dimensions.
## How to Run Locally with PostgreSQL
1. Install PostgreSQL (or use a Docker container).
2. Create a database, for example: `data_warehouse_demo`.
3. Run `create_tables.sql` in your SQL client to create the tables.
4. Insert some dummy rows into the dimensions and fact tables (manual INSERTs or small scripts).
5. Run `example_queries.sql` to validate that the schema works for analytics.
## Common Errors and Debugging Tips
**Error 1: Foreign key constraint failures**
- Cause: Attempting to insert into `fact_sales` with a `customer_key` or `date_key` that doesn't exist in
the dimension tables.
- Fix: Insert dimension rows first, then insert into `fact_sales`.
**Error 2: Table or column does not exist**
- Cause: Typos or running queries before creating tables.
- Fix: Re-run `create_tables.sql` and ensure names match exactly.
```
sql/create_tables.sql
```sql
CREATE TABLE dim_customer (
 customer_key SERIAL PRIMARY KEY,
 customer_id VARCHAR(50) NOT NULL,
 first_name VARCHAR(100),
 last_name VARCHAR(100),
 email VARCHAR(255),
 city VARCHAR(100),
 state VARCHAR(100),
 country VARCHAR(100),
 start_date DATE NOT NULL DEFAULT CURRENT_DATE,
 end_date DATE,
 is_current BOOLEAN DEFAULT TRUE
);
CREATE TABLE dim_date (
 date_key INTEGER PRIMARY KEY,
 full_date DATE NOT NULL,
 year INTEGER,
 month INTEGER,
 day INTEGER,
 month_name VARCHAR(20),
 day_of_week VARCHAR(20)
);
CREATE TABLE fact_sales (
 sales_key SERIAL PRIMARY KEY,
 customer_key INTEGER REFERENCES dim_customer(customer_key),
 date_key INTEGER REFERENCES dim_date(date_key),
 product_id VARCHAR(50),
 quantity INTEGER,
 sales_amount NUMERIC(10,2)
);
```
sql/example_queries.sql
```sql
-- Total sales by year
SELECT d.year, SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY d.year
ORDER BY d.year;
-- Top 10 customers by sales
SELECT c.customer_id, c.first_name, c.last_name,
 SUM(f.sales_amount) AS total_sales
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_sales DESC
LIMIT 10;
