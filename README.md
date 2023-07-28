# SQL-Data-Cleaning-Data-Exploration---Fasoos-Roll-Metrics

**Created a database for Fasoos.**
pgAdmnin-->server-->right click-->create database--> name the database

****Next step is to create tables to the database. Below are the details for the same:**

CREATE TABLE driver(driver_id integer,reg_date date); 

INSERT INTO driver(driver_id,reg_date) 
 VALUES (1,'01-01-2021'),(2,'01-03-2021'),(3,'01-08-2021'),(4,'01-15-2021');

CREATE TABLE ingredients(ingredients_id integer,ingredients_name varchar(60)); 

INSERT INTO ingredients(ingredients_id ,ingredients_name) 
 VALUES (1,'BBQ Chicken'),(2,'Chilli Sauce'),(3,'Chicken'),(4,'Cheese'),(5,'Kebab'),(6,'Mushrooms'),(7,'Onions'),(8,'Egg'),(9,'Peppers'),(10,'schezwan sauce'),
(11,'Tomatoes'),(12,'Tomato Sauce');

CREATE TABLE rolls(roll_id integer,roll_name varchar(30)); 

INSERT INTO rolls(roll_id ,roll_name) 
 VALUES (1	,'Non Veg Roll'),(2	,'Veg Roll');

CREATE TABLE rolls_recipes(roll_id integer,ingredients varchar(24)); 

INSERT INTO rolls_recipes(roll_id ,ingredients) 
 VALUES (1,'1,2,3,4,5,6,8,10'),(2,'4,6,7,9,11,12');

CREATE TABLE driver_order(order_id integer,driver_id integer,pickup_time datetime,distance VARCHAR(7),duration VARCHAR(10),cancellation VARCHAR(23));
INSERT INTO driver_order(order_id,driver_id,pickup_time,distance,duration,cancellation) 
 VALUES(1,1,'01-01-2021 18:15:34','20km','32 minutes',''),(2,1,'01-01-2021 19:10:54','20km','27 minutes',''),(3,1,'01-03-2021 00:12:37','13.4km','20 mins','NaN'),
(4,2,'01-04-2021 13:53:03','23.4','40','NaN'),(5,3,'01-08-2021 21:10:57','10','15','NaN'),(6,3,null,null,null,'Cancellation'),
(7,2,'01-08-2020 21:30:45','25km','25mins',null),(8,2,'01-10-2020 00:15:02','23.4 km','15 minute',null),(9,2,null,null,null,'Customer Cancellation'),
(10,1,'01-11-2020 18:50:20','10km','10minutes',null);

CREATE TABLE customer_orders(order_id integer,customer_id integer,roll_id integer,not_include_items VARCHAR(4),extra_items_included VARCHAR(4),order_date datetime);
INSERT INTO customer_orders(order_id,customer_id,roll_id,not_include_items,extra_items_included,order_date)
VALUES (1,101,1,'','','01-01-2021  18:05:02'),(2,101,1,'','','01-01-2021 19:00:52'),(3,102,1,'','','01-02-2021 23:51:23'),(3,102,2,'','NaN','01-02-2021 23:51:23'),
(4,103,1,'4','','01-04-2021 13:23:46'),(4,103,1,'4','','01-04-2021 13:23:46'),(4,103,2,'4','','01-04-2021 13:23:46'),(5,104,1,null,'1','01-08-2021 21:00:29'),
(6,101,2,null,null,'01-08-2021 21:03:13'),(7,105,2,null,'1','01-08-2021 21:20:29'),(8,102,1,null,null,'01-09-2021 23:54:33'),(9,103,1,'4','1,5','01-10-2021 11:22:59'),
(10,104,1,null,null,'01-11-2021 18:34:49'),(10,104,1,'2,6','1,4','01-11-2021 18:34:49');

**So I basically tried to code few roll metrics which would help in analysing and take important business decisons. Below mentioned are the questions and the code that I used:**

**1. How many rolls were ordered?**

select
count(roll_id)
from customer_orders

**2. How many rolls were ordered in each order?**

select
order_id,
count(roll_id)
from customer_orders
group by order_id

**3. How many unique customer orders were made?**

select
count(distinct(customer_id))
from customer_orders

**4. How many unique orders were placed by each customer?**

select
customer_id,
count(distinct(order_id))
from customer_orders
group by customer_id

**5. How many successful orders were delivered by each driver?**

with order_table as (select
*,
case when cancellation in ('Cancellation','Customer Cancellation') then 'c' else 'nc' end as order_cancel_details
from driver_order)

