# SQL Project 3 - Exploratory Data Analysis - Pizza Restaurant Database Analysis
## Microsoft Power BI Dashboard
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/9b444361-4ba1-4f4f-8cdb-837f3cc4a860)

## Complete code attached - Pizza Restaurant Database Analysis.sql

## Building the Database.
```
-- Creating a database. 
-- Pizza Restraunt Database.
create database PizzaRestrauntDatabase
GO

-- Selecting the Database.
use PizzaRestrauntDatabase
GO

-- Creating tables and inserting data.
-- Creating Pizzas Types Table.
create table Pizza_Types(
	Pizza_Type_ID varchar(25) not null constraint PK_Pizza_Types_Pizza_Type_ID primary key,
	Pizza_Name varchar(100) not null,
	Category varchar(25) not null constraint Check_Pizza_Types_Category check(Category in ('Classic', 'Supreme', 'Veggie', 'Chicken')),
	Ingredients varchar(500) null constraint Default_Pizza_Types_Ingredients default 'N/A'
)

-- Altering table Pizza_Types and adding GUID newsequenceid() along with getdate()
alter table Pizza_Types
add UniqueID uniqueidentifier constraint Default_Pizza_Types_UniqueID default NEWSEQUENTIALID()
GO

alter table Pizza_Types
add Updated_Timestamp datetime null constraint Default_Pizza_Types_Updated_Timestamp default getdate()
GO

-- Inserting data into Pizza_Types. 1 row with insert command, remaining with GUI.
insert into Pizza_Types([Pizza_Type_ID], [Pizza_Name], [Category], [Ingredients])
values('bbq_ckn', 'The Barbecue Chicken Pizza', 'Chicken', 'Barbecued Chicken, Red Peppers, Green Peppers, Tomatoes, Red Onions, Barbecue Sauce')
GO

-- Creating Pizzas Table.
create table Pizza(
	Pizza_ID varchar(25) not null constraint PK_Pizza_Pizza_ID primary key,
	Pizza_Type_ID varchar(25) not null constraint FK_Pizza_Pizza_Type_ID foreign key references [dbo].[Pizza_Types]([Pizza_Type_ID]) on update cascade,
	Size varchar(3) not null constraint Check_Pizza_Size check(Size in ('S', 'M', 'L', 'XL', 'XXL')),
	Price decimal(5,2) not null,
	UniqueID uniqueidentifier constraint Default_Pizza_UniqueID default NEWSEQUENTIALID(),
	Updated_Timestamp datetime constraint Default_Pizza_Updated_Timestamp default getdate()
)
GO

-- Inserting data into Pizza. 1 row with insert command, remaining with GUI.
insert into Pizza([Pizza_ID], [Pizza_Type_ID], [Size], [Price])
values('bbq_ckn_s', 'bbq_ckn', 'S', '12.75')
GO

-- Creating table Orders.
create table Orders(
	Order_ID int not null constraint PK_Orders_Order_ID primary key,
	Order_Date date not null,
	Order_Time time not null,
	UniqueID uniqueidentifier constraint Default_Orders_Unique default NEWSEQUENTIALID(),
	Updated_Timestamp datetime constraint Default_Orders_Updated_Timestamp default getdate()
)
GO

-- Inserting data into Orders. 1 row with insert command, remaining with GUI.
insert into Orders([Order_ID], [Order_Date], [Order_Time])
values(1, '01-01-2015', '11:38:36')
GO

-- Creating Order_Details Table.
create table Order_Details(
      Order_Details_ID int not null constraint Unique_Order_Details_Order_Details_ID unique(Order_Details_ID),
      Order_ID int not null constraint FK_Order_Details_Order_ID foreign key references [dbo].[Orders]([Order_ID]),
      Pizza_ID varchar(25) not null constraint FK_Pizza_ID foreign key references [dbo].[Pizza]([Pizza_ID]),
      Quantity tinyint not null,
	  UniqueID uniqueidentifier constraint Default_Order_Details_Unique default NEWSEQUENTIALID(),
	  Updated_Timestamp datetime constraint Default_Order_Details_Updated_Timestamp default getdate()
)
GO

-- Inserting data into Order_Details. 1 row with insert command, remaining with GUI.
insert into Order_Details([Order_Details_ID], [Order_ID], [Pizza_ID], [Quantity])
values(1, 1, 'hawaiian_m', 1)
GO
```

