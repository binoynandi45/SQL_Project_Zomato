# SQL_Project_Zomato

DROP TABLE IF EXISTS goldusers_signup;
CREATE TABLE goldusers_signup (
    userid INTEGER,
    gold_signup_date DATE
);

INSERT INTO goldusers_signup(userid, gold_signup_date) 
VALUES 
(1, '2017-09-22'),
(3, '2017-04-21');

DROP TABLE IF EXISTS users;
CREATE TABLE users (
    userid INTEGER,
    signup_date DATE
);

INSERT INTO users(userid, signup_date) 
VALUES 
(1, '2014-09-02'),
(2, '2015-01-15'),
(3, '2014-04-11');

DROP TABLE IF EXISTS sales;
CREATE TABLE sales (
    userid INTEGER,
    created_date DATE,
    product_id INTEGER
);

INSERT INTO sales(userid, created_date, product_id) 
VALUES 
(1, '2017-04-19', 2),
(3, '2019-12-18', 1),
(2, '2020-07-20', 3),
(1, '2019-10-23', 2),
(1, '2018-03-19', 3),
(3, '2016-12-20', 2),
(1, '2016-11-09', 1),
(1, '2016-05-20', 3),
(2, '2017-09-24', 1),
(1, '2017-03-11', 2),
(1, '2016-03-11', 1),
(3, '2016-11-10', 1),
(3, '2017-12-07', 2),
(3, '2016-12-15', 2),
(2, '2017-11-08', 2),
(2, '2018-09-10', 3);

DROP TABLE IF EXISTS product;
CREATE TABLE product (
    product_id INTEGER,
    product_name TEXT,
    price INTEGER
);

INSERT INTO product(product_id, product_name, price) 
VALUES
(1, 'p1', 980),
(2, 'p2', 870),
(3, 'p3', 330);

-- Verify the contents of each table
SELECT * FROM sales;
SELECT * FROM product;
SELECT * FROM goldusers_signup;


1. Total amount spend by each customer?

select s.userid, sum(price) as totalcost
from sales s
Left join Product P
on s.product_id=p.product_id
group by s.userid

2. How many days each customer visited zomato?

 select userid, Count(distinct created_date) as no_of_days_visited
 from sales
 group by userid
 
 3.What was the first product purcahsed by each customer?
 
 Select s.userid, Min(s.created_date) as first_date_brought, p.product_name
 from sales s
 Join product p
 on s.product_id=p.product_id
 group by s.userid, p.product_name
 
 
 select *
 from 
 (select *, rank() Over( partition by userid order by created_date asc) as Ranking
 from sales) A
 where A.ranking=1
 
 4. what is the most purchsed item by the customers?
 
 select p.product_id, count(s.userid) as No_of_times_purchased
 from sales s
Join product p on s.product_id=p.product_id
group by p.product_id

5. which item was most popular for each customers?

Select *
from (select userid, product_id,
rank() Over(partition by userid order by cnt desc) as ranking
from 
(select s.userid,s.product_id, count(*) as cnt
from sales s
join product p
on s.product_id=p.product_id
group by s.userid,s.product_id) A ) B
where ranking =1

 6. which item was first purchased after membership?
Select *,
from
( select A.*, 
 rank() over( partition by userid order by created_date asc) as ranking
 from
 (select *
 from sales s
 join goldusers_signup g
 on s.userid=g.userid and s.created_date >= g.gold_signup_date) A) D)
 where ranking =1;
 
7. which item was first purchased before membership?
Select *,
from
( select A.*, 
 rank() over( partition by userid order by created_date desc) as ranking
 from
 (select *
 from sales s
 join goldusers_signup g
 on s.userid=g.userid and s.created_date <= g.gold_signup_date) A) D)
 where ranking =1;
 
 8. what is the total order and amount spend by each member before they became member?
 
 
 Select A.userid, sum(A.price) as total_amount, count(A.userid) as No_of_item_brought
 from
 (select s.userid, s.product_id, p.price, g.gold_signup_date
 from sales s
 inner Join product p on s.product_id=p.product_id
 inner Join goldusers_signup g
 on s.userid=g.userid 
 and s.created_date <=g.gold_signup_date) A
 group by A.userid
 
 
 9. For buying each product generates point 
 p1-5rs- 2, p2-10rs - 5pt, p3- 5rs 1 pt
 calculate points collected by each customer and for which products most points given?
 
 ( select e.*, amt,points
 from
 (select A.*,
 Case when product_1=1 then 2 when product_1=2 then 2.5 when product_1=3 then 1 else 0 as Points
from 
( select s.product_id, p.product_name,sum(p.price) as total_price
 from sales s
 Join product p
 On s.product_id=p.product_id
 group by p.product_id, p.product_name
 order by p.product_id ) A) D);
 
 
 10. In the first one year after a customer joins the gold memebership. What is the point earned.
 
 
Select A.*,p.price, p.price*0.5 as zomato_points
from 
(select s.userid, s.product_id, s.created_date, g.gold_signup_date
 from sales s
 join goldusers_signup g
 on s.userid=g.userid and s.created_date >= g.gold_signup_date 
 and s.created_date <= DATE_ADD(g.gold_signup_date, INTERVAL 1 YEAR)) A
 Join product p on A.product_id=p.product_id
 
 11. rank all the transcations of customers?
 
 select *, rank() over(partition by s.userid order by s.created_date desc) as Ranking
 from sales s
 Join product p on s.product_id=p.product_id
 
