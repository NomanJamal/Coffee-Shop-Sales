# Power BI Build Guide - Coffee Shop Sales Dashboard

## 1. Load data (Home > Get Data > Text/CSV)
Load all five CSVs from this folder: FactSales, DimDate, DimTime, DimStore, DimProduct.
In Power Query, set types: transaction_date -> Date, transaction_time -> Time, revenue/unit_price -> Fixed decimal, hour/ids -> Whole number.

## 2. Model (Model view) - star schema, all 1:* single direction (dim -> fact)
- DimDate[Date]            -> FactSales[transaction_date]
- DimTime[Hour]            -> FactSales[hour]
- DimStore[store_id]       -> FactSales[store_id]
- DimProduct[product_id]   -> FactSales[product_id]
Mark DimDate as the date table. Sort DimDate[Month] by MonthNum and DimDate[Weekday] by WeekdayNum.

## 3. DAX measures (create in a table named "Measures")
```
Total Revenue      = SUM ( FactSales[revenue] )
Total Transactions = DISTINCTCOUNT ( FactSales[transaction_id] )
AOV                = DIVIDE ( [Total Revenue], [Total Transactions] )
Quantity Sold      = SUM ( FactSales[transaction_qty] )
Items per Tx       = DIVIDE ( [Quantity Sold], [Total Transactions] )
Prev Month Revenue = CALCULATE ( [Total Revenue], DATEADD ( DimDate[Date], -1, MONTH ) )
MoM Growth %       = DIVIDE ( [Total Revenue] - [Prev Month Revenue], [Prev Month Revenue] )
Revenue Share %    = DIVIDE ( [Total Revenue], CALCULATE ( [Total Revenue], ALL ( DimProduct ) ) )
Hour Share %       = DIVIDE ( [Total Revenue], CALCULATE ( [Total Revenue], ALL ( DimTime ) ) )
Avg Tx per Hour per Day =
    DIVIDE ( [Total Transactions], DISTINCTCOUNT ( DimDate[Date] ) )
Revenue Rank (Product) =
    RANKX ( ALL ( DimProduct[product_detail] ), [Total Revenue], , DESC, DENSE )
Peak Hour =
    VAR t = ADDCOLUMNS ( VALUES ( DimTime[Hour] ), "r", [Total Revenue] )
    RETURN MAXX ( TOPN ( 1, t, [r], DESC ), DimTime[Hour] )
```
Note: each transaction_id is one product line (there is no basket ID), so "Total Transactions" counts lines.

## 4. Pages and visuals (matches the brief's dashboard checklist)
Page 1 - Sales Overview: 4 KPI cards (Total Revenue, Total Transactions, AOV, Quantity Sold); line chart Revenue by DimDate[Month]; slicers: Store, Category, Month.
Page 2 - Product Performance: bar chart Top 10 products (Top N filter on product_detail by Total Revenue); bar chart Bottom 10 (Bottom N); donut/bar Revenue by product_category; table with Revenue, Qty, Revenue Share %.
Page 3 - Time Analysis: column chart Revenue by DimTime[HourLabel] (highlight 07:00-10:00); line chart daily revenue; matrix Weekday x Hour with Total Transactions and conditional-format heat map.
Page 4 - Customer Behavior and Operations: card Items per Tx; clustered bar Avg Tx per Hour per Day by Store x Hour; table by DimTime[Daypart] and Store.
Theme suggestion: dark espresso #3B2314, caramel #C8873A, cream #F4E9D8, accent teal #2A7F7B.

## 5. Save
File > Save As > CoffeeShop_Dashboard.pbix
