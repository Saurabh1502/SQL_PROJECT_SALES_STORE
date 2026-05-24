# SQL_PROJECT_SALES_STORE
it's normal SQL project using sql server, we will write sql query using advance funcion like window funtion, subquery
bulk insert, group by,order by,format function etc.

# PROBLEM SOLVING 
Q1 find out top 5 selling prodcut by quantity 
Q2 find out which product is mostly canceled
Q3 what time on the date has highest purchase
Q4 who are the top spending customer
Q5 which product category is generate highest revenue
Q6 what is the return/canellation rate at product category
Q7 what is most prefered payment mode
Q8 how does age group effect purchasing behaviour
Q 9 find out monthly sales trend
Q10 Are certain gender buying more product



----------------- project 1 Sales Store Analysis for SQL server ------------------


/* you can create the table with attached sales_store_table_script file. for that 
you just need to execute that query on server that will geneate table for you or
follow below step to create table */

/*

NOTE - Below question and query is for learning purpose please do not follow as it is
if required in your talbel then only apply or leave it.

*/

-------------------- creating table ----------------------------
create table Sales_store (
transaction_id varchar(15)
,customer_id varchar(15)
,customer_name  varchar(15)
,customer_age int
,gender varchar(15)
,product_id varchar(15)
,product_name varchar(15)
,product_category varchar(15)
,quantiy int
,prce float
,payment_mode varchar(15)
,purchase_date date
,time_of_purchase time 
,status varchar(15)
)

---------- update data type as varchar 25 for customer name -------------

alter table Sales_store alter  column customer_name varchar(25)

set dateformat dmy ------------- set the dd-mm-yy formate for purchase date column
bulk insert Sales_store
from 'C:\Users\91996\Downloads\archive\sales_store_updated_allign_with_video.csv' -- put your CSV file path
with (
firstrow =2,
fieldterminator=',',
rowterminator='\n',
tablock
)

select * from Sales_store

----------- data cleaning ---------

/* Before data cleaning we should take backup of original file and do our data cleaning 
activity on backup table */

select * into sales_store_BKP from Sales_store -- for backup run this query

/* check all table inserted succefully */

-- step 1  /* if it's show zero record then data is match */
select * from sales_store_BKP
except
select * from Sales_store

-- step 2  /* if count is match then data is match */
select count(*) from sales_store_BKP
select count(*) from sales_store

----------- checking for duplicate value in table --------

select transaction_id, count(*) as duplicate
from Sales_store 
group by transaction_id having count(*)>1

----------- find duplicate record which have to remove ----------

with CTE as (
select *, row_number() over(partition by transaction_id order by transaction_id) as ranking
from sales_store
)
select * from CTE where ranking >1

----------------- remove duplicate record --------- 

with CTE as (
select *, row_number() over(partition by transaction_id order by transaction_id) as ranking
from sales_store
)
delete from CTE where ranking>1

----- change wrong column name with correct one 

exec sp_rename 'sales_store.quantiy','Quantity', 'column'
exec sp_rename 'sales_store.prce','Price', 'column'

select * from sales_store

---- check table datatype 

select COLUMN_NAME,DATA_TYPE from INFORMATION_SCHEMA.COLUMNS
Where TABLE_NAME='Sales_store'

----- to check null value -----

DECLARE @TableName NVARCHAR(255) = 'sales_store';
DECLARE @sql NVARCHAR(MAX) = '';

