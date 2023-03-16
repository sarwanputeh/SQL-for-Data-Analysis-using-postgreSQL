# SQL-for-Data-Analysis-using-postgreSQL
--HW2
--Creating a DB, Table and importing data
--1.	Open pgAdmin4
--2.	Create a database such as DS522DB
--3.	Create a table named orders
-- Table: public.orders

-- DROP TABLE IF EXISTS public.orders;

CREATE TABLE IF NOT EXISTS public.orders
(
    row_id integer NOT NULL,
    order_id text COLLATE pg_catalog."default",
    order_date date,
    delivery_date date,
    ship_mode text COLLATE pg_catalog."default",
    customer_id text COLLATE pg_catalog."default",
    customer_name text COLLATE pg_catalog."default",
    segment text COLLATE pg_catalog."default",
    country text COLLATE pg_catalog."default",
    city text COLLATE pg_catalog."default",
    state text COLLATE pg_catalog."default",
    postal_code text COLLATE pg_catalog."default",
    reqion text COLLATE pg_catalog."default",
    product_id text COLLATE pg_catalog."default",
    category text COLLATE pg_catalog."default",
    sub_category text COLLATE pg_catalog."default",
    product_name text COLLATE pg_catalog."default",
    sales text COLLATE pg_catalog."default",
    quantity numeric,
    discount numeric,
    profit text COLLATE pg_catalog."default",
    CONSTRAINT orders_pkey PRIMARY KEY (row_id)
)

TABLESPACE pg_default;

ALTER TABLE IF EXISTS public.orders
    OWNER to postgres;

--Q2
--If you encounter a problem with numeric data stored as text, solve the problem by changing the data type to numeric. (See the columns sales and profit)
--Hint: Read this https://help.openstreetmap.org/questions/37638/how-to-deal-with-commas-in-integers
--1.	Use Alter table and Update to create sales and profit column with proper data types: 
--แทนที่ Comma ด้วย nothing
--replace commas with nothing
update orders set sales = replace(sales,',','');
update orders set profit = replace(profit,',','');
--cast sales column as an numeric
Alter table orders ADD COLUMN sales2 numeric;
UPDATE orders SET sales2 =cast(sales as numeric) WHERE sales IS NOT NULL;
ALTER TABLE orders DROP COLUMN sales;
ALTER TABLE orders RENAME COLUMN sales2 TO sales;

--cast profit column as an numeric
ALTER TABLE orders ADD COLUMN profit2 numeric;
UPDATE orders SET profit2 = cast(profit as numeric) WHERE profit IS NOT NULL;
ALTER TABLE orders DROP COLUMN profit;
ALTER TABLE orders RENAME COLUMN profit2 TO profit;

--Q3 Create a delay column based on order date and delivery date.
ALTER TABLE orders ADD COLUMN delay integer

SELECT order_date,delivery_date,DATE_PART('day', AGE(delivery_date, order_date)) AS delay
FROM orders;

UPDATE orders
SET delay = DATE_PART('day', AGE(delivery_date, order_date));

--Q4 : Create a delay type column based on the following conditions.
--Fast: delay < 3
--Medium: 3 <= delay < 5
--Slow: delay >= 5

ALTER TABLE orders ADD COLUMN delay_type text;

UPDATE orders
SET delay_type = 
    CASE 
        WHEN delay < 3 THEN 'Fast'
        WHEN delay >= 3 AND delay < 5 THEN 'Medium'
        ELSE 'Slow'
    END;

--Q5Count the number of items classified by shipping speed (delay type).
SELECT delay_type, COUNT(*) AS Count
FROM orders
GROUP BY delay_type;
--Q6Question 6: Find the sum of sales and the sum of profit for each year.
SELECT DATE_TRUNC('year', order_date) AS year,
       SUM(sales) AS sum_of_sale,
       SUM(profit) AS sum_of_profit
FROM orders
GROUP BY year;

--Q7 Question 7: Find the total of sales and the total of profit for each subcategory in ascending order of profit

SELECT sub_category ,
       SUM(sales) AS sum_of_sale,
       SUM(profit) AS sum_of_profit
FROM orders
GROUP BY sub_category 
ORDER BY sum_of_profit ASC;
--Question 8: Who is the customer with the highest cumulative orders with the sum of the profit also shown?
SELECT customer_name, SUM(sales) AS num_orders, SUM(profit) AS total_profit
FROM orders
GROUP BY customer_name
ORDER BY SUM(sales) DESC, SUM(profit) DESC
LIMIT 3;
--Q9Find the top 3 most sold orders and show their profits. No need to group customers.
SELECT customer_name, sales, profit
FROM orders
ORDER BY sales DESC
LIMIT 3;
