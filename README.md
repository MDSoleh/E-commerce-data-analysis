# E-commerce-data-analysis
Data analysis of e-commerce retail data

Bussiness context:
Using the data from retail store to analyse the behaviour of customers and order patterns.

Data Availibility:
Three tables are used with names:

Customers (customer demographics)

Transactions (customer transaction details)

Product category (product category and sub-category information)

Data Analysis using SQL : 

-- --------BASIC Queries to use database and view tables within the database.Here db created is retail_salesl

use retail_salesl
select * from ecommerce_data
select * from prod_cat_info
select * from transactions

------------------------DATA PREPARATION AND UNDERSTANDING-------------------------------------------
-- ---- To calculate total number of rows present in tables of database retail_sales-----------------------

select count(*) from ecommerce_data union
select count(*) from prod_cat_info  union
select count(*) from transactions;

-- --for transactions in which we have returns that is profit 'sales - price column';

select count(*) from transactions
where total_amt < 0;