SELECT @sql = STRING_AGG(
    'SELECT ''' + COLUMN_NAME + ''' AS [ColumnName], 
     COUNT(*) - COUNT(' + QUOTENAME(COLUMN_NAME) + ') AS [NullCount] 
     FROM ' + QUOTENAME(TABLE_SCHEMA) + '.' + QUOTENAME(sales_store_BKPsales_store_BKP),
    ' UNION ALL '
)
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = @TableName;

EXEC sp_executesql @sql;

------- treating null value ---------------

select * from sales_store
where 
transaction_id is null or 
customer_id is null or 
customer_name is null or 
customer_age is null or 
gender is null

------ 
/* this data is not useful hence deleted */
delete from Sales_store
where transaction_id is null 

select * from Sales_store
where customer_id='CUST1003'

--- updating customer_id 
/* please check null column and update if posible or delete the record */

begin transaction 

update Sales_store set customer_id='CUST9494' 
where transaction_id='TXN977900'

update Sales_store set customer_name='Mahika Saini',customer_age= 35, gender='Male'
where transaction_id='TXN432798'

commit; 
------------- checking gender data --------

select gender 
from sales_store 
group by gender

-- if you find differnt type of value in gender column then please update it

update Sales_store set gender ='Female' 
where gender ='F'

---------- checking data for payment mode ------

select payment_mode 
from Sales_store 
group by payment_mode

update sales_store set payment_mode ='Credit Card' 
where payment_mode='CC'


------ Q1 find out top 5 selling prodcut by quantity ------

select * from Sales_store


select top 5 product_name, sum(Quantity) 
from Sales_store
where status='delivered'
group by product_name
order by sum(Quantity) desc

------ Q2 find out which product is mostly canceled -------

select distinct status from Sales_store

select top 5 product_name,count(*) from Sales_store
where status='cancelled'
group by product_name
order by count(status) desc


------ Q3 what time on the date has highest purchase -----------

select 
case 
when datepart(hour,time_of_purchase) between 6 and 12 then '6:00 AM to 12:00 PM '
when datepart(hour,time_of_purchase) between 13 and 18 then '1:00 PM to 6:00  PM '
when datepart(hour,time_of_purchase) between 19 and 23 then '7:00 PM to 12:00 PM '
else '12:00 AM to 5:00 AM' end as [purchase_time]
,count(*) as [number of user]
from Sales_store
group by 
case 
when datepart(hour,time_of_purchase) between 6 and 12 then '6:00 AM to 12:00 PM '
when datepart(hour,time_of_purchase) between 13 and 18 then '1:00 PM to 6:00  PM '
when datepart(hour,time_of_purchase) between 19 and 23 then '7:00 PM to 12:00 PM '
else '12:00 AM to 5:00 AM' end
order by count(*) desc


------ Q4 who are the top spending customer ----------

select top 5 customer_name,format(sum(price*quantity),'C0','en-in') 
from Sales_store
group by customer_id, customer_name
order by sum(price*quantity) desc

-------- Q5 which product category is generate highest revenue -------

select product_category, format(sum(Quantity*Price),'C0','en-IN') [revenue]
from Sales_store
group by product_category
order by sum(Quantity*price) desc

---------- Q6 what is the return/canellation rate at product category ---------

select product_category, format(count(case when status='cancelled' then 1 end)*100.0/count(*),'N2')+'%'
from Sales_store
group by product_category

/* checking how many ordered is cancelled, pending or delivered for eatch product category */

select product_category,count(case when status='cancelled' then 1 end) [cancelled] 
,count(case when status='pending' then 1 end) pending 
,count(case when status='delivered' then 1 end) delivered 
from Sales_store
group by product_category

----- Q7 what is most prefered payment mode -------

select payment_mode,count(1) 
from Sales_store
group by payment_mode
order by count(1) desc

---- Q8 how does age group effect purchasing behaviour ---

select min(customer_age),max(customer_age) from Sales_store

select 
case when customer_age between 18 and 28 then '18 to 28'
when customer_age between 29 and 40 then '29 to 40'
when customer_age between 41 and 60 then '40 to 60'
end [age grouping ] ,
/*displaying with indain rupees sign with the help of format function */
format(sum(Quantity*Price),'C0','en-in') [Total spending] 
from Sales_store
group by
case when customer_age between 18 and 28 then '18 to 28'
when customer_age between 29 and 40 then '29 to 40'
when customer_age between 41 and 60 then '40 to 60'
end
order by sum(Quantity*Price) desc

-------- Q9 find out monthly sales trend -----

select format(purchase_date,'yyyy-MM') , 
sum(Quantity*Price), sum(Quantity) 
from Sales_store
group by format(purchase_date,'yyyy-MM')

--- Q10 are certain gender buying more product ---

-- method 1 

select product_category,gender, count(product_name) from Sales_store
group by gender, product_category

-- method 2 pivot method

select * 
from (select gender,product_category from sales_store) as source_table
pivot(
 count(gender) for gender in([MALE],[FEMALE]) 
 ) as pivot_table
order by product_category
