# SQL Project: Superstore Data Analysis

This project covers the complete SQL pipeline for cleaning, transforming, and analyzing insights from a retail dataset using **MySQL**.

---

## 1. Database and Table Setup

### Create and Use Database

```sql
CREATE DATABASE db_sql_projects;
USE db_sql_projects;
```

### Create `Store` Table (with Keys)

```sql
CREATE TABLE Store (
    Row_ID           INT AUTO_INCREMENT PRIMARY KEY,
    Order_ID         VARCHAR(100),
    Order_Date       DATE,
    Ship_Date        DATE,
    Ship_Mode        VARCHAR(100),
    Customer_ID      VARCHAR(100),
    Customer_Name    VARCHAR(100),
    Segment          VARCHAR(100),
    Postal_Code      INT,
    City             VARCHAR(100),
    State            VARCHAR(100),
    Country          VARCHAR(100),
    Region           VARCHAR(100),
    Market           VARCHAR(100),
    Product_ID       VARCHAR(100),
    Category         VARCHAR(100),
    Sub_Category     VARCHAR(100),
    Product_Name     VARCHAR(1000),
    Sales            FLOAT,
    Quantity         INT,
    Discount         INT,
    Profit           FLOAT,
    Shipping_Cost    FLOAT,
    Order_Priority   VARCHAR(100),
    UNIQUE (Order_ID, Product_ID)
);
```

### Load CSV File

```sql
SHOW VARIABLES LIKE 'secure_file_priv';

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/superstore_2016.csv'
INTO TABLE Store
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(
    Row_ID,
    Order_ID,
    Order_Date,
    Ship_Date,
    Ship_Mode,
    Customer_ID,
    Customer_Name,
    Segment,
    @Postal_Code,
    City,
    State,
    Country,
    Region,
    Market,
    Product_ID,
    Category,
    Sub_Category,
    Product_Name,
    Sales,
    Quantity,
    Discount,
    Profit,
    Shipping_Cost,
    Order_Priority
);
```

---

## 2. Data Cleaning

### Convert Date Columns

```sql
SET SQL_SAFE_UPDATES = 0;

UPDATE store SET order_date = STR_TO_DATE(order_date, '%Y-%m-%d');
UPDATE store SET ship_date = STR_TO_DATE(ship_date, '%Y-%m-%d');

ALTER TABLE store
MODIFY order_date DATE,
MODIFY ship_date DATE;
```

### Convert Sales & Profit to FLOAT

```sql
UPDATE store SET profit = CAST(REPLACE(REPLACE(profit, '$', ''), ',', '') AS FLOAT);
ALTER TABLE store MODIFY profit FLOAT;

UPDATE store SET sales = CAST(REPLACE(REPLACE(sales, '$', ''), ',', '') AS FLOAT);
ALTER TABLE store MODIFY sales FLOAT;
```

---

## 3. Basic Data Checks

```sql
SELECT COUNT(*) AS Total_Rows FROM Store;
SELECT * FROM Store LIMIT 5;

SELECT COUNT(*) AS Postal_Code_Null_Count
FROM Store
WHERE Postal_Code IS NULL;

SELECT Order_ID, COUNT(*) AS Occurrences
FROM Store
GROUP BY Order_ID
HAVING Occurrences > 1;
```

---

## 4. Exploratory Data Analysis (EDA)

### Sales by Category & Sub-Category

```sql
SELECT Category, Sub_Category, SUM(Sales) AS Total_Sales
FROM Store
GROUP BY Category, Sub_Category
ORDER BY Total_Sales DESC;
```

### Average Discount by Segment

```sql
SELECT Segment, AVG(Discount) AS Avg_Discount
FROM Store
GROUP BY Segment;
```

### Monthly Sales Trend

```sql
SELECT MONTHNAME(Order_Date) AS Month, SUM(Sales) AS Total_Sales
FROM Store
GROUP BY Month
ORDER BY MONTH(Order_Date);
```

### Yearly Profit by Region

```sql
SELECT YEAR(Order_Date) AS Year, Region, SUM(Profit) AS Total_Profit
FROM Store
GROUP BY Year, Region
ORDER BY Year, Region;
```

### Categorize Orders by Sales Value

```sql
SELECT Order_ID, Sales,
  CASE
    WHEN Sales >= 1000 THEN 'High Value'
    WHEN Sales >= 500 THEN 'Medium Value'
    ELSE 'Low Value'
  END AS Value_Category
FROM Store;
```

### Running Total Sales per Region

```sql
SELECT Region, Order_Date,
       SUM(Sales) OVER (PARTITION BY Region ORDER BY Order_Date) AS Running_Sales
FROM Store;
```

### Rank Products by Profit

```sql
SELECT Product_Name, SUM(Profit) AS Total_Profit,
       RANK() OVER (ORDER BY SUM(Profit) DESC) AS Profit_Rank
FROM Store
GROUP BY Product_Name;
```

### Detect Outliers

```sql
-- High Sales
SELECT * FROM Store WHERE Sales > 5000 ORDER BY Sales DESC;

-- High Discounts
SELECT * FROM Store WHERE Discount > 0.5;
```

### Profit by Sales Buckets

```sql
SELECT
  CASE
    WHEN Sales >= 1000 THEN 'High'
    WHEN Sales >= 500 THEN 'Medium'
    ELSE 'Low'
  END AS Sales_Range,
  AVG(Profit) AS Avg_Profit
FROM Store
GROUP BY Sales_Range;
```

### General Summary

```sql
SELECT
  COUNT(DISTINCT Customer_ID) AS Total_Customers,
  COUNT(*) AS Total_Orders,
  SUM(Sales) AS Total_Sales,
  SUM(Profit) AS Total_Profit,
  AVG(Discount) AS Avg_Discount
FROM Store;
```