## Data Insights Using SQL.

#### 1) Number of Pizza varieties sold by the restaurant.
```
select COUNT(distinct Pizza_Name) as Pizza_Variety_Count from Pizza_Types
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/74e2a010-152f-4d21-93c6-2a89732cceed)

#### 2) Number of Pizzas currently in each category.
```
select Category, COUNT(Pizza_Name) as Number_Of_Pizzas from Pizza_Types
group by Category
order by Category desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/96bf7240-140a-4fff-a14c-7b496c6d0800)

#### 3) Number of ingredients used in each pizza.
```
select Pizza_Name, Ingredients, LEN(Ingredients) - LEN(REPLACE(Ingredients, ',', '')) + 1 AS Number_Of_Ingredients -- Replaced the commas with an empty string, added 1 to that value and subtracted this length with the original Ingridient column length.
from Pizza_Types
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/55263202-ed0b-45e2-a7aa-21573a055d53)

#### 4) The average number of ingredients used to make one single pizza.
```
with Pizza_Ingredients as
(select Ingredients, LEN(Ingredients) - LEN(REPLACE(Ingredients, ',', '')) + 1 AS Number_Of_Ingredients
from Pizza_Types)
select AVG(Number_Of_Ingredients) as Average_Number_Of_Ingredients from Pizza_Ingredients
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/b9a9be0a-d986-484b-bcc4-880b455cc961)

#### 5) The topmost and least used ingredients in the Pizzas.
```
with Ingredient_Popularity as
(
select Pizza_Name, VALUE as Result
from Pizza_Types 
cross apply string_split(Ingredients, ',') -- Using cross apply as we are joining a table with the result of a function.
) 
select Result as Ingredient, COUNT(Result) as Ingredient_Count, DENSE_RANK() over(order by COUNT(Result) desc) as DenseRnk
from Ingredient_Popularity
group by Result
order by Ingredient_Count desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/800cc3c9-67fb-4d02-82f2-78f7fa8d2896)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/5e7cc73f-54ca-416a-af85-98adf37374a9)\

#### 6) The average price of Pizzas based on Size.
```
select Size, 
case when Size = 'S' 
	 then 'The Average Price for a Small Pizza is' + SPACE(1) + cast(format(AVG(Price), 'C', 'en-US') as varchar(10))
	 when Size = 'M'
	 then 'The Average Price for a Medium Pizza is' + SPACE(1) + cast(format(AVG(Price), 'C', 'en-US') as varchar(10))
	 when Size = 'L'
	 then 'The Average Price for a Large Pizza is' + SPACE(1) + cast(format(AVG(Price), 'C', 'en-US') as varchar(10))
	 when Size = 'XL'
	 then 'The Average Price for a X Large Pizza is' + SPACE(1) + cast(format(AVG(Price), 'C', 'en-US') as varchar(10))
	 when Size = 'XXL'
	 then 'The Average Price for a XX Large Pizza is' + SPACE(1) + cast(format(AVG(Price), 'C', 'en-US') as varchar(10))
	 end Pizza_Price -- SPACE function used adding a space between words. Cast used to convert numeric to varchar. Format used to add USD currency symbol.
from Pizza
group by Size
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/b3f16125-58c4-493f-86d2-093cbf804e7f)

#### 7) Average price of all pizzas regardless of size.
```
select format(AVG(Price), 'C', 'en-US') as Avg_Pizza_Price from Pizza
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/b249d8a1-ed35-4472-b898-0c8b6f41a2a8)

#### 8) Average price of pizzas under each category.
```
select pt.Category, format(AVG(p.Price), 'C', 'En-US') as Avg_Price from Pizza_Types pt
inner join Pizza p on pt.Pizza_Type_ID = p.Pizza_Type_ID
group by pt.Category
order by Avg_Price
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/8df94299-4b96-49e5-a360-24a0e81760d4)

