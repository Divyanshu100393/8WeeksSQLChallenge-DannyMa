## 1. What is the total amount each customer spent at the restaurant?
SELECT <br/> 
&nbsp;&nbsp;&nbsp;&nbsp; s.customer_id AS "Customer ID", <br/> 
&nbsp;&nbsp;&nbsp;&nbsp; SUM(m.price) AS "Total Spent" <br/> 
FROM dannys_diner.sales AS s <br/> 
JOIN dannys_diner.menu AS m <br/> 
ON s.product_id = m.product_id <br/> 
GROUP BY s.customer_id<br/> 


|Customer ID                          |Total Spent                         |
|-------------------------------|-----------------------------|
|B         |74            |
|C            |36            |
|A|76|

<br/>

## 2. How many days has each customer visited the restaurant?
SELECT <br/>
&nbsp;&nbsp;&nbsp;&nbsp; customer_id AS "Customer ID", <br/>
&nbsp;&nbsp;&nbsp;&nbsp; COUNT(DISTINCT(order_date))  AS "Number of Days Visited <br/>
FROM dannys_diner.sales <br/>
GROUP BY customer_id <br/>

|Customer ID                          |Total Visits                         |
|-------------------------------|-----------------------------|
|A         |4            |
|B            |6            |
|C|2|

<br/>

## 3. What was the first item from the menu purchased by each customer?
SELECT <br/>
&nbsp;&nbsp;&nbsp;&nbsp; t.customer_id AS Customer, <br/>
&nbsp;&nbsp;&nbsp;&nbsp; t.order_date AS "Order Date", <br/>
&nbsp;&nbsp;&nbsp;&nbsp; m.product_name AS "Order item"  <br/>
FROM (<br/>
&nbsp;&nbsp;&nbsp;&nbsp; SELECT customer_id, <br/>
&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;  product_id, <br/> 
&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; order_date, <br/>
&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date ASC) AS "ROW NUMBER" <br/>
&nbsp;&nbsp;&nbsp;&nbsp; FROM dannys_diner.sales) t <br/>
JOIN dannys_diner.menu AS m <br/>
ON t.product_id = m.product_id <br/>
WHERE "ROW NUMBER" = 1 <br/>

| customer | Order Date               | Order item |
| -------- | ------------------------ | ---------- |
| A        | 2021-01-01T00:00:00.000Z | sushi      |
| B        | 2021-01-01T00:00:00.000Z | curry      |
| C        | 2021-01-01T00:00:00.000Z | ramen      |

<br />

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT <br/>
&nbsp;&nbsp;&nbsp;&nbsp; m.product_name AS "Menu Item",<br/>
&nbsp;&nbsp;&nbsp;&nbsp; MAX(t.maxprod) as "Quantity" <br/>
&nbsp;&nbsp;&nbsp;&nbsp; FROM <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(SELECT <br/>
    	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DISTINCT ON (product_id) product_id, <br/>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;COUNT(product_id) OVER(PARTITION BY product_id) AS maxprod <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FROM dannys_diner.sales) t <br/>
JOIN dannys_diner.menu AS m <br/>
ON t.product_id = m.product_id <br/>
GROUP BY product_name <br/>
LIMIT 1 <br/>

| Menu Item | Quantity |
| --------- | -------- |
| ramen     | 8        |



##-- 6. Which item was purchased first by the customer after they became a member?

SELECT  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;t.customer_id AS Customer,  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;t.order_date AS "Order Date",  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;m.product_name AS "Order item"   <br/>
FROM ( <br/>
&nbsp;&nbsp;&nbsp;&nbsp;SELECT  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; s.customer_id,  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s.product_id,  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s.order_date,  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS "ROW NUMBER"  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;FROM dannys_diner.sales AS s  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;JOIN dannys_diner.members AS mem  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;ON s.customer_id = mem.customer_id  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;WHERE s.order_date >= mem.join_date) t  <br/>
JOIN dannys_diner.menu AS m  <br/>
ON t.product_id = m.product_id  <br/>
WHERE "ROW NUMBER" = 1 <br/>

| customer | Order Date               | Order item |
| -------- | ------------------------ | ---------- |
| B        | 2021-01-11T00:00:00.000Z | sushi      |
| A        | 2021-01-07T00:00:00.000Z | curry      |



## 8. -- 7. Which item was purchased just before the customer became a member?

SELECT <br/>
&nbsp;&nbsp;&nbsp;&nbsp;t.customer_id AS Customer,<br/>
&nbsp;&nbsp;&nbsp;&nbsp;t.order_date AS "Order Date", <br/>
&nbsp;&nbsp;&nbsp;&nbsp;m.product_name AS "Order item" <br/>
&nbsp;&nbsp;&nbsp;&nbsp;FROM ( <br/>
&nbsp;&nbsp;&nbsp;&nbsp;SELECT <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s.customer_id, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s.product_id, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s.order_date, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS "ROW NUMBER" <br/>
&nbsp;&nbsp;&nbsp;&nbsp;FROM dannys_diner.sales AS s <br/>
&nbsp;&nbsp;&nbsp;&nbsp;JOIN dannys_diner.members AS mem <br/>
&nbsp;&nbsp;&nbsp;&nbsp;ON s.customer_id = mem.customer_id <br/>
&nbsp;&nbsp;&nbsp;&nbsp;WHERE s.order_date < mem.join_date) t <br/>
JOIN dannys_diner.menu AS m <br/>
ON t.product_id = m.product_id <br/>
WHERE "ROW NUMBER" = 1 <br/>

| customer | Order Date               | Order item |
| -------- | ------------------------ | ---------- |
| B        | 2021-01-04T00:00:00.000Z | sushi      |
| A        | 2021-01-01T00:00:00.000Z | sushi      |

## -- 8. What is the total items and amount spent for each member before they became a member?
SELECT  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;s.customer_id AS "Customer ID", <br/>
&nbsp;&nbsp;&nbsp;&nbsp;SUM(m.price) AS "Total Spent",  <br/>
&nbsp;&nbsp;&nbsp;&nbsp;COUNT(m.price) AS "Total Orders"  <br/>
FROM dannys_diner.sales AS s  <br/>
JOIN dannys_diner.menu AS m  <br/>
ON s.product_id = m.product_id  <br/>
JOIN dannys_diner.members AS mem  <br/>
ON s.customer_id = mem.customer_id  <br/>
WHERE s.order_date < mem.join_date  <br/>
GROUP BY s.customer_id  <br/>

| Customer ID | Total Spent | Total Orders |
| ----------- | ----------- | ------------ |
| B           | 40          | 3            |
| A           | 25          | 2            |

