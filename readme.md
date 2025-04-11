# Data Analysis SQL
In this project I have used advanced SQL queries to perform various types of Analysis and generated business report.

![Advanced Data Analytics](./docs/Advanced%20Data%20Analytics.png)

# Types of Analyses Covered:

- **Changes Over Time Analysis:** Tracking trends and seasonality by aggregating measures over time (e.g., sales by year or month).
- **Cumulative Analysis:** Calculating running totals or moving averages to understand business growth or decline.
- **Performance Analysis:** Comparing current metrics (e.g., sales) with targets (e.g., averages or previous years) to measure success.
- **Part-To-Whole Analysis:** Determining the contribution of individual categories to the whole (e.g., sales percentage by product category).
- **Data Segmentation:** Grouping data into ranges or categories (e.g., customer segments based on spending behavior or product segments based on cost ranges).

# Building Reports:

- **Customer Report:** Consolidates customer metrics (e.g., total orders, sales, recency) and segments customers (e.g., VIP, regular, new).
- **Product Report:** Provides product details (e.g., category, cost) and segments products (e.g., high, medium, low revenue).
- Both reports include KPIs like average order value and monthly spending.


## Changes Over Time Analysis
Change Over Time Analysis is a technique used to track how a specific measure (e.g., sales, revenue, customer count) evolves over a time dimension (e.g., days, months, years). It helps identify trends, seasonality, and performance fluctuations in business data.

### Key Concepts

**Formula:**

*Aggregate a measure (e.g., sum, average) and group it by a date dimension.*

Example:

```sql
SELECT 
    YEAR(OrderDate) AS OrderYear, 
    SUM(SalesAmount) AS TotalSales
FROM Sales
GROUP BY YEAR(OrderDate)
ORDER BY OrderYear;
```
**Purpose:**

- Identify growth or decline (e.g., sales increasing year-over-year).
- Detect seasonal patterns (e.g., higher sales in December).
- Support strategic decisions (e.g., allocating budgets based on trends).

**Common Time Granularities:**
- Yearly → Long-term trends.
- Monthly/Quarterly → Seasonal patterns.
- Daily/Weekly → Short-term fluctuations.

### Example in SQL
1. Basic Yearly Sales Trend

```sql
SELECT 
    YEAR(OrderDate) AS OrderYear,
    SUM(SalesAmount) AS TotalSales,
    COUNT(DISTINCT CustomerKey) AS TotalCustomers
FROM Sales
WHERE OrderDate IS NOT NULL
GROUP BY YEAR(OrderDate)
ORDER BY OrderYear;
```
```
Output:

OrderYear	TotalSales	TotalCustomers
2020	    100,000	    500
2021	    150,000	    700
2022	    120,000	    600
```
Insight: Sales peaked in 2021 but declined in 2022.

2. Monthly Seasonality

```sql

SELECT 
    FORMAT(OrderDate, 'yyyy-MM') AS OrderMonth,
    SUM(SalesAmount) AS MonthlySales
FROM Sales
GROUP BY FORMAT(OrderDate, 'yyyy-MM')
ORDER BY OrderMonth;
```
```
Output:

OrderMonth	MonthlySales
2022-01	    10,000
2022-02	    8,000
...	...
2022-12	    25,000
```
Insight: December has the highest sales (holiday season).

## Advanced Techniques
Date Truncation (Simplify grouping):

```sql
SELECT 
    DATETRUNC(month, OrderDate) AS OrderMonth,
    SUM(SalesAmount) AS TotalSales
FROM Sales
GROUP BY DATETRUNC(month, OrderDate);
```
Comparing Time Periods (YoY/MoM changes):

```sql
WITH MonthlySales AS (
    SELECT 
        YEAR(OrderDate) AS Year,
        MONTH(OrderDate) AS Month,
        SUM(SalesAmount) AS Sales
    FROM Sales
    GROUP BY YEAR(OrderDate), MONTH(OrderDate)
)
SELECT 
    Year, Month, Sales,
    LAG(Sales, 12) OVER (ORDER BY Year, Month) AS PrevYearSales,
    (Sales - LAG(Sales, 12) OVER (ORDER BY Year, Month)) AS YoYGrowth
FROM MonthlySales;
```