#### 9) Check whether all sizes are available for each Pizza type.
```
select Pizza_Type_ID, 
	   case when Size = 'S' then 'Small Available' when Size = 'M' then 'Medium Available'
			when Size = 'L' then 'Large Available' when Size = 'XL' then 'X Large Available'
			when Size = 'XXL' then 'XX Large Available' end Is_Available
from Pizza
group by Pizza_Type_ID, Size
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/6f9374e2-cf58-45fc-b314-5120696a1185)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/6da9cc92-5794-4cc5-b46d-dc71865351ab)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/a330422b-88e9-4c47-a693-c0519212a824)

#### 10) The most expensive small, medium, and large pizza.
```
with Pizza_Size_Price as
(
select Pizza_Type_ID, Size, Price, DENSE_RANK() over(partition by size order by Price desc) as Price_Rank
from Pizza
)
select * from Pizza_Size_Price
where Size = 'L' -- Size filter
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/13be5272-d48a-4b84-a697-57d067d6b555)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/66fc708f-56ba-485b-b521-d7f1d91d6f66)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/4d7bb085-d52a-4eb3-a890-0dcffffda958)

#### 11) Total number of pizzas sold in 2015.
```
select format(SUM(Quantity), '##,###') as Number_Of_Pizzas_Sold from Order_Details
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/9776c7f4-106e-4254-93a5-e883deafab0b)

#### 12) Least popular to most popular pizza category. Based on the number of orders.
```
select case when pt.Category is null then 'TOTAL = ' else pt.Category end Category, 
	   format(SUM(Quantity), '##,###') as Pizza_Order_Count from Order_Details od
inner join Pizza p on od.Pizza_ID = p.Pizza_ID
inner join Pizza_Types pt on p.Pizza_Type_ID = pt.Pizza_Type_ID
group by rollup (pt.Category)
order by SUM(Quantity) asc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/d09bd5ce-1a07-4d81-b0e1-6c84a0427b86)

#### 13) Top 5 best-selling pizzas and top 5 least-selling pizzas. Based on the number of orders.
```
select top 5 Pizza_ID, 
SUM(Quantity) as Count_Pizzas_Sold, DENSE_RANK() over(order by SUM(Quantity) desc) as Rnk
from Order_Details
group by Pizza_ID 
order by Count_Pizzas_Sold desc
GO
select top 5 Pizza_ID, 
SUM(Quantity) as Count_Pizzas_Sold, DENSE_RANK() over(order by SUM(Quantity) asc) as Rnk
from Order_Details
group by Pizza_ID 
order by Count_Pizzas_Sold
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/d243111c-08cf-4e24-8903-ebec7d888f8b)

#### 14) Most popular pizza size. Based on the number of orders.
```
select p.Size, SUM(od.Quantity) as Number_Of_Pizzas_Sold from Pizza p
inner join Order_Details od on p.Pizza_ID = od.Pizza_ID
group by p.Size
order by Number_Of_Pizzas_Sold desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/26a16b0e-6ebd-4e5b-9bd9-f7ad1eec6678)

#### 15) Number of orders received in 2015.
```
select format(COUNT(Order_ID), '##,###') as Number_Of_Orders from Orders
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/279e7e2d-1fc6-4aa7-bbc5-ac635034f5f5)

#### 16) Average number of orders received per day.
```
with Avg_Orders_Per_Day as
(
	select DATENAME(DAYOFYEAR, Order_Date) as Order_Day, COUNT(Order_ID) as Number_Of_Orders 
	from Orders
	group by DATENAME(DAYOFYEAR, Order_Date)
)
select AVG(Number_Of_Orders) as Avg_Orders_Per_Day from Avg_Orders_Per_Day
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/b2b2e036-6871-418f-a391-459c5a761051)

