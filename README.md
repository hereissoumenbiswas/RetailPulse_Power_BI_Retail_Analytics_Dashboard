# Main Objectives

To build an end-to-end Power BI analytics solution that turns two years of raw UK online retail transactions (December 2009 to December 2011) into a trusted data model and an interactive 10-page dashboard. A business manager should be able to see sales performance, product and customer behaviour, returns and stock risk, and make decisions with confidence.

# Background

A retailer records thousands of invoice lines every month, but the raw rows don’t show what is selling, who is buying, or where money is lost. This project was assigned in my Data Science and Analytics internship at Zidio Development. It is the Power BI track of the RetailPulse project.

# Business objectives

What the dashboard must help the business do:

1. Track performance: monitor revenue, orders, average order value, estimated profit and margin, with month-over-month growth and seasonal patterns.
2. Find the best and worst products: rank products by net revenue (after returns), leaving out non-product items such as postage and manual entries.
3. Know the customers: group customers by recency and number of orders (Champions, Loyal, New / Recent, At Risk, Lost), so the business can reward loyal customers and win back those at risk.
4. Understand markets: show which countries bring revenue and how dependent the business is on the United Kingdom.
5. Control returns: measure returned value, return rate and net revenue.
6. Spot inventory risk: find products that have stopped selling or are slowing down, using sales patterns.
7. Plan timing: find the peak months, weekdays and hours for staffing, stock and promotions.

# Data objectives
1. Combine the two CSV files into one Sales table with no lost or doubled rows.
2. Read every date correctly (month/day/year) and verify the first and last dates.
3. Clean the data: correct types, trimmed text, duplicates removed, sales separated from returns and adjustments, non-product items and unknown customers flagged.
4. Record every cleaning rule with row counts before and after.
5. Handle the rows without a Customer_ID openly: keep them for sales analysis and leave them out of customer analysis.

# Modeling objectives
1. Build a star schema with Sales as the fact table, and Date and Customers as dimension tables.
2. Create correct one-to-many relationships with single filter direction.
3. Create customer fields for recency, frequency and revenue, and use them to assign segments.
4. Put every KPI in a documented DAX measure, so the same number means the same thing on every page.

# Dashboard objectives
1. Build 10 pages, each answering one business question, with a consistent theme.
2. Add slicers for year, quarter, month and country, plus drill-through and page navigation.

## Data

**Source:** Online Retail II, transactions of a UK online retailer from 1 December 2009 to 9 December 2011.

| Table | What it holds | Columns used in this project |
|---|---|---|
| `online_retail_09_10.csv`, `online_retail_10_11.csv` | Source files: raw transactions, one row per invoice line, split by period | `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country` |
| `Sales` | Fact table: both files combined and cleaned, one row per invoice line | `Invoice`, `Stock_Code`, `Description`, `Quantity`, `Invoice_Date`, `Unit_Price`, `Customer_ID`, `Country`, and the added columns `Transaction_Type`, `Invoice_Date_Only`, `Revenue`, `Estimated_Cost`, `Estimated_Profit`, `Hour`, `Customer_Known`, `Item_Type` |
| `Date` | Dimension table, one row per day from the first to the last sale date | `Date`, `Year`, `Month No`, `Month`, `Quarter`, `Weekday No`, `Weekday` |
| `Customers` | Dimension table, One row per known customer who has at least one sale | `Customer_ID`, `Orders`, `Customer_Revenue`, `Last_Purchase`, `Recency_Days`, `Segment` |
| `OrderSummary` | Dimension table, One row per invoice that has at least one sale line | `Invoice`, `Order_Value` |
| `KPI Measures` | No data rows; holds the DAX measures | `Total Revenue`, `Total Orders`, `Total Customers`, `Total Quantity Sold`, `Total Estimated Profit`, `Average Order Value`, `Estimated Profit Margin %`, `Repeat Customers`, `New Customers`, `At Risk Customers`, `Lost Customers`, `Previous Month Revenue`, `Revenue Growth %`, `Return Value`, `Net Revenue`, `Net Units Sold` |

**Notes on the data**

- The raw columns were renamed in Power Query (for example `InvoiceNo` became `Invoice` and `UnitPrice` became `Unit_Price`).
- `Transaction_Type` has three values: `Sale`, `Return` (invoices starting with "C") and `Adjustment` (zero or negative quantity or price).
- About 1.3 lakhs of rows have no `Customer_ID`. They stay in `Sales` for sales analysis, and `Customer_Known` flags them.
- `Item_Type` separates real products from non-product codes such as postage and fees.
- `Estimated_Cost` is assumed to be 65% of revenue, because the dataset has no cost column.
- Size: 10,67,371 rows before cleaning, 10,33,036 after removing duplicates.