### Why It Matters
- Helps businesses forecast demand.
- Reveals underperforming periods (e.g., low sales in Q1).
- Supports resource planning (e.g., hiring seasonal staff).


## Cumulative Analysis
Cumulative Analysis calculates progressive totals of a measure over time (e.g., running sums, moving averages). Unlike standard aggregations (e.g., monthly sales), it shows how values accumulate across a timeline, helping track growth or decline trends.

**Key Concepts**

Formula:

*Aggregate a measure (e.g., sales) and sum it sequentially over a sorted dimension (e.g., dates).*

Example:

```sql
SUM(Sales) OVER (ORDER BY Date) AS RunningTotal;
```

**Purpose:**

- Track business growth (e.g., YTD revenue).
- Calculate moving averages (e.g., 3-month rolling average).
- Identify milestones (e.g., when cumulative sales hit a target).

**Window Functions:**

- `SUM() OVER (ORDER BY ...)` → Running total.
- `AVG() OVER (ORDER BY ... ROWS N PRECEDING)` → Moving average.
---

**Examples in SQL**

1. Running Total of Monthly Sales

```sql
SELECT 
    MONTH(OrderDate) AS Month,
    SUM(SalesAmount) AS MonthlySales,
    SUM(SUM(SalesAmount)) OVER (ORDER BY MONTH(OrderDate)) AS RunningTotal
FROM Sales
WHERE YEAR(OrderDate) = 2023
GROUP BY MONTH(OrderDate)
ORDER BY Month;
```
```
Output:

Month	MonthlySales	RunningTotal
1	    10,000	        10,000
2	    8,000	        18,000
3	    12,000	        30,000
```
Insight: By March, cumulative sales reached $30K.

2. Running Total by Year (Resets Annually)

```sql
SELECT 
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    SUM(SalesAmount) AS MonthlySales,
    SUM(SUM(SalesAmount)) OVER (
        PARTITION BY YEAR(OrderDate) 
        ORDER BY MONTH(OrderDate)
    ) AS YTDSales
FROM Sales
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
ORDER BY Year, Month;
```
```
Output:

Year	Month	MonthlySales	YTDSales
2023	1	    10,000	        10,000
2023	2	    8,000	        18,000
2024	1	    15,000	        15,000	(Resets for new year)
```
Insight: Tracks yearly progress separately.

3. 3-Month Moving Average

```sql

SELECT 
    OrderDate,
    SalesAmount,
    AVG(SalesAmount) OVER (
        ORDER BY OrderDate 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS MovingAvg3Months
FROM Sales;
```
```
Output:

OrderDate	SalesAmount	MovingAvg3Months
2023-01-01	10,000	    10,000
2023-02-01	8,000	    9,000	(Avg of Jan+Feb)
2023-03-01	12,000	    10,000	(Avg of Jan+Feb+Mar)
```
Insight: Smooths out short-term fluctuations.

**Why It Matters**

- **Trend Analysis:** Shows if growth is accelerating or slowing.
- **Goal Tracking:** Compare cumulative performance against targets.
- **Anomaly Detection:** Spot deviations from expected progress.

### Advanced Use Cases

1. **Partitioned Cumulative Sums** (e.g., by product category):

```sql
SUM(Sales) OVER (PARTITION BY Category ORDER BY Date);
```

2. **Cumulative Percentage of Total:**

```sql
SUM(Sales) OVER (ORDER BY Date) / SUM(Sales) OVER () * 100;
```

3. **Running Difference vs. Target:**

```sql
SUM(Sales - Target) OVER (ORDER BY Date)
```
---

### Key Takeaway
Cumulative Analysis transforms raw data into actionable trends by answering:
- *"How much have we earned so far this year?"*
- *"Is our 6-month performance improving?"*

## Performance Analysis in SQL

Performance Analysis compares a current metric (e.g., sales, revenue) against a target (e.g., past performance, averages, benchmarks) to evaluate success or identify gaps. It answers:

- Is this product performing better than last year?
- How does this month’s sales compare to the 6-month average?

**Key Concepts**

Formula:

```
Performance = Current Value − Target Value  
```

Targets can be:

- Historical data (e.g., previous year).
- Aggregates (e.g., average, max, min).
- External benchmarks (e.g., industry standards).

SQL Tools:

- **Window Functions:** `LAG()`, `LEAD()`, `AVG() OVER()`.
- **CASE Statements:** Flag performance (e.g., "Above Target").

**Examples in SQL**

1. **Year-over-Year (YoY) Comparison**
    
    Compare current sales to the previous year:

```sql
WITH YearlySales AS (  
    SELECT  
        YEAR(OrderDate) AS Year,  
        SUM(SalesAmount) AS TotalSales  
    FROM Sales  
    GROUP BY YEAR(OrderDate)  
)  
SELECT  
    Year,  
    TotalSales,  
    LAG(TotalSales) OVER (ORDER BY Year) AS PrevYearSales,  
    TotalSales - LAG(TotalSales) OVER (ORDER BY Year) AS YoYGrowth,  
    (TotalSales - LAG(TotalSales) OVER (ORDER BY Year)) / LAG(TotalSales) OVER (ORDER BY Year) * 100 AS YoYGrowthPercent  
FROM YearlySales  
ORDER BY Year; 
```
``` 
Output:

Year	TotalSales	PrevYearSales	YoYGrowth	YoYGrowth %
2022	120,000	    NULL	        NULL	    NULL
2023	150,000	    120,000	        +30,000	    +25%
```
Insight: 2023 sales grew by 25% vs. 2022.

2. **Performance vs. Average**

    Flag products selling above/below average:

```sql
SELECT  
    ProductName,  
    SUM(SalesAmount) AS CurrentSales,  
    AVG(SUM(SalesAmount)) OVER () AS AvgSales,  
    SUM(SalesAmount) - AVG(SUM(SalesAmount)) OVER () AS DiffFromAvg,  
    CASE  
        WHEN SUM(SalesAmount) > AVG(SUM(SalesAmount)) OVER () THEN 'Above Avg'  
        ELSE 'Below Avg'  
    END AS Performance  
FROM Sales  
JOIN Products ON Sales.ProductKey = Products.ProductKey  
GROUP BY ProductName;  
```
```
Output:

ProductName	CurrentSales	AvgSales	DiffFromAvg	Performance
Product A	50,000	        30,000	    +20,000	Above Avg
Product B	10,000	        30,000	    -20,000	Below Avg
```
Insight: Product A outperforms the average by $20K.

3. **Rolling 3-Month Performance**

    Track recent trends:

```sql
SELECT  
    FORMAT(OrderDate, 'yyyy-MM') AS Month,  
    SUM(SalesAmount) AS MonthlySales,  
    AVG(SUM(SalesAmount)) OVER (  
        ORDER BY FORMAT(OrderDate, 'yyyy-MM')  
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW  
    ) AS Rolling3MonthAvg,  
    SUM(SalesAmount) - AVG(SUM(SalesAmount)) OVER (  
        ORDER BY FORMAT(OrderDate, 'yyyy-MM')  
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW  
    ) AS DiffFromRollingAvg  
FROM Sales  
GROUP BY FORMAT(OrderDate, 'yyyy-MM');  
```
```
Output:

Month	MonthlySales	Rolling3MonthAvg	DiffFromRollingAvg
2023-01	10,000	        10,000	            0
2023-02	8,000	        9,000	            -1,000
2023-03	12,000	        10,000	            +2,000
```
Insight: March sales exceeded the 3-month average by $2K.

**Why It Matters**
- **Decision-Making:** Identify top performers and underperformers.
- **Goal Tracking:** Measure progress against KPIs.
- **Trend Alerts:** Spot declines early (e.g., sales dropping below average).

### Advanced Techniques

**Ranking Performance:**

```sql
RANK() OVER (PARTITION BY Year ORDER BY Sales DESC) AS SalesRank
```

