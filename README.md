### Data_Analyst_Prep

1.) Write a query to transform the data so that each fruite becomes its own column and     values should be "Yes" if person has that fruit in their basket otherwise "No"
```sql
CREATE TABLE baskets (
    Person VARCHAR(10),
    Basket VARCHAR(100)
);
INSERT INTO baskets (Person, Basket) VALUES
('A', 'Apple,Mango,Orange'),
('B', 'Apple'),
('C', 'Guava,Cherry'),
('D', 'Mango,Cherry,Orange');

--- by using CASE WHEN 

SELECT Person,
CASE WHEN Basket LIKE "%Apple" THEN "Yes" ELSE "No" END AS "Apple",
CASE WHEN Basket LIKE "%Mango" THEN "Yes" ELSE "No" END AS "Mango",
CASE WHEN Basket LIKE "%Orange" THEN "Yes" ELSE "No" END AS "Orange",
CASE WHEN Basket LIKE "%Guava" THEN "Yes" ELSE "No" END AS "Guava",
CASE WHEN Basket LIKE "%Cherry" THEN "Yes" ELSE "No" END AS "Cherry"
FROM baskets


--- second approach by using find_in_set() function 

 SELECT Person,
 IF(find_in_set("Apple",Basket)>0,"Yes","No") AS "Apple",
 IF(find_in_set("Mango",Basket)>0,"Yes","No") AS "Mango",
 IF(find_in_set("Orange",Basket)>0,"Yes","No") AS "Orange",
 IF(find_in_set("Guava",Basket)>0,"Yes","No") AS "Guava",
 IF(find_in_set("Cherry",Basket)>0,"Yes","No") AS "Cherry"
 FROM baskets


 ```


2.) Write a query to return the capital gain/loss for each stock

The capital GAIN/LOSS of a stock is the total gain or loss after buying and selling the stock one or many times return the result table ordered by capital_gain_loss in descending order

```sql
CREATE TABLE stocks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    stock_name VARCHAR(50),
    operation ENUM('Buy', 'Sell'),
    operation_day INT,
    price INT
);

INSERT INTO stocks (stock_name, operation, operation_day, price) VALUES
('Apple', 'Buy', 1, 1500),
('Tesla', 'Buy', 2, 1200),
('Apple', 'Sell', 5, 5000),
('Samsung', 'Buy', 17, 20000),
('Tesla', 'Sell', 3, 1300),
('Tesla', 'Buy', 4, 1500),
('Tesla', 'Sell', 5, 1100),
('Tesla', 'Buy', 6, 1400),
('Samsung', 'Sell', 29, 15000),
('Tesla', 'Sell', 10, 1200); 


--- using row_number() 

WITH cte1 AS (
    SELECT 
        stock_name,
        operation,
        operation_day,
        price,
        ROW_NUMBER() OVER(PARTITION BY stock_name, operation ORDER BY operation_day) AS rnk
    FROM stocks
),
cte2 AS (
    SELECT 
        c1.stock_name,
        c2.price - c1.price AS stock_gain_loss
    FROM cte1 AS c1
    JOIN cte1 AS c2
        ON c1.rnk = c2.rnk
       AND c1.stock_name = c2.stock_name
       AND c1.operation = 'Buy'
       AND c2.operation = 'Sell'
)
SELECT 
    stock_name,
    SUM(stock_gain_loss) AS gain_loss
FROM cte2
GROUP BY stock_name
ORDER BY stock_name;



--- by using case when statement

SELECT 
stock_name,
sum(
case 
when operation="Buy" then -1*price
when operation="Sell" then price
end) as capital_gain_loss
from stocks
GROUP BY stock_name
ORDER BY stock_name;
```

3.)Write a query to find moving average of how much the customer paid in seven days window ( current day+6 day before )  return the result table ordered by visited_on in ascending order and the average_amount should be rounded to two decimal place

```sql
CREATE TABLE customer (
    customer_id INT NOT NULL,
    name VARCHAR(50) NOT NULL,
    visited_on DATE NOT NULL,
    amount DECIMAL(10,2) NOT NULL
);
INSERT INTO customer (customer_id, name, visited_on, amount) VALUES
(1, 'Jhon',    '2019-01-01', 100),
(2, 'Daniel',  '2019-01-02', 110),
(3, 'Jade',    '2019-01-03', 120),
(4, 'Khaled',  '2019-01-04', 130),
(5, 'Winston', '2019-01-05', 110),
(6, 'Elvis',   '2019-01-06', 140),
(7, 'Anna',    '2019-01-07', 150),
(8, 'Maria',   '2019-01-08', 80),
(9, 'Jaze',    '2019-01-09', 110),
(1, 'Jhon',    '2019-01-10', 130),
(3, 'Jade',    '2019-01-10', 150);


with cte as(
select visited_on,sum(amount) as total_amount
from customer
group by visited_on
),
cte2 as(
select
visited_on,
sum(total_amount) over(order by visited_on rows between 6 preceding and current row) as 7_day_rolling_sum,
round(avg(total_amount) over( order by visited_on rows between 6 preceding and current row),2) as 7_day_rolling_avg
from 
cte)
select * 
from cte2
limit 10 offset 6
```

