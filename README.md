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
| `KPI Measures` | No data rows; holds the DAX measures | `Total Revenue`, `Total Orders`, `Total Customers`, `Total Quantity Sold`, `Total Estimated Profit`, `Average Order Value`, `Estimated Profit Margin %`, `Repeat Customers`, `New Customers`, `At Risk Customers`, `Lost Customers`, `Previous Month Revenue`, `Revenue Growth %`, `Return Value` |

**Notes on the data**

- The raw columns were renamed in Power Query (for example `InvoiceNo` became `Invoice` and `UnitPrice` became `Unit_Price`).
- `Transaction_Type` has three values: `Sale`, `Return` (invoices starting with "C") and `Adjustment` (zero or negative quantity or price).
- About 1.3 lakhs of rows have no `Customer_ID`. They stay in `Sales` for sales analysis, and `Customer_Known` flags them.
- `Item_Type` separates real products from non-product codes such as postage and fees.
- `Estimated_Cost` is assumed to be 65% of revenue, because the dataset has no cost column.
- Size: 10,67,371 rows before cleaning, 10,33,036 after removing duplicates.

# Step-by-step: what I did and the problems I faced

### Step 1: Load the two CSV files

**Problem:** The preview showed dates like `12-01-2009`, which looked wrong.

**Fix:** Fixed in Step 2.

### Step 2: Fix the date column

**Problem:** Power BI read the month/day/year dates as day/month/year (12 January instead of 1 December). My first fix also went on the wrong column, and the Date type gave errors because the values include time.

**Fix:** Set `InvoiceDate` to Date/Time with the English (United States) locale, in both files.

### Step 3: Combine the two files

**Problem:** The plain Append would have merged file 2 into file 1, and the other columns went back to text after I removed the automatic Changed Type step.

**Fix:** Used Append Queries as New, named it `Sales`, and set the data types again.

### Step 4: Check the loaded data

**Problem:** The column statistics showed the same min and max date, because they only used the first 1000 rows. About 1.3 lakh rows also have no `CustomerID`.

**Fix:** Checked the full data with Earliest and Latest Card visuals (1 Dec 2009 to 9 Dec 2011). Kept the rows without a customer, because they are real sales, and left them out only of customer analysis.

### Step 5: Clean the data

**What I did in Power Query (the Applied Steps are: Renamed Columns, Changed Type, Trimmed Text, Removed Duplicates):**

1. Renamed the columns to `Invoice`, `Stock_Code`, `Description`, `Quantity`, `Invoice_Date`, `Unit_Price`, `Customer_ID`, `Country`
2. Set the data types
3. Trimmed extra spaces in Description
4. Removed duplicate rows (all columns selected)

**Problem:** some instructions I followed used column names that were different from my real names, for example Invoice_No.

**How I solved it:** I always checked the real column name in Power Query before writing a formula.

### Step 6: Add Extra Columns

**What I did in Power Query: Added these columns using Add Column → Custom Column:**
1. Transaction_Type
2. Invoice_Date_Only
3. Revenue
4. Estimated_Cost
5. Estimated_Profit
6. Hour
7. Customer_Known
8. Item_Type

**Purpose:** These columns support transaction classification, date analysis, revenue and estimated profit calculations, hourly analysis, customer identification, and product filtering.

**Problem:** The Transaction_Type column initially showed errors because the formula referenced an incorrect column name. The Hour column was also missing.

**How I solved it:** Corrected the formula using the actual Invoice column name and added the missing Hour column.

### Step 7: Create the Date Table

**Purpose:** To support time-based sales analysis and consistent date filtering across dashboard pages.

**What I did:**
Created a separate Date table in Power BI using the sales data's date range.

**Columns Created:**

| Column Name | Description |
|---|---|
| `Date` | Contains a continuous sequence of dates. |
| `Year` | Identifies the year. |
| `Month No` | Represents the month number from 1 to 12. |
| `Month` | Displays the full month name. |
| `Quarter` | Identifies the quarter of the year. |
| `Weekday No` | Represents the weekday number, starting with Monday as 1. |
| `Weekday` | Displays the weekday name. |

**Additional Configuration:**

Sorted Month by Month No to maintain the correct month order.

Sorted Weekday by Weekday No to maintain the correct weekday order.

Marked the table as a Date table.

Created a relationship between the Date table and the Sales table using their date columns.

**Result:** The Date table enables consistent filtering and analysis of sales by year, quarter, month, and weekday.

### Step 8: Create the Customer Table

**Purpose:** To create a separate list of unique, identified customers for customer-level analysis.

**What I did:**
Created a Customers table in Power BI using the customer information available in the Sales table.

**Columns Created:**

| Column Name | Description |
|---|---|
| `Customer_Type` | Identifies the customer type based on customer information. |
| `Customer_ID` | Stores the unique identification number of each customer. |
| `Customer_Revenue` | Represents the total revenue generated by each customer. |
| `Last_Purchase` | Stores the date of the customer's most recent purchase. |
| `Orders` | Represents the total number of orders placed by each customer. |
| `Recency_Days` | Calculates the number of days since the customer's last purchase. |
| `Segment` | Categorizes customers into different segments based on their purchasing behavior. |
