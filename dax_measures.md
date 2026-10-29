# DAX Measures

Table names assume a fact table called `Sales` and a `Date` table. Adjust to match your model.

## Date table

```dax
Date =
ADDCOLUMNS (
    CALENDAR ( DATE ( 2023, 1, 1 ), DATE ( 2025, 12, 31 ) ),
    "Year", YEAR ( [Date] ),
    "Month Number", MONTH ( [Date] ),
    "Month", FORMAT ( [Date], "MMM" ),
    "Quarter", "Q" & QUARTER ( [Date] )
)
```

Mark it as a date table and relate `Date[Date]` to `Sales[Order Date]` (one-to-many).

## Core KPIs

```dax
Total Sales = SUM ( Sales[Sales] )

Total Profit = SUM ( Sales[Profit] )

Profit Margin % = DIVIDE ( [Total Profit], [Total Sales] )

Total Orders = DISTINCTCOUNT ( Sales[Order ID] )

Average Order Value = DIVIDE ( [Total Sales], [Total Orders] )

Total Quantity = SUM ( Sales[Quantity] )
```

## Time intelligence

```dax
Sales PY = CALCULATE ( [Total Sales], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )

Sales YoY % = DIVIDE ( [Total Sales] - [Sales PY], [Sales PY] )

Sales YTD = TOTALYTD ( [Total Sales], 'Date'[Date] )

Profit YoY % =
VAR ProfitPY = CALCULATE ( [Total Profit], SAMEPERIODLASTYEAR ( 'Date'[Date] ) )
RETURN DIVIDE ( [Total Profit] - ProfitPY, ProfitPY )
```

## Ranking and segmentation

```dax
Top 10 Products Sales =
CALCULATE (
    [Total Sales],
    TOPN ( 10, ALL ( Sales[Product Name] ), [Total Sales], DESC )
)

Sales % of Total =
DIVIDE ( [Total Sales], CALCULATE ( [Total Sales], ALL ( Sales ) ) )

Discount Band =
SWITCH (
    TRUE (),
    Sales[Discount] = 0, "No discount",
    Sales[Discount] <= 0.2, "10-20%",
    "30-40%"
)
```

`Discount Band` is a calculated column on the `Sales` table.

## Loss-making flag

```dax
Loss Making Orders = CALCULATE ( [Total Orders], FILTER ( Sales, Sales[Profit] < 0 ) )

Margin Status =
IF ( [Profit Margin %] < 0, "Loss", IF ( [Profit Margin %] < 0.05, "Low", "Healthy" ) )
```

Use `Margin Status` for conditional formatting (red for Loss, amber for Low, blue for Healthy).