**Percentile Analysis:**

```sql
PERCENT_RANK() OVER (ORDER BY Sales) AS Percentile 
```

**Target Variance:**

```sql
(Actual / Target * 100) - 100 AS VariancePercent  
```
**Key Takeaway**

Performance Analysis turns raw data into actionable insights by answering:
- Are we improving?
- What’s underperforming?
- Where should we focus resources?

## Part-to-Whole Analysis in SQL

Part-to-Whole Analysis measures how individual categories (parts) contribute to the total (whole), typically expressed as percentages. It answers questions like:

- What % of total sales comes from each product category?
- Which customer segment drives the most revenue?

**Key Concepts**
Formula:

```
% Contribution = (Part Value / Total Value) × 100  
```

**SQL Tools:**

- **Window Functions:** `SUM() OVER()` to calculate totals.
- **Aggregations:** `GROUP BY` to segment data.

Formatting: `ROUND()` and `CONCAT()` for percentages.

**Examples in SQL**
1. **Sales Contribution by Product Category**

```sql
SELECT  
    Category,  
    SUM(SalesAmount) AS CategorySales,  
    SUM(SUM(SalesAmount)) OVER () AS TotalSales,  
    ROUND(SUM(SalesAmount) / SUM(SUM(SalesAmount)) OVER () * 100, 2) AS ContributionPercent  
FROM Sales  
JOIN Products ON Sales.ProductKey = Products.ProductKey  
GROUP BY Category  
ORDER BY ContributionPercent DESC; 
```
``` 
Output:

Category	CategorySales	TotalSales	ContributionPercent
Bikes	    $500,000	    $720,000	69.44%
Accessories	$150,000	    $720,000	20.83%
Clothing	$70,000	        $720,000	9.72%
```
Insight: Bikes contribute 69.44% of total sales.

2. **Customer Segment Share of Orders**
```sql
SELECT  
    CustomerSegment,  
    COUNT(OrderID) AS SegmentOrders,  
    COUNT(OrderID) / SUM(COUNT(OrderID)) OVER () * 100 AS OrderSharePercent  
FROM Customers  
JOIN Orders ON Customers.CustomerID = Orders.CustomerID  
GROUP BY CustomerSegment;  
```
```
Output:

CustomerSegment	SegmentOrders	OrderSharePercent
VIP	            1,200	        60%
Regular	        700	            35%
New	            100	            5%
```
Insight: VIP customers drive 60% of orders.

3. **Monthly Revenue Breakdown**
```sql
SELECT  
    FORMAT(OrderDate, 'yyyy-MM') AS Month,  
    SUM(SalesAmount) AS MonthlyRevenue,  
    SUM(SalesAmount) / SUM(SUM(SalesAmount)) OVER () * 100 AS RevenuePercent  
FROM Sales  
GROUP BY FORMAT(OrderDate, 'yyyy-MM');  
```
```
Output:

Month	MonthlyRevenue	RevenuePercent
2023-12	$200,000	    25%
2024-01	$120,000	    15%
```
Insight: December contributes 25% of annual revenue.

**Why It Matters**
- **Prioritization:** Focus resources on high-impact categories (e.g., top-selling products).
- **Risk Assessment:** Detect over-reliance on a single category (e.g., bikes making up 70% of sales).
- **Benchmarking:** Compare segment performance (e.g., regional sales shares).

### Advanced Techniques
**Cumulative Percentage:**

```sql
SUM(ContributionPercent) OVER (ORDER BY ContributionPercent DESC) AS CumulativePercent  
```
*(Useful for Pareto analysis—identifying the "80/20" contributors.)*

**Dynamic Grouping:**

```sql
CASE  
    WHEN ContributionPercent > 10 THEN 'Major Contributor'  
    ELSE 'Minor Contributor'  
END AS ImpactCategory  
```

**Time-Based Trends:**