4.)You are given a table named NUM with two columns SN->serial number and NUMB->number
Find all unique values in NUMB that form a "sandwich pattern" in other word the same value should be appear at rows I and i+2 while the row in between that is i+1 should contain different value

```sql
CREATE TABLE NUM (
    SN INT,
    NUMB INT
);

-- Step 2: Insert sample data
INSERT INTO NUM (SN, NUMB) VALUES
(1, 4),
(2, 7),
(3, 4),
(4, 9),
(5, 9),
(6, 7),
(7, 9),
(8, 4);

with cte as(
select
SN,NUMB as "first_numb",
lead(NUMB) over() as "second_numb",
lead(NUMB,2) over() as "third_numb"
from NUM)
select SN,first_numb FROM cte 
where first_numb=third_numb and second_numb!=first_numb 
```

5.)Find users with above average conversion probability

Calculate the conversion rate(probability) for each variant (A and B) and list users who had a higher probability of converting based on their variant compared to the overall average conversion rate.

Conversion rate = total conversions / total users

```sql
CREATE TABLE ab_test (
    user_id INT,
    variant CHAR(1),
    converted TINYINT
);
INSERT INTO ab_test (user_id, variant, converted) VALUES
(1, 'A', 1),
(2, 'A', 0),
(3, 'A', 1),
(4, 'A', 0),
(5, 'B', 1),
(6, 'B', 1),
(7, 'B', 1),
(8, 'B', 0);

with overall as(
select sum(converted)/count(*) overall_conversion_rate
from ab_test
),
variant as(
select variant,sum(converted)/count(*) variant_conversion_rate
from ab_test
group by variant
)

select 
ab.variant,
v.variant_conversion_rate,
overall_conversion_rate
from ab_test ab 
join variant v on ab.variant=v.variant
cross join overall 
where v.variant_conversion_rate>overall_conversion_rate
```

6.)Match combiations write a query to generate all possible unique match between the teams in the format (team1 vs team2)

Each pair should be appear only once means KKR vs CSK is valid but CSK vs KKR is not valid

Team cannot play with itself like CSK vs CSK

```sql
CREATE TABLE teams (
    team_name VARCHAR(50) NOT NULL
);

-- Insert team names
INSERT INTO teams (team_name) VALUES 
('CSK'),
('KKR'),
('GT'),
('DC'),
('LSG');

select 
concat(t1.team_name," vs ",t2.team_name) as matches
from teams t1 
join teams t2 on t1.team_name<t2.team_name
```


7.)Write a query that outputs the name of each credit card and the difference in the number of issued card between the month with the highest issuance card and the number of lowest issuance arrange the result based on the largest disparity.

```sql
CREATE TABLE credit_card_issuance (
    card_name VARCHAR(50) NOT NULL,
    issued_amount INT NOT NULL,
    issue_month TINYINT NOT NULL CHECK (issue_month BETWEEN 1 AND 12),
    issue_year YEAR NOT NULL,
    PRIMARY KEY (card_name, issue_month, issue_year)
);

-- Insert the data
INSERT INTO credit_card_issuance (card_name, issued_amount, issue_month, issue_year) VALUES
('Chase Freedom Flex', 55000, 1, 2021),
('Chase Freedom Flex', 60000, 2, 2021),
('Chase Freedom Flex', 65000, 3, 2021),
('Chase Freedom Flex', 70000, 4, 2021),
('Chase Sapphire Reserve', 170000, 1, 2021),
('Chase Sapphire Reserve', 175000, 2, 2021),
('Chase Sapphire Reserve', 180000, 3,2021)
;

select card_name,max(issued_amount)-min(issued_amount) as difference
from credit_card_issuance
group by card_name
order by difference desc
```

8.)
```sql
CREATE TABLE transactions (
    transaction_id INT PRIMARY KEY,
    user_id INT NOT NULL,
    transaction_date DATE NOT NULL,
    transaction_amount DECIMAL(10, 2) NOT NULL
);

-- Insert the data into the table
INSERT INTO transactions (transaction_id, user_id, transaction_date, transaction_amount)
VALUES
    (1, 269, '2018-08-15', 500),
    (2, 478, '2018-11-25', 400),
    (3, 269, '2019-01-05', 1000),
    (4, 123, '2020-10-20', 600),
    (5, 478, '2021-07-05', 700),
    (6, 123, '2022-03-05', 900);
    
SELECT 
    YEAR(transaction_date) AS year,
    user_id,
    ROUND(AVG(transaction_amount), 2) AS avg_amount
FROM transactions
WHERE YEAR(transaction_date) BETWEEN 2018 AND 2022
GROUP BY YEAR(transaction_date), user_id;
```






