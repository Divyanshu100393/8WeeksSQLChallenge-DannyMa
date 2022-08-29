## 1. What is the total amount each customer spent at the restaurant?
select 
	s.customer_id as "Customer ID",
    SUM(m.price) as "Total Spent"
from dannys_diner.sales as s
join dannys_diner.menu as m 
on s.product_id = m.product_id 
group by s.customer_id


|Customer ID                          |Total Spent                         |
|-------------------------------|-----------------------------|
|B         |74            |
|C            |36            |
|A|76|