```sql
-- Yearly contribution changes  
WITH YearlyCategorySales AS (  
    SELECT  
        YEAR(OrderDate) AS Year,  
        Category,  
        SUM(SalesAmount) AS CategorySales  
    FROM Sales  
    GROUP BY YEAR(OrderDate), Category  
)  
SELECT  
    Year,  
    Category,  
    CategorySales / SUM(CategorySales) OVER (PARTITION BY Year) * 100 AS YearlyContribution  
FROM YearlyCategorySales;  
```

**Key Takeaway**
Part-to-Whole Analysis reveals composition and dependencies in your data. It’s essential for:
- **Strategic Planning:** Allocate budgets to high-impact areas.
- **Diversification:** Reduce risks from over-concentration.

**Pro Tip:** Visualize results as pie charts or stacked bar charts for stakeholders!

## Data Segmentation in SQL
Data Segmentation divides a dataset into meaningful groups (segments) based on shared characteristics, such as customer behavior, product performance, or transaction patterns. It enables targeted analysis and decision-making.

**Key Concepts**

1. Purpose:
    - Identify high-value customers (e.g., VIPs).
    - Group products by price range (e.g., budget vs. premium).
    - Analyze trends within specific cohorts (e.g., users who signed up in Q1).

2. SQL Tools:
    - **CASE WHEN:** Define segment rules (e.g., "IF age > 50 THEN 'Senior'").
    - **Window Functions:** Calculate metrics for segmentation (e.g., `SUM() OVER()` for spending tiers).
    - **CTEs/Subqueries:** Organize multi-step logic.
---

**Examples in SQL**

1. Customer Segmentation by Spending

```sql
WITH CustomerSpending AS (  
    SELECT  
        CustomerID,  
        SUM(SalesAmount) AS TotalSpent,  
        DATEDIFF(MONTH, MIN(OrderDate), MAX(OrderDate)) AS LifespanMonths  
    FROM Sales  
    GROUP BY CustomerID  
)  
SELECT  
    CustomerID,  
    TotalSpent,  
    LifespanMonths,  
    CASE  
        WHEN LifespanMonths >= 12 AND TotalSpent > 5000 THEN 'VIP'  
        WHEN LifespanMonths >= 12 AND TotalSpent <= 5000 THEN 'Regular'  
        ELSE 'New'  
    END AS CustomerSegment  
FROM CustomerSpending;  
```
```
Output:

CustomerID	TotalSpent	LifespanMonths	CustomerSegment
1001	    8,000	    14	            VIP
1002	    3,000	    18	            Regular
1003	    1,200	    2	            New
```
Insight: Segment customers for tailored marketing campaigns.

2. Product Segmentation by Price Range
```sql
SELECT  
    ProductName,  
    Cost,  
    CASE  
        WHEN Cost < 100 THEN 'Budget'  
        WHEN Cost BETWEEN 100 AND 500 THEN 'Mid-Range'  
        ELSE 'Premium'  
    END AS PriceSegment  
FROM Products;  
```
```
Output:

ProductName	    Cost	PriceSegment
Helmet	        50	    Budget
Road Bike	    1200	Premium
Water Bottle    150	    Mid-Range
```
Insight: Optimize inventory based on price tiers.

3. Time-Based Segmentation (Cohort Analysis)
```sql
WITH FirstPurchases AS (  
    SELECT  
        CustomerID,  
        MIN(OrderDate) AS FirstOrderDate  
    FROM Sales  
    GROUP BY CustomerID  
)  
SELECT  
    YEAR(FirstOrderDate) AS CohortYear,  
    MONTH(FirstOrderDate) AS CohortMonth,  
    COUNT(CustomerID) AS NewCustomers  
FROM FirstPurchases  
GROUP BY YEAR(FirstOrderDate), MONTH(FirstOrderDate);
```
```  
Output:

CohortYear	CohortMonth	NewCustomers
2023	    1	        150
2023	    2	        200
```
Insight: Track customer acquisition trends over time.

**Why It Matters**
- **Personalization:** Tailor strategies to segments (e.g., discounts for VIPs).
- **Efficiency:** Focus resources on high-impact groups.
- **Pattern Detection:** Spot outliers or emerging trends (e.g., growing "Premium" demand).

