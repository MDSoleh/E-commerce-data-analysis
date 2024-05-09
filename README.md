# E-commerce-data-analysis
Data analysis of e-commerce retail data

# Bussiness context:
Using the data from retail store to analyse the behaviour of customers and order patterns.

# Data Availibility:
Three tables are used with names:

Customers (customer demographics)

Transactions (customer transaction details)

Product category (product category and sub-category information)


# -- -----BASIC SQL CASE STUDY-----

CREATE DATABASE P_Retail_sales;
USE P_Retail_sales;

select * from p_retail_sales.customers;
select * from p_retail_sales.prod_cat_info;
select * from p_retail_sales.transactions;
select * from p_retail_sales.transactions1;

# -- ------DATA PREPARATION AND UNDERSTANDING-----

-- #1. Total number of rows in each table of the database:

select count(*) as Cust_rows from p_retail_sales.customers union
select count(*) as Prod_rows from p_retail_sales.prod_cat_info union
select count(*) as tran_rows from p_retail_sales.transactions;

-- #2. Number of transactions that have a return:

select count(transaction_id) as tran_returns from p_retail_sales.transactions
where total_amt <0 

-- #3. date variables into valid date format:

-- update transactions set tran_date = str_to_date(tran_date,'%d-%m-%y')
-- WHERE tran_date IS NOT NULL AND tran_date != '';

-- #4. Time range of transaction data available for analysis:

select distinct year(tran_date) from p_retail_sales.transactions1;

select distinct month(tran_date) from p_retail_sales.transactions1

select tran_date, day(tran_date) as Days, month(tran_date) as Month, year(tran_date) as Years 
from p_retail_sales.Transactions1;

select tran_date, day(tran_date) as Date, month(tran_date) as Month_date from p_retail_sales.transactions1 
order by tran_date desc;

-- select datediff(year,'2011-01-25','2014-02-22') from transactions1;

-- SELECT  DATEDIFF(year,'2011-01-02','2014-12-02')as Years from transactions
-- limit 100,
-- DATEDIFF(month, '2011-01-02','2014-12-02')as Months,
-- DATEDIFF(day, '2011-01-02','2014-12-02')as Days from p_retail_sales.transactions1

-- #5. Sub-category DIY belongs to which product category

select prod_subcat, prod_cat from p_retail_sales.prod_cat_info
where prod_subcat ='DIY';

# --------------------------------------------------DATA ANALYSIS--------------------------------------------------

-- #1. Most frequently used channel for tansactions. 

select Store_type, count(store_type) as Count_of_stores 
from P_Retail_sales.Transactions
group by store_type 
order by Count_of_stores desc
Limit 1;

-- #2. Count of Male and Female customers in the database.

update P_Retail_sales.Customers
set gender = 'Unknown'
where gender = ' '

select gender, count(gender) as Total_count 
from P_Retail_sales.Customers
group by gender
order by 
case 
when gender = 'M' then 1
when gender = 'F' then 2
else 3
end;

-- #3. Maximim number of customers belong to which city.

select (city_code), count(city_code) as Count_city 
from P_Retail_sales.Customers
group by city_code
order by Count_city desc 
limit 1;

-- #4. Number of sub_categories under the Category Books.

select prod_cat, count(prod_subcat) as No_of_subcat  
from P_Retail_sales.prod_cat_info
where prod_cat = 'Books'
group by prod_cat  

-- #5. Maximum number of products ever ordered.

#join on transactions1 table with Prod_cat_info table using key column "prod_cat_code"

select (tran_date), count(prod_cat) as No_of_products 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P
on T.prod_cat_code = P.prod_cat_code  
group by tran_date     
order by No_of_products desc
Limit 1;

-- #6. Net total revenue generated from Books and Electronics category.

select sum(total_amt) as Total_amount 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P
on T.prod_subcat_code = P.prod_sub_cat_code
where prod_cat in ('Books','Electronics')

-- #7. Number of customers having more than 10 transactions,excluding returns.

with Alpha
as (select cust_id ,count(transaction_id)as Count_of_trans 
from P_Retail_sales.Transactions1
where total_amt > 0 
group by cust_id
having count(transaction_id)>10)
select count(cust_id) as Num_of_customers from Alpha

-- #8. Combined revenue earned from Electronics and Clothing categories from Flagship stores.