### Sales and Profit by Category

```sql
SELECT Category, SUM(Sales) AS Total_Sales, SUM(Profit) AS Total_Profit
FROM Store
GROUP BY Category
ORDER BY Total_Sales DESC;
```

---

## 5. Advanced Analyses

### Month-over-Month Sales Growth

```sql
WITH MonthlySales AS (
  SELECT DATE_FORMAT(Order_Date, '%Y-%m') AS Month, SUM(Sales) AS Total_Sales
  FROM Store
  GROUP BY Month
),
SalesGrowth AS (
  SELECT Month, Total_Sales,
         LAG(Total_Sales) OVER (ORDER BY Month) AS Prev_Month_Sales
  FROM MonthlySales
)
SELECT Month, Total_Sales, Prev_Month_Sales,
       ROUND(((Total_Sales - Prev_Month_Sales) / Prev_Month_Sales)*100, 2) AS Sales_Growth_Percentage
FROM SalesGrowth
WHERE Prev_Month_Sales IS NOT NULL;
```

### Market Basket (Product Pairing)

```sql
SELECT o1.Product_Name AS Product_A, o2.Product_Name AS Product_B, COUNT(*) AS Pair_Count
FROM Store o1
JOIN Store o2 ON o1.Order_ID = o2.Order_ID AND o1.Product_Name < o2.Product_Name
GROUP BY Product_A, Product_B
ORDER BY Pair_Count DESC
LIMIT 10;
```

### Delivery Delay by Shipping Mode

```sql
SELECT Ship_Mode, AVG(DATEDIFF(Ship_Date, Order_Date)) AS Avg_Delivery_Days, COUNT(*) AS Total_Orders
FROM Store
GROUP BY Ship_Mode
ORDER BY Avg_Delivery_Days;
```

---

## 6. People Table Setup and Analysis

### Create Table

```sql
CREATE TABLE People (
    Person VARCHAR(100) PRIMARY KEY,
    Region VARCHAR(100)
);
```

### Load Data

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/People.csv'
INTO TABLE People
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(
    Person,
    Region
);
```

### Sample Analysis

```sql
-- Count of People per Region
SELECT Region, COUNT(*) AS People_Count FROM People GROUP BY Region;

-- Orders Managed per Person
SELECT p.Person, COUNT(s.Order_ID) AS Orders_Managed
FROM Store s
JOIN People p ON s.Region = p.Region
GROUP BY p.Person
ORDER BY Orders_Managed DESC;

-- Sales by Person
SELECT p.Person, SUM(s.Sales) AS Total_Sales
FROM Store s
JOIN People p ON s.Region = p.Region
GROUP BY p.Person;

-- Profit by Person
SELECT p.Person, SUM(s.Profit) AS Total_Profit
FROM Store s
JOIN People p ON s.Region = p.Region
GROUP BY p.Person;

-- Average Discount by Person
SELECT p.Person, AVG(s.Discount) AS Avg_Discount
FROM Store s
JOIN People p ON s.Region = p.Region
GROUP BY p.Person;
```

---

## 7. Returns Table Setup and Analysis

### Create Table

```sql
CREATE TABLE Returns (
    Return_ID INT PRIMARY KEY AUTO_INCREMENT,
    Returned VARCHAR(100),
    Order_ID VARCHAR(100),
    Region VARCHAR(100),
    FOREIGN KEY (Order_ID) REFERENCES Store(Order_ID) ON DELETE CASCADE
);
```

### Load Data

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Returns.csv'
INTO TABLE Returns
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(
    Returned,
    Order_ID,
    Region
);
```

### Sample Analysis

```sql
-- Return Rate
SELECT
  COUNT(r.Order_ID) AS Total_Returns,
  COUNT(DISTINCT s.Order_ID) AS Total_Orders,
  ROUND(COUNT(r.Order_ID) / COUNT(DISTINCT s.Order_ID) * 100, 2) AS Return_Percentage
FROM Store s
LEFT JOIN Returns r ON s.Order_ID = r.Order_ID;

-- Returns by Region
SELECT r.Region, COUNT(*) AS Return_Count
FROM Returns r
GROUP BY r.Region;

-- Monthly Return Trend
SELECT MONTHNAME(s.Order_Date) AS Month, COUNT(r.Order_ID) AS Returns
FROM Returns r
JOIN Store s ON r.Order_ID = s.Order_ID
GROUP BY Month
ORDER BY MONTH(s.Order_Date);

-- Profit Lost due to Returns
SELECT
  r.Region,
  COUNT(r.Order_ID) AS Return_Count,
  SUM(s.Profit) AS Profit_Lost
FROM Returns r
JOIN Store s ON r.Order_ID = s.Order_ID
GROUP BY r.Region;

-- Most Returned Products
SELECT s.Product_Name, COUNT(r.Order_ID) AS Return_Count
FROM Returns r
JOIN Store s ON r.Order_ID = s.Order_ID
GROUP BY s.Product_Name
ORDER BY Return_Count DESC
LIMIT 10;

-- Ship Mode with Highest Return Rate
SELECT s.Ship_Mode,
       COUNT(r.Order_ID) AS Returns,
       COUNT(DISTINCT s.Order_ID) AS Total_Orders,
       ROUND(COUNT(r.Order_ID)/COUNT(DISTINCT s.Order_ID) * 100, 2) AS Return_Rate
FROM Store s
LEFT JOIN Returns r ON s.Order_ID = r.Order_ID
GROUP BY s.Ship_Mode
ORDER BY Return_Rate DESC;
```
