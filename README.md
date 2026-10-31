# RetailPulse – Interactive Retail Sales & Customer Analytics Dashboard (Power BI)

## Main Objectives

To build an end-to-end Power BI analytics solution that turns two years of raw UK online retail transactions (December 2009 to December 2011) into a trusted data model and an interactive 10-page dashboard. A business manager should be able to see sales performance, product and customer behaviour, returns and stock risk, and make decisions with confidence.

## Background

A retailer records thousands of invoice lines every month, but the raw rows don’t show what is selling, who is buying, or where money is lost. This project was assigned in my Data Science and Analytics internship at Zidio Development. It is the Power BI track of the RetailPulse project.

## Business objectives

What the dashboard must help the business do:

1. Track performance: monitor revenue, orders, average order value, estimated profit and margin, with month-over-month growth and seasonal patterns.
2. Find the best and worst products: rank products by net revenue (after returns), leaving out non-product items such as postage and manual entries.
3. Know the customers: group customers by recency and number of orders (Champions, Loyal, New / Recent, At Risk, Lost), so the business can reward loyal customers and win back those at risk.
4. Understand markets: show which countries bring revenue and how dependent the business is on the United Kingdom.
5. Control returns: measure returned value, return rate and net revenue.
6. Plan timing: find the peak months and quarters for staffing, stock and promotions.

## Data

**Source:** Online Retail II, transactions of a UK online retailer from 1 December 2009 to 9 December 2011.

| Table | What it holds | Columns used in this project |
|---|---|---|
| `online_retail_09_10.csv`, `online_retail_10_11.csv` | Source files: raw transactions, one row per invoice line, split by period | `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country` |
| `Sales` | Fact table: both files combined and cleaned, one row per invoice line | `Invoice`, `Stock_Code`, `Description`, `Quantity`, `Invoice_Date`, `Unit_Price`, `Customer_ID`, `Country`, and the added columns `Transaction_Type`, `Invoice_Date_Only`, `Revenue`, `Estimated_Cost`, `Estimated_Profit`, `Hour`, `Customer_Known`, `Item_Type` |
| `Date` | Dimension table, one row per day from the first to the last sale date | `Date`, `Year`, `Month No`, `Month`, `Quarter`, `Weekday No`, `Weekday` |
| `Customers` | Dimension table, One row per known customer who has at least one sale | `Customer_ID`, `Orders`, `Customer_Revenue`, `Last_Purchase`, `Recency_Days`, `Segment` |
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

**Problem:** A formula failed because it used a column name that did not exist.

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

- Sorted Month by Month No to maintain the correct month order.
- Sorted Weekday by Weekday No to maintain the correct weekday order.
- Marked the table as a Date table.
- Created a relationship between the Date table and the Sales table using their date columns.

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

## Step 9: Create DAX Measures

### Objective
Created a centralized `KPI Measures` table to organize DAX measures and perform dynamic calculations for business analysis.

### DAX Measures Created

| Analysis Area | Key Measures |
|---|---|
| Sales & Revenue | Total Revenue, Net Revenue, Revenue Growth %, Previous Month Revenue, Average Order Value |
| Customer Analysis | Total Customers, New Customers, Repeat Customers, Average Customer Spend, At Risk Customers, Lost Customers |
| Profitability | Total Estimated Cost, Total Estimated Profit, Estimated Profit Margin % |
| Returns Analysis | Return Quantity, Return Rate %, Return Value, Net Units Sold |
| Data Validation | Total Sales Rows, Duplicate Customer IDs |

### Purpose
- Monitor key business KPIs.
- Analyze sales, revenue, and profitability.
- Understand customer behavior and retention.
- Evaluate product returns and net sales.
- Support interactive dashboards using filters and slicers.

**Note:** Estimated Cost is assumed to be 65% of revenue; therefore, profit figures are estimates, not actual accounting values.

## Step 10: Build Interactive Power BI Dashboards

### Purpose
Created 10 interactive dashboard pages to analyze sales, customers, products, countries, profitability, and returns.

### Dashboard Pages

| # | Dashboard Page |
|---|---|
| 1 | Executive Summary |
| 2 | Sales Performance Analysis |
| 3 | Customer Analysis |
| 4 | Product Analysis |
| 5 | Region and Country Analysis |
| 6 | Monthly & Seasonal Analysis |
| 7 | Customer Behavior Analysis |
| 8 | Profit & Revenue |
| 9 | Revenue and Transaction Analysis |
| 10 | Sales Returns & Adjustments |

### Interactive Features
- Year, Quarter, Month, and Country slicers.
- Cross-filtering between visuals.
- Interactive KPI cards and charts.
- Chart tooltips and page navigation.
- Consistent dashboard theme and layout.

### Problems & Fixes
- **Top 10 filter issue:** Applied Top N filters to display only the highest-performing products.
- **Non-product entries:** Used `Item_Type` to exclude non-product items from product analysis.
- **Cancelled orders affecting revenue:** Used `Net Revenue` to account for returns.
- **Incorrect transaction-type values:** Used an appropriate revenue measure without a fixed Sale filter.

### Outcome
Developed a 10-page interactive Power BI report that helps users explore business performance through KPIs, charts, slicers, and cross-filtering.

## Author

**Soumen Biswas**
(Data Science graduate)

- GitHub: (https://github.com/hereissoumenbiswas)
- LinkedIn: (https://www.linkedin.com/in/hereissoumenbiswas/?isSelfProfile=true)