-- ~~~~ Product category and Product item count ~~~~~

select prod_subcat, count(prod_cat) as Item_count 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P 
on T. prod_cat_code = P.prod_cat_code
where prod_cat in ('Electronics','Clothing')
group by prod_subcat
order by Item_count desc;

-- ~~~~~~~ TOTAL REVENUE GENERATED ~~~~~~~
select sum(total_amt) as Total_Revenue 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P 
on T. prod_subcat_code = P.prod_sub_cat_code
where prod_cat in ('Electronics','Clothing') and store_type = 'Flagship store';

-- #9. Total revenue generated from Male customers in Electronics segment.{output w.r.t. prod_subcat}

select prod_subcat,sum(total_amt) as Amt_by_males 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.Customers as C
on T.cust_id = C.customer_Id
inner join P_Retail_sales.prod_cat_info as P 
on P.prod_cat_code = T.prod_cat_code
where gender = 'M' and prod_cat = 'Electronics'
group by prod_subcat;

-- #10. Percentage of sales and returns by product-sub-category.{TOP 5}

with perABS 
as(select (prod_subcat), 
    ABS(sum(case when Qty < 0 then Qty else 0 end)) as Returns, 
    sum(case when Qty > 0 then Qty else 0 end) as Sales
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P
on T.prod_cat_code = P.prod_cat_code
group by prod_subcat
order by Sales desc)

select prod_subcat,round(((Returns /(Returns + Sales))*100),2) as Return_percent,
round(((Sales /(Returns + Sales))*100),2) as Sales_percent from perABS
limit 5;

-- #11. Total revenue generated by customers aged between 25 to 30 years in the last 30 days of transaction

select  (tran_date) from P_Retail_sales.Transactions
group by tran_date
order by tran_date desc
limit 30;
ALTER TABLE customers MODIFY COLUMN DOB date;
                                                      
with Alpha
as(select tran_date,sum(total_amt) as Total_amount 
from P_Retail_sales.customers as C
inner join P_Retail_sales.Transactions1 as T
on T.cust_id = C.customer_id
where datediff(year,DOB,getdate()) between 25 and 35
group by tran_date
order by tran_date desc)
select sum(Total_amount) as Final_revenue from Alpha;

-- #12. Product category with max amount of returns in last three months.

select prod_cat,count(Qty) as No_of_returns
from P_Retail_sales.Transactions1 as T
inner join P_Retail_Sales.prod_cat_info as P
on T.prod_cat_code = P.prod_cat_code
where total_amt < 0 
and datediff(month,'2014-09-01',tran_date)= 3   
group by prod_cat ;

-- #13. Store-type selling maximum no of products in terms of sales_amount and no_of_products.

select (Store_type),count(Qty) as No_of_products,
sum(total_amt) as Sales_Amount from P_Retail_sales.Transactions1
where total_amt > 0
group by Store_type
order by No_of_products desc
limit 1;

-- #14. Product categories having average revenues greater than the overall average revenue.

select prod_cat,round(avg(total_amt), 2) as Averages 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P
on T.prod_cat_code = P.prod_cat_code
group by prod_cat
having avg(total_amt) > (select avg(total_amt) 
from P_Retail_sales.Transactions1)

-- #15. Average and total revenue by each subcategory belonging to top five categories as per quantity sold

select (prod_cat), count(Qty) as Quantity_sold 
from P_Retail_sales.Transactions1 as T
inner join P_Retail_sales.prod_cat_info as P
on T.prod_cat_code = T.prod_cat_code
where total_amt > 0
group by prod_cat
order by Quantity_sold desc
limit 5;

select prod_cat, prod_subcat,
round(sum(total_amt),3) as Total_amount, 
round(avg(total_amt),3) as Avg_amount 
from P_Retail_sales.Transactions1 as T 
inner join P_Retail_sales.prod_cat_info as P
on T.prod_cat_code = P.prod_cat_code
where total_amt > 0 and 
prod_cat in ('Books','Electronics','Home and kitchen','Footwear','Clothing') 
group by prod_cat, prod_subcat
order by 
case    when prod_cat = 'Books' then 1
        when prod_cat = 'Electronics' then 2
	when prod_cat = 'Home and kitchen' then 3
        when prod_cat = 'Footwear' then 4
	else 5
	end;