#### 17) Number of orders received each day in 2015.
```
with Orders_Per_Day as
(
	select DATEPART(DAYOFYEAR, Order_Date) as Day_Number, COUNT(Order_ID) as Number_Of_Orders
	from Orders
	group by DATEPART(DAYOFYEAR, Order_Date)
)
select * from Orders_Per_Day
order by Day_Number
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/2d6c4921-e83a-45e8-9a3f-abe9b41b5e25)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/bf0fbc75-90dc-4349-add4-54f27972db79)

#### 18) The number of orders received on each weekday.
```
with Order_Per_Week_Day as
(
	select case when DATENAME(WEEKDAY, Order_Date) = 'Monday' then 1
				when DATENAME(WEEKDAY, Order_Date) = 'Tuesday' then 2
				when DATENAME(WEEKDAY, Order_Date) = 'Wednesday' then 3
				when DATENAME(WEEKDAY, Order_Date) = 'Thursday' then 4
				when DATENAME(WEEKDAY, Order_Date) = 'Friday' then 5
				when DATENAME(WEEKDAY, Order_Date) = 'Saturday' then 6
				when DATENAME(WEEKDAY, Order_Date) = 'Sunday' then 7 end Day_Number,
	DATENAME(WEEKDAY, Order_Date) as Week_Day, count(Order_ID) as Number_Of_Orders 
	from Orders
	group by DATENAME(WEEKDAY, Order_Date)
)
select Week_Day, Number_Of_Orders from Order_Per_Week_Day
order by Day_Number asc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/bf3a1c23-09e3-4f65-a997-0140ad52355c)

#### 19) The number of orders received each month.
```
with Orders_Per_Month as
(
	select MONTH(Order_Date) as Month_Number, DATENAME(MONTH, Order_Date) as Month_Name, COUNT(Order_ID) as Number_Of_Orders
	from Orders
	group by MONTH(Order_Date), DATENAME(MONTH, Order_Date)
)
select Month_Name, Number_Of_Orders from Orders_Per_Month
order by Month_Number
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/0fe32c7b-70bb-4302-9b18-ed78ab4210fa)

#### 20) Number of orders received each quarter.
```
with Order_Per_Quarter as
(
	select DATENAME(QUARTER, Order_Date) as Quarter_Number,
		   case when DATENAME(QUARTER, Order_Date) = 1 then 'QTR 1 - JAN/FEB/MAR'
				when DATENAME(QUARTER, Order_Date) = 2 then 'QTR 2 - APR/MAY/JUN'
				when DATENAME(QUARTER, Order_Date) = 3 then 'QTR 3 - JUL/AUG/SEP'
		        when DATENAME(QUARTER, Order_Date) = 4 then 'QTR 4 - OCT/NOV/DEC' end Quarter,
			COUNT(Order_ID) as Number_Of_Orders
	from Orders
	group by DATENAME(QUARTER, Order_Date)
)
select Quarter, Number_Of_Orders from Order_Per_Quarter
order by Quarter_Number
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/288e3c5d-ef3d-4aa5-a609-2d6337ac79ae)

#### 21) Total number of orders received in each hour in 2015.
```
with Orders_Per_Hour as 
(
	select DATEPART(HOUR, Order_Time) as Hour_Number, COUNT(Order_ID) as Number_Of_Orders from Orders
	group by DATEPART(HOUR, Order_Time)
)
select * from Orders_Per_Hour
order by Hour_Number
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/85b0e266-3ffe-49f7-8fb2-75e414d3d3c1)

#### 22) Number of orders received based on brunch, lunch, snack, and dinner.
```
select case when Order_Time < '11:59:59' then 'Brunch'
				   when Order_Time between '12:00:00' and '15:00:00' then 'Lunch' 
				   when Order_Time between '15:00:01' and '17:29:59' then 'Snack'
				   when Order_Time >= '17:30:00' then 'Dinner' 
				   end Order_Time_Category,
			  COUNT(Order_ID) as Number_Of_Orders
	from Orders
	group by case when Order_Time < '11:59:59' then 'Brunch'
				  when Order_Time between '12:00:00' and '15:00:00' then 'Lunch' 
				  when Order_Time between '15:00:01' and '17:29:59' then 'Snack'
				  when Order_Time >= '17:30:00' then 'Dinner' end
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/ad18d136-814b-4bb2-93be-a6adcf18eb06)