select
driver_id,
count(distinct (order_id))
from order_table
where order_cancel_details in ('nc')
group by driver_id

**6. How many of each type of roll was delivered?**

with order_table as (select
*,
case when cancellation in ('Cancellation','Customer Cancellation') then 'c' else 'nc' end as order_cancel_details
from driver_order)

select
c.roll_id,
count(c.roll_id)
from customer_orders c
inner join order_table o
on c.order_id = o.order_id
where o.order_cancel_details = 'nc'
group by c.roll_id

**7. How many veg and non-veg rolls were ordered by each customer?**

select
c.customer_id,
r.roll_name,
count(c.roll_id)
from customer_orders as c
inner join rolls as r
on c.roll_id = r.roll_id
group by c.customer_id,r.roll_name
order by c.customer_id,r.roll_name ASC

**8. What was the maximum number of rolls delivered in a single order?**

with order_table as (select
*,
case when cancellation in ('Cancellation','Customer Cancellation') then 'c' else 'nc' end as order_cancel_details
from driver_order)

select
o.order_id,
count(c.roll_id) as total_rolls
from order_table o
inner join customer_orders c
on o.order_id = c.order_id
where o.order_cancel_details = 'nc'
group by o.order_id
order by total_rolls DESC
limit 1

**9. For each customer, how many delivered rolls had at least 1 change and how many had no change?**

with change_in_order as (select
*,
case 
when not_include_items is null then 'Not Applicable' 
when not_include_items = '' then 'Not Applicable' 
else 'Change' end as not_included_changes,
case
when extra_items_included is null then 'Not Applicable' 
when extra_items_included = '' then 'Not Applicable' 
when extra_items_included = 'NaN' then 'Not Applicable'
else 'Change' end as extra_included_changes						 
from customer_orders),

atleat_one_change as (select
*,
case 
when not_included_changes = 'Change' then 'min_change'
when extra_included_changes = 'Change' then 'min_change'
when not_included_changes = 'Not Applicable' then 'no_change'
when extra_included_changes = 'Not Applicable' then 'no_change'
end as change_of_order
from change_in_order),

order_table as (select
*,
case when cancellation in ('Cancellation','Customer Cancellation') then 'c' else 'nc' end as order_cancel_details
from driver_order),

table1 as(select
at.customer_id,
at.order_id,
at.not_included_changes,
at.extra_included_changes
from atleast_one_change as at
inner join order_table as o
on at.order_id = o.order_id
where order_cancel_details = 'nc'
order by customer_id, order_id ASC)

select
customer_id,
order_id,
case
when not_included_changes = 'Not Applicable' and extra_included_changes = 'Not Applicable' then 'No Change'
when not_included_changes = 'Not Applicable' or extra_included_changes = 'Change' then 'Change'
when not_included_changes = 'Change' or extra_included_changes = 'Not Applicable' then 'Change'
else '' end as sorted
from table1
order by customer_id, order_id, sorted ASC

**10. How many rolls were delivered that had both exclusions and extras?**

with table1 as(select
*,
case 
when not_include_items is null then '' 
when not_include_items = '' then '' 
else 'not_include_items' end as not_included_changes,
case
when extra_items_included is null then '' 
when extra_items_included = '' then '' 
when extra_items_included = 'NaN' then ''
else 'extra_items_included' end as extra_included_changes						 
from customer_orders),

table2 as(select
customer_id,
order_id,
count(roll_id) as count_of_orders_with_exclusion_inclusion
from table1
where not_included_changes != '' and extra_included_changes != ''
group by customer_id,order_id),

order_table as (select
*,
case when cancellation in ('Cancellation','Customer Cancellation') then 'c' else 'nc' end as order_cancel_details
from driver_order)

select
t.customer_id,
t.order_id,
t.count_of_orders_with_exclusion_inclusion
from table2 t
inner join order_table o
on t.order_id=o.order_id
where order_cancel_details = 'nc'


**11. What was the total number of rolls ordered for each hour of the day?**

select
concat(DATE_PART('hour', order_date),'-',DATE_PART('hour', order_date)+1) as hour_bucket,
count(roll_id) as total_rolls_ordered
from customer_orders
group by hour_bucket


**12. What was the number of orders for each day of the week?**


with table1 as(select
*,
to_char(order_date,'Day') as day_of_week
from customer_orders)

select
day_of_week,
count(distinct order_id)
from table1
group by day_of_week