### Advanced Techniques
1. **RFM Segmentation (Recency, Frequency, Monetary):**

```sql

WITH RFM AS (  
    SELECT  
        CustomerID,  
        DATEDIFF(DAY, MAX(OrderDate), CURRENT_DATE) AS Recency,  
        COUNT(OrderID) AS Frequency,  
        SUM(SalesAmount) AS Monetary  
    FROM Sales  
    GROUP BY CustomerID  
)  
SELECT  
    CustomerID,  
    NTILE(5) OVER (ORDER BY Recency DESC) AS R_Score,  -- 1=Least recent  
    NTILE(5) OVER (ORDER BY Frequency) AS F_Score,  
    NTILE(5) OVER (ORDER BY Monetary) AS M_Score  
FROM RFM;  
```
*(Combines scores to classify customers like "Champions" or "At Risk".)*

2. **Behavioral Segmentation:**

```sql
-- Segment users by activity level  
SELECT  
    UserID,  
    COUNT(LoginDate) AS Logins,  
    CASE  
        WHEN COUNT(LoginDate) > 50 THEN 'Power User'  
        WHEN COUNT(LoginDate) > 10 THEN 'Active'  
        ELSE 'Inactive'  
    END AS ActivitySegment  
FROM UserLogins  
GROUP BY UserID;  
```

3. **Dynamic Thresholds:**

```sql
-- Auto-calculate segment bounds using AVG/STDEV  
WITH Stats AS (  
    SELECT  
        AVG(SalesAmount) AS AvgSales,  
        STDEV(SalesAmount) AS SalesStdDev  
    FROM Sales  
)  
SELECT  
    ProductID,  
    SalesAmount,  
    CASE  
        WHEN SalesAmount > (AvgSales + SalesStdDev) THEN 'High Performer'  
        WHEN SalesAmount < (AvgSales - SalesStdDev) THEN 'Low Performer'  
        ELSE 'Average'  
    END AS PerformanceSegment  
FROM Sales, Stats;  
```

**Key Takeaway**

Data Segmentation transforms raw data into actionable groups using:

- **Rules:** Define segments with CASE WHEN.
- **Context:** Combine metrics (e.g., spending + tenure).
- **SQL Power:** Leverage window functions and aggregations.

**Pro Tip:** Visualize segments with bar charts or heatmaps for stakeholders!

## How to Build Customer & Product Reports in SQL
Building comprehensive customer and product reports involves consolidating key metrics, segmenting data, and calculating KPIs. Below are step-by-step SQL templates for both reports.

1. **Customer Report**

    **Objective:** Track customer behavior, demographics, and spending patterns.

```sql
-- Step 1: Base data (customer details + sales)
WITH CustomerBase AS (
    SELECT
        c.CustomerKey,
        c.FirstName + ' ' + c.LastName AS CustomerName,
        DATEDIFF(YEAR, c.BirthDate, GETDATE()) AS Age,
        COUNT(DISTINCT s.OrderNumber) AS TotalOrders,
        SUM(s.SalesAmount) AS TotalSales,
        SUM(s.OrderQuantity) AS TotalQuantity,
        MAX(s.OrderDate) AS LastOrderDate,
        DATEDIFF(MONTH, MIN(s.OrderDate), MAX(s.OrderDate)) AS LifespanMonths
    FROM Sales s
    JOIN Customer c ON s.CustomerKey = c.CustomerKey
    WHERE s.OrderDate IS NOT NULL
    GROUP BY c.CustomerKey, c.FirstName, c.LastName, c.BirthDate
),

-- Step 2: Segment customers (VIP, Regular, New)
CustomerSegments AS (
    SELECT
        *,
        CASE
            WHEN LifespanMonths >= 12 AND TotalSales > 5000 THEN 'VIP'
            WHEN LifespanMonths >= 12 AND TotalSales <= 5000 THEN 'Regular'
            ELSE 'New'
        END AS CustomerSegment,
        CASE
            WHEN Age < 20 THEN 'Under 20'
            WHEN Age BETWEEN 20 AND 29 THEN '20-29'
            WHEN Age BETWEEN 30 AND 39 THEN '30-39'
            WHEN Age BETWEEN 40 AND 49 THEN '40-49'
            ELSE '50+'
        END AS AgeGroup
    FROM CustomerBase
)

-- Final Report
SELECT
    CustomerKey,
    CustomerName,
    Age,
    AgeGroup,
    CustomerSegment,
    TotalOrders,
    TotalSales,
    TotalQuantity,
    LastOrderDate,
    DATEDIFF(MONTH, LastOrderDate, GETDATE()) AS RecencyMonths,
    ROUND(TotalSales / NULLIF(TotalOrders, 0), 2) AS AvgOrderValue,
    ROUND(TotalSales / NULLIF(LifespanMonths, 0), 2) AS AvgMonthlySpend
FROM CustomerSegments
ORDER BY TotalSales DESC;
```