#### 23) Average number of pizzas sold each day.
```
with Avg_Pizza_Sold_Per_Day as
(
	select DATEPART(DAYOFYEAR, o.Order_Date) as DayOfYear, SUM(od.Quantity) as Total_Pizzas_Sold from Order_Details od
	inner join Orders o on od.Order_ID = o.Order_ID
	group by DATEPART(DAYOFYEAR, o.Order_Date)
)
select AVG(Total_Pizzas_Sold) as Avg_Pizza_Sold_Per_Day from Avg_Pizza_Sold_Per_Day
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/b86e10a6-97db-449b-a520-452166017278)

#### 24) Average cart value of a customer.
```
with Cart_Total_Amount as
(
	select od.Order_ID, sum(od.Quantity * p.Price) as Cart_Value from Order_Details od
	inner join Pizza p on od.Pizza_ID = p.Pizza_ID
	group by od.Order_ID
)
select format(AVG(Cart_Value), 'C', 'en-US') as Avg_Customer_Cart_Value from Cart_Total_Amount
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/877c6ad5-dd0f-4175-a728-7afe5abd8822)

#### 25) Total revenue generated from selling Pizzas in 2015.
```
select format(SUM(od.Quantity * p.Price), 'C', 'en-US') as Revenue_2015 from Order_Details od
inner join Pizza p on od.Pizza_ID = p.Pizza_ID
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/57aef6e8-18fe-4d3e-8297-6aba6b2b84ae)

#### 26) Total revenue generated from each pizza category.
```
select pt.Category, format(SUM(od.Quantity * p.Price), 'C', 'en-US') as Total_Revenue,
	   left((SUM(od.Quantity * p.Price)/(select SUM(od.Quantity * p.Price) as Revenue_2015 from Order_Details od -- Used sub query to divide revenue from each category with total revenue. Used left to get only 5 digits of percentage.
			 inner join Pizza p on od.Pizza_ID = p.Pizza_ID)) * 100, 5) + replicate('%', 1)  as Percentage_Of_Revenue -- -- Used replicate to append % sign once.
from Pizza_Types pt
inner join Pizza p on pt.Pizza_Type_ID = p.Pizza_Type_ID
inner join Order_Details od on p.Pizza_ID = od.Pizza_ID
group by pt.Category
order by Total_Revenue desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/e22062d6-c950-49ac-b3d4-64d38cbc4fe2)

#### 27) Total revenue generated based on pizza size.
```
select p.Size, format(SUM(p.Price * od.Quantity), 'C', 'en-US') as Revenue_Generated,
	   left((SUM(od.Quantity * p.Price)/(select SUM(od.Quantity * p.Price) as Revenue_2015 from Order_Details od -- Used sub query to divide revenue from each category with total revenue.
			 inner join Pizza p on od.Pizza_ID = p.Pizza_ID)) * 100, 5) + replicate('%', 1)  as Percentage_Of_Revenue -- Used replicate to append % sign once.
from Pizza p
inner join Order_Details od on p.Pizza_ID = od.Pizza_ID
group by p.Size
order by Revenue_Generated desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/a49ee141-1b7b-4547-8148-1d6dbf8f9664)

#### 28) Best-selling and worst-selling pizzas, based on revenue.
```
select od.Pizza_ID, pt.Category, pt.Pizza_Name, format(SUM(p.Price * od.Quantity), 'C') as Revenue_Generated,
	   DENSE_RANK() over(Order by SUM(p.Price * od.Quantity) desc) as Rnk
from Order_Details od
inner join Pizza p on od.Pizza_ID = p.Pizza_ID
inner join Pizza_Types pt on p.Pizza_Type_ID = pt.Pizza_Type_ID
group by od.Pizza_ID, pt.Category, pt.Pizza_Name
order by SUM(p.Price * od.Quantity) desc
GO
```
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/dd05dd47-ed2a-4797-96cd-f57687ba2dd6)
![image](https://github.com/JoshuaSequeira2000/SQL-Project3-Exploratory-Data-Analysis/assets/92262753/7eee609f-d9f9-4783-a118-6658f524ec1a)


