**Key Metrics:**
- Customer Segmentation (VIP/Regular/New)
- Age Group Analysis
- Recency (Months since last order)
- Avg. Order Value (AOV) & Monthly Spend

2. **Product Report**

    **Objective:** Analyze product performance, revenue contribution, and pricing tiers.

```sql
-- Step 1: Base data (product sales + categories)
WITH ProductBase AS (
    SELECT
        p.ProductKey,
        p.ProductName,
        p.Category,
        p.Subcategory,
        p.ProductCost,
        COUNT(DISTINCT s.OrderNumber) AS TotalOrders,
        SUM(s.SalesAmount) AS TotalSales,
        SUM(s.OrderQuantity) AS TotalQuantity,
        MAX(s.OrderDate) AS LastSaleDate,
        DATEDIFF(MONTH, MIN(s.OrderDate), MAX(s.OrderDate)) AS SalesLifespan
    FROM Sales s
    JOIN Product p ON s.ProductKey = p.ProductKey
    WHERE s.OrderDate IS NOT NULL
    GROUP BY p.ProductKey, p.ProductName, p.Category, p.Subcategory, p.ProductCost
),

-- Step 2: Segment products (High/Medium/Low Revenue)
ProductSegments AS (
    SELECT
        *,
        CASE
            WHEN TotalSales > 50000 THEN 'High Revenue'
            WHEN TotalSales BETWEEN 10000 AND 50000 THEN 'Medium Revenue'
            ELSE 'Low Revenue'
        END AS RevenueSegment,
        ROUND(TotalSales / SUM(TotalSales) OVER () * 100, 2) AS RevenueContributionPercent
    FROM ProductBase
)

-- Final Report
SELECT
    ProductKey,
    ProductName,
    Category,
    Subcategory,
    ProductCost,
    RevenueSegment,
    RevenueContributionPercent,
    TotalOrders,
    TotalSales,
    TotalQuantity,
    LastSaleDate,
    DATEDIFF(MONTH, LastSaleDate, GETDATE()) AS RecencyMonths,
    ROUND(TotalSales / NULLIF(TotalQuantity, 0), 2) AS AvgSellingPrice,
    ROUND(TotalSales / NULLIF(SalesLifespan, 0), 2) AS AvgMonthlyRevenue
FROM ProductSegments
ORDER BY TotalSales DESC;
```

**Key Metrics:**

- Revenue Segmentation (High/Medium/Low)
- % Contribution to Total Sales
- Avg. Selling Price & Monthly Revenue
- Sales Recency (Last sold date)

### How to Deploy These Reports
1. **Save as Views for easy access:**

```sql
CREATE VIEW Gold.Report_Customers AS <Customer Query>;
CREATE VIEW Gold.Report_Products AS <Product Query>;
```

2. **Connect to BI Tools** (Power BI, Tableau):

```sql
SELECT * FROM Gold.Report_Customers;  -- Use in dashboards
```

### Why These Reports Matter
- **Customers:** Identify VIPs, track churn risk, optimize marketing.
- **Products:** Find best-sellers, retire underperformers, adjust pricing.

**Pro Tip:** Add visualizations (pie charts for segments, trend lines for sales).